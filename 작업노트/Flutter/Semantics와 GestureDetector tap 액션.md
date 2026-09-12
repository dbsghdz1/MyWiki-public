---
audience: ai
분야: Flutter
---

# Semantics와 GestureDetector tap 액션

## 핵심 정리

- `GestureDetector`는 `onTap`이 null이어도 **`onTapDown`·`onTapUp`·`onTapCancel` 중 하나라도 있으면 시맨틱에 tap 액션을 싣는다.** 그래서 "비활성" 상태에서 `onTap`만 떼고 `Semantics(enabled: false)`를 줘도 스크린 리더는 `actions: [tap]`인 버튼으로 읽는다
- 해결: tap 액션은 `Semantics(button: true, enabled: ..., onTap: ...)`가 소유하고, `GestureDetector(excludeFromSemantics: true)`로 터치만 맡긴다. 눌림 색을 위한 up·cancel 핸들러는 그대로 둘 수 있다
- `matchesSemantics`에서 `isEnabled`·`hasTapAction`을 **안 주는 것이 곧 "그 플래그·액션이 없다"**는 단언이다. `isEnabled: false`를 명시하면 `avoid_redundant_argument_values`가 잡는다

## 기록

### 2026-09-13 — 보험찾개냥 `AppUploadRow(busy:)` (SSH-557 #138 AI 리뷰 P2)

- 올리는 중 줄을 `onTap: null` + `Semantics(enabled: !busy)`로 만들었더니 테스트 `matchesSemantics(isButton: true, hasEnabledState: true)`가 `unexpected actions: [tap]`로 실패. 원인은 남아 있던 `onTapUp`/`onTapCancel`. 위 해결로 초록 — `Client/lib/ui/core/ui/app_upload_row.dart`, 커밋 `a47096a`
