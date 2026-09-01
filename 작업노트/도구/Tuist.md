---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-01
updated: 2026-09-01
projects:
  - "[[프로젝트/개인/WristNote/README|WristNote]]"
---

# Tuist

Xcode 프로젝트를 `Project.swift`로 생성하는 도구. 로컬엔 mise로 4.202.5가 깔려 있다(`(로컬 경로)`).

## 핵심 정리

- **루트 디렉터리 판정은 `.git` 또는 `Tuist/` 폴더다.** 둘 다 없으면 `Couldn't locate the root directory from path …`로 실패한다. 새 프로젝트는 `git init`을 먼저 한다.
- **빈 `Tuist/Config.swift`를 만들면 안 된다.** 루트 판정용으로 `touch Tuist/Config.swift`를 했더니 `The encoded data for the manifest is corrupted. The given data was not valid JSON.`로 죽는다 — 빈 매니페스트가 JSON을 안 뱉기 때문. `Tuist/` 폴더를 지우고 `.git`으로 루트를 잡으면 된다.
- **watchOS 앱 임베드**: iOS 타깃 `dependencies: [.target(name: "WristNoteWatch")]`만으로 `WristNote.app/Watch/WristNoteWatch.app`에 들어간다. 워치 타깃은 `destinations: [.appleWatch]`, `product: .app`, Info.plist에 `WKApplication: true`·`WKCompanionAppBundleIdentifier`·`WKBackgroundModes`. 워치 번들 ID는 `<iOS 번들 ID>.watchkitapp`.
- `tuist generate --no-open` 후 `xcodebuild -workspace WristNote.xcworkspace -scheme WristNote …`. 생성물(`*.xcodeproj`·`*.xcworkspace`·`Derived/`)은 gitignore.

## 기록

### 2026-09-01 — WristNote 스캐폴딩

- 맥락: [[프로젝트/개인/WristNote/README|WristNote]] iOS + watchOS 앱을 처음부터 만들며 pbxproj 손작성 대신 Tuist 사용.
- 배운 것: 위 「핵심 정리」. 루트 판정 실패와 빈 Config.swift 에러를 연달아 밟았다.
- 근거: `(로컬 경로)`, 커밋 `f8c4bd3`.
