---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-01
updated: 2026-09-05
projects:
  - "[[프로젝트/개인/WristNote/README|WristNote]]"
  - "탭탭"
---

# Tuist

Xcode 프로젝트를 `Project.swift`로 생성하는 도구. 로컬엔 mise로 4.202.5가 깔려 있다(`(로컬 경로)`).

## 핵심 정리

- **루트 디렉터리 판정은 `.git` 또는 `Tuist/` 폴더다.** 둘 다 없으면 `Couldn't locate the root directory from path …`로 실패한다. 새 프로젝트는 `git init`을 먼저 한다.
- **빈 `Tuist/Config.swift`를 만들면 안 된다.** 루트 판정용으로 `touch Tuist/Config.swift`를 했더니 `The encoded data for the manifest is corrupted. The given data was not valid JSON.`로 죽는다 — 빈 매니페스트가 JSON을 안 뱉기 때문. `Tuist/` 폴더를 지우고 `.git`으로 루트를 잡으면 된다.
- **watchOS 앱 임베드**: iOS 타깃 `dependencies: [.target(name: "WristNoteWatch")]`만으로 `WristNote.app/Watch/WristNoteWatch.app`에 들어간다. 워치 타깃은 `destinations: [.appleWatch]`, `product: .app`, Info.plist에 `WKApplication: true`·`WKCompanionAppBundleIdentifier`·`UIBackgroundModes`. 워치 번들 ID는 `<iOS 번들 ID>.watchkitapp`.
- `tuist generate --no-open` 후 `xcodebuild -workspace WristNote.xcworkspace -scheme WristNote …`. 생성물(`*.xcodeproj`·`*.xcworkspace`·`Derived/`)은 gitignore.
- **바이너리 xcframework를 포함한 SPM 패키지(Firebase)도 그냥 붙는다.** `Tuist/Package.swift`에 `.package(url: "https://github.com/firebase/firebase-ios-sdk.git", from: "12.18.0")`를 넣고 `.external(name: "FirebaseAnalytics")`로 쓰면 끝. `tuist install`이 Firestore·gRPC zip까지 전부 받아 느리지만(약 250MB) 실제 링크는 참조한 product만 된다. **`packageSettings.productTypes`에 Firebase 항목을 넣지 않는 편이 안전하다** — 넣지 않고도 `Generating project Firebase / GoogleAppMeasurement / GoogleUtilities`가 정상 생성됐다.
- **`Tests/` 타깃을 추가해도 별도 스킴은 안 생긴다.** `-scheme AnalyticsKitTests`는 `does not contain a scheme named`로 실패하고, 모듈 스킴(`-scheme AnalyticsKit test`)이 테스트까지 돌린다.
- **모듈 이름이 의존 SDK의 타입명과 겹치지 않게 한다.** `Analytics`라는 모듈을 만들면 `FirebaseAnalytics`의 `Analytics` 클래스와 이름이 부딪힌다 — `AnalyticsKit`으로 지었다.
- 새 모듈 추가는 세 곳: `Tuist/ProjectDescriptionHelpers/ProjectName.swift`의 `Module` enum + `TargetDependency+Module.swift`의 헬퍼 + `Projects/<이름>/Project.swift`. `Workspace.swift`가 `Projects/*`를 글롭하므로 워크스페이스 등록은 자동이다. **반대로 `Project.swift` 없이 생성물만 남은 폴더는 조용히 무시된다** — 브랜치를 갈아탈 때 이런 유령 폴더가 남는다.

## 기록

### 2026-09-01 — WristNote 스캐폴딩

- 맥락: [[프로젝트/개인/WristNote/README|WristNote]] iOS + watchOS 앱을 처음부터 만들며 pbxproj 손작성 대신 Tuist 사용.
- 배운 것: 위 「핵심 정리」. 루트 판정 실패와 빈 Config.swift 에러를 연달아 밟았다.
- 근거: `(로컬 경로)`, 커밋 `f8c4bd3`.

### 2026-09-05 — 탭탭에 Firebase·Amplitude SPM 추가

- 맥락: 탭탭에 GA4·Amplitude 계측 모듈(`AnalyticsKit`)을 붙이며 Tuist 매니페스트를 고쳤다.
- 배운 것: 위 「핵심 정리」의 뒤 네 줄. 특히 모듈명 `Analytics`가 `FirebaseAnalytics`의 `Analytics` 클래스와 충돌하는 것과, `Tests` 타깃에 전용 스킴이 안 생기는 것.
- 근거: `(로컬 경로)`, `Projects/AnalyticsKit/Project.swift`. 브랜치 `feat/analytics-ga4-amplitude`.
