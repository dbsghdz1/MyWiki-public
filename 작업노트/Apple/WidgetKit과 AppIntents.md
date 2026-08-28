---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-19
updated: 2026-08-25
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
- **`reloadAllTimelines()`는 공짜가 아니다.** 요청마다 chronod가 위젯 확장 프로세스를 깨워 각 사이즈의 아카이브를 다시 만들고, NotificationCenter가 그걸 전환 애니메이션으로 다시 그린다. 게다가 리로드는 **시스템 예산제**라 헤프게 쏘면 정작 상태가 바뀔 때 갱신을 못 받는다. 리로드는 "보이는 상태가 실제로 바뀌었는가"로 걸러야 하고, 그 판단에 들어가는 입력이 흔들리는 값이면(아래 IsCharging) 스냅샷 비교만으로는 못 막는다 — 히스테리시스 + 최소 간격 하한, 두 겹이 필요하다.
- **SMC의 `IsCharging`은 AC에 꽂힌 채 잔량이 유지되는 구간에서 몇 초 주기로 껐다 켜진다.** 최적화된 충전의 80% 홀드 같은 상태에서 실제로 충전이 on/off를 반복하기 때문이고, IOPS(IOPowerSources) 알림도 같은 횟수로 온다 — 실측 30초에 17회. "충전 상태 변화"를 트리거로 쓰는 코드는 전부 이 흔들림을 걸러야 한다.

## 기록

### 2026-08-19 — 위젯마다 테마를 고르게 만들기
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] 1.7에서 Zappy+ 가치를 늘리려고 "위젯마다 다른 캐릭터"를 넣었다. 이 앱은 `.lproj` 없이 `L10n.pick(...)`으로 5개 언어를 런타임 분기하는 구조라, 인텐트 문구도 같은 방식으로 쓰려다 빌드가 15개 오류로 깨졌다.
- 배운 것: 위 핵심 정리 전부. 요점은 **AppIntents는 빌드 타임 세계**라서 런타임 로컬라이제이션과 층이 다르다는 것 — 이 앱처럼 리소스를 안 쓰는 프로젝트라도 인텐트를 넣는 순간 문자열 카탈로그를 도입해야 한다.
- 근거: 커밋 `a791917`. `Widget/ZappyThemeWidget.swift`(리터럴), `Widget/Localizable.xcstrings`(21키 × ko/ja/es/zh-Hant), pbxproj `knownRegions`. 검증: 빌드 산출물 `ZappyWidget.appex/Contents/Resources/{ko,ja,es,zh-Hant}.lproj/Localizable.strings`에 번역이 들어갔고 `extract.actionsdata`(4,976B)에 `ZappyThemeIntent`·`ThemePick`·`StylePick`이 들어갔다.

### 2026-08-25 — 충전 중 위젯 리로드 폭주
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]]에서 AC에 꽂아둔 채 쓰는 동안 NotificationCenter 77% + WindowServer 37% CPU를 먹는 것이 관측됐다. 원인은 SMC `IsCharging` 흔들림(30초에 17회)이 스냅샷 비교를 그대로 통과해 2초에 한 번 `reloadAllTimelines()`가 돌던 것.
- 배운 것: 위 핵심 정리의 리로드 예산·IsCharging 항목. 스냅샷 문자열 비교는 "같은 값 반복"만 거르지 "진동하는 값"은 못 거른다 — 진동은 시간축으로만(20초 버텨야 인정) 걸러진다.
- 근거: Zappy 1.10.1 커밋 `05f927d` (`App.swift`의 `settledCharging` 히스테리시스 + 리로드 최소 간격 30초).
