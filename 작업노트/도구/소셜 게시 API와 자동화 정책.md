---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-12
updated: 2026-09-12
projects:
  - "홍보 자동화"
  - "[[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]]"
---

# 소셜 게시 API와 자동화 정책 — Reddit · X · Threads · Instagram

한 줄 요약 — **Instagram·Threads는 공식 API로 무료 자동 게시가 되고, X는 건당 과금, Reddit은 초안까지만이 안전선이다.** 자동 답글·DM은 네 곳 모두 "상대가 먼저 반응한 경우"로 묶여 있다. 서버가 알아서 답하게 두지 말고 초안 → 사람 승인으로 간다.

## 핵심 정리 (2026-09-12 확인)

- **X — 무료 등급 없음, 쓴 만큼 과금.**
  - 가격: 게시 `$0.015`/건, **URL 포함 게시 `$0.20`/건**, DM `$0.015`, 게시물 읽기 `$0.005`. 크레딧을 먼저 충전하고 월 결제 한도를 설정할 수 있다.
  - 호출 한도: 게시 사용자당 100건/15분, DM 15건/15분.
  - 크레딧이 없으면 쓰기에서 `402 credits depleted`가 난다.
- **X 자동화 규칙.**
  - 자동 답글은 상대가 먼저 반응했을 때 반응 1건당 1개까지만.
  - **AI가 쓴 답글은 "Requires prior approval from X".**
  - 키워드로 찾은 글에 자동 답글·자동 환영 DM을 보내면 안 되고, 여러 계정에 같은 글을 올려도 안 된다.
  - 남을 대신해 게시할 때는 "Show exactly what will be published" 후 권한을 받는다 — Slack 승인 흐름이 이 요건에 맞는다.
  - API 자동화 계정은 프로필에 "Automated" 라벨이 필요하다. 사람이 승인한 글만 올리는 계정도 해당하는지는 문서가 모호하다.
- **Threads — 무료, 본인 계정은 심사 없이.**
  - 인증과 심사: Threads OAuth를 쓴다. 앱에 **본인을 tester로 등록하면 App Review가 필요 없다**(공개 사용자에게 권한을 받을 때만 필요). 2026-08-12 조사표의 "Tech Provider Verification + App Review 필요"는 이 경우에 해당하지 않는다.
  - 토큰: 장기 토큰은 60일이고 `refresh_access_token`으로 갱신한다.
  - 게시: Instagram처럼 컨테이너 생성 → 발행 2단계다. 본문 500자, 링크 5개 이하, 캐러셀 2~20개.
  - 한도: **게시 250건·답글 1,000건/24시간**. 예약 파라미터가 없어 대기열은 서버에 둔다.
  - 답글 웹훅은 Advanced Access + 비즈니스 인증이 필요하다. 1인 운영은 `GET {media-id}/replies` 폴링이 현실적이다.
- **Instagram — 이미 발행 중인 경로 그대로.**
  - 게시: `graph.instagram.com`, 프로페셔널 계정, FB 페이지는 필요 없다. 게시 100건/24시간, 캐러셀 10개, 이미지는 JPEG만, 예약 파라미터 없음.
  - **AI 생성 표시는 컨테이너를 만들 때 `is_ai_generated=true`.**
  - DM: 상대가 먼저 보낸 뒤 24시간 안에만 보낼 수 있다. 댓글 작성자에게 보내는 private reply는 댓글 7일 안에 1건.
  - Meta 정책 1.4 "Don't … spam or surprise anyone". 커뮤니티 기준은 "at very high frequencies"로 수동·자동 게시하는 것과 반복 콘텐츠를 금지한다.
- **Reddit — 초안까지만.**
  - API 앱: 2025-11 Responsible Builder Policy 이후 **새 OAuth 앱은 수동 승인**을 받는다. 자동화 계정은 "App" 라벨이 필요하고, 자동 게시·댓글·DM으로 스팸하면 안 된다(원문은 차단돼 스니펫으로만 확인).
  - Data API Terms(2026-07-20 개정): "spam, incentivize, or harass users"에 쓰면 안 되고, 상업적 용도는 별도 계약이다.
  - 자기 홍보: 일부 커뮤니티가 **10% 규칙**을 쓰고 karma 최소치를 요구한다. 새 계정은 곧바로 섀도밴을 당한 사례가 있다. 링크를 올릴 때는 "(I'm the founder)"라고 밝힌다.
  - **서브레딧 규칙 원문(r/macapps, r/iOSProgramming, r/SideProject, r/apple)은 403으로 못 봤다** — 게시 전에 사이드바를 직접 확인한다.
