---
type: study
area: AppStore
audience: ai
status: active
created: 2026-09-06
updated: 2026-09-06
projects:
  - "탭탭"
---

# TestFlight 내부 그룹과 수출 규정

업로드한 빌드를 "테스트 가능" 상태로 만드는 두 가지 — 수출 규정(암호화) 답변과 테스터 그룹 — 을 자동화하려다 알게 된 것.

## 핵심 정리

- **내부(internal) 그룹에는 빌드를 붙일 수 없다.** App Store Connect 사용자로 구성된 내부 그룹은 처리(processing)와 수출 규정 답변이 끝나면 **모든 빌드를 자동으로 받는다.** API로 붙이려 하면 `Builds cannot be assigned to this internal group. - Cannot add internal group to a build.`가 떨어진다(`POST /v1/builds/{id}/relationships/betaGroups`, spaceship `add_beta_groups_to_build`). 그러므로 fastlane `upload_to_testflight(groups: [...])`에 **내부 그룹 이름을 주면 레인이 그 자리에서 죽는다** — 자동화할 것이 없는 게 정답이다. 그룹이 내부인지 외부인지는 `betaGroups.isInternalGroup`으로 확인한다.
- **외부 그룹만 빌드 배정 대상이다.** 그리고 외부 배포는 매 빌드 **베타 앱 심사**를 태우므로 `distribute_external: true` + `changelog`가 필요하고, `skip_waiting_for_build_processing: false`로 처리 완료까지 기다려야 한다.
- **수출 규정 질문은 Info.plist로 없앤다.** `ITSAppUsesNonExemptEncryption = false`를 앱 타깃 Info.plist에 넣으면 업로드마다 뜨던 "암호화 알고리즘 해당 없음" 선택이 아예 사라진다(HTTPS만 쓰는 앱의 표준 선언). Tuist면 `infoPlist: .extendingDefault(with: [...])`에 한 줄.
- **이미 올라간 빌드의 답변은 못 고친다.** 값이 한 번 정해지면 `PATCH /v1/builds/{id}`가 `You cannot update when the value is already set. - /data/attributes/usesNonExemptEncryption`로 거부한다.
- 빌드가 실제로 테스트 가능한지는 `buildBetaDetails`의 `internalBuildState`로 본다 — `IN_BETA_TESTING`이면 내부 테스터에게 이미 나가 있다(`externalBuildState`는 `READY_FOR_BETA_SUBMISSION`으로 따로 논다). spaceship의 `build.ready_for_internal_testing?`는 이 상태에서 `false`를 돌려주므로 판단 근거로 쓰지 않는다.

- **macOS 스크린샷은 세트가 하나다** — `AppScreenshotSet::DisplayType`에 `APP_DESKTOP` 하나뿐이고 크기는 2880×1800(또는 2560×1600 등 허용 크기). iOS처럼 기기별 세트를 고를 일이 없다.
- **업로드 전 알파 채널을 없앤다.** 피그마에서 뽑은 PNG는 RGBA로 나오는데 App Store 이미지에 알파가 있으면 거부된다 — 흰 배경에 합성해 RGB로 저장한다.
- **deliver 대신 spaceship로 직접 올리면 이중 업로드를 피한다.** `loc.create_app_screenshot_set(attributes: { screenshotDisplayType: ... })` → `set.upload_screenshot(path:, wait_for_processing: true)`를 순서대로 부르면 그 순서가 그대로 진열 순서가 되고, 올린 뒤 `app_screenshots`의 `assetDeliveryState`가 `COMPLETE`인지로 검증한다(deliver의 중복 업로드 함정은 [[작업노트/도구/fastlane 배포 알림|배포 자동화]] 계열 기록 참고).

## 기록

### 2026-09-06 — macOS 앱스토어 스크린샷 4장 업로드 (탭탭)

- 맥락: 디자이너가 피그마(`C6 디자인 4차 HI-FI` → `macOS 프로모션 이미지 최종 2880*1800`, 노드 `15125:44633`)에 만들어 둔 1~4번 프레임을 macOS 앱(6795730513)의 ko 로케일에 순서대로 올렸다. 그 전까지 `ko: 0 screenshots`라 제출이 막혀 있었다.
- 배운 것: 위 「핵심 정리」 뒤쪽 세 항목. Figma MCP `get_screenshot`에 `maxDimension: 2880`을 주면 프레임 원본 크기 그대로 PNG를 준다(URL은 수명이 짧아 바로 `curl -L -o`).
- 근거: 업로드 후 `asc state 6795730513` → `ko: 4 screenshots`, 세트 id `c5413881-…`, 4장 모두 `assetDeliveryState=COMPLETE`. 버전 레코드는 여전히 `1.0`이고 빌드 미연결이라 제출 전 정합성 작업이 남았다.

### 2026-09-06 — "알고리즘 해당 없음·테스터 그룹까지 자동으로" 요청 (탭탭)

- 맥락: 탭탭 macOS 1.1.0 build 5를 TestFlight에 올린 뒤, 홍이 매번 손으로 하던 두 가지(수출 규정 답변·테스팅 그룹 설정)를 다음 업로드부터 자동으로 해 달라고 요청.
- 배운 것: 위 「핵심 정리」 전부. 조회 결과 macOS 앱(6795730513)의 그룹은 `TapTap`(internal) 하나, iOS 앱(6754357960)은 `UT1`(internal)·`TOT`(external, public link)였다. 즉 **맥은 자동화할 그룹 작업 자체가 없고**, 남는 것은 plist 한 줄이었다.
- 근거: spaceship 스크립트로 `add_beta_groups_to_build` 시도 → 위 에러. build 5는 이미 `usesNonExemptEncryption=false`·`internalBuildState=IN_BETA_TESTING`. 조치는 `Projects/TapTapMac/Project.swift`·`Projects/App/Project.swift`에 `ITSAppUsesNonExemptEncryption: false` 추가(미커밋 작업 트리).
