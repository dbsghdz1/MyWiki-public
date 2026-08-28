---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-22
updated: 2026-08-22
projects:
  - "[[프로젝트/개인/DayTune/README|DayTune]]"
---

# Swift 동시성과 UIKit 메인 스레드

`Task { await something(); navigate() }`는 **`navigate()`가 메인에서 돈다는 보장이 전혀 없다.** `await` 뒤에 이어지는 코드는 "호출한 스레드"가 아니라 **마지막으로 재개시킨 실행자(executor)** 에서 실행된다. `@MainActor`가 아닌 클래스 안의 비동기 함수는 기본적으로 협력 스레드 풀(generic executor)에서 재개되므로, 그 자리에 UIKit 호출(`setViewControllers`, `pushViewController`, 뷰 갱신)이 있으면 백그라운드에서 UIKit을 건드리는 것이 된다.

## 핵심 정리

- **`Task {}`는 감싼 컨텍스트의 actor를 물려받는다** — 그런데 물려받을 게 있을 때만이다. `UIViewController`의 메서드 안에서 만든 `Task`는 VC가 `@MainActor`라 메인에서 시작하지만, 거기서 `await viewModel.foo()`로 **non-isolated 클래스의 async 메서드**에 들어가면 그 메서드 본문은 제네릭 실행자로 넘어가고, 그 안에서 부르는 모든 동기 호출(콜백·클로저·코디네이터)도 백그라운드다.
- **증상은 바로 크래시가 아니라 조용한 스레드 위반**이다. 대부분은 Main Thread Checker 보라색 경고로 끝나지만, `UINavigationController.setViewControllers` 같은 경로는 내부에서 `NSAssertionHandler`가 걸려 **SIGABRT**로 죽는다. 크래시 로그 특징: `-[UIApplication _performAfterCATransactionCommitsWithLegacyRunloopObserverBasedTiming:block:]` → `handleFailureInMethod` → `objc_exception_throw`. 이 스택이 보이면 "UIKit을 메인 밖에서 불렀다"로 읽으면 된다.
- **가장 안전한 고치기**는 ViewModel 자체를 `@MainActor`로 선언하는 것이다. 그러면 `await useCase.execute()` 뒤의 코드가 **자동으로 메인으로 돌아온다**(await 경계에서 actor hop). 국소 수정이면 `await MainActor.run { finish() }`. `DispatchQueue.main.async`도 동작하지만 구조적 동시성 안에서는 actor 방식이 일관된다.
- HealthKit `requestAuthorization(toShare:read:)`의 async 버전은 **완료 콜백을 백그라운드 큐에서 돌려준다.** 완료 직후 화면 전환을 하는 흐름은 거의 항상 이 함정에 걸린다.
- **왜 "가끔"만 죽나**: `await` 이전에 이미 끝난 작업(캐시 히트, 권한이 이미 허용됨)은 suspension 없이 동기적으로 지나가 메인에 남기도 한다. "처음 설치했을 때만 죽는다"가 전형적 증상이다.

## 기록

### 2026-08-22 — 건강 앱 연결 → 권한 허용 직후 SIGABRT
- 맥락: [[프로젝트/개인/DayTune/README|DayTune]] CTA 버튼 디자인 수정 후 "건강앱연결에서 크래시 난다" 보고. 디자인 변경과 무관한 기존 버그였다.
- 근거: `(로컬 경로)` — `HealthConnectionViewModel.requestHealthKitPermission()` 안에서 `try await requestHealthAuthorizationUseCase.execute()` 뒤 `finishHealthConnection()` → `onFinish` → `AppCoordinator.showSleepAnalyzing()` → `setViewControllers(_:animated:)` → `NSAssertionHandler` → SIGABRT. 시뮬레이터(iPhone 17 Pro, iOS 26.5) 새 설치 → CTA 탭 → 허용에서 100% 재현.
- 수정: `await MainActor.run { finishHealthConnection() }` (프로젝트 기존 관례 — `HomeViewModel`·`SleepAnalyzingViewModel`도 같은 방식). 수정 후 같은 경로로 "수면 데이터 없음" 화면까지 정상 진입 확인.
- 덤: 시뮬레이터 자동화 중 macOS에는 GNU `timeout`이 없어 `timeout 30 cmd`가 **command not found로 조용히 아무것도 안 한 채 "성공"처럼 보였다.** 샌드박스 안에서 `simctl launch/io`가 멈춰 보인 것도 실제론 이 조합이었다 — macOS에서 시간 제한이 필요하면 백그라운드 실행 + `sleep` 또는 `gtimeout`(coreutils).

## 참고 자료

- The Swift Programming Language — Concurrency 장 (원본 소스): https://github.com/swiftlang/swift-book/blob/main/TSPL.docc/LanguageGuide/Concurrency.md (2026-08-22 확인)
- SE-0306 Actors: https://github.com/swiftlang/swift-evolution/blob/main/proposals/0306-actors.md (2026-08-22 확인)
- SE-0316 Global actors (`@MainActor`의 hop 규칙): https://github.com/swiftlang/swift-evolution/blob/main/proposals/0316-global-actors.md (2026-08-22 확인)
