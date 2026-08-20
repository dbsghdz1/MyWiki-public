---
type: study
area: 언어·프레임워크
status: active
created: 2026-08-19
updated: 2026-08-19
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# WidgetKit과 AppIntents

설정 가능한 위젯의 문구는 **컴파일 타임 상수여야 한다.** 런타임에 언어를 고르는 수동 로컬라이제이션과 공존할 수 없고, 번역은 문자열 카탈로그로 붙여야 한다.

## 핵심 정리

- **AppIntents 메타데이터는 빌드 타임에 추출된다.** `appintentsmetadataprocessor`가 소스를 훑어 `Metadata.appintents/extract.actionsdata`를 만드는데, 이때 `static var title`, `IntentDescription`, `@Parameter(title:)`, `AppEnum`의 `typeDisplayRepresentation`·`caseDisplayRepresentations`는 **리터럴이거나 리터럴로 초기화된 저장 프로퍼티**여야 한다. computed property나 `LocalizedStringResource(stringLiteral: 런타임값)`을 쓰면 빌드가 깨진다:
  - `'LocalizedStringResource' must be initialized with a call to its initializer or a string literal`
  - `The property 'caseDisplayRepresentations' must have a compile-time static value and cannot be computed or dynamic`
  - `Protocol 'AppEnum' requires 'caseDisplayRepresentations' to be exhaustive`
  - 하나라도 걸리면 `At least one halting error produced during export. No AppIntents metadata have been exported` — 메타데이터가 통째로 비고 인텐트가 동작하지 않는다.
- **그래서 런타임 분기형 로컬라이제이션(`L10n.pick(ko, en, …)`)은 이 자리에 못 쓴다.** 해법은 표준 경로로 돌아가는 것: 소스에는 **영어 리터럴**을 두고 그것을 키로 `Localizable.xcstrings`에 번역을 넣는다. 프로젝트의 `knownRegions`에 언어를 추가해야 `.lproj`가 산출물에 들어간다 (넣지 않으면 조용히 en만 남는다).
- **WidgetKit 쪽 문구는 제약이 없다.** `.configurationDisplayName(_:)` / `.description(_:)`은 그냥 `String`을 받는 런타임 값이라 수동 L10n을 그대로 쓸 수 있다. 즉 **한 위젯 안에서 인텐트 문구만 카탈로그, 나머지는 런타임 분기**로 섞여도 된다.
- **`AppIntentConfiguration`은 macOS 14+**, 그 아래는 `StaticConfiguration`뿐이다. `WidgetBundle`의 body는 result builder라 `if #available(macOS 14.0, *) { … }`로 위젯 하나를 조건부 포함할 수 있다 — 배포 타깃을 올리지 않고 설정형 위젯을 추가하는 방법.
- **`strings`로 Swift 문자열을 찾을 때의 함정**: 15바이트 이하 문자열은 small-string 최적화로 레지스터에 인라인되어 `__cstring` 섹션에 남지 않는다. 위젯 kind(`"ZappyThemePick"`, 14자)가 `strings -a`에 안 잡히는 이유가 이것이다. 바이너리에서 확인하려면 더 긴 식별자(타입명 등)를 찾거나 `Metadata.appintents/extract.actionsdata`를 본다.
- **위젯 갤러리 미리보기는 `placeholder(in:)`로 그려진다.** 여기에 하드코딩한 값이 그대로 상품 진열이 되므로, 유료 게이팅이 있는 위젯은 placeholder에서도 **실제 구매 상태를 읽어야** 한다. 안 그러면 구매자에게 갤러리에서 자물쇠 화면이 보인다.

## 기록

### 2026-08-19 — 위젯마다 테마를 고르게 만들기
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] 1.7에서 Zappy+ 가치를 늘리려고 "위젯마다 다른 캐릭터"를 넣었다. 이 앱은 `.lproj` 없이 `L10n.pick(...)`으로 5개 언어를 런타임 분기하는 구조라, 인텐트 문구도 같은 방식으로 쓰려다 빌드가 15개 오류로 깨졌다.
- 배운 것: 위 핵심 정리 전부. 요점은 **AppIntents는 빌드 타임 세계**라서 런타임 로컬라이제이션과 층이 다르다는 것 — 이 앱처럼 리소스를 안 쓰는 프로젝트라도 인텐트를 넣는 순간 문자열 카탈로그를 도입해야 한다.
- 근거: 커밋 `a791917`. `Widget/ZappyThemeWidget.swift`(리터럴), `Widget/Localizable.xcstrings`(21키 × ko/ja/es/zh-Hant), pbxproj `knownRegions`. 검증: 빌드 산출물 `ZappyWidget.appex/Contents/Resources/{ko,ja,es,zh-Hant}.lproj/Localizable.strings`에 번역이 들어갔고 `extract.actionsdata`(4,976B)에 `ZappyThemeIntent`·`ThemePick`·`StylePick`이 들어갔다.
