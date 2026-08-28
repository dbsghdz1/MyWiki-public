---
type: study
area: JS
audience: me
status: active
created: 2026-08-18
updated: 2026-08-28
aliases: [서버 컴포넌트, Route Handler, revalidate, App Router, NEXT_PUBLIC, 환경변수]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# Next.js 서버와 캐싱

Next.js는 **React를 서버에서 실행해 HTML을 만들어 브라우저로 보내는 프레임워크**다. React가 Next에게 결과를 주는 게 아니라 **Next이 React를 부른다**. 그래서 "서버"이자 "백엔드"이고, 이 프로젝트에 별도 백엔드가 없는 이유다.

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

### 축이 두 개다 — 빌드 타임 / 런타임(서버·브라우저)

| | 빌드 타임 | 런타임 |
|---|---|---|
| 언제 | 코드 저장·배포할 때 한 번 | 유저가 페이지 열 때마다 |
| 누가 | tsc, Tailwind, 번들러 | Node(서버), 브라우저 |
| 하는 일 | 타입 검사 후 제거, CSS 생성, JS 묶기 | React 실행 → HTML, fetch, DOM 조작 |

```
빌드 타임 (tsc, Tailwind)
   ↓ 결과물
런타임 ─ 서버 (Node): 서버 컴포넌트, route handler, DB, API 키
      └ 브라우저:     'use client' 컴포넌트, 클릭, useState
```

**Tailwind는 런타임에 호출되지 않는다.** 빌드 때 소스 전체를 훑어 `glass`·`text-rise` 같은 완성된 클래스명을 수집해 CSS 파일을 한 번 만들고 끝난다. 서버는 `className="glass"`를 문자열로 HTML에 박아 보낼 뿐 그게 무슨 색인지 모르고, 브라우저가 CSS 파일을 보고 판단한다. → **Tailwind 클래스명을 `` `bg-${color}-500` ``처럼 조립하면 안 되는 이유**: 빌드 때 소스에 완성된 이름이 없어 수집되지 않고, CSS에 그 클래스가 없어서 런타임에 아무 일도 일어나지 않는다.

tsc와 Tailwind는 같은 칸(빌드 타임)에 있다 — 둘 다 실행 중에는 존재하지 않는다. [[공부/JS/TypeScript 타입 시스템|타입 소거]]와 같은 이야기.

**이 프로젝트는 모노리스다.** 프론트엔드/백엔드가 별도 저장소로 나뉘지 않고 한 프로젝트에 있다 — `app/api/*/route.ts`가 백엔드, `_pages`·`widgets`가 프론트엔드이며 같은 `npm run dev`로 함께 뜬다. (React는 프레임워크가 아니라 화면 그리는 **라이브러리**고, 프레임워크는 Next.js다.)

### 환경변수 — `NEXT_PUBLIC_`은 허가가 아니라 명령이다

`process`는 **Node의 물건**이다([[공부/JS/JavaScript 런타임|런타임]]). 브라우저엔 `process.env`가 없다. 그래서 브라우저로 갈 코드에 `process.env.X`를 쓰면 Next가 **빌드 타임에 그 자리를 값 문자열로 치환**한다. 읽어오는 게 아니라 **값이 코드에 새겨진다**(인라인).

```js
const key = process.env.NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY;
// 빌드 후 번들에 실제로 들어가는 것
const key = "pk_test_a1b2c3...";
```

아무 변수나 박아넣으면 서버 비밀이 전부 새므로 규칙이 하나 있다 — **`NEXT_PUBLIC_`으로 시작하는 것만 브라우저 번들에 박고, 나머지는 서버에만 남긴다.**

> **`NEXT_PUBLIC_`은 "공개해도 된다"는 표시가 아니라 "브라우저에 박아넣어라"는 명령이다.** 허가가 아니라 지시라서, 붙이면 그게 비밀이든 아니든 시키는 대로 한다.

