---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-18
updated: 2026-09-05
projects:
  - "탭탭"
  - "[[프로젝트/개인/WristNote/README|WristNote]]"
---

# SwiftUI

한 줄 요약 — 레이아웃·히트 영역·이펙트가 "보이는 것"과 다르게 동작하는 지점들. macOS 앱을 만들며 하나씩 부딪힌 것.

## 핵심 정리

- **`.buttonStyle(.plain)` 버튼의 히트 영역은 라벨의 불투명 픽셀뿐이다.** 투명 배경 행·패딩 영역까지 눌리게 하려면 라벨 쪽에 `.contentShape(...)`를 준다. 배경을 Button 바깥에 두면 그 배경은 클릭 영역이 아니다.
- **`.blur(radius:)`는 네 가장자리를 전부 흐린다.** 한 방향 페이드 그라데이션에 블러를 얹으면 반대편·좌우 가장자리도 반투명해져 밑 콘텐츠가 비친다. 흐려도 되는 쪽만 남기려면 뷰를 블러 반경만큼 음수 패딩으로 부모 클립 밖으로 밀거나(`.padding(.bottom, -12)` + 부모 `clipShape`), 가려야 하는 쪽에 불투명 오버레이를 덧댄다.
- **고정 폭 텍스트는 잘린다.** 메뉴처럼 항목 길이가 가변인 곳은 `Text.fixedSize(horizontal: true, vertical: false)` + 컨테이너 `frame(minWidth:)` + `fixedSize(horizontal: true, vertical: false)`로 "최소 폭은 유지, 내용이 길면 늘어남"을 만든다.
- **SwiftUI에는 배경 블러(backdrop blur)가 없다.** `.blur`는 그 뷰 자체를 흐릴 뿐 뒤 콘텐츠는 그대로다. 피그마의 "Background blur"를 맞추려면 macOS에선 `NSViewRepresentable`로 레이어를 만들고 `layer.backgroundFilters = [CIAffineClamp, CIGaussianBlur]`를 건다 — `view.layerUsesCoreImageFilters = true`가 있어야 적용되고, `CIAffineClamp`을 앞에 두지 않으면 가장자리가 투명을 샘플링해 어두워진다. `hitTest`를 `nil`로 넘겨야 밑의 스크롤·클릭이 막히지 않는다. **`CIGaussianBlur.inputRadius`는 픽셀 단위**라 Retina에서 4를 주면 2pt로 먹는다 — `window.backingScaleFactor`를 곱해야 피그마 4px와 같아진다(`viewDidChangeBackingProperties`에서 재적용). `Material`은 시스템 틴트가 섞여 순수 블러가 아니다.
- **`.scrollIndicators(.hidden)`은 "숨김 권장"이지 강제가 아니다.** macOS에서 시스템 설정이 스크롤 막대 항상 표시면 그대로 보인다. 확실히 없애려면 `.never`. NSScrollView를 뒤져 스크롤러를 숨기는 우회는 macOS 26 SwiftUI ScrollView에선 효과가 없었다(NSScrollView 기반이 아닌 듯 — 추정).
- **`.onExitCommand`는 포커스를 가진 뷰에만 온다.** 딤 위에 얹은 팝오버·모달에 붙이면 컴파일은 되지만 Escape가 한 번도 호출되지 않는다. `NSEvent.addLocalMonitorForEvents(matching: .keyDown)`으로 keyCode 53을 직접 받는다.
- **`Text("\(n)개")`는 천 단위 구분 기호가 붙고, String을 먼저 만들어 넘기면 안 붙는다.** `LocalizedStringKey` 이니셜라이저와 `StringProtocol` 오버로드가 갈리는 탓이라 화면마다 "4,009개"/"4009개"로 조용히 어긋난다. 표기는 포매터 한 곳에서 정한다.
- **다크 모드의 모달은 배경을 낮추는 게 아니라 카드를 올려서 띄운다.** 알파 딤은 이미 검은 배경을 더 어둡게 못 만들므로, 카드가 배경과 같은 토큰이면 대비가 3/255가 된다. 카드에는 한 단계 위 면 토큰(n0)을 준다.
- **"호버가 안 된다"는 신고는 이벤트가 아니라 대비를 먼저 의심한다.** 실제로 `.onHover`는 멀쩡히 발화하는데 색 차이가 눈에 안 잡히는 경우가 많다. 토큰 값을 직접 열어 배경과의 차이를 계산해 보면 바로 갈린다 — `n20`은 라이트에서 `#F6F6F6`이라 `n0`(흰색) 위에서 **9/255**, 다크에서도 10/255 차이다. 팝업·버튼처럼 `n0` 면 위에 얹히는 호버는 `n40`(라이트 Δ31, 다크 Δ20)을 쓴다. DS의 `MacArrowButton`이 이미 `n0 → n40`이다.
- **호버 색과 같은 색을 기본 배경으로 깔아 두면 그 항목은 영원히 호버 상태로 보인다.** 위험 액션에 `bgDimDanger`를 상시 배경으로 주고 호버 색도 같은 토큰으로 주면 호버가 아무 변화도 못 만든다. 기본은 `.clear`, 강조는 전경색(텍스트·아이콘)으로 준다.
- **경과 시간 표시를 내 `Timer` tick으로 갱신하면 워치에서 건너뛰며 올라간다.** 손목을 내리거나 Always-On 감광 상태에선 tick이 멈추고, 화면을 다시 보는 순간 밀린 값이 한 번에 반영돼 `0:03 → 0:07`처럼 보인다. 시스템이 직접 다시 그리는 **`Text(timerInterval:countsDown:)`** 을 쓴다 — 앱이 tick하지 않아도 갱신되고, 감광 중엔 시스템이 알아서 분 단위로 낮춘다. 같은 계열: 시간이 흐르는 것만으로는 `@Published`가 울리지 않는다, 애니메이션은 트랜잭션 대신 시계(`TimelineView(.animation)`)로.
- **매초 바뀌는 값을 `@Published`로 두면 뷰가 초당 한 번 다시 그려진다.** 타이머 텍스트를 시스템에 넘겨도 같은 객체의 다른 `@Published`가 매초 `objectWillChange`를 쏘면 이득이 사라진다. 뷰가 안 읽는 진행값은 `private var`로 내리고, 뷰에 필요한 것(시작 시각)만 노출한다.
- **알파만 있는 어두운 오버레이 토큰은 다크 모드에서 사라진다.** `#272146 @7%` 같은 호버 색은 흰 배경에선 회색, 검은 배경에선 검정 위 검정이라 안 보인다. 색 토큰에 다크 appearance 변형이 있는지(`Contents.json`의 `appearances`) 확인하고, 없으면 밝기 변형이 있는 중립색(n20 등)을 쓴다.

