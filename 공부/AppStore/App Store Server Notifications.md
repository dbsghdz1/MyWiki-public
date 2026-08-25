---
type: study
area: 도구·인프라
audience: ai
status: active
created: 2026-08-19
updated: 2026-08-19
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# App Store Server Notifications

Apple이 구매·환불 이벤트를 내 서버로 밀어주는 V2 웹훅. **내 서버가 2xx를 못 돌려주면 Apple이 시간을 두고 재시도**하고, 보냈는지·실패했는지는 App Store Server API의 알림 이력으로 Apple 쪽에서 확인할 수 있다.

## 핵심 정리

- **비소모성 일회 구매도 프로덕션에서 `ONE_TIME_CHARGE`가 온다** (2026-08 관측, Zappy+ ₩3,300). 초기 문서엔 샌드박스 한정이었으나 지금은 프로덕션 실구매에서 수신됨.
- **재시도**: 서버가 200 아닌 응답을 주면 Apple이 다시 보낸다. 관측: 최초 실패 1시간 뒤 재시도. Apple 문서 기준 최대 5회(1h·12h·24h·48h·72h 간격). 즉 **72시간 안에 서버를 고치면 놓친 알림이 저절로 도착**한다 — 재전송 API는 없다.
- **전송 이력 조회**: `POST https://api.storekit.itunes.apple.com/inApps/v1/notifications/history` body `{"startDate":ms,"endDate":ms}` → 각 알림의 `signedPayload`(JWS, base64url 디코드하면 notificationType·환경·거래정보)와 `sendAttempts[]`(시각·`SUCCESS`/`UNSUCCESSFUL_HTTP_RESPONSE_CODE`). "우리 서버에 요청이 왔나?"는 내 로그보다 이게 정확하다.
- **테스트 알림**: `POST /inApps/v1/notifications/test` → `testNotificationToken`; `GET /inApps/v1/notifications/test/<token>`으로 결과. 등록된 프로덕션 URL로 실제 `TEST` 타입이 날아오므로 종단 점검용.
- **인증**: App Store Server API JWT는 `kid`(키 ID)·`iss`(issuer)·`aud: appstoreconnect-v1`·`bid`(번들 ID)·`iat/exp`를 ES256으로 서명. **App Store Connect API 팀 키(fastlane용 AuthKey.p8)로도 호출됐다** — 별도 In-App Purchase 키가 필수는 아니었음(적어도 notifications 계열은).
- 페이로드 검증은 서버가 직접: `x5c` 체인(리프→중간→Apple Root CA G3) 확인 + ES256 서명 검증. 검증 실패는 400(위조 가능성, 재시도 유도 안 함).
- 구매자 이메일은 오지 않는다 — 상품·금액(milliunits)·스토어프런트·시각·환경만.

## 기록

### 2026-08-19 — 구매 알림 미수신 원인 규명: 서버 404 → Apple 재시도 대기
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] — Slack 구매 알림 웹훅(`landing/api/appstore-webhook.js`)이 08-18 밤 구매에 반응하지 않음
- 배운 것:
  - 알림 이력 API로 3건 확인: 08-08·08-10 `SUCCESS`, **08-18 23:03 ONE_TIME_CHARGE는 23:03·00:03 두 번 `UNSUCCESSFUL_HTTP_RESPONSE_CODE`** — 우리 웹훅이 404였기 때문([[공부/도구/Vercel 배포|Vercel 배포]]). 파이프라인 코드·Slack·ASC URL 등록은 전부 정상이었다.
  - 서버 복구 후 테스트 알림 2회 모두 `SUCCESS`(04:04:11). 놓친 구매 건은 Apple의 12h 재시도(08-19 ~11:03)에 실리도록 두기로 함 — 수동 재전송하면 재시도와 중복.
  - fastlane의 ASC 팀 키로 App Store Server API가 200으로 열렸다(`bid` 클레임 포함).
- 근거: `notifications/history` 응답(스크래치 `hist.json`), 테스트 토큰 `76b6cd44…`·`0634aa67…` 결과 `SUCCESS`; 호출 스크립트는 node `crypto.sign('sha256', …, {dsaEncoding:'ieee-p1363'})`로 JWT 생성 → `fetch`. [[프로젝트/개인/Zappy/Zappy 마케팅 플랜]] 장애 기록

## 참고 자료
- [Apple — Receiving App Store Server Notifications](https://developer.apple.com/documentation/appstoreservernotifications/receiving-app-store-server-notifications) — 재시도 정책·응답 요건 (2026-08-19 열림 확인, 본문은 JS 렌더)
- [Apple — Get Notification History](https://developer.apple.com/documentation/appstoreserverapi/get-notification-history) — 이력 API 요청/응답 스키마 (2026-08-19 열림 확인)
- [Apple — Request a Test Notification](https://developer.apple.com/documentation/appstoreserverapi/request-a-test-notification) — 테스트 알림 엔드포인트 (2026-08-19 열림 확인)
