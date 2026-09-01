---
type: study
area: AppStore
audience: ai
status: active
created: 2026-09-01
updated: 2026-09-01
projects:
  - "탭탭"
---

# App Store Connect 분석 리포트 API

한 줄 요약 — ASC 웹의 「분석」 숫자를 API로 받으려면 **앱마다 `analyticsReportRequests`를 먼저 만들어야 하고**, 요청을 만든 순간부터 리포트 *정의*는 즉시 보이지만 **실제 데이터(instances)는 애플이 만들 때까지 비어 있다**. 안 켜두면 그 기간 데이터는 나중에 소급해서 못 본다.

## 핵심 정리

- **리포트 요청이 없으면 데이터가 0이다.** `GET /v1/apps/{appId}/analyticsReportRequests` → `total: 0`이면 이 앱은 분석 API를 한 번도 안 켠 것이다. 켜는 건 POST 한 번:
  ```
  POST /v1/analyticsReportRequests
  {"data":{"type":"analyticsReportRequests",
           "attributes":{"accessType":"ONGOING"},
           "relationships":{"app":{"data":{"type":"apps","id":"<appId>"}}}}}
  ```
  `accessType`은 **`ONGOING`(앞으로 매일 누적)**과 **`ONE_TIME_SNAPSHOT`(과거분 한 번)** 두 가지이고, 둘은 별개 요청이라 **둘 다 필요하면 두 번 POST**한다. 응답 201.
- **요청 직후 `…/reports`는 바로 채워진다 — 탭탭 iOS 기준 156개.** 카테고리는 `COMMERCE`(App Downloads Standard/Detailed, Purchases, Pre-Orders…), `APP_STORE_ENGAGEMENT`(App Store Discovery and Engagement — 임프레션·제품 페이지 조회), `APP_USAGE`(App Sessions, App Store Installation and Deletion, App Crashes…), `PERFORMANCE`, 그리고 대부분을 차지하는 `FRAMEWORK_USAGE`(103개).
- **그런데 `GET /v1/analyticsReports/{reportId}/instances`는 한동안 `total: 0`이다.** 리포트 "정의"와 "데이터 파일"이 분리돼 있어서, 요청 생성 당일에는 정의만 보이고 instance는 애플이 생성한 뒤에 나타난다. 데이터를 받으려면 instance → `GET /v1/analyticsReportInstances/{id}/segments`의 다운로드 URL(gzip CSV)까지 가야 한다.
- **급하면 분석이 아니라 판매 및 추세(Sales and Trends)를 쓴다.** `GET /v1/salesReports`는 동기 호출이고 과거분이 바로 나오지만 **`filter[vendorNumber]`가 필수**다. 벤더 번호를 알려주는 API 엔드포인트는 없다 — ASC 웹 「지급 및 재무 보고서」에서 사람이 확인해야 한다.
- **"단위(Units)"에는 첫 다운로드·재다운로드·업데이트가 섞여 있다.** 추세 화면에서 숫자가 커 보이는 건 대개 업데이트다. 홍보 성과는 **첫 번째 다운로드**만 떼어봐야 하고, 그 구분은 `App Downloads Detailed`의 다운로드 유형 차원에 있다.
- **미출시 앱은 분석 데이터가 존재하지 않는다.** 앱 레코드가 있어도 `PREPARE_FOR_SUBMISSION`이면 TestFlight 빌드뿐이라 볼 게 없다. 다만 **출시 전에 `ONGOING`을 미리 걸어두면 출시 첫날부터 데이터가 쌓인다.**
- 도구: `(로컬 경로)`에 인증된 raw 호출 `asc get <path>` · `asc post <path> <json>`을 추가했다(2026-09-01). 키는 `fastlane/.env`를 읽고, 키 파일 경로가 `AuthKey.p8`이 아니면 `ASC_KEY_PATH=<경로>`로 넘긴다 — 탭탭은 `.env`의 `ASC_KEY_FILEPATH` 값을 그대로 쓰면 된다.

## 기록

### 2026-09-01 — 탭탭 iOS·macOS 분석 지표를 보려다 파이프라인부터 켠 건

- 맥락: 탭탭 홍보 착수를 앞두고 iOS·macOS 분석을 정리하려 했다.
- 배운 것: 위 「핵심 정리」 전부. 결론은 **볼 데이터가 아직 없었다는 것** — iOS는 리포트 요청 자체가 0건이었고, macOS는 앱이 미출시였다.
- 근거: `asc get "/v1/apps/6754357960/analyticsReportRequests"` → `total: 0`. `asc post`로 iOS에 `ONGOING`(`d20791e4…`)·`ONE_TIME_SNAPSHOT`(`303fca7a…`), macOS(appId 6795730513)에 `ONGOING`(`e28d80a8…`) 생성(201). 직후 `…/reports` 156개 조회되지만 `App Downloads Standard`(r3)·`App Store Discovery and Engagement Standard`(r14)·`App Store Installation and Deletion Standard`(r6) 전부 instances `total: 0`.
- 부수 확인: iOS 앱(6754357960)은 1.0(2025-10-22)~1.2.0(2026-08-02) 9개 버전 출시, 리뷰 7건(5★ 6·4★ 1). 요청이 **iPad 최적화 2회·기기 간 연동·저장 링크를 사파리로 열기·위젯**에 몰려 있다. macOS(6795730513)는 `PREPARE_FOR_SUBMISSION`·빌드 미연결·스크린샷 0장인데 TestFlight엔 1.1.0을 올리는 중이라 제출 전에 버전 정합성을 맞춰야 한다.