- **Oracle E2.1.Micro(RAM 1GB)에서는 헤드리스 브라우저로 게시할 수 없다** — 게이트웨이 혼자 swap을 쓴다(인프라 실측). 서버 자동화는 공식 API 호출로만 한다.
- **예약 도구가 API 문제를 대신 풀어주지는 않는다.**
  - Buffer는 IG·X·Threads를 지원하고 서버 API가 있으며, 무료 3채널이다. Reddit은 지원하지 않는다.
  - Typefully는 IG·Reddit을 지원하지 않는다. Hypefury는 X 지원을 종료했다.
  - Postiz(오픈소스)는 Reddit·X까지 붙지만 결국 **본인 개발자 앱**이 필요하다.
- **한국 이용자**: 모바일인덱스 2025-07 기준 스레드 MAU 543만(전년 대비 약 180% 증가), X는 600만 명대로 정체다. 한국어 앱 홍보에서 Threads를 X보다 뒤로 둘 이유가 없다.

## 기록

### 2026-09-12 — 홍보 오토파일럿 설계를 위한 4개 플랫폼 조사

- **맥락**: 홍이 "앱 홍보(Reddit·X·Threads)와 인스타 운영을 맥을 안 쓰는 시간에 맡기고 싶다"고 요청했다 → 오토파일럿 설계안. 인프라 실측과 함께 진행했다.
- **배운 것**: 위 핵심 정리 전부. 설계에 가장 크게 작용한 것은 셋이다.
  - ① Threads는 본인 tester 토큰이면 심사 없이 바로 된다 — 08-12 조사의 "심사 1~4주" 전제가 무너졌다.
  - ② X의 AI 자동 답글은 사전 승인 사항이다 → 답글은 전 채널 초안만.
  - ③ Reddit은 API 승인과 커뮤니티 규칙 둘 다 막혀 있다 → 사람이 직접 게시한다.
- **근거**: 아래 참고 자료(WebFetch로 연 페이지). reddit.com·help.x.com·support.reddithelp.com은 WebFetch가 차단돼 redditinc.com 약관만 curl로 원문을 받았다.
- **못 확인한 것**:
  - Reddit Responsible Builder Policy 원문과 서브레딧 규칙 전부
  - Reddit API 요금·한도(제3자 글 기준만 봤다)
  - X 멘션 조회 단가와 미디어 업로드 과금
  - 사람이 승인한 API 게시 계정에도 X "Automated" 라벨 의무가 있는지
  - Threads API의 DM 지원
  - Typefully 가격

## 참고 자료

- [X API Pricing](https://docs.x.com/x-api/getting-started/pricing) — 사용량 과금 단가 (2026-09-12 확인)
- [X API Rate limits](https://docs.x.com/x-api/fundamentals/rate-limits) (2026-09-12 확인)
- [X Developer Guidelines](https://docs.x.com/developer-guidelines) · [Developer Policy](https://docs.x.com/developer-terms/policy) — 자동 답글·AI 답글 사전 승인·"Show exactly what will be published" (2026-09-12 확인)
- [Threads API — Get Started](https://developers.facebook.com/docs/threads/get-started) · [Posts](https://developers.facebook.com/docs/threads/posts) · [Overview(한도)](https://developers.facebook.com/docs/threads/overview) · [Webhooks](https://developers.facebook.com/docs/threads/webhooks) (2026-09-12 확인)
- [What's new in the Threads API (2026-04-14)](https://developers.facebook.com/blog/post/2026/04/14/whats-new-in-the-threads-api/) (2026-09-12 확인)
- [Instagram API with Instagram Login](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login) · [Private Replies](https://developers.facebook.com/docs/instagram-platform/private-replies) (2026-09-12 확인)
- [Meta Platform Terms / Developer Policies](https://developers.facebook.com/devpolicy/?locale=en_US) · [Meta Community Standards — Spam](https://transparency.meta.com/policies/community-standards/spam/) (2026-09-12 확인)
- [Reddit Data API Terms](https://redditinc.com/policies/data-api-terms) · [Developer Terms](https://redditinc.com/policies/developer-terms) · [Reddit Rules](https://redditinc.com/policies/reddit-rules) — curl로 원문 열람 (2026-09-12)
- [Self-promotion on Reddit the right way — Vadim Kravcenko](https://vadimkravcenko.com/qa/self-promotion-on-reddit-the-right-way/) — 인디 개발자 관점 (2026-09-12 확인)
- [Buffer Pricing](https://buffer.com/pricing) · [Postiz providers](https://docs.postiz.com/providers/overview) (2026-09-12 확인)
- [헤럴드경제 — 모바일인덱스 2025-07 스레드·X 국내 MAU](https://biz.heraldcorp.com/article/10546627) (2026-09-12 확인)