그래서 `CLERK_SECRET_KEY` 앞에 `NEXT_PUBLIC_`을 붙이면 값이 JS 번들에 문자열로 새겨져 사이트를 여는 **모든 사람**에게 전송된다(개발자 도구 Sources, `view-source`, 크롤러 전부). Clerk secret key는 백엔드 API 키라 아무 유저나 흉내내 토큰을 발급하고 유저 목록을 조회·삭제할 수 있다 — 계정 시스템 전체를 넘겨주는 것과 같다.

**가장 나쁜 건 되돌릴 수 없다는 점이다.** 고쳐서 다시 배포해도 이미 나간 번들은 브라우저 캐시·CDN·아카이브에 남는다. 복구는 **키 폐기 후 재발급**뿐이다.

반대로 publishable key는 이름 그대로 **퍼블리시를 전제로 설계된** 키다. 할 수 있는 일이 제한적이고(로그인 UI 띄우기, 세션 유효성 문의), Clerk 서버가 **요청 도메인**을 확인해 남이 훔쳐 써도 막는다. 애초에 브라우저에 있어야 로그인 창을 그릴 수 있다.

**판단 기준은 하나다 — 이 코드가 유저 컴퓨터로 가는가.** 업비트 때는 CORS·레이트리밋 때문에 route handler 뒤로 숨겼고 여기서는 `NEXT_PUBLIC_`을 안 붙이는 것으로 해결하지만, 묻는 질문은 같다.

### 함수로 부르기 vs HTTP로 부르기 — 부르는 사람이 어디 있는가

같은 로직을 두 가지로 노출한다.

```
src/shared/api/upbit/client.ts    함수     getTickers(['KRW-BTC'])
app/api/upbit/ticker/route.ts     URL      GET /api/upbit/ticker?markets=KRW-BTC
```

`route.ts`는 `client.ts`를 감싼 얇은 껍데기다(쿼리 읽기 → 검증 → shared 함수 호출 → JSON 반환). 구간을 나눠 세면 명확해진다 — **①누가 우리 코드를 부르는가**와 **②우리 코드가 외부 API를 부른다**는 별개이고, ②는 항상 네트워크다.

```
서버 컴포넌트: LiveMarketCard ──함수 호출──▶ getTickers() ──네트워크──▶ 업비트   (네트워크 1회)
브라우저:      브라우저 ──네트워크──▶ route.ts ──함수 호출──▶ getTickers() ──네트워크──▶ 업비트  (2회)
```

**브라우저는 `import { getTickers }`를 할 수 없다 — 그 파일이 유저 컴퓨터에 없기 때문이다.** 서버 컴포넌트 코드는 브라우저로 전송되지 않는다. 그래서 "그 함수 실행해서 결과 줘"라고 부탁하는 수밖에 없고, 그 부탁이 HTTP 요청이며 받는 창구가 route handler다. 인자 전달 방식이 다른 것도 여기서 나온다 — 함수는 배열을 그대로 넘기지만 URL은 문자열만 실을 수 있어 `join(',')` → `split(',')`으로 오간다.

**브라우저가 외부 API를 직접 부르면 안 되는 이유** ① CORS — 브라우저만의 안전장치라 서버끼리는 해당 없음 ② 캐시 공유 — 우리 서버를 거쳐야 `revalidate` 캐시가 전 유저에게 공유된다. 각자 부르면 유저 수만큼 호출되어 레이트리밋에 걸린다.

> 지금 시점에 route handler를 실제로 쓰는 코드는 아직 없다. 홈 연결은 서버 컴포넌트가 `client.ts`를 직접 부른다. route handler는 D1 DoD(curl 검증)와 이후 클라이언트 폴링을 위해 미리 만든 것.

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

### Route Handler — 우리 서버의 API 창구

`page.tsx` 대신 **`route.ts`**를 두면 HTML이 아니라 데이터를 반환하는 API가 된다. 파일 위치가 곧 URL인 건 페이지와 같다.

```
app/api/upbit/ticker/route.ts   →  GET /api/upbit/ticker
```

