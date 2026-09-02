---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-27
updated: 2026-09-02
projects:
  - "보험찾개냥"
---

# Maestro

## 핵심 정리

- 모바일 UI 자동화(YAML 플로우). 설치는 `curl -fsSL https://get.maestro.mobile.dev | bash` → `(로컬 경로)`. iOS 시뮬레이터는 `--device <UDID>`로 XCTest 러너를 자동 설치해 조작한다 — AppleScript·접근성 권한 불필요.
- **텍스트 셀렉터(`tapOn: "문구"`)는 접근성 트리 의존이다.** Flutter 앱이라도 트리에 있는 텍스트는 잡지만, [[작업노트/Flutter/iOS 접근성 트리 공백|트리가 비는 화면]]에서는 전부 실패한다. Flutter TextField의 **placeholder 힌트는 트리에 없어** `tapOn`이 안 된다 — 필드는 좌표 탭이 필요.
- **좌표 탭(`point: "50%,28%"`)은 요소를 기다리지 않는다** — 화면 전환·애니메이션 중에 날아가 허공을 때리고도 COMPLETED로 넘어간다. 좌표 탭 앞엔 반드시 `waitForAnimationToEnd`나 텍스트 기반 `extendedWaitUntil`을 둔다. 검증 없는 좌표 체이닝은 한 번 어긋나면 이후 입력이 엉뚱한 필드에 쏟아진다(실측: 07a용 이름이 04c 진료 내용 칸에 입력됨).
- 실패 시 `(로컬 경로)`에 스텝별 스크린샷·`screen-hierarchy` JSON·`maestro.log`가 남는다 — 디버깅 정본. 라이브 트리는 `maestro hierarchy`.
- Android 대안: adb는 트리 무관(좌표+`input text`)이지만 **한글 입력 불가**·`keyevent 4`(back)가 키보드 없을 때 화면을 pop하는 함정. Maestro `inputText`는 유니코드 가능.

## 기록

### 2026-09-01 — 같은 시뮬레이터를 다른 세션이 쓰면 그 앱의 시스템 알림이 주행을 깬다

- 맥락: 한능검 2.0 콘텐츠 최종 회귀 주행(`maestro/record.sh v2.0-content-complete`). 첫 단계 `assertVisible: "한국사 정복"`이 실패했는데 앱은 정상 렌더 상태였다.
- 원인: 디버그 스크린샷에 **다른 프로젝트(WristNote)의 음성 인식 권한 알림**이 우리 앱 위에 떠 있었다. 계층 로그가 `Find the Application 'com.apple.springboard'` — 시스템 알림이 포그라운드를 가져가면 Maestro는 스프링보드 계층을 잡아 앱 텍스트를 못 찾는다. 같은 iPhone 17 Pro(E6EFBDB2…)를 다른 세션이 동시에 쓰고 있었다.
- 해법: 주행 전용 시뮬레이터를 분리한다 — `UDID=<다른 기기> record.sh …`. 실패 로그가 springboard를 가리키면 앱 회귀보다 먼저 겹친 알림을 의심할 것.
- 근거: `web/maestro/shots/20260901-2117-v2.0-content-complete/.maestro/tests/2026-09-01_211716/flow/screenshots/step-004-*.png`.

### 2026-08-30 — Capacitor(WKWebView) 앱은 텍스트 셀렉터로 완주된다, 단 세 가지 함정

- 맥락: [[프로젝트/개인/한능검/README|한능검]] v1.1 — UI 변경마다 핵심 플로우를 스크린샷으로 기록하는 `maestro/record.sh` 구축. Flutter와 달리 **WKWebView는 접근성 트리가 충실해** 좌표 탭 없이 10스텝을 완주했다.
- 배운 것:
  1. **텍스트 셀렉터는 정규식 "전체 일치"다.** WKWebView는 버튼 안 span들을 한 라벨로 합치므로(`제79회 48문항`, `① 주로 동굴이나…`) 부분 문자열은 `.*`로 감싸야 한다. 괄호 등 정규식 문자가 든 발문은 그대로 못 쓴다.
  2. 반대로 **React 보간(`{n}회차`)은 텍스트를 별개 노드로 조각내** `기본 14회차`가 어떤 요소와도 전체 일치하지 않는다 — 조각 하나(`640`)를 잡는다. 합쳐질지 조각날지는 hierarchy 덤프로 확인하는 게 빠르다.
  3. `takeScreenshot`은 cwd가 아니라 **디버그 출력 폴더에 저장된다** — `--debug-output <dir>`로 받아 `*/takeScreenshot/*.png`를 꺼낸다.
  4. **탭이 COMPLETED인데 아무 일도 안 일어나면 오버레이가 삼킨 것이다.** 실측: 하단 고정 유리 캡슐(`pointer-events:auto`)이 그 뒤 강의 카드의 탭을 먹었다 — 도구 문제가 아니라 **실제 UX 버그**(사용자도 못 누른다)라서 앱을 고치는 게 답이었다.
- 근거: `(로컬 경로)` · 기록 `maestro/shots/20260830-*`

### 2026-08-27 — 보험찾개냥 iOS 07b 스크린샷 자동화 시도

- 목표: 04c→07b 자동 주행 후 캡처. 텍스트 셀렉터로 홈까지는 완주했으나 04c에서 접근성 트리 공백에 막혔고, 전면 좌표 전환은 드리프트로 3회 실패 — 최종적으로 사람이 이동하고 캡처만 자동화했다.
- 교훈: **Flutter 앱에 Maestro를 제대로 쓰려면 시맨틱스가 먼저다.** 화면별 Semantics 보강(특히 오버레이 화면·TextField 라벨) 없이는 좌표 놀음이 된다. E2E 자산화하려면 그 선행 작업부터.

### 2026-09-02 — 기기가 둘 이상이면 `--device`가 필수다

- 맥락: 소프트웨어마에스트로 SSH-441 — 대표 상태 3장(iOS 라이트·다크·Android)을 찍으려고 iOS 시뮬과 Android 에뮬을 동시에 띄운 상태
- 증상: 시뮬레이터에서 잘 되던 같은 flow가 갑자기 `Tap on "건너뛰기"... FAILED`로 죽었다. **앱은 멀쩡히 그 화면에 있었다**
- 원인: 그사이 Android 에뮬레이터가 부팅을 마쳤고, `maestro test`가 **그쪽을 잡았다.** 어느 기기를 골랐는지는 `Running on …` 한 줄에만 나온다
- 해법: `maestro --device <UDID 또는 emulator-5554> test flow.yaml`. **`--device`는 `test` 앞에 온다**
- 함께 겪은 것: `takeScreenshot: /tmp/…`는 `it resolves outside this run's takeScreenshot output folder`로 거절된다 — 절대경로를 못 쓴다. 그냥 `xcrun simctl io <udid> screenshot` / `adb exec-out screencap -p >` 로 찍는 게 빠르다