## 기록

### 2026-09-05 — 워치 녹음 타이머가 "0:03 → 0:07"로 건너뛴다 → `Text(timerInterval:)` (WristNote)

- 맥락: [[프로젝트/개인/WristNote/README|WristNote]] 1.1.0 심사 중, 홍이 실기기에서 "시간초가 갑자기 올라가 7초 이렇게" 라고 보고. 워치 녹음 화면의 경과 시간 표시.
- 배운 것:
  - `Timer.scheduledTimer(withTimeInterval: 1, repeats: true)`로 `elapsed`를 갱신하고 `String(format: "%02d:%02d", ...)`로 그리면, **손목 내림·Always-On 감광 중 run loop tick이 멈춘다.** `elapsed`는 `Date().timeIntervalSince(startedAt)` 절대차라 값 자체는 정확하지만, **갱신 시점이 없어서** 화면을 다시 보는 순간 밀린 초가 한 번에 반영된다. 증상은 "타이머가 느리다"가 아니라 **"건너뛴다"** 로 나타난다.
  - 해결은 시작 시각만 넘기고 그리기를 시스템에 맡기는 것: `Text(timerInterval: startedAt...Date.distantFuture, countsDown: false)`. 앱이 tick하지 않아도 매초 다시 그려진다. 포맷은 시스템이 정해서 `00:05`가 `0:05`, 1시간 넘으면 `1:00:00`이 된다(직접 포맷하던 `%02d:%02d`는 65분에 `65:23`을 찍었다 — 새 쪽이 낫다).
  - **`@Published var elapsed`를 같이 지워야 의미가 있다.** 남겨 두면 매초 `objectWillChange`가 나가 뷰가 어차피 초당 한 번 다시 그려진다. `startedAt`만 `@Published private(set)`으로 노출하면 시작·정지 두 번만 갱신된다.
  - **곁가지로 드러난 것: tick에 의존한 판정 로직도 같이 깨진다.** 실기기 코덱 사다리의 성장 판정이 `if wall == 2 { base = size }` / `if wall == 5, size <= base`였는데, tick이 밀려 그 초를 건너뛰면 **판정 자체가 한 번도 안 돈다.** 부등호(`wall >= 2` / `wall >= baselineWall + 3`)로 바꾸되, 기준을 잡은 그 tick에서 바로 비교하면 `size == base`라 "안 자람" 오탐이 나므로 기준 시각 +3초 이후 tick에서만 한 번 비교하도록 했다. **`==`로 특정 tick을 노리는 코드는 tick이 보장될 때만 성립한다.**
