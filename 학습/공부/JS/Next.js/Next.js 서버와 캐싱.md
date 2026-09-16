---
type: study
area: JS
audience: me
status: active
created: 2026-08-18
updated: 2026-09-16
aliases: [서버 컴포넌트, revalidate, App Router, fetch 캐싱, Next 16]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# Next.js 서버와 캐싱

Next.js는 **React를 서버에서 실행해 HTML을 만들어 브라우저로 보내는 프레임워크**다. React가 Next에게 결과를 주는 게 아니라 **Next이 React를 부른다**. 그래서 "서버"이자 "백엔드"이고, 이 프로젝트에 별도 백엔드가 없는 이유다.

**2026-09-16 재편**: 한 파일(239줄)에 있던 8개 절을 폴더로 쪼갰다. 이 파일은 파이프라인·서버 컴포넌트·fetch 캐싱만 든다.

## 이 폴더

- [[학습/공부/JS/Next.js/빌드 타임과 런타임|빌드 타임과 런타임]] — 축이 두 개(tsc·Tailwind는 빌드 타임), `NEXT_PUBLIC_`은 허가가 아니라 「번들에 박아라」는 명령
- [[학습/공부/JS/Next.js/proxy 미들웨어와 matcher|proxy 미들웨어와 matcher]] — 요청마다 돈다, 정적 파일을 안 빼면 성능이 아니라 파손, `matcher: []`는 실행 0회, Route Group layout으로 인증 경계
- [[학습/공부/JS/Next.js/Route Handler와 내부 API|Route Handler와 내부 API]] — `app/api`는 HTTP 창구, `src/**/api`는 내부 함수. 브라우저는 서버 파일을 import할 수 없어서 HTTP가 필요하다

## 핵심 정리

### 파이프라인

```
1. 브라우저: localhost:3000 요청
2. Next 서버: 라우팅으로 실행할 컴포넌트 결정 (app/page.tsx)
3. React: 컴포넌트 함수 실행 → JS 객체 트리 → HTML
4. Next 서버: HTML + CSS를 전송
5. 브라우저: 그린다
```

- **JSX는 HTML도 CSS도 만들지 않는다.** `<div className="glass">안녕</div>`은 빌드 때 `jsx('div', { className: 'glass', children: '안녕' })`라는 **자바스크립트 객체**가 된다. React는 이 설명서를 받아 서버에선 HTML로, 브라우저에선 DOM 조작으로 바꾼다.
- CSS는 완전히 별개 경로다. `className`은 이름표 문자열일 뿐이고 실제 스타일은 `globals.css` + Tailwind가 빌드 때 만든 CSS 파일에 있다.
- **서버/백엔드**는 다른 단어를 가리킨다: 서버 = 실행되는 곳(node 프로세스), 백엔드 = 하는 역할(DB·외부 API·비밀키). Next는 둘 다다.

### 서버 컴포넌트

`'use client'`가 없는 컴포넌트는 **서버에서만 실행**된다. 컴포넌트의 JS가 브라우저로 가지 않고 실행 결과 HTML만 간다. 그래서 컴포넌트 함수 자체를 `async`로 만들고 `await`로 데이터를 기다렸다가 완성된 화면을 반환할 수 있다 — 로딩 상태 관리가 필요 없다.

```tsx
export default async function LiveMarketCard() {
  const tickers = await getTickers(['KRW-BTC']);
  return <div>{tickers[0].trade_price}</div>;
}
```

iOS에서 `viewDidLoad` → URLSession → completion에서 UI 갱신하던 흐름과 근본적으로 다르다.

| | 서버 (Node) | 브라우저 |
|---|---|---|
| 파일·DB 접근, 비밀키 | ✅ | ❌ (개발자도구에 다 보임) |
| `window`, 클릭 이벤트, `useState` | ❌ | ✅ |
| 우리 코드에서 | 기본 | `'use client'` 붙인 것 |

### fetch 캐싱 — Next 16은 기본이 "캐시 안 함"

유저 100명이 홈에 들어오면 서버 컴포넌트가 100번 실행되고 외부 API를 100번 부른다. 업비트 레이트리밋(~10 req/s)에 바로 걸린다. 그래서 재사용 기간을 **명시**한다.

```ts
fetch(url, { next: { revalidate: 5 } })   // 5초 안에 같은 URL이면 저장된 응답을 준다
```

- 브라우저 캐시가 아니라 **Next 서버 안의 캐시**다. 100명이 5초 안에 들어와도 업비트 요청은 1번.
- `next`는 표준 fetch에 없는 **Next이 추가한 옵션**이다.
- **`revalidate` 값은 데이터가 변하는 속도로 정한다.** MyCryptoDiary: ticker 5초(시세는 자주 변함) · 일봉 60초(하루 단위 데이터) · 마켓 목록 3600초(상장 목록은 거의 안 변함).

### Next 16에서 바뀐 것

- `params` / `searchParams`는 **Promise** — `await` 필수 (route handler의 `ctx.params`도).
- `fetch`는 기본 캐시 안 함 — `next: { revalidate: N }` 명시.
- middleware → **`proxy.ts`**로 개명.
- `next lint` 제거 → `eslint` 직접 실행.

## 기록

### 2026-08-18 — 업비트 연동 (D1)

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]])에서 `shared/api/upbit` + `app/api/upbit/ticker/route.ts`를 만들며.
- 배운 것: 위 핵심 정리 전체. 특히 **"React가 Next에게 HTML을 준다"고 거꾸로 이해하고 있었던 것**을 바로잡았고, JSX가 HTML 문자열이 아니라 JS 객체를 만든다는 것을 처음 알았다.
- 검증: `curl "localhost:3000/api/upbit/ticker?markets=KRW-BTC"` → 실시세 JSON, 파라미터 없이 호출 → 400. D1 DoD 항목.
- 근거: 커밋 `c761e7a`(client), `2551b96`(route handler)

## 참고 자료

- [Next.js — Server Components](https://nextjs.org/docs/app/getting-started/server-and-client-components) — 서버/클라이언트 컴포넌트 경계 (2026-08-18 확인)
- [Next.js — fetch 옵션](https://nextjs.org/docs/app/api-reference/functions/fetch) — `next.revalidate` (2026-08-18 확인)
