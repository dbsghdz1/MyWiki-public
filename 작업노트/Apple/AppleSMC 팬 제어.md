---
type: study
area: Apple
audience: ai
status: active
created: 2026-09-28
updated: 2026-09-28
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
