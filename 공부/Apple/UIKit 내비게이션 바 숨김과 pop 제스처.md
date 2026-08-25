---
type: study
area: 언어·프레임워크
audience: ai
status: active
created: 2026-08-22
updated: 2026-08-22
projects:
  - "[[프로젝트/개인/DayTune/README|DayTune]]"
---

# UIKit 내비게이션 바 숨김과 pop 제스처

`setNavigationBarHidden(true)`를 하면 **엣지 스와이프 pop이 같이 죽는다.** 버그가 아니라 설계다 — `interactivePopGestureRecognizer`의 delegate가 기본으로 `UINavigationController` 자신인데, 그 delegate 로직이 "내비게이션 바에 back 버튼이 보이는가"를 기준으로 제스처 시작을 허용한다. 바가 없으면 back 버튼도 없으니 항상 거부한다.

## 핵심 정리

- **고치는 법은 delegate를 빼앗는 것 하나다.** `UINavigationController` 서브클래스에서 `interactivePopGestureRecognizer?.delegate = self`로 잡고, `gestureRecognizerShouldBegin`에서 `viewControllers.count > 1 && transitionCoordinator == nil`을 돌려준다. 두 조건이 빠지면 (1) 루트에서 스와이프 시 화면이 먹통이 되거나 (2) push 애니메이션 중 스와이프로 내비게이션 스택이 꼬인다.
- **바 숨김은 한 곳에서만 한다.** 화면마다 `viewWillAppear`에서 숨기고 `viewWillDisappear`에서 다시 켜는 패턴은 push/pop 전환 중 바가 한 프레임 나타났다 사라지며 **상단 레이아웃이 튄다**(safe area가 바 높이만큼 순간 변해 스크롤 뷰 inset이 흔들림). 커스텀 헤더를 전면 사용하는 앱이면 `UINavigationController` 서브클래스의 `viewDidLoad`에서 한 번 숨기고 끝낸다.
- **숨긴 바 아래의 스크롤 뷰는 `contentInsetAdjustmentBehavior = .never`.** 기본 `.automatic`은 내비게이션 컨트롤러 안의 스크롤 뷰에 safe-area 상단 inset을 넣는데, 커스텀 헤더 밑에 붙은 스크롤 뷰는 이미 safe area 아래라 불필요한 여백·점프의 원인이 된다.
- `childForStatusBarStyle`을 `topViewController`로 넘겨야 화면별 상태바 스타일이 먹는다 — 바가 숨겨지면 기본 위임 경로가 사라지기 때문.

## 기록

### 2026-08-22 — "push된 상태에서 제스처로 pop이 안 되고 스크롤할 때 위쪽이 이상하다"
- 맥락: [[프로젝트/개인/DayTune/README|DayTune]]. 7개 화면이 각자 `viewWillAppear`에서 바를 숨기고 HealthConnection만 `viewWillDisappear`에서 다시 켜고 있었다. delegate 설정은 어디에도 없음.
- 수정: `Core/Navigation/DTNavigationController.swift` 신설(바 숨김 1회 + pop 제스처 delegate), 7개 VC의 토글 제거, `AppCoordinator`가 서브클래스 사용, TodayPlan 스크롤 뷰 `.never`. 시뮬레이터에서 홈·오늘의 계획·설정·추천 상세 렌더링 동일 확인. 제스처는 실기기 확인 필요.

## 참고 자료

- Apple — UINavigationController.interactivePopGestureRecognizer: https://developer.apple.com/documentation/uikit/uinavigationcontroller/interactivepopgesturerecognizer (2026-08-22 확인)
- Apple — UIScrollView.ContentInsetAdjustmentBehavior: https://developer.apple.com/documentation/uikit/uiscrollview/contentinsetadjustmentbehavior-swift.enum (2026-08-22 확인)