- 근거: `(로컬 경로)` 커밋 `7e557ec` (`WristNoteWatch/Sources/RecorderView.swift`·`WatchRecorder.swift`). 워치 시뮬레이터(Series 11 46mm) `WRISTNOTE_AUTOTEST=1` 자동 테스트로 `0:01 → 0:04 → 0:05` 정상 증가·정지 후 `0:00` 확인, 로그에서 `ulaw-16k` 오탐 전환 없음(`tick wall=5 … fileSize=79432`), `finished … audio=8.15s` 후 `transferFile didFinish error=nil`.
- 원전 확인(WWDC21 "What's new in watchOS 8"): Always-On에서 **활성 세션이 있는 앱은 초당 1회, 없는 앱은 분당 1회**까지 갱신된다. 즉 감광 중에도 갱신 예산 자체는 있는데 **내 run loop `Timer`가 그 예산을 못 쓰는 것**이다. 애플 권고는 "손목을 내린 상태에서 UI를 갱신하려면 `TimelineView`" — `Text(timerInterval:)`이 타이머 표시에 대한 그 권고의 기성품이다. 감광 상태 자체는 `@Environment(\.isLuminanceReduced)`로 읽어 민감 정보를 가릴 수 있다(WristNote는 해당 없음).

### 2026-09-02 — `Text(String)`은 로컬라이즈를 **안 한다** ★★ (Fadeo)

번역이 `Localizable.xcstrings`에 **멀쩡히 있는데도** 영어·일본어·중국어·스페인어 사용자가 상세 화면 버튼 네 개(적기·저장·공유·삭제)를 한국어로 봤다. **1.0에 이 상태로 출시됐다.**

원인은 헬퍼의 인자 타입 하나.

```swift
// ❌ Text(_ content: some StringProtocol) 오버로드로 잡힌다 → 카탈로그를 안 본다
private func action(_ title: String, ...) -> some View { Text(title) }

// ✅ Text(_ key: LocalizedStringKey) 오버로드 → 카탈로그를 본다
private func action(_ title: LocalizedStringKey, ...) -> some View { Text(title) }
```

호출부(`action("적기", ...)`)는 **양쪽 다 컴파일된다.** 문자열 리터럴이 `LocalizedStringKey`로 자동 변환되므로, 타입을 `String`으로 두면 **조용히 로컬라이즈만 사라진다. 경고도 없다.**

**인자 타입만 보고 판정하면 안 되고 호출부까지 봐야 한다.** 같은 앱의 `AlbumScreen.header(_ title: String, _ count: String)`은 타입이 같은데 멀쩡했다 — 호출부에서 이미 `String(localized:)`로 풀어 넘기고 있었기 때문이다.

**찾는 법: 로케일별 스크린샷을 실제로 찍어 눈으로 비교한다.** 이 버그는 스토어 스크린샷을 en-US로 다시 뽑다가 발견됐다. 코드 리뷰로는 세 번을 지나쳤다.


### 2026-08-27 — 삽입되는 뷰의 `withAnimation`은 삼켜질 수 있다 → 시계로 움직여라 (즉석카메라)

새로 나타나는 화면의 `.task`에서 `withAnimation(.linear(2.4)) { t = 1 }`로 등장 연출을 밀었는데, **메인 스레드 블록을 없애자 애니메이션이 사라졌다** — 요소가 끝 위치에 즉시 그려졌다.

