---
type: study
area: Flutter
audience: ai
status: active
created: 2026-09-12
updated: 2026-09-12
projects:
  - "보험찾개냥"
---

# 위젯 테스트와 ListView 지연 빌드

## 핵심 정리

- `testWidgets`의 기본 뷰포트는 **800×600 논리 픽셀**. `ListView`는 보이는 범위(+ `cacheExtent` 기본 250)만 자식을 만들기 때문에, 화면 아래쪽 위젯은 **빌드 자체가 안 돼** `find.text(...)`가 `findsNothing`이다 — 렌더링 실패가 아니라 존재하지 않는 것.
- 스크롤 대신 뷰포트를 키우는 게 싸다: `tester.view.physicalSize = const Size(800, 2400); tester.view.devicePixelRatio = 1; addTearDown(tester.view.reset);` — `reset`이 둘 다 되돌린다.
- `ChangeNotifier` ViewModel이 생성자에서 `unawaited(load())`를 부르고 Fake Repository가 `async`로 즉시 값을 주면, `pumpWidget` 뒤 **`await tester.pump()` 한 번**이면 로드 상태다. `pumpAndSettle`은 로딩 위젯에 무한 애니메이션이 있으면 안 끝난다.

## 기록

### 2026-09-12 — 보험찾개냥 SSH-552 09 청구 상세 반려 줄 위젯 테스트

- 맥락: AI 리뷰 P2(반려 상태 스크린샷 없음)를 위젯 테스트로 대체. 결과 카드(`_OutcomeCard`)가 `ListView` 맨 아래라 기본 뷰포트에서는 안 만들어질 위치.
- 근거: `Client/test/ui/claim_detail/claim_detail_screen_test.dart`(`6d870f6`, PR #129). 뷰포트를 세로 2400으로 키우고 `pump()` 한 번으로 4건 통과.
