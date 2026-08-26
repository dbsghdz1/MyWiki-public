---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-26
updated: 2026-08-26
projects:
  - "탭탭"
---

# macOS 앱 QA 자동화

한 줄 요약 — 빌드한 맥 앱을 스크립트로 클릭·호버·캡처해 QA하는 법. 시뮬레이터가 없는 macOS에서 실제 앱을 돌려보는 유일한 수단이다.

## 핵심 정리

- **`CGEvent`로 만든 마우스 다운/업은 `mouseEventClickState`를 1로 세팅하지 않으면 클릭으로 인식되지 않는다.** 기본값 0은 "클릭이 아님"이라 AppKit/SwiftUI 버튼이 무시한다. **호버(`mouseMoved`)는 멀쩡히 동작하기 때문에** "이벤트는 가는데 버튼만 안 눌린다"로 보여 앱 버그로 오진하기 쉽다.
- **다운과 업 사이에 60ms쯤 두고, 그 전에 `mouseMoved`를 한 번 보낸다.** 즉시 연속으로 쏘면 뷰가 트래킹을 못 잡는 경우가 있다.
- **`screencapture -R x,y,w,h`는 그 화면 영역을 찍을 뿐 앱 창을 찍지 않는다.** 다른 창이 위에 있으면 그게 찍힌다. 창만 확실히 찍으려면 `screencapture -x -o -l <windowID>`.
- **windowID는 `CGWindowListCopyWindowInfo`로 얻는다.** pyobjc가 없는 macOS 기본 python3로는 못 하니 Swift 한 파일을 `swiftc`로 말아 두고 쓴다. `kCGWindowBounds`가 그대로 클릭 좌표계(주 디스플레이 좌상단 원점)라 **창 좌표 + 원점 = 클릭 좌표**로 바로 매핑된다.
- **매 조작 직전에 앱을 활성화한다.** 캡처·셸 명령이 포커스를 가져가면 창이 비활성 상태가 되고, 그러면 호버가 아예 안 걸린다. `osascript -e 'tell application "TapTapMac" to activate'`.
- **`osascript ... keystroke "한글"`은 한글을 입력하지 못한다** (알파벳으로 떨어진다). 클립보드에 넣고 `keystroke "v" using command down`으로 붙여넣는다.
- **창 리사이즈는 System Events로**: `tell process "X" to set size of window 1 to {640, 450}`. 앱의 `minWidth/minHeight`보다 작게 요청하면 그 값으로 잘린다 — 최소 크기 검증에 그대로 쓸 수 있다.
- **판정은 눈이 아니라 픽셀로.** "딤이 사이드바를 안 덮은 것 같다", "모달이 반투명한 것 같다" 같은 인상은 스크린샷에서 자주 틀린다. `NSBitmapImageRep.colorAt(x:y:)`로 rgb를 찍어 비교하면 오진이 사라진다. Retina 캡처는 2배이므로 **포인트 좌표 × (픽셀폭/포인트폭)** 으로 환산한다.

## 기록

### 2026-08-26 — TapTapMac QA 하네스 (탭탭)

- 맥락: 탭탭 macOS 앱을 실행해 QA하려는데 시뮬레이터가 없어 실제 앱을 스크립트로 조작해야 했다.
- 배운 것: 위 「핵심 정리」 전부. 특히 **클릭이 안 먹던 30분이 전부 `mouseEventClickState` 때문**이었고, 그 사이 "사이드바 카테고리 클릭이 동작하지 않는다"를 앱 버그로 적을 뻔했다.
- 근거: 아래 스크립트로 2026-08-26 QA의 결함 6건을 재현·검증했다.

빌드부터 조작까지의 전체 흐름:

```bash
# 1) 빌드 (Tuist 프로젝트)
tuist generate --no-open
xcodebuild -workspace TapTap.xcworkspace -scheme TapTapMac \
  -configuration Debug -destination 'platform=macOS' \
  -derivedDataPath ./dd build CODE_SIGNING_ALLOWED=NO

# 2) 실행 (-n: 이미 떠 있어도 새 인스턴스)
open -n ./dd/Build/Products/Debug/TapTapMac.app
```

windowID 조회 (`swiftc -O winid.swift -o winid`):

```swift
import CoreGraphics
let list = CGWindowListCopyWindowInfo([.optionOnScreenOnly, .excludeDesktopElements],
                                      kCGNullWindowID) as! [[String: Any]]
for w in list where (w[kCGWindowOwnerName as String] as? String ?? "").contains(name) {
  let b = w[kCGWindowBounds as String] as! [String: Any]
  print(w[kCGWindowNumber as String] as! Int, b["X"]!, b["Y"]!, b["Width"]!, b["Height"]!)
}
```

클릭 (`swiftc -O click.swift -o click`) — **`setIntegerValueField(.mouseEventClickState, value: 1)`이 핵심**:

```swift
let p = CGPoint(x: x, y: y)
CGEvent(mouseEventSource: nil, mouseType: .mouseMoved,
        mouseCursorPosition: p, mouseButton: .left)?.post(tap: .cghidEventTap)
usleep(50_000)
for type in [CGEventType.leftMouseDown, .leftMouseUp] {
  let e = CGEvent(mouseEventSource: nil, mouseType: type,
                  mouseCursorPosition: p, mouseButton: .left)!
  e.setIntegerValueField(.mouseEventClickState, value: 1)   // ← 없으면 무시된다
  e.post(tap: .cghidEventTap)
  usleep(60_000)
}
```

호버는 `mouseMoved` 하나로 충분하지만 **여러 좌표를 거쳐 이동**시키는 편이 안정적이다. 드래그는 `.leftMouseDragged`를 10여 단계로 나눠 보낸다.

픽셀 판정:

```swift
let rep = NSBitmapImageRep(cgImage: cg)
let scale = Double(cg.width) / img.size.width      // Retina면 2
let c = rep.colorAt(x: Int(px * scale), y: Int(py * scale))!
```

이 판정으로 **"모달 딤이 사이드바를 안 덮는다"(→ 실제로는 255→160으로 덮고 있었음)** 와 **"다크 모드에서 모달이 반투명하다"(→ 실제로는 카드와 딤 배경 명도가 같아서 경계가 안 보이던 것)** 를 각각 오진과 진짜 결함으로 갈랐다.

## 참고 자료

- [CGEvent — Apple Developer](https://developer.apple.com/documentation/coregraphics/cgevent) — `mouseEventClickState` 등 이벤트 필드 정의 (2026-08-26 확인)
- [CGWindowListCopyWindowInfo(_:_:) — Apple Developer](https://developer.apple.com/documentation/coregraphics/cgwindowlistcopywindowinfo(_:_:)) — windowID·bounds 조회 (2026-08-26 확인)