상태 변경이 **화면 삽입과 같은 트랜잭션**에 들어가면, 새로 삽입되는 뷰는 출발값(t=0)이 커밋된 적이 없어 보간할 대상이 없다. 그전까지 움직였던 것은 블록이 삽입 커밋과 상태 변경을 다른 트랜잭션으로 갈라준 **우연**이었다.

`sleep` 한 프레임 같은 미봉책 대신 **시계 기반**으로 바꾸는 것이 정답이다:

```swift
TimelineView(.animation) { ctx in
    let p = min(max(ctx.date.timeIntervalSince(start) / duration, 0), 1)
    content.offset(y: -height * (1 - p))
}
```

- 트랜잭션이 무엇을 하든 **확정적으로** 움직인다
- 소리·햅틱 등 **다른 시계와의 동기**도 공짜로 보장된다 (Fadeo 배출음 2.55초와 정합)
- "일정한 속도로 민다" 같은 물리적 성질을 이징 함수가 아니라 수식으로 직접 쓴다

같은 원리의 응용: 모터 떨림도 같은 TimelineView에서 `sin(e·2π·13)·0.7pt`로 얹었다. 연출 주파수는 소리(26Hz)를 그대로 쓰면 잔상만 남는다 — 눈에는 절반 이하로.

### 2026-08-26 — `.onExitCommand`는 포커스 없는 오버레이에 오지 않는다 (탭탭 macOS)

