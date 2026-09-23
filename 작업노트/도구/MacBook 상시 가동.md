---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-12
updated: 2026-09-23
projects:
  - "홍보 자동화"
  - "[[프로젝트/개인/BookMini/README|BookMini]]"
---

# MacBook 상시 가동 — 맥미니처럼 쓰기

한 줄 요약 — **전원 연결 시 잠자기를 끄고(`sudo pmset -c sleep 0`) 외부 모니터를 꽂은 채 덮개를 닫으면 MacBook도 상시 가동된다.** 배터리는 충전 한도 80%로 보호한다. 남는 함정은 둘이다: **FileVault 때문에 재부팅 뒤 멈추는 것**과 **원격으로 살릴 경로가 없는 것**.

## 핵심 정리

- **이 맥 실측 (2026-09-12)**
  - MacBook Pro `Mac17,9`(M5 Pro, 48GB), macOS 26.5.1, AC 연결 100% charged
  - `pmset` AC 설정: `sleep 1` · `displaysleep 30` · `womp 1` · `powernap 1`
  - FileVault On
  - `com.openssh.sshd`·`com.apple.screensharing` 미로드, Tailscale 없음
  - 외부 모니터 DELL P2723QE(5K)·P2419H(24" 세로)
- **Claude Code 세션이 거는 `caffeinate`를 상시 잠자기 방지로 착각하지 않는다.**
  - `caffeinate -i -t 300`의 부모 프로세스가 `claude`였다. 작업하는 동안만 5분씩 갱신되고, 세션이 멈추면 사라진다.
  - 확인: `ps -o pid,ppid,args -p $(pgrep caffeinate)`, `pmset -g assertions`
  - 2026-09-12에 이걸 보고 "밤에도 안 잠든다"고 잘못 말했다.
- **잠자기 끄기**: `sudo pmset -c sleep 0`
  - 전원 연결 시에만 적용된다.
  - 모니터는 `displaysleep`대로 꺼져도 된다.
  - 덮개를 닫아도 외부 모니터와 전원이 연결돼 있으면 clamshell 모드로 깨어 있다. Apple 권장 구성은 전원 + 외부 디스플레이 + 외부 키보드·마우스다.
- **모니터 없이 덮개를 닫고 돌려야 할 때만 앱이나 강제 설정을 쓴다.**
  - Amphetamine의 Closed-Display Mode: Apple 실리콘 노트북은 5.3부터 스크립트 설치와 관리자 인증이 필요하고, 5.3.1의 Power Protect는 별도로 설치한다(검색 요약 기준).
  - 강제 대안 `sudo pmset -a disablesleep 1`: 배터리 모드에도 적용돼 가방 속에서 발열할 위험이 있다. 되돌리기는 `0`.
- **배터리 보호**: macOS 26.4+의 설정 > 배터리 > 충전 옆 (i) > 충전 한도(80~100%). macOS가 켜져 있을 때만 작동한다.
- **FileVault On이면 자동 로그인이 안 된다.**
  - 그래서 재부팅(macOS 업데이트, 전원 차단) 뒤에는 사람이 비밀번호를 넣기 전까지 로그인 세션 기반 자동화(Aside 브라우저, launchd 사용자 에이전트)가 돌지 않는다.
  - 계획된 재시작은 `sudo fdesetup authrestart`로 한다.
  - macOS 업데이트 자동 설치는 끄고, 사람이 있을 때 올린다.
- **launchd로 도는 bash 스크립트에서 `set -o pipefail` + `cmd | head -1`을 쓰면 exit 141(SIGPIPE)로 조용히 죽는다** (2026-09-12 BookMini 실측).
  - 증상: `launchctl print gui/$(id -u)/<label>`의 `last exit code = 141`, 로그 파일은 빈 채.
  - 원인: 읽는 쪽(`head -1`, `awk '… {exit}'`)이 먼저 끝나면 앞 명령(`pmset -g batt`, `ioreg`)이 SIGPIPE를 받고, pipefail이 그걸 실패로 올린다.
  - 수정: 출력을 먼저 변수에 담고(`out="$(pmset -g batt)"`) here-string으로 파싱한다.
- **구현체**: 이 문서의 구성을 스크립트로 묶은 것이 [[프로젝트/개인/BookMini/README|BookMini]] v0다 — 현재는 `caffeinate -s` launchd(전원 연결 시에만 잠자기 방지), 5분 하트비트 → Oracle 감시 → Slack. 2026-09-23에 배터리에서도 적용되던 `-i`를 제거했다.
- **2026-09-12 홍 설정 반영**: 충전 한도 80%(CLI로 값은 못 읽고 94% `not charging`으로 간접 확인), macOS 자동 업데이트 끔(`defaults read /Library/Preferences/com.apple.SoftwareUpdate AutomaticallyInstallMacOSUpdates` → `0`). App Store 앱 자동 업데이트(`com.apple.commerce AutoUpdate=1`)는 재부팅을 일으키지 않아 그대로 둔다.
- **2026-09-12 원격 로그인 켬(홍)** — 확인: `nc -z -G 3 127.0.0.1 22` 성공(`launchctl print system/com.openssh.sshd`는 온디맨드라 로드 여부로 판단하지 않는다). 밖에서 닿으려면 Tailscale 같은 사설망이 아직 없다.
- **원격 복구 경로를 먼저 만든다.**
  - 원격 로그인(SSH)과 화면 공유를 켠다(시스템 설정 > 일반 > 공유).
  - 휴대폰에서 닿으려면 Tailscale 같은 사설망을 둔다.
  - Oracle에서 Mac이 살아 있는지 보는 신호도 둔다 — 예: publisher 워커가 폴링할 때 서버에 마지막 시각을 남기고, 07:20 브리핑이 확인한다.

## 기록

### 2026-09-23 — Claude RC 대기를 전원 연결 중으로 한정

- **맥락**: 홍이 전원 연결 중에만 Claude RC를 계속 쓰려는 목적을 설명하고 설정 변경을 승인했다. [[프로젝트/개인/BookMini/README|BookMini]] 로컬 커밋 `21772cd`.
- **이전 설명 정정**: 09-12에는 `caffeinate -i -s` 전체를 AC 전용으로 설명했으나, 로컬 `man caffeinate`에 따르면 `-i`는 전원 종류와 무관하게 유휴 잠자기를 막고 `-s`만 AC에서 유효하다. 09-23 변경 전 배터리에서 BookMini PID에 무기한 `PreventUserIdleSystemSleep`이 실제로 있었다. 기존 설명보다 매뉴얼과 실행 상태를 우선한다.
- **수정**: 소스 `bin/bookmini-awake`와 설치본을 `exec /usr/bin/caffeinate -s`로 맞추고 `com.hong.bookmini.awake`만 `launchctl kickstart -k`로 재시작했다. 전체 설치기는 실행하지 않았다. 화면 자동 꺼짐은 macOS Lock Screen UI에서 배터리 5분·전원 10분으로 적용했다.
- **검증**: `bash -n`, `git diff --check`, 소스/설치본 `cmp` 통과. `pmset -g custom`에서 `displaysleep` 5/10, 양쪽 `sleep 1`·`powermode 0` 유지. 배터리 상태에서 awake 새 PID에는 `PreventSystemSleep`만 있고 시스템 집계는 0이었다. BookMini의 `PreventUserIdleSystemSleep`은 사라졌다. 다른 프로세스의 300초 유휴 잠자기 방지는 별도로 남아 있으므로 집계값 1만 보고 수정 실패로 판정하지 않는다.
- **검증 범위**: 실제 잠자기·덮개 닫힘·전원 연결 후 화면 꺼짐 상태의 RC 연결은 시험하지 않았다. 기본 잠자기 타이머만으로 정확한 잠드는 시간을 단정하지 않는다. 전원 차단 시 RC를 즉시 종료하는 설정도 아니다.
- **성능 관찰**: 변경 전 짧은 표본에서 M5 Pro·48GB, 전체 CPU 약 10% 사용, 스왑 약 288MB였고 `pmset -g therm`에는 온도·성능 경고 기록이 없었다. 온도 직접 측정이나 지속 부하 벤치마크 결과는 아니다.
- **근거**: 로컬 `man caffeinate`, `pmset -g custom`·`assertions`·`batt`·`therm`, `launchctl print`, `top`, `sysctl`; [Claude RC 공식 문서](https://code.claude.com/docs/en/remote-control), [Apple 배터리 설정](https://support.apple.com/en-sa/guide/mac-help/mchlfc3b7879/mac). RC의 코드 실행은 로컬에 남으며 시스템 잠자기와 화면 꺼짐은 구분한다.

### 2026-09-12 — 화면 조작 홍보를 위해 MacBook을 상시 가동 서버로

- **맥락**: 오토파일럿 설계안에서 홍이 게시를 API 대신 화면(Aside 브라우저) 조작으로 하자고 했다 → Mac이 밤새 깨어 있어야 한다. 홍이 이렇게 요청했다: "맥북을 맥미니처럼 만들 수 있는 무언가".
- **배운 것**: 위 핵심 정리. 가장 컸던 건 두 가지다.
  - ① 떠 있던 `caffeinate`가 Claude Code 세션 것이었다는 오판.
  - ② FileVault 때문에 재부팅 뒤 무인 복구가 안 된다는 점.
- **근거**: `system_profiler SPHardwareDataType`, `pmset -g custom`, `pmset -g assertions`, `fdesetup status`, `launchctl print system/com.openssh.sshd`.
- **아직 모르는 것**: 화면이 잠기거나 모니터가 꺼진 상태에서 `aside repl` 조작이 되는지. 첫 드라이런에서 확인한다.

## 참고 자료 (검색 결과 요약 기준, 원문 미열람 — 2026-09-12)

- [How to use a MacBook with the lid closed — Macworld](https://www.macworld.com/article/673295/how-to-use-macbook-with-lid-closed-stop-closed-mac-sleeping.html)
- [Apple Brings iPhone-Style Battery Charge Limits to the Mac in macOS Tahoe 26.4 — MacRumors](https://www.macrumors.com/2026/02/16/mac-charge-limit-macos-tahoe-26-4/)
- [Amphetamine — App Store](https://apps.apple.com/us/app/amphetamine/id937984704)
