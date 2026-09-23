---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-22
updated: 2026-09-23
projects:
  - "[[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]]"
  - "[[작업노트/도구/Ghostty 설정|Ghostty 설정]]"
---

# macOS 파일 접근 권한(TCC)과 휴지통

**Claude Code 셸에서 `(로컬 경로)` 안을 읽으면 `Operation not permitted`가 난다 — 폴더 문제가 아니라 터미널 앱에 Full Disk Access가 없어서다.** `sudo`·`osascript … with administrator privileges`(root)도 같은 이유로 막히고, `osascript`로 Finder를 시켜도 거부는 Finder가 아니라 **호출자인 osascript에 귀속**된다. 이 상태에서 본 "권한 없음"은 진단 근거가 아니다.

## 핵심 정리

- **TCC 판정은 프로세스 사슬(responsible process) 기준이다.** `claude` → `zsh` → `osascript` → Finder AppleEvent여도 커널 로그는 `System Policy: osascript(PID) deny(1) file-write-unlink (로컬 경로)`로 찍힌다. Finder가 직접 하는 작업과 다르다.
- **root도 예외가 아니다.** `sudo rm -rf (로컬 경로)`를 Full Disk Access 없는 터미널에서 치면 `rm: … Operation not permitted`. 암호를 제대로 넣어도 같다.
- **`mv`로 휴지통에 넣는 건 되는데 `ls (로컬 경로)`는 막힌다.** 디렉터리 열람(read)이 보호 대상이라 `stat`·`mv` 대상 지정은 통과한다. 그래서 "일부는 되고 일부는 안 된다"로 보인다.
- **파일별 예외가 있다.** `com.apple.macl` xattr(과거에 접근 허용된 앱 기록)이 붙은 파일은 같은 폴더 안에서도 옮겨진다 — 42개 중 1개만 이동된 이유였다.
- **진짜 원인 확인 순서**: ① `log stream --predicate 'eventMessage CONTAINS "<파일명>"'`를 켜고 재현 → `Sandbox … deny` 주체가 내 프로세스면 TCC ② 사용자에게 Finder가 띄운 **오류 문구 원문**을 받는다. ③ 그 다음에야 파일 플래그(`ls -laO`의 `uchg`/`restricted`/`dataless`)·열린 핸들(`lsof +D`)을 본다.
- **Finder "Trash can't be opened … being used by another task"** = Finder 내부 작업(이동·비우기)이 걸려 있는 상태. `killall Finder`로 재시작하면 풀리고 그 뒤 비우기가 정상 동작했다. AppleScript로 휴지통 항목을 여러 번 옮기려 시도한 뒤 나타났으므로, **osascript로 Finder 휴지통을 건드리는 진단은 하지 않는다.**
- 해법은 하나다: 시스템 설정 → 개인정보 보호 및 보안 → **Full Disk Access에 터미널 앱(Ghostty/Terminal)을 추가하고 앱을 완전히 재시작**. 설정 창은 `open "x-apple.systempreferences:com.apple.preference.security?Privacy_AllFiles"`로 바로 연다.

## 기록
### 2026-09-22 — 휴지통의 hyunji-birthday 폴더가 안 지워진다
- 맥락: [[작업노트/도구/Ghostty 설정|Ghostty]] 안의 Claude Code `co` 세션에서 맥 정리(안 쓰는 앱·캐시 삭제) 뒤 홍이 "휴지통의 폴더가 왜 안 지워지나" 질문
- 배운 것:
  - `ls (로컬 경로)` → `Operation not permitted`. `sudo rm -rf`(Terminal, 암호 입력)도 동일. 폴더 `stat`은 되고 `mv … (로컬 경로)`도 됨.
  - `log stream`에 `kernel (Sandbox) System Policy: osascript(73460) deny(1) file-write-unlink (로컬 경로)` — Finder를 시켰는데 거부 주체는 osascript.
  - Finder로 옮기기를 시도한 파일 중 `com.apple.macl` xattr가 있는 `26.1.0.JPG` 하나만 이동됨. 다른 xattr(`com.apple.cscachefs`, `com.apple.dataprotection.policy.exception-applied-by: com.apple.mediastream.mstreamd`, `com.apple.quarantine …;sharingd;` = AirDrop)는 원인과 무관.
  - 최종적으로 홍의 Finder가 "Trash can't be opened right now because it's being used by another task"를 띄움 → `killall Finder` 후 비우기 성공. 처음 Finder가 왜 걸렸는지는 미확정(iCloud Desktop 동기화 중인 폴더였음).
- 근거: 이 세션의 `log stream` 출력, `xattr -l` 결과, `brctl status`(Desktop iCloud 동기화 활성)

### 2026-09-23 — launchd 작업은 `(로컬 경로)`을 못 읽는다 (exit 127)

- 맥락: [[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]] 릴스 밤 작업을 launchd에 걸었더니 `launchctl print`에 `last exit code = 127`, 로그에 `/bin/zsh: can't open input file: (로컬 경로)` — [[프로젝트/개인/인스타카드뉴스/릴스 발행 시작 2026-09-23|릴스 발행 시작]]
- 배운 것:
  - launchd가 띄운 `/bin/zsh`·python은 TCC상 `(로컬 경로)`(여기선 iCloud 동기화 중) 접근 권한이 없다. 대화형 터미널에서 되는 것과 다르다. **옛 `com.instacardnews.publish`·`.report`가 `last exit code = 127`로 죽어 있던 것도 같은 원인으로 보인다**(09-12 인프라 실측에서 127만 기록됨)
  - 해법은 권한을 주는 대신 **실행본을 보호되지 않은 곳에 두는 것**: 이 Mac의 멀쩡한 launchd 작업은 전부 `(로컬 경로)`·`(로컬 경로)`에서 돈다. 릴스는 `(로컬 경로)`에 코드·폰트·`.env`(600)·venv를 복사하고 plist 템플릿을 채우는 `scripts/install_reel_runtime.sh`를 만들었다
  - 확인은 일회성 launchd 작업(`RunAtLoad true`)으로 한다 — 같은 환경에서 `claude -p`(키체인 OAuth)·`uvx`·mise shim `node`·`hermes send`가 모두 됐다
- 근거: `launchctl print gui/501/com.instacardnews.reel`, `output/reel-launchd.log`(실행본), 설치 후 수동 kickstart `exit 0`
