---
title: "AppleSMC 팬 제어"
summary: "팬 회전수와 온도를 읽을 때 SMC 키 이름, 구조체 정렬과 접근 권한을 점검한 기록"
type: study
area: Apple
audience: ai
status: active
created: 2026-09-28
updated: 2026-11-25
projects: []
---

# AppleSMC 팬 제어

한 줄 요약 — **Apple Silicon 맥의 온도·팬은 IOKit `AppleSMC`로 root 없이 읽히고, 팬 쓰기만 root가 필요하다.** Swift로 SMC 구조체를 옮길 때 **꼬리 패딩** 때문에 `kIOReturnBadArgument`가 난다.

## 기록

### 2026-09-28 — Macs Fan Control 유료 기능(온도 기준 팬 구동) 직접 구현

- **맥락**: 홍 요청 "CPU·GPU가 45°C를 넘으면 팬을 틀고 싶다". 위키 프로젝트가 아닌 개인 도구 `(로컬 경로)`(`SMC.swift`·`main.swift`·`install.sh`·`com.hong.fanauto.plist`).
- **증상**: `IOConnectCallStructMethod(conn, 2, …)`가 `-536870206`(`0xE00002C2`, `kIOReturnBadArgument`).
  - **원인**: C `SMCKeyData_keyInfo_t`는 `dataSize u32 + dataType u32 + dataAttributes u8` = 9바이트지만 stride 12다. **Swift는 중첩 구조체 다음 필드를 size(9) 자리에 붙인다** — `result`가 C의 40이 아닌 37에 놓여 전체 레이아웃이 어긋난다.
  - **해결**: `SMCKeyInfo`에 `pad: (UInt8, UInt8, UInt8)`를 넣어 12바이트로 맞춘다. 전체 `MemoryLayout<SMCParam>.stride`는 80이어야 한다.
- **호출 규약**(selector 2, `data8`로 명령): `5` 읽기 · `6` 쓰기 · `8` 인덱스→키 · `9` 키 정보. `#KEY`로 키 개수.
- **M5 Pro `Mac17,9` 실측 (macOS 26)**
  - `FNum` = 2, `F0Mn`/`F1Mn` 2317, `F0Mx`/`F1Mx` 7826 (`flt `, 리틀엔디언 Float32)
  - 유휴 시 `F0Ac` 0rpm — 팬이 아예 멈춰 있다
  - **모드 키는 소문자 `F0md`**(`ui8`). 인텔·구형 자료의 `F0Md`는 없다. `Ftst`(M3 이후 잠금 해제 키라는 자료가 있음)도 이 기기엔 없다.
  - 목표 속도 `F0Tg`(`flt `)
  - 온도: `Tp*` 23개 = CPU, `Tg*` 42개 = GPU, 모두 `flt `. 유휴 CPU 37~50°C, GPU 33~45°C — **45°C 임계값이면 가벼운 작업에도 팬이 돈다.**
- **쓰기 검증 (2026-09-28)**: root 데몬에서 `F0md=1` → `F0Tg` 쓰기가 **`Ftst` 잠금 해제 없이 바로 먹는다**. `yes` 8개 부하로 CPU 53°C → 수 초 뒤 양쪽 팬 0 → 3744rpm, 목표 rpm을 따라 오르내림 확인.
- **설치 지뢰**: Claude Code `!`에서는 `sudo`가 `a terminal is required to read the password`로 실패 → `osascript -e 'do shell script "…" with administrator privileges'`로 비밀번호 창을 띄운다. 이때 **`(로컬 경로)` 안 스크립트는 `Operation not permitted (126)`** — Desktop은 TCC 보호 폴더라 관리자 셸도 못 읽는다. 보호 밖 폴더로 복사해서 실행. 관리자 창 경유 시 `SUDO_USER`도 비어 있다.
- **종료 시 자동 모드 복귀 필수**: `F0md=1`로 두고 프로세스가 죽으면 팬이 마지막 속도에 고정된다. SIGTERM·SIGINT·SIGHUP에서 `F*md=0`.
- **Macs Fan Control과 동시 실행 금지**: 둘 다 같은 키를 덮어쓴다.

### 2026-11-25: 목표 rpm으로 바로 뛰지 않고 램프로 따라가게 변경