```ts
export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const markets = searchParams.get('markets');       // string | null
  if (!markets) {
    return NextResponse.json({ error: 'markets is required' }, { status: 400 });
  }
  const tickers = await getTickers(markets.split(','));
  return NextResponse.json(tickers);
}
```

- **함수 이름이 곧 HTTP 메서드**다(`GET`/`POST`, 대문자 필수).
- `new URL(request.url)`로 감싸야 `searchParams`로 쿼리스트링을 꺼낼 수 있다.
- 파라미터가 없으면 외부 API에 갈 것도 없이 **400으로 먼저 막는다** — 입력 검증의 가장 싼 형태.

**왜 필요한가**: 서버 컴포넌트는 `getTickers`를 직접 부르면 된다. 하지만 나중에 브라우저에서 N초마다 시세를 갱신하려면 브라우저가 외부 API를 직접 부르게 되는데, ① CORS에 막히고 ② 레이트리밋이 유저 브라우저마다 따로 걸려 서버 캐시를 공유하지 못한다. 그래서 같은 출처(same-origin)의 창구를 두고 브라우저 → 우리 서버 → (캐시) → 외부 API로 간다.

### Next 16에서 바뀐 것

- `params` / `searchParams`는 **Promise** — `await` 필수 (route handler의 `ctx.params`도).
- `fetch`는 기본 캐시 안 함 — `next: { revalidate: N }` 명시.
- middleware → **`proxy.ts`**로 개명.
- `next lint` 제거 → `eslint` 직접 실행.

## 기록

### 2026-08-28 — Clerk 키 두 개 (D3 블록 1)

맥락: MyCryptoDiary D3 인증 착수. `.env.local`에 `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`와 `CLERK_SECRET_KEY`를 넣으며 **왜 하나만 접두사가 붙는지** 물어서 정리했다(위 「환경변수」 절).

핵심은 접두사의 성격이다 — 나는 "공개해도 안전하다는 표시"로 읽었는데 실제로는 **"브라우저 번들에 박아라"는 명령**이다. 그래서 secret key에 붙이면 Next가 순순히 박아넣고, 그때부터는 배포를 되돌려도 키를 폐기하는 것 말고 복구 방법이 없다.

빌드 타임 인라인이라는 점에서 [[공부/JS/JavaScript 런타임|런타임]]의 "API 키를 서버에만 둔다", Tailwind 클래스명 조립 금지와 **같은 축**의 이야기다 — 셋 다 "빌드 때 결정되어 번들에 새겨지는 것"이 무엇인지의 문제다.

번들에 실제로 박히는지는 D3 블록 4(Clerk 클라이언트 컴포넌트 투입)에서 빌드 산출물을 직접 뒤져 확인하기로 했다. 지금은 Clerk 코드가 없어 어느 키도 번들에 없다 — 안 나오는 게 당연해서 증명이 되지 않는다.

### 2026-08-18 — 업비트 연동 (D1)

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]])에서 `shared/api/upbit` + `app/api/upbit/ticker/route.ts`를 만들며.
- 배운 것: 위 핵심 정리 전체. 특히 **"React가 Next에게 HTML을 준다"고 거꾸로 이해하고 있었던 것**을 바로잡았고, JSX가 HTML 문자열이 아니라 JS 객체를 만든다는 것을 처음 알았다.
- 검증: `curl "localhost:3000/api/upbit/ticker?markets=KRW-BTC"` → 실시세 JSON, 파라미터 없이 호출 → 400. D1 DoD 항목.
- 근거: 커밋 `c761e7a`(client), `2551b96`(route handler)

## 참고 자료

- [Next.js — Server Components](https://nextjs.org/docs/app/getting-started/server-and-client-components) — 서버/클라이언트 컴포넌트 경계 (2026-08-18 확인)
- [Next.js — Route Handlers](https://nextjs.org/docs/app/api-reference/file-conventions/route) — `route.ts`의 메서드 export 규약 (2026-08-18 확인)
- [Next.js — fetch 옵션](https://nextjs.org/docs/app/api-reference/functions/fetch) — `next.revalidate` (2026-08-18 확인)
