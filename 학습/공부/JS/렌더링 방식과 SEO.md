---
type: study
area: JS
audience: me
status: active
created: 2026-08-30
updated: 2026-08-30
---

# 렌더링 방식과 SEO

배포 플랫폼(Vercel)이 SEO를 결정하는 게 아니라 **크롤러가 받는 첫 HTML에 콘텐츠가 있느냐**가 결정한다.

## 핵심 정리

- **"Vercel로 배포하면 SEO 되나?" → 된다. 안 되는 경우는 Vercel 탓이 아니라 SPA 탓이다.** HTTPS·CDN·빠른 응답은 오히려 SEO에 유리한 조건.
- 갈림길은 렌더링 방식:
    - **SSR/SSG** (Next.js 등) — 서버가 완성된 HTML을 내려줌 → 모든 크롤러가 콘텐츠를 바로 읽음. ✅
    - **순수 CSR/SPA** (Vite+React 등) — 첫 HTML이 빈 `<div id="root">` + JS 번들 → 크롤러가 JS를 실행해야만 콘텐츠가 보임. ⚠️
- **Google은 3단계로 JS를 처리한다**: 크롤링 → **렌더링 큐**(headless Chromium으로 JS 실행) → 색인. SPA도 색인은 되지만 렌더링 큐를 거치므로 느리고 불완전할 수 있다.
- **한국 서비스면 더 중요**: 네이버 검색 크롤러·카카오톡 링크 미리보기(OG 태그) 봇은 JS 실행을 거의 안 한다. 순수 SPA는 네이버 노출·카톡 공유 미리보기가 깨질 가능성이 높다. → SEO가 목표면 **Next.js SSR/SSG로 만들어 Vercel에 올리는 게 정석**.
- **Vercel 프리뷰 배포는 자동 noindex**: 프로덕션이 아닌 브랜치 배포(`프로젝트-xxx.vercel.app`)에는 `X-Robots-Tag: noindex` 헤더가 자동으로 붙는다(프로덕션 URL과 색인 경쟁 방지). 프로덕션 배포는 색인 가능. 단, 프리뷰 브랜치에 **커스텀 도메인을 붙이면 이 헤더가 빠지므로** staging을 숨기려면 직접 noindex를 넣어야 한다.
- 나머지는 앱에서 챙기는 것: `<title>`·메타 태그(Next.js Metadata API), OG 태그, `sitemap.xml`, `robots.txt`.

## 기록

### 2026-08-30 — "Vercel로 배포하면 SEO 할 수 있어?"

- 맥락: 배포 방식을 고민하다 나온 질문. 특정 프로젝트에 속하지 않은 개념 질문.
- 배운 것: 위 핵심 정리 전체. 플랫폼이 아니라 **렌더링 방식이 SEO를 가른다**는 것, Google의 렌더링 큐, Vercel 프리뷰 noindex 동작.
- 근거: 아래 참고 자료 (2026-08-30 전부 열어서 확인).

## 참고 자료

- [Understand JavaScript SEO Basics — Google Search Central](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics) — Google이 JS를 크롤링→렌더링 큐→색인 3단계로 처리하는 원리, JS SEO의 원전 (2026-08-30 확인)
- [Are Vercel Preview Deployments indexed by search engines? — Vercel KB](https://vercel.com/kb/guide/are-vercel-preview-deployment-indexed-by-search-engines) — 프리뷰 배포 자동 `X-Robots-Tag: noindex`와 커스텀 도메인 예외 (2026-08-30 확인)
- [Server and Client Components — Next.js 공식 문서](https://nextjs.org/docs/app/getting-started/server-and-client-components) — 서버 컴포넌트가 완성된 HTML을 내려주는 구조(SSR이 SEO에 유리한 이유의 메커니즘) (2026-08-30 확인)
- 네이버 서치어드바이저의 JS 렌더링 가이드는 URL 접근 확인에 실패해 링크를 남기지 않음 — "네이버 크롤러는 JS 실행이 제한적"이라는 본문 서술은 널리 알려진 실무 통념 수준으로만 받아들일 것.
