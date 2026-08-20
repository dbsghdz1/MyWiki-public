---
type: source
source_type: 웹 조사
title: 홍보 자동화 채널 API 현황 조사
publisher: 각 출처 병기
captured: 2026-08-12
---

# 홍보 자동화 채널 API 현황 조사 (2026-08-12)

> 2026-08-12에 웹 검색으로 수집. 홍보 자동화 Phase 0 조사 결과 원본.

## X API — 무료 티어 폐지, pay-per-use

- 2026-02-06부터 신규 개발자에게 무료 티어 없음 — pay-per-use가 기본. 기존 무료 티어 사용자는 $10 크레딧과 함께 이전됨.
- 요율: 포스트 작성 $0.015, **URL 포함 포스트 $0.20**, 읽기 $0.005/post.
- (구 무료 티어는 월 ~1,500 write였으나 폐지)
- 출처: [Postproxy X API Pricing 2026](https://postproxy.dev/blog/x-api-pricing-2026/), [Blotato Twitter API Pricing](https://www.blotato.com/blog/twitter-api-pricing), [Sorsa X API Pricing 2026](https://api.sorsa.io/blog/twitter-api-pricing-2026)

## Threads API — 무료, 사전 심사 필요

- 무료. graph.threads.net, 컨테이너 생성 → publish 2단계.
- **Tech Provider Verification 필요 (~1주)** + `threads_content_publish` 권한 App Review 필요.
- 한도: 250 posts/24h. 텍스트 500자, 이미지 JPEG/PNG 8MB.
- Instagram 비즈니스 계정·Facebook 페이지 불필요, Threads 프로필만 연결하면 됨.
- 출처: [Postproxy Threads 가이드](https://postproxy.dev/blog/how-to-post-to-threads-via-api/), [Threads App Review 경험](https://singhamandeep.com/threads-api-app-review-permissions/), [Blotato Threads API Pricing](https://www.blotato.com/blog/threads-api-pricing)

## App Store Connect Analytics Reports API — 지표 루프 구현 가능

- 다운로드·수익·구독 리포트를 벌크 export — ONGOING 요청 시 daily/weekly/monthly 반복 생성, ONE_TIME_SNAPSHOT으로 과거 전체.
- 형식: .txt.gz (tab-delimited). 첫 리포트는 요청 후 24–48시간.
- 권한: 첫 리포트 타입 요청은 Admin 키, 이후 다운로드는 Sales and Reports/Finance 역할 키.
- 출처: [Apple 공식 — Analytics reports API](https://developer.apple.com/help/app-store-connect-analytics/overview/analytics-reports-api), [App Analytics](https://developer.apple.com/app-store-connect/analytics/)

## Instagram Content Publishing API — 요건 무거움

- Instagram Professional(비즈니스/크리에이터) 계정 + Facebook 페이지 연결 + Meta 개발자 앱 필요. 개인 계정 불가.
- `instagram_business_content_publish` App Review 필요 — 스크린캐스트 제출, **2–4주 소요**.
- 발행: media 컨테이너 → media_publish 2단계.
- 출처: [Postproxy Instagram 가이드](https://postproxy.dev/blog/post-to-instagram-via-api/), [Elfsight Instagram Graph API 2026](https://elfsight.com/blog/instagram-graph-api-complete-developer-guide-for-2026/), [Storrito — 2026 API 규칙](https://storrito.com/resources/Instagram-API-2026/)
