---
type: hub
title: "AppStore 작업노트"
summary: "앱 배포, 스토어 심사, 인앱 구매, 분석 리포트와 개발자 계정"
status: active
created: 2026-11-19
updated: 2026-11-19
---

# AppStore 작업노트

[전체 분야](../README.md#주제-목록)

- [인앱 구매 등록 API](%EC%9D%B8%EC%95%B1%20%EA%B5%AC%EB%A7%A4%20%EB%93%B1%EB%A1%9D%20API.md) — ASC API로 IAP를 만들면 `MISSING_METADATA`로 시작하고 **다섯 조각**(생성·로컬라이제이션·가격·**지역**·심사 스크린샷)을 다 채워야 `READY_TO_SUBMIT`이 된다. 빠진 것을 알려주지 않는다. 관계 이름은 `inAppPurchaseV2`, 가격 스케줄에 `data.id`를 넣으면 409 (2026-09-09)
- [스토어 스크린샷 중복 업로드](%EC%8A%A4%ED%86%A0%EC%96%B4%20%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%20%EC%A4%91%EB%B3%B5%20%EC%97%85%EB%A1%9C%EB%93%9C.md) — `overwrite_screenshots: true`인데도 deliver가 두 벌을 올리는 일이 있고, **심사 제출 뒤엔 삭제가 막힌다**(409 "Can't Delete Screenshot After Submit for review"). 라이브 제품 페이지는 새 버전으로만 고친다. 방어는 업로드→dedupe→제출로 쪼개는 것 (2026-09-09)
- [App Store 성장 도구](App%20Store%20%EC%84%B1%EC%9E%A5%20%EB%8F%84%EA%B5%AC.md) — **Mac 전용 앱은 Apple Ads·인앱 이벤트·커스텀 제품 페이지가 전부 막힌다**(전부 iPhone/iPad 전용). 남는 것은 피처링 노미네이션(최소 3주 전)·프로모션 텍스트 (2026-08-19)
- [App Store Connect 분석 리포트 API](App%20Store%20Connect%20%EB%B6%84%EC%84%9D%20%EB%A6%AC%ED%8F%AC%ED%8A%B8%20API.md) — 앱마다 `analyticsReportRequests`를 POST해야 데이터가 쌓이기 시작한다(`ONGOING`·`ONE_TIME_SNAPSHOT` 별개). 요청 직후 리포트 정의 156개는 보이지만 **instances는 한동안 `total: 0`**. 급하면 `salesReports`인데 `vendorNumber`는 API로 못 얻는다. 추세의 '단위'엔 업데이트가 섞여 있다 (2026-09-01)
- [TestFlight 내부 그룹과 수출 규정](TestFlight%20%EB%82%B4%EB%B6%80%20%EA%B7%B8%EB%A3%B9%EA%B3%BC%20%EC%88%98%EC%B6%9C%20%EA%B7%9C%EC%A0%95.md) — 내부 그룹엔 빌드를 못 붙인다(자동 배포), 수출 규정 질문은 `ITSAppUsesNonExemptEncryption`로 없앤다 · **맥 TestFlight가 리딤 코드를 요구하면 그 Apple ID가 macOS 테스터 그룹에 없는 것**(iOS·macOS 그룹은 별개) · **리젝된 reviewSubmission은 되살릴 수 없다** — `PATCH submitted:true`는 409 `STATE_ERROR`(메타데이터 문제처럼 읽히지만 아니다), 옛 제출 `canceled:true` → 새 reviewSubmission 생성·항목 추가·제출 (2026-09-16)
- [App Store Server Notifications](App%20Store%20Server%20Notifications.md) — 비소모성도 프로덕션 ONE_TIME_CHARGE 수신, 실패 시 1h→12h→24h→48h→72h 재시도(72h 안에 고치면 자동 도착), 전송 이력·테스트는 App Store Server API(ASC 팀 키로 가능) (2026-08-19)
- [유료 앱 판매 알림](%EC%9C%A0%EB%A3%8C%20%EC%95%B1%20%ED%8C%90%EB%A7%A4%20%EC%95%8C%EB%A6%BC.md) — **유료 다운로드 앱은 App Store Server Notifications가 안 온다**(IAP·구독 전용). 대안은 `salesReports` DAILY를 Vercel Cron(07:00 KST)으로 당겨 다음 날 Slack 요약. 거래 0건이면 404, 유료 판매는 `Customer Price > 0`으로 가른다. 수동 실행은 대시보드 Run 버튼뿐 (2026-09-03)
- [미국 세금 양식 W-8BEN](%EB%AF%B8%EA%B5%AD%20%EC%84%B8%EA%B8%88%20%EC%96%91%EC%8B%9D%20W-8BEN.md) — **ASC 웹 세금 양식에는 Line 10(Article) 칸이 없다.** 조약 조항을 지정하려면 IRS 원본 PDF를 작성해 Finance Support 링크에 올리고 **파일명을 Case ID 8자리로** + **메일 회신까지** 해야 접수된다. 갱신 결과는 ASC 화면에 절대 반영되지 않는다. 한미조세조약 로열티는 Article 14(일반 15%, 저작권 10%) · **이름·주소의 정답지는 ASC 비즈니스 화면의 Legal Entity**이지 사업자등록증명이 아니다 (2026-09-03)
- [Google Play 개발자 계정](Google%20Play%20%EA%B0%9C%EB%B0%9C%EC%9E%90%20%EA%B3%84%EC%A0%95.md) — **Android 첫 출시의 병목은 코드가 아니라 계정 유형이다.** 개인 계정은 신규 앱마다 **테스터 12명 × 14일 연속 비공개 테스트** 필수, 조직(사업자) 계정은 면제 — 개인사업자로도 만들 수 있다. 단 조직 계정은 **D-U-N-S(최대 30일) + 도메인 연결 웹사이트**를 요구한다. **개인사업자에게 D-U-N-S는 구글 전용** — 애플은 개인사업자를 Organization으로 안 받고 Individual엔 애초에 불필요하다 (2026-08-26) · DUNS 없이도 개인으로 결제한 뒤 조직으로 전환 가능(09-01, 08-26 정정) · 발급됐다는 DUNS가 다른 회사 레코드 — 미해결 (2026-09-02)
- [앱 이름 속 Apple 상표 (5.2.5)](%EC%95%B1%20%EC%9D%B4%EB%A6%84%20%EC%86%8D%20Apple%20%EC%83%81%ED%91%9C%20%285.2.5%29.md) — 5.2.5 리젝은 스토어 등재명이 아니라 **기기에 표시되는 앱 이름**을 본다. `CFBundleDisplayName`이 없으면 조용히 `CFBundleName = $(PRODUCT_NAME)` = **타깃명**이 노출돼 `TapTapMac`처럼 Apple 제품명이 샌다. 빌드 설정에 이름을 넣어도 Info.plist 매핑이 없으면 에러 없이 버려진다 — 제출 전 아카이브 plist를 읽어 확인한다 (2026-09-11)
- [앱 이름과 검색 노출](%EC%95%B1%20%EC%9D%B4%EB%A6%84%EA%B3%BC%20%EA%B2%80%EC%83%89%20%EB%85%B8%EC%B6%9C.md) — **짧은 조어 브랜드명은 흔한 단어에 먹힌다.** `Fadeo`는 `Video`에 퍼지 매칭돼 **자기 이름 검색 200위 밖**(KR·US). 이름 짓기 전 iTunes Search API로 *정확 일치가 없을 때 무엇이 대신 나오는지*를 봐야 한다. 키워드는 단어 뜻이 아니라 **그 검색어가 속한 시장** 기준 — `인화`는 찍스·스냅스 같은 사진인화 서비스 시장이라 카메라 앱이 못 잡는다 (2026-09-02) · 한능검: "한능검" 3위인데 "한국사"는 색인 없음 — 부제 개정 때 단어가 빠졌다 (2026-09-04)
- [앱 내 계정 삭제 요건](%EC%95%B1%20%EB%82%B4%20%EA%B3%84%EC%A0%95%20%EC%82%AD%EC%A0%9C%20%EC%9A%94%EA%B1%B4.md) — 계정 생성 앱은 **앱 안에서 계정 삭제**가 필수(2022-06-30~)이고 **Sign in with Apple을 쓰면 삭제 시 REST API로 토큰 해지**까지가 요건이다. 해지하려면 서버가 애플 refresh 토큰을 갖고 있어야 하므로 ID 토큰만 검증하는 구조는 선행 설계가 필요 (2026-09-04)
- [실물을 흉내 내는 앱의 상표 위험](%EC%8B%A4%EB%AC%BC%EC%9D%84%20%ED%9D%89%EB%82%B4%20%EB%82%B4%EB%8A%94%20%EC%95%B1%EC%9D%98%20%EC%83%81%ED%91%9C%20%EC%9C%84%ED%97%98.md) — 폴라로이드 흰 테두리는 **classic border logo** 등록 상표이고 재판 계류 중, 실질 위험은 소송이 아니라 **App Store 신고 한 건** (2026-08-26)