TapTapMac QA 중 **Escape로 아무 오버레이도 닫히지 않는 것**을 발견했다. 코드에는 분명 `.onExitCommand`가 붙어 있었다 (#132에서 "편집 메뉴/카테고리 이동 오버레이에 Escape 취소 동작 추가"로 머지된 것).

```swift
ZStack { dim; MacPopup(...) }
  .onExitCommand { editMenuArticleID = nil }   // ✗ 호출되지 않는다
```

**`.onExitCommand`는 포커스 체인에 있는 뷰에만 전달된다.** 딤 레이어 위에 얹은 팝오버·모달은 `focusable`도 아니고 포커스를 받지도 않으므로 이벤트가 도달하지 않는다. **컴파일도 되고 코드 리뷰도 통과하지만 런타임에 아무 일도 안 일어난다** — 리뷰에서 못 잡히는 종류의 버그다.

포커스와 무관하게 받으려면 AppKit 이벤트를 직접 가로챈다. 오버레이가 떠 있는 동안에만 모니터를 설치하도록 표시 분기 안쪽에 붙인다.

```swift
.onAppear {
  monitor = NSEvent.addLocalMonitorForEvents(matching: .keyDown) { event in
    guard event.keyCode == 53 else { return event }   // 53 = Escape
    action(); return nil                              // nil = 이벤트 소비
  }
}
.onDisappear { NSEvent.removeMonitor(monitor) }
```

대안인 `Button("") { }.keyboardShortcut(.cancelAction)`은 포커스된 `TextField`가 Escape를 먼저 먹을 수 있어 모달에서 신뢰하기 어렵다. 로컬 모니터는 responder chain·IME보다 앞이라 확실하다 (대신 한글 조합 중 Escape도 가로챈다 — 검색창처럼 조합 취소가 의미 있는 곳은 트레이드오프를 따져야 한다).

- 근거: `DesignSystem/Sources/macOS/Foundation/EscapeDismiss.swift`, 커밋 `dcc77a5`

### 2026-08-26 — `Text("\(count)개")`와 `Text(문자열)`은 숫자 표기가 다르다 (탭탭 macOS)

같은 링크 개수가 사이드바에서는 **"4,009개"**, 링크 추가 화면에서는 **"4009개"**로 나왔다. 데이터도 로케일도 같았다.

```swift
Text("\(totalLinkCount)개")            // → "4,009개"
Text("\(totalLinkCount)개" as String)  // → "4009개"
```

`Text("...")`는 **`LocalizedStringKey`** 를 받는 이니셜라이저이고, 여기 정수를 보간하면 `_FormatSpecifiable` 경로를 타 **로케일 천 단위 구분 기호가 붙는다.** 반면 `String`을 먼저 만들어 넘기면 `Text(_ content: S) where S: StringProtocol` 오버로드라 그대로 출력된다. **오버로드가 갈리는 지점이 문자열을 넘기는 방식뿐이라** 화면마다 표기가 달라져도 아무도 눈치채지 못한다.

`countText: "\(count)개"`처럼 String으로 만들어 자식 뷰에 넘기는 컴포넌트가 섞여 있으면 반드시 갈라진다. **표기는 `Text`에 맡기지 말고 포매터 한 곳에서 정한다.**

- 근거: `DesignSystem/Sources/Foundation/LinkCountFormat.swift`, 커밋 `15dcbb1`

### 2026-08-26 — 다크 모드에서 "딤 + 모달"은 카드 면을 한 단계 올려야 한다 (탭탭 macOS)

라이트에서 멀쩡하던 모달들이 **다크에서 경계가 사라졌다.** 카드가 어디서 시작하는지 안 보인다.

측정해 보니 원인이 분명했다. 카드 배경이 `Color.background`(다크 `#17171C`)인데, 그 뒤 배경도 같은 `background`이고 딤(`#121221` @65%)을 씌워봐야 **rgb(23,23,28) vs rgb(23,23,34)** — 3/255 차이다.

- 라이트: 카드 250 vs 딤 배경 160 → 충분한 대비
- 다크: 카드 23 vs 딤 배경 23 → **없음**

**알파 딤은 이미 검은 배경을 더 어둡게 만들지 못한다.** 그래서 다크에서 모달을 띄우는 방식은 "배경을 낮추기"가 아니라 **"카드를 올리기"** 여야 한다. 카드에는 배경 토큰(`background`)이 아니라 한 단계 위 면 토큰(`n0`, 다크 `#202027`)을 준다. 같은 앱 안에서 `n0`을 쓰던 팝오버 하나만 다크에서 정상이었던 게 힌트였다.

카드 안에서 스크롤 끝을 페이드하는 그라데이션도 **카드 색과 같은 토큰**으로 맞춰야 한다 — 배경색으로 페이드하면 카드 위에 다른 색 띠가 생긴다.

**일반화: 색을 "무엇처럼 보이는가"가 아니라 "어느 층인가"로 고른다.** 배경층·표면층·강조층을 토큰으로 나눠 두면 다크에서 층이 무너지지 않는다. [[작업노트/Apple/macOS 템플릿 아이콘 그리기|템플릿 아이콘]]에서 알파만 남는 것과 같은 종류의 함정이다.

- 근거: `Assets.xcassets/Color/Background/{background,bgDim}.colorset`, 커밋 `8a9a1b9`

### 2026-08-26 — "처음인가"를 `@AppStorage`로 직접 판정하면 안 된다 (즉석카메라)

첫 실행에만 다른 문구를 보여주려고 이렇게 썼다가 **첫 실행에 그 문구가 안 나왔다.**

```swift
@AppStorage("didExplain") var didExplain = false
var caption: some View { didExplain ? Text("짧게") : Text("길게") }   // ✗
.task { ...; didExplain = true }
```

플래그를 세우는 **그 순간 뷰가 다시 그려져서** 이미 "본 적 있음" 분기로 넘어간다. 화면이 뜰 때 값을 `@State`로 받아 두고, 사라질 때 세운다.

```swift
@State private var isFirstEver = false
.onAppear  { isFirstEver = !didExplain }
.onDisappear { didExplain = true }
```

**일반화: `@AppStorage`·`@Published`처럼 쓰기가 곧 무효화인 값은 "읽고 나서 그 자리에서 바꾸는" 용도로 못 쓴다.**

### 2026-08-26 — 겹침 방지는 상수가 아니라 `PreferenceKey`로 (즉석카메라)

프리뷰 위에 떠 있는 요소를 조작부 위까지만 내리려고 높이를 상수로 박았더니, **안전 영역이 다른 기기(iPhone 17e)에서 겹쳤다.** 자식의 실제 크기를 부모가 알아야 하는 상황은 `PreferenceKey`가 정답이다.

```swift
private struct ControlsHeightKey: PreferenceKey {
    static let defaultValue: CGFloat = 0
    static func reduce(value: inout CGFloat, nextValue: () -> CGFloat) { value = max(value, nextValue()) }
}
controls.background { GeometryReader { g in Color.clear.preference(key: ControlsHeightKey.self, value: g.size.height) } }
.onPreferenceChange(ControlsHeightKey.self) { controlsHeight = $0 }
```

`.background`에 `GeometryReader`를 넣는 것이 요령이다 — 오버레이와 달리 **자식의 레이아웃에 영향을 주지 않는다.**

### 2026-08-26 — `ignoresSafeArea`를 섞으면 좌표계가 어긋난다 (즉석카메라)

프리뷰는 화면 끝까지 가야 하고 조작부는 안전 영역을 지켜야 하는 화면에서, 둘을 한 `GeometryReader` 안에 두면 **`geo.size`가 안전 영역 크기라 마스크·가이드가 상단 인셋(약 59pt)만큼 밀린다.**

**바깥은 안전 영역을 지키고 안쪽 `GeometryReader`만 무시하게 나눈다.** 안쪽 `geo.size`가 전체 화면이 되고, 두 값이 필요하면 바깥 `GeometryReader`의 `safeAreaInsets`를 같이 읽는다.

### 2026-08-25 — 화면 밖으로 나가는 요소는 여백을 잡지 말고 **잘라낸다** (즉석카메라)

카메라 본체 아래로 인화지가 밀려 나오는 연출을 만들면서, 처음에는 배출될 자리를 `VStack`에 **여백으로 확보**했다. 그 대가로 본체가 화면 높이의 32%로 눌려 뷰파인더가 226pt밖에 안 됐다.

해법은 **고정 높이 컨테이너 + `.clipped()`**다. 나올 만큼의 높이를 잡고 잘라내면, 요소는 `-h`(완전히 숨음)에서 `0`(완전히 나옴)까지 내려오면서 컨테이너 밖으로 새지 않는다.

```swift
ZStack(alignment: .top) {
    Color.clear                      // 컨테이너 폭을 준다
    if let item { PaperView(...).offset(y: -h + t * h) }
}
.frame(height: h)
.clipped()
```

- **`.overlay`는 클리핑되지 않는다.** 오버레이 안에서 `offset`으로 위로 밀면 상위 뷰를 덮는다. 클리핑은 `.frame` + `.clipped()`로 **명시적으로** 걸어야 한다.
- `.clipped()`는 그림자도 자른다. 물리적으로는 그게 맞다(슬롯 안쪽은 안 보인다).
- `t`를 1을 넘겨 2까지 보내면 아래로 빠져나가며 사라진다 — 별도의 퇴장 전환이 필요 없다.

**얻은 것: 여백 344pt가 사라지고 뷰파인더가 262pt로 커졌다(면적 +34%).** 배출·서랍·시트처럼 "화면 밖에서 들어오는" 연출 전반에 그대로 쓰인다.

### 2026-08-25 — 시간이 흐르는 것만으로는 `@Published`가 울리지 않는다 (즉석카메라)

`developing`/`developed`처럼 **현재 시각으로 계산되는 computed property**로 목록을 나누면, 시간이 지나 분류가 바뀌어도 **뷰가 다시 그려지지 않는다.** 저장소가 바뀐 게 아니기 때문이다. 실제로 앨범을 열어둔 채 현상이 끝나면 다 된 사진이 "현상 중" 칸에 갇혀 탭도 안 됐다.

1초 타이머로 폴링하는 건 낭비다(초당 목록 전체 재평가). **다음 전환 시각에 딱 한 번 깨우는** 편이 낫고, `.task(id:)`가 그 도구다.

```swift
@State private var now = Date()
private var nextFinish: Date? { items.filter { !$0.isDone(at: now) }.map(\.finishesAt).min() }

.task(id: nextFinish) {
    guard let t = nextFinish else { return }
    let wait = t.timeIntervalSinceNow
    if wait > 0 { try? await Task.sleep(nanoseconds: UInt64(wait * 1_000_000_000)) }
    guard !Task.isCancelled else { return }
    now = Date()
}
```

`now`가 갱신되면 → 분류가 바뀌고 → `nextFinish`가 바뀌고 → **`.task(id:)`가 스스로 다시 걸린다.** 폴링 없이 연쇄가 이어진다. 값이 nil이면(남은 게 없으면) 아무도 안 깨운다.

### 2026-08-25 — 목록의 "고정 랜덤"에 `hashValue`를 쓰면 안 된다 (즉석카메라)

사진마다 미세하게 다른 기울기를 주려고 `abs(photo.id.hashValue) % 41`을 썼다. 스크롤 중에는 고정이라 맞아 보였지만, **Swift의 `Hashable`은 실행마다 시드가 바뀐다.** 앱을 다시 열 때마다 모든 항목의 각도가 재배치됐다 — "실물은 반듯하게 놓이지 않는다"는 연출인데 실물은 다시 봐도 같은 각도로 놓여 있다.

UUID 바이트에서 직접 뽑으면 영구적으로 고정되고, `abs(Int.min)` 트랩도 같이 사라진다.

```swift
let b = photo.id.uuid                                   // (UInt8, UInt8, ...) 튜플
let h = Int(b.0) &+ Int(b.1) &* 3 &+ Int(b.2) &* 7
let angle = Double(h % 41) / 10.0 - 2.0
```

**규칙: 디스크에 남는 것과 짝을 이뤄야 하는 값에는 `hashValue`를 쓰지 않는다.**

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


### 2026-09-01 — 호버가 안 되는 게 아니라 안 보이는 것이었다 (탭탭 macOS)

- 맥락: 탭탭 macOS 디자인 QA(이슈 #143). "카테고리 ⋮ 팝업 버튼들이 호버가 안 된다"는 신고를 받고 원인을 찾는 중.
- 배운 것:
  - 앱을 빌드해 실제로 호버를 걸고 픽셀을 재 보니 **호버는 정상 발화하고 있었다** — 팝업 배경 `rgb(42,43,51)` → 호버 `rgb(55,55,64)`, 차이 13. 2026-08-18에 다크 대비 때문에 `bgDimHover` → `n20`으로 바꿨던 그 색인데, `n20`은 라이트에서 `#F6F6F6`이라 `n0`(흰색) 위에서 9/255 차이다. **다크를 고치면서 라이트를 잃은 셈.** `n40`으로 올리니 차이 25(라이트 31).
  - '카테고리 삭제하기'만 `backgroundColor: bgDimDanger`를 상시로 깔고 호버 색도 같은 토큰이라, 기본 상태가 이미 호버처럼 보이고 호버해도 변화가 없었다. 상시 배경을 지우니 기본은 투명, 호버할 때만 빨간 딤이 된다.
  - 사이드바가 접혔을 때 뜨는 '사이드바 열기' 버튼은 배경이 `.clear`였다 — 콘텐츠 배경 위에 아이콘만 떠 있어 옆의 네비 버튼(`MacArrowButton`: `n0` 배경 + 그림자 2겹 + radius 12)과 따로 놀았다. 같은 뷰가 사이드바 안에서도 쓰이므로 `isFloating` 플래그로 갈랐다.
- 근거: `taptap-ios` `TapTapMac/Sources/Sidebar/SidebarCategoryRow.swift`·`MacSidebarAssets.swift`·`RootView.swift`. 수정 전후 모두 앱을 띄워 `NSBitmapImageRep.colorAt`으로 픽셀 확인 ([[작업노트/도구/macOS 앱 QA 자동화|QA 하네스]]).

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

## 참고 자료

- [Text `init(timerInterval:pauseTime:countsDown:showsHours:)` — Apple Developer Documentation](https://developer.apple.com/documentation/swiftui/text/init(timerinterval:pauseTime:countsDown:showsHours:)) — 시스템이 갱신하는 타이머 텍스트 이니셜라이저 (2026-09-05 확인)
- [What's new in watchOS 8 — WWDC21](https://developer.apple.com/videos/play/wwdc2021/10002/) — Always-On 갱신 예산(활성 세션 초당 1회 / 그 외 분당 1회), `isLuminanceReduced`, `TimelineView` 스케줄 (2026-09-05 확인)
- [Designing your app for the Always On state — Apple Developer Documentation](https://developer.apple.com/documentation/watchkit/designing_your_app_for_the_always_on_state) — 감광 상태 설계 지침 (2026-09-05 확인, 밑줄 없는 `-` 경로는 404)
