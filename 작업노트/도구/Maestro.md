---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-27
updated: 2026-09-12
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

### 2026-09-09 — 접근성 트리가 빈 화면은 좌표 탭인데, 좌표를 틀리면 **다른 앱**이 눌린다

- 맥락: 보험찾개냥 SSH-457(PR #119) Mock 실기동. 07a까지 가려면 온보딩(펫 등록 → 보험 등록)을 통과해야 했다
- 배운 것:
  - **화면마다 다르다.** 온보딩 1단계(펫)는 `tapOn: "강아지"`·`"다음"`이 그대로 먹었는데, **보험 등록 화면은 `maestro hierarchy`가 통째로 비어 있었다**(`accessibilityText`가 루트 말고 전부 빈 문자열) — 08-27에 기록한 「iOS 접근성 트리 공백」과 같은 계열이다. 텍스트 셀렉터가 하나도 안 잡힌다
  - **좌표 탭은 백분율이 화면 pt 기준이다.** 스크린샷은 `1206×2622`px(@3x = `402×874`pt)이고, `sips -Z 700`으로 줄여 눈으로 잰 y는 **`y/700×874`로 환산**해야 맞는다. 눈대중으로 4%를 틀렸더니(47% vs 43%) 필드 사이 빈 곳을 눌러 **아무 일도 안 일어났고**, 실패가 조용해서 원인을 찾는 데 시간이 들었다
  - **가장 비싼 실수는 조용한 실패가 아니라 오탭이다.** 탭이 안 먹은 줄 알고 다음 좌표를 이어 눌렀더니 앱이 백그라운드로 간 사이 **카메라 앱과 다른 프로젝트 앱(즉석카메라)이 열렸다.** 시뮬레이터는 앱 경계를 안 지켜 준다 — **한 스텝마다 스크린샷으로 현재 화면을 확인하고 다음 좌표를 정한다**
  - 대안이 있으면 그쪽이 싸다 — 이 건은 **라우터에 화면을 실물로 걸어 도는 위젯 테스트**(`claimant_info_screen_test.dart`)로 배선을 고정했고, 시뮬레이터는 «눈으로 보는 것»에만 남겼다
- 근거: PR #119 · `docs/spec/SSH-457/tasks.md` 「Mock 실기동」 항목의 중단 기록

### 2026-09-12 — 보험찾개냥 09 결과 입력(SSH-434) iOS 주행 완주 — 정규식 셀렉터·좌표·키패드
- 맥락: 보험찾개냥 SSH-434(PR #134) `main_mock` 실기동 + 스크린샷 10장. 온보딩 통과는 Mock `isOnboarded`를 **임시로 true**로 바꿔 건너뛰었다(찍고 되돌림 — 02b 접근성 공백을 피하는 가장 싼 길).
- 배운 것:
  1. **`Semantics`로 묶인 카드는 라벨이 합쳐진다** — `AppSelectableCard`(제목+설명, `inMutuallyExclusiveGroup`)는 `tapOn: "지급 완료"`가 FAILED, **`tapOn: ".*지급 완료.*"`(정규식)** 로 잡힌다. 텍스트 셀렉터는 전체 일치라서다(8/30 WKWebView 기록과 같은 규칙이 Flutter Semantics에도 적용).
  2. **홈 청구 카드(`AppClaimCard`) 텍스트는 트리에 없다** — `tapOn: "행복동물병원"` FAILED. 좌표 탭(`point: "50%,48%"`)으로. 반면 `AppButton` 라벨·`AppTopBar` 뒤로가기(`Semantics(label: '뒤로가기')`)·`AppSectionTitle`·다이얼로그 「확인」은 잡힌다 → `scrollUntilVisible: element: "결과 알려주기"`도 된다.
  3. **숫자 키패드에서 `hideKeyboard`는 FAILED다**(완료 키가 없다). `AppDismissKeyboard` 덕에 빈 곳 탭으로 내리는데, **키보드 영역을 피해야 한다** — 62%는 키보드를 눌렀고(아무 일 없음) 46%가 맞았다. 스크린샷(1206×2622)에서 키보드 상단 ≈ 50%.
  4. `flutter run`을 백그라운드로 띄우면 「Lost connection to device」로 호스트가 떨어져도 **앱은 시뮬레이터에 살아 있다**(8/29 기록과 같음) — Maestro `launchApp`으로 다시 띄우면 된다. `timeout` 명령은 macOS에 없다.
- 근거: `docs/spec/SSH-434/tasks.md` 실기동 표, 플로우 파일은 세션 스크래치(`maestro/f*.yaml`) — 레포에는 안 남겼다.
- 추가(같은 날): **Android 에뮬레이터(`emulator-5554`)에서도 같은 플로우가 그대로 돈다** — `건너뛰기`·`카카오 로그인`·`뒤로가기`·정규식 카드 셀렉터·`scrollUntilVisible` 전부 동일, 홈 카드 좌표(50%,48%)도 같았다(1080×2400). 캡처는 `adb exec-out screencap -p >`. SDK는 `(로컬 경로)`(표준 경로 아님).
- 추가(9/12 저녁): **iOS에서 `launchApp` 없는 플로우가 끝나면 앱이 백그라운드로 가 있을 수 있다**(스크린샷이 스프링보드) — `xcrun simctl launch <udid> <bundleId>`로 앞으로 올리면 상태 그대로 돌아온다(재시작 아님). 제목만 있는 `AppSelectableCard`는 정규식·정확 텍스트 둘 다 앱이 백그라운드면 FAILED — 먼저 앞으로 올릴 것.

### 2026-09-13 — 사진 선택기(PHPicker·Android Photo Picker)를 지나 업로드 순간을 찍기 (보험찾개냥 SSH-557)

- **`takeScreenshot`은 절대 경로를 거부한다** — `Invalid path "/tmp/x.png" ... resolves outside this run's takeScreenshot output folder`. 이름만 주면 `(로컬 경로)`에 남는다. 실패한 스텝의 자동 캡처는 같은 폴더의 `screenshots/step-NNN-*.png`
- `extendedWait`라는 커맨드는 없다(`Invalid Command`). 고정 대기는 `waitForAnimationToEnd: {timeout: N}`
- **iOS PHPicker**: 「앨범에서 선택」 뒤 시뮬레이터 기본 사진 그리드가 뜬다. 첫 장은 `tapOn: point: "17%,25%"`(3열 그리드 첫 칸). 탭 직후 바로 돌아오고 앱은 그대로 포그라운드
- **Android Photo Picker**: 에뮬레이터에 사진이 없으면 빈 그리드다 — `adb push x.jpg /sdcard/Pictures/` + `am broadcast -a android.intent.action.MEDIA_SCANNER_SCAN_FILE -d file:///sdcard/Pictures/x.jpg`로 심으면 뜬다. 접근성 배너("will only have access to the photos you select") 아래 그리드 첫 칸이 대략 (180,1400)px(1080×2400). Maestro 없이 `adb shell input tap`으로도 된다
- **1~2초짜리 전이 상태(업로드 중 스피너)는 Maestro 스텝 사이로 못 잡는다** — 스텝 지연이 상태보다 길다. 대신 **탭을 백그라운드로 던지고 셸 루프로 0.25초마다 스크린샷**해 md5로 바뀐 프레임만 남긴다: iOS `xcrun simctl io <udid> screenshot`, Android `adb exec-out screencap -p`. Android는 3번째 프레임에 잡혔고, iOS는 PHPicker 닫힘 애니메이션(~1s)이 Mock 지연(1.2s)을 거의 다 먹어 못 잡았다 — 잡으려면 Mock 지연을 3초쯤으로 늘려 빌드해야 한다
- Semantics로 합쳐진 카드(`AppSummaryCard`)는 `tapOn: "보호자 정보를 등록해 주세요"`가 FAILED — 여전히 좌표 탭(`50%,32%`)이다(9/12와 같은 함정)