- **맥락**: 홍 「갑자기 켜졌다가 엄청 시끄럽다가 갑자기 내려가」. 설치본은 ON 60°C, OFF 55°C, FULL 80°C였다. 바탕화면 plist는 50/46으로 설치본과 달랐고, 이번에 60/55로 맞췄다.
- **원인**: 2초마다 센서 65개(`Tp*` 23, `Tg*` 42)의 **최댓값**으로 목표 rpm을 계산해 그대로 썼다. 센서 하나가 순간 80°C를 찍으면 곧바로 7826rpm이 되고 20초 뒤 2330rpm으로 떨어졌다(11-25 12:22 로그). OFF 아래로 내려가면 곧바로 `F*md=0`으로 넘겼고, 맥 자동 모드는 유휴 때 0rpm이라 소리가 뚝 끊겼다.
- **변경** (`main.swift`, 백업 `main.swift.bak-20261125`)
  - 지금 준 rpm(`current`)을 따로 들고, 매 틱 목표 쪽으로 `FAN_RAMP_UP`(기본 150rpm/s) 또는 `FAN_RAMP_DOWN`(기본 80rpm/s)만큼만 움직인다. 최저에서 최고까지 약 37초, 내려올 때 약 70초.
  - OFF 미만이면 목표를 최소 rpm으로 두고 천천히 내린 뒤, 최소에 닿았을 때 자동 모드로 넘긴다.
  - 켜질 때 `F0Ac`가 최소보다 높으면 그 속도에서 이어받는다. 최소 rpm(2317) 아래로는 쓰지 않는다.
  - 로그는 500rpm 이상 움직였거나 목표에 닿았을 때만 남긴다. 기존의 「100rpm 미만 변화는 쓰지 않음」 조건은 없앴다.
- **검증**: `FAN_ON=40 FAN_OFF=35 FAN_FULL=50 FAN_RAMP_UP=1000 ./fanauto run --dry-run`에서 2317 → 4317 → 6317 → 목표 도달 순으로 단계 상승 확인. 설치 뒤 `/var/log/fanauto.log` 첫 줄 `램프 +150/-80 rpm/s`, `launchctl print system/com.hong.fanauto` running, 설치 바이너리와 빌드 결과 `cmp` 일치. 실제 고온에서 램프가 도는 모습은 아직 보지 못했다.
- **설치 지뢰 재확인**: Claude Code auto 모드가 `osascript ... with administrator privileges` 실행을 거부했다. 설치 파일을 scratchpad에 복사하고 홍이 `!`로 같은 명령을 실행해 설치했다.
- **남은 문제 (변경 전 로그 실측)**: 11-25 0시부터 12시 22분까지 수동 구동이 72번 켜졌다 꺼졌고, 그중 22번은 10초 이하, 29번은 11~30초였다. 램프는 속도 변화만 부드럽게 할 뿐, 0rpm에서 최소 2317rpm으로 켜지는 순간과 짧은 온도 튐에 켜지는 문제는 남는다. 다음 후보는 온도를 지수 이동 평균(EMA)으로 눌러 쓰는 것이다.

### 2026-11-25: 판단 온도를 지수 이동 평균으로 바꿈

- **맥락**: 램프 설치 뒤 홍이 위 남은 문제의 해결을 요청했다.
- **변경** (`main.swift`, 램프 버전 백업 `main.swift.bak-20261125-ramp`): 센서 최댓값(`raw`)을 매 틱 `t = t + α(raw - t)`로 섞어, 켜고 끄기와 목표 rpm을 이 값으로 정한다. `α = 1 - exp(-interval / FAN_SMOOTH)`, 기본 `FAN_SMOOTH` 20초, 2초 간격이면 α는 약 0.095다. `FAN_SMOOTH=0`이면 예전처럼 순간값을 쓴다. 첫 틱은 평균 없이 순간값으로 시작한다. 로그에 `평균`을 같이 찍는다.
- **검증**: `FAN_ON=47 FAN_OFF=44 FAN_FULL=55 ./fanauto run --dry-run`에서 순간 온도가 48.4에서 44.1로 떨어질 때 평균은 48.4, 47.8, 47.4로 천천히 내려가고 rpm도 그에 맞춰 움직임을 확인했다.
- **대가 (추론)**: 실제로 뜨거워질 때 반응이 20~30초쯤 늦다. 맥은 과열되면 스스로 성능을 낮추므로 위험하지 않다고 봤지만 실측은 아니다.
- **확인할 것**: 하루 쓴 뒤 같은 방식으로 켜짐 횟수와 10초 이하, 11~30초 구간 수를 세어 위의 72번, 22번, 29번과 비교한다.
