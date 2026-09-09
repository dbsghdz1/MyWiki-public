---
type: study
area: AppStore
audience: ai
status: active
created: 2026-09-09
updated: 2026-09-09
projects:
  - "[[프로젝트/개인/한능검/README|한국사 정복]]"
---

# 인앱 구매 등록 API — READY_TO_SUBMIT까지 필요한 다섯 조각

App Store Connect API로 IAP를 만들면 `MISSING_METADATA`로 시작하고, **다섯 가지를 다 채워야** `READY_TO_SUBMIT`으로 넘어간다. 어느 것이 빠졌는지는 알려주지 않는다 — 상태 문자열 하나뿐이다.

## 핵심

1. **생성** `POST /v2/inAppPurchases` — `name`(30자)·`productId`·`inAppPurchaseType`(`NON_CONSUMABLE`)·`reviewNote`, 관계로 `app`. `availableInAllTerritories`는 **이 리소스의 속성이 아니다**(409 `ENTITY_ERROR.ATTRIBUTE.UNKNOWN`) — 지역은 4번에서 따로 만든다.
2. **로컬라이제이션** `POST /v1/inAppPurchaseLocalizations` — `locale`·`name`(30자)·`description`(**45자**). 관계 이름은 `inAppPurchase`가 아니라 **`inAppPurchaseV2`**다.
3. **가격** `POST /v1/inAppPurchasePriceSchedules` — `data.id`를 넣으면 409 `ENTITY_ERROR.UNEXPECTED_ID`로 튕긴다. 가격은 `included`에 임시 id(`${price1}`)로 `inAppPurchasePrices`를 만들고 `manualPrices`가 그 id를 가리키는 형태다. 가격 포인트 id는 `GET /v2/inAppPurchases/{id}/pricePoints?filter[territory]=KOR`에서 `customerPrice`로 찾는다.
    - **기준 지역(baseTerritory)을 KOR로 잡는다.** 이 앱은 앱 가격표에서 기준 지역이 USA인데 USA 가격이 0이라 유럽 42개 지역이 통째로 무료가 된 전례가 있다 — 기준 지역을 실제로 파는 나라에 두면 그 사고가 안 난다.
4. **지역** `POST /v1/inAppPurchaseAvailabilities` — `availableInNewTerritories` + `availableTerritories`에 지역 목록(`GET /v1/territories?limit=200`, 175개). **이게 마지막 조각이었다.** 다른 넷을 다 채워도 이게 없으면 `MISSING_METADATA`에서 안 움직인다.
5. **심사 스크린샷** `POST /v1/inAppPurchaseAppStoreReviewScreenshots`(fileSize·fileName) → 응답의 `uploadOperations[0].url`로 바이트 PUT → `PATCH`로 `uploaded: true` + `sourceFileChecksum`(md5). 앱 스크린샷 업로드와 같은 3단 흐름이다.

상태 전이는 즉시가 아니다 — 5초쯤 뒤에 다시 조회해야 `READY_TO_SUBMIT`이 보인다.

## 앱 버전과 함께 심사에 넣기

fastlane의 `submit_for_review`는 **버전만** 넣는다. IAP를 같이 올리려면 심사 제출을 직접 만든다:

1. `POST /v1/reviewSubmissions` — `attributes.platform: "IOS"` + 관계 `app`. 응답 상태는 `READY_FOR_REVIEW`.
2. `POST /v1/reviewSubmissionItems` × 2 — 하나는 관계 `appStoreVersion`, **다른 하나는 `inAppPurchaseVersion`이다.**
   - `inAppPurchase` · `inAppPurchaseV2` · `iapItem` 전부 409 `ENTITY_ERROR.RELATIONSHIP.UNKNOWN`이다. 넣어야 하는 건 IAP 자체가 아니라 **그 IAP의 버전 리소스**이고, id는 `GET /v2/inAppPurchases/{id}/versions`에서 얻는다(비소모성도 `version: 1`짜리가 하나 있다).
3. `PATCH /v1/reviewSubmissions/{id}` — `attributes.submitted: true`. 이때 버전과 IAP가 **같이** `WAITING_FOR_REVIEW`로 넘어간다.

직접 제출할 때 fastlane이 대신 해주던 것 두 개를 잊지 말 것:
- **수출 규정** — `PATCH /v1/builds/{id}` `usesNonExemptEncryption: false`. `null`인 채로 두면 제출이 막힌다.
- **출시 방식** — `PATCH /v1/appStoreVersions/{id}` `releaseType`. 기본값은 `AFTER_APPROVAL`(승인 즉시 출시)이라, 가격 변경과 출시를 맞춰야 하면 `MANUAL`로 바꾼다.

## 기록

### 2026-09-09 — 한국사 정복을 무료+잠금해제 IAP로 돌리며 등록
- 맥락: [[프로젝트/개인/한능검/README|한국사 정복]]을 ₩6,600 유료 앱에서 무료+비소모성 IAP로 바꾸는 작업. 위키에 예약돼 있던 트립와이어(09-17 판정)를 홍이 앞당겨 결정했다.
- 만든 것: `com.hong.hangeom.full`(ASC id `6810125328`), ₩6,600, 기준 지역 KOR, 175개 지역, ko·en-US 로컬라이제이션, 심사 스크린샷은 Maestro가 찍은 잠금해제 시트.
- 밟은 것: 위 1·2·3번의 세 가지 409와, 4번이 없어 `MISSING_METADATA`에 머무른 것. 관계 이름이 `inAppPurchaseV2`인 것은 문서보다 에러 메시지가 먼저 알려줬다.
- 도구: `(로컬 경로)`의 범용 `get`/`post`/`patch`로 전부 처리했다. 이때 `delete`도 추가했다(아래 [[작업노트/AppStore/스토어 스크린샷 중복 업로드|스크린샷 중복]]에서 필요했다).
