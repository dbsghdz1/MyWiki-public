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

- **제출을 막는 항목은 에러 한 번에 다 나온다.** `reviewSubmission.submit_for_review`가 거부되면 사유가 줄바꿈으로 나열된다(실측: `contentRightsDeclaration` 누락 + App Privacy 미게시 + `is not in valid state` 세 줄). **한 줄씩 고치지 말고 전부 읽고 한 번에 채운 뒤 재시도**한다.
- **`contentRightsDeclaration`은 버전이 아니라 앱 속성이다** — `App#update(attributes: { contentRightsDeclaration: "DOES_NOT_USE_THIRD_PARTY_CONTENT" })`. 새 앱 레코드에는 비어 있고, 이것 하나 때문에 제출이 통째로 막힌다. 값 확인은 새로 `App.find` 하거나 `/v1/apps/<id>`를 직접 읽는다(기존 객체는 캐시라 `nil`로 보인다).
- **App Privacy(데이터 수집)는 이제 공개 API에 없다.** `apps/<id>/dataUsages`·`/v1/appDataUsages`·`appDataUsagePublishStates` 모두 404(`The relationship 'dataUsages' does not exist`)다. spaceship의 `get_app_data_usages`도 같은 404를 낸다 — **ASC 웹에서만** 채울 수 있다고 보고 계획을 짠다.
- **제출 흐름은 reviewSubmissions다.** `app.get_ready_review_submission(platform:, includes: "items")` → 없으면 `create_review_submission(platform:)` → `add_app_store_version_to_review_items(app_store_version_id:)` → `submit_for_review`. 제출되고 나면 `get_ready_review_submission`이 `nil`을 주므로, **성공 판정은 반환값이 아니라 버전 상태(`WAITING_FOR_REVIEW`)로** 한다.
- **버전 문자열과 빌드는 제출 전에 맞춘다.** `version.update(attributes: { versionString: "1.1.0" })` + `version.select_build(build_id:)`. TestFlight 빌드가 1.1.0인데 스토어 레코드가 1.0인 상태로 두면 제출 단계에서 걸린다.

## 기록

### 2026-09-06 — macOS 첫 제출: 진짜 막고 있던 건 `contentRightsDeclaration`이었다 (탭탭)

- 맥락: 홍 "출시해줘". 점검 문서에서 제출을 막는다고 본 세 가지(빌드 미연결·리뷰 정보·웹 전용 항목) 중 앞의 둘을 채우고 제출을 시도했다.
- 배운 것: 위 「핵심 정리」 다섯 항목. 특히 **점검 단계에서 "웹에서만 되는 App Privacy 때문"이라고 본 것이 실제로는 `contentRightsDeclaration` 하나였다** — 앱 속성 한 줄을 채우자 App Privacy 경고까지 사라지고 제출이 통과했다(iOS 레코드 상속 여부는 미검증).
- 근거: 첫 시도 에러 3줄 → `App#update(contentRightsDeclaration:)` → 재시도 통과. `asc state 6795730513` = `1.1.0 | WAITING_FOR_REVIEW | build 6 (VALID) | ko 4장`, reviewSubmission `4d37df45-d9f5-44a9-9b54-4938e4a14c22` submitted `2026-09-06T14:44:02Z`.

### 2026-09-06 — macOS 앱스토어 스크린샷 4장 업로드 (탭탭)

- 맥락: 디자이너가 피그마(`C6 디자인 4차 HI-FI` → `macOS 프로모션 이미지 최종 2880*1800`, 노드 `15125:44633`)에 만들어 둔 1~4번 프레임을 macOS 앱(6795730513)의 ko 로케일에 순서대로 올렸다. 그 전까지 `ko: 0 screenshots`라 제출이 막혀 있었다.
- 배운 것: 위 「핵심 정리」 뒤쪽 세 항목. Figma MCP `get_screenshot`에 `maxDimension: 2880`을 주면 프레임 원본 크기 그대로 PNG를 준다(URL은 수명이 짧아 바로 `curl -L -o`).
- 근거: 업로드 후 `asc state 6795730513` → `ko: 4 screenshots`, 세트 id `c5413881-…`, 4장 모두 `assetDeliveryState=COMPLETE`. 버전 레코드는 여전히 `1.0`이고 빌드 미연결이라 제출 전 정합성 작업이 남았다.

### 2026-09-06 — "알고리즘 해당 없음·테스터 그룹까지 자동으로" 요청 (탭탭)

- 맥락: 탭탭 macOS 1.1.0 build 5를 TestFlight에 올린 뒤, 홍이 매번 손으로 하던 두 가지(수출 규정 답변·테스팅 그룹 설정)를 다음 업로드부터 자동으로 해 달라고 요청.
- 배운 것: 위 「핵심 정리」 전부. 조회 결과 macOS 앱(6795730513)의 그룹은 `TapTap`(internal) 하나, iOS 앱(6754357960)은 `UT1`(internal)·`TOT`(external, public link)였다. 즉 **맥은 자동화할 그룹 작업 자체가 없고**, 남는 것은 plist 한 줄이었다.
- 근거: spaceship 스크립트로 `add_beta_groups_to_build` 시도 → 위 에러. build 5는 이미 `usesNonExemptEncryption=false`·`internalBuildState=IN_BETA_TESTING`. 조치는 `Projects/TapTapMac/Project.swift`·`Projects/App/Project.swift`에 `ITSAppUsesNonExemptEncryption: false` 추가(미커밋 작업 트리).
