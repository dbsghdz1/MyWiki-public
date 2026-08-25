---
type: study
area: 언어·프레임워크
audience: ai
status: active
created: 2026-08-18
updated: 2026-08-18
projects:
  - "탭탭"
---

# SwiftUI

한 줄 요약 — 레이아웃·히트 영역·이펙트가 "보이는 것"과 다르게 동작하는 지점들. macOS 앱을 만들며 하나씩 부딪힌 것.

## 핵심 정리

- **`.buttonStyle(.plain)` 버튼의 히트 영역은 라벨의 불투명 픽셀뿐이다.** 투명 배경 행·패딩 영역까지 눌리게 하려면 라벨 쪽에 `.contentShape(...)`를 준다. 배경을 Button 바깥에 두면 그 배경은 클릭 영역이 아니다.
- **`.blur(radius:)`는 네 가장자리를 전부 흐린다.** 한 방향 페이드 그라데이션에 블러를 얹으면 반대편·좌우 가장자리도 반투명해져 밑 콘텐츠가 비친다. 흐려도 되는 쪽만 남기려면 뷰를 블러 반경만큼 음수 패딩으로 부모 클립 밖으로 밀거나(`.padding(.bottom, -12)` + 부모 `clipShape`), 가려야 하는 쪽에 불투명 오버레이를 덧댄다.
- **고정 폭 텍스트는 잘린다.** 메뉴처럼 항목 길이가 가변인 곳은 `Text.fixedSize(horizontal: true, vertical: false)` + 컨테이너 `frame(minWidth:)` + `fixedSize(horizontal: true, vertical: false)`로 "최소 폭은 유지, 내용이 길면 늘어남"을 만든다.
- **SwiftUI에는 배경 블러(backdrop blur)가 없다.** `.blur`는 그 뷰 자체를 흐릴 뿐 뒤 콘텐츠는 그대로다. 피그마의 "Background blur"를 맞추려면 macOS에선 `NSViewRepresentable`로 레이어를 만들고 `layer.backgroundFilters = [CIAffineClamp, CIGaussianBlur]`를 건다 — `view.layerUsesCoreImageFilters = true`가 있어야 적용되고, `CIAffineClamp`을 앞에 두지 않으면 가장자리가 투명을 샘플링해 어두워진다. `hitTest`를 `nil`로 넘겨야 밑의 스크롤·클릭이 막히지 않는다. **`CIGaussianBlur.inputRadius`는 픽셀 단위**라 Retina에서 4를 주면 2pt로 먹는다 — `window.backingScaleFactor`를 곱해야 피그마 4px와 같아진다(`viewDidChangeBackingProperties`에서 재적용). `Material`은 시스템 틴트가 섞여 순수 블러가 아니다.
- **`.scrollIndicators(.hidden)`은 "숨김 권장"이지 강제가 아니다.** macOS에서 시스템 설정이 스크롤 막대 항상 표시면 그대로 보인다. 확실히 없애려면 `.never`. NSScrollView를 뒤져 스크롤러를 숨기는 우회는 macOS 26 SwiftUI ScrollView에선 효과가 없었다(NSScrollView 기반이 아닌 듯 — 추정).
- **알파만 있는 어두운 오버레이 토큰은 다크 모드에서 사라진다.** `#272146 @7%` 같은 호버 색은 흰 배경에선 회색, 검은 배경에선 검정 위 검정이라 안 보인다. 색 토큰에 다크 appearance 변형이 있는지(`Contents.json`의 `appearances`) 확인하고, 없으면 밝기 변형이 있는 중립색(n20 등)을 쓴다.

## 기록

### 2026-08-25 — 입력 없는 `AVCaptureSession`에 프리뷰를 붙이면 화면이 통째로 죽는다 (즉석카메라)
- 맥락: 시뮬레이터에서 카메라 권한을 부여하자 **앱 화면 전체가 검게** 나왔다. 로그에 `AttributeGraph: cycle detected`.
- 원인: **시뮬레이터엔 카메라가 없다.** `AVCaptureDevice.default(...)`가 nil이라 세션에 입력이 하나도 안 붙었는데, 코드는 권한만 보고 `AVCaptureVideoPreviewLayer`를 만들었다. 뷰파인더만 검어지는 게 아니라 **SwiftUI 렌더 자체가 멈춘다.**
- 배운 것: **권한(`authorizationStatus`)과 가용성(입력이 실제로 붙었는지)은 별개다.** 프리뷰를 만들기 전에 `!session.inputs.isEmpty`를 확인해야 한다. 실기기에선 안 나지만, 카메라를 못 잡는 상황에서 **사용자가 아무 설명 없는 검은 화면만 보게 되는 결함**이다.
- 근거: `(로컬 경로)` 커밋 `fa62033` (`CameraSession.hasCamera` 가드).

### 2026-08-25 — ~~TabView는 보이지 않는 탭도 미리 만든다~~ → **오진이었다. 시뮬레이터의 낡은 다이얼로그를 증거로 읽었다** (즉석카메라)
> [!warning] 아래 진단은 틀렸다 — 같은 날 로그로 반증됨
> **iOS 26의 `TabView`는 선택되지 않은 탭을 지연 생성한다.** `.task`에 디버그 로그를 넣어 보니 **한 번도 실행되지 않았다.**
> 권한 다이얼로그가 뜬 진짜 이유는 **`simctl`이 화면을 탭할 수 없어서, 최초 실행(기본 탭이 촬영일 때) 때 뜬 다이얼로그가 닫히지 않고 이후 모든 스크린샷에 계속 얹혀 있었기 때문**이다. 앱을 종료해도 남는다.
> **교훈: 시뮬레이터 스크린샷의 다이얼로그는 현재 실행의 것이 아닐 수 있다.** 권한은 `xcrun simctl privacy <dev> grant camera <bundle>`로 미리 부여하고 시작한다.
> 아래 가드 자체는 무해해서 코드에 남겨뒀다(다른 OS 버전에서는 유효할 수 있다).

- 맥락: [[프로젝트/개인/즉석카메라/README|즉석카메라]] v1 골격. 시뮬레이터에서 **앨범 탭을 보고 있는데 카메라 권한 요청이 떴다.**
- 배운 것: **`TabView`는 선택되지 않은 탭의 뷰도 미리 만든다.** 그래서 그 안의 `.task`·`.onAppear`가 **화면에 보이기 전에 실행된다.** 카메라·마이크·위치처럼 **권한을 묻거나 하드웨어를 켜는 작업**을 탭 루트에 두면 사용자가 그 탭을 열지도 않았는데 실행된다.
- 해결: 부모가 선택 상태를 내려주고 `.task(id: isActive)`로 **실제로 보일 때만** 실행한다. `onDisappear`만으로는 부족하다 — 애초에 나타난 적이 없어도 `.task`는 이미 돌았기 때문이다.
```swift
TabView(selection: $tab) {
    CameraScreen(isActive: tab == .camera).tag(Tab.camera)
    ...
}
// CameraScreen
.task(id: isActive) {
    guard isActive else { camera.stop(); return }
    await camera.requestAccess()
}
```
- 근거: `(로컬 경로)` 커밋 `ba78190`. 시뮬레이터 재현·수정 후 확인.


### 2026-08-18 — 사이드바 호버·블러·팝업 폭 (탭탭 macOS)
- 맥락: 탭탭 디자인 QA #133 (작업 기록)에서 사이드바 호버·그라데이션 블러·⋮ 팝업을 손보다가.
- 배운 것:
  - 하단 페이드에 `.blur(10)`을 올리니 아래 가장자리가 반투명해져 맨 밑 행이 비쳤다 → `.padding(.bottom, -12)`로 사이드바 `clipShape` 밖으로 밀어 해결. 같은 걸 카테고리 헤더 스크림에 하려다 스크림이 첫 행(38pt) 위로 넘어와 선택 배경을 덮었다 → 블러 자체를 원복. 이펙트는 "가장자리가 어디에 닿는지"부터 봐야 한다.
  - ⋮ 팝업 항목 텍스트가 `.frame(width: 95)`로 고정돼 "즐겨찾기에서 제거하기"가 잘렸다 → `fixedSize` + `minWidth`로 hug.
  - 팝업 호버를 DS `MacPopupItem`과 같은 `bgDimHover`로 넣었는데 다크 모드에서 안 보였다. 에셋을 열어 보니 라이트/다크 둘 다 `#272146 @7%` — 다크 배경 위에서는 사실상 투명. `n20`(다크 변형 있음)으로 교체. DS의 `MacPopupItem`도 같은 문제를 안고 있을 것으로 추정.
  - `.plain` 버튼 히트 영역 — 빈 목록의 "새 링크 추가하기" 버튼은 배경이 Button 밖에 있어 텍스트/아이콘 픽셀만 눌리던 상태였다. 배경을 라벨 안으로 옮기고 `contentShape`.
  - 위 블러 작업은 결국 **잘못된 해석**이었다 — 디자이너가 "블러가 없다"고 해서 피그마 `get_design_context`로 노드를 열어 보니 하단 페이드·헤더 스크림 둘 다 `backdrop-blur 4px` + 그라데이션이었다. 그라데이션을 흐린 게 아니라 뒤 행을 흐리는 효과. `CALayer.backgroundFilters`로 다시 구현하고 화면 캡처로 하단 행이 실제로 흐려지는 것을 확인. 이펙트 스펙은 스크린샷 육안이 아니라 노드의 effect 속성으로 확인해야 한다.
  - 카테고리 목록 스크롤바가 `.scrollIndicators(.hidden)`에도 계속 보였다. `.never`로 바꾸니 사라짐. 기존 `HiddenScrollIndicatorsConfigurator`(NSScrollView 탐색 후 스크롤러 숨김)를 붙여도 변화 없었다.
- 근거: `taptap-ios` 커밋 `0fba04a`(블러), `9c2b140`(배경 블러), `e5bf988`(스크롤바), `6a2ae97`(스크림 원복), `88ef2ff`(팝업 폭), `3032087`(호버 색), `908a3cc`(버튼 히트) / `DesignSystem/Resources/Assets.xcassets/Color/Background/bgDimHover.colorset/Contents.json`
