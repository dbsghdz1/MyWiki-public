---
type: study
area: JS
audience: me
status: active
created: 2026-08-18
updated: 2026-09-16
aliases: [Route Handler, route.ts, app/api, src api 세그먼트, same-origin 창구]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# Next.js Route Handler와 내부 API

**같은 로직을 함수(`src/**/api`)와 HTTP(`app/api/**/route.ts`) 두 가지로 노출한다.** 서버 컴포넌트는 함수를 직접 부르고, 브라우저는 그 파일이 없으니 HTTP로 부탁한다 — 그 창구가 Route Handler다. (2026-09-16 [[학습/공부/JS/Next.js/Next.js 서버와 캐싱|Next.js 서버와 캐싱]]에서 분리)

## 핵심 정리

### 함수로 부르기 vs HTTP로 부르기 — `src/**/api`와 `app/api`

같은 로직을 두 가지로 노출한다.

```
src/shared/api/upbit/client.ts    함수     getTickers(['KRW-BTC'])
app/api/upbit/ticker/route.ts     URL      GET /api/upbit/ticker?markets=KRW-BTC
```

`route.ts`는 `client.ts`를 감싼 얇은 껍데기다(쿼리 읽기 → 검증 → shared 함수 호출 → JSON 반환). 구간을 나눠 세면 명확해진다 — **①누가 우리 코드를 부르는가**와 **②우리 코드가 외부 API를 부른다**는 별개이고, ②는 항상 네트워크다.

여기서 두 `api`는 서로 다른 분류 체계의 단어다. `app/api`는 Next 라우팅 경로라 HTTP URL을 만들고, `src/**/api`는 FSD 세그먼트라 외부 데이터와 통신하는 내부 함수를 묶는다. `src`에 `api` 폴더를 만든다고 URL이 생기지 않으며, `app/api`도 폴더명 자체가 특별한 것이 아니라 그 아래 **`route.ts`가 있을 때** HTTP endpoint가 된다.

```
서버 컴포넌트: LiveMarketCard ──함수 호출──▶ getTickers() ──네트워크──▶ 업비트   (네트워크 1회)
브라우저:      브라우저 ──네트워크──▶ route.ts ──함수 호출──▶ getTickers() ──네트워크──▶ 업비트  (2회)
```

**브라우저는 `import { getTickers }`를 할 수 없다 — 그 파일이 유저 컴퓨터에 없기 때문이다.** 서버 컴포넌트 코드는 브라우저로 전송되지 않는다. 그래서 "그 함수 실행해서 결과 줘"라고 부탁하는 수밖에 없고, 그 부탁이 HTTP 요청이며 받는 창구가 route handler다. 인자 전달 방식이 다른 것도 여기서 나온다 — 함수는 배열을 그대로 넘기지만 URL은 문자열만 실을 수 있어 `join(',')` → `split(',')`으로 오간다.

**브라우저가 외부 API를 직접 부르면 안 되는 이유** ① CORS — 브라우저만의 안전장치라 서버끼리는 해당 없음 ② 캐시 공유 — 우리 서버를 거쳐야 `revalidate` 캐시가 전 유저에게 공유된다. 각자 부르면 유저 수만큼 호출되어 레이트리밋에 걸린다.

> 지금 시점에 route handler를 실제로 쓰는 코드는 아직 없다. 홈 연결은 서버 컴포넌트가 `client.ts`를 직접 부른다. route handler는 D1 DoD(curl 검증)와 이후 클라이언트 폴링을 위해 미리 만든 것.

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

## 기록

### 2026-08-29 (2) — `app/api`는 HTTP, `src/**/api`는 내부 통신 코드

맥락: [[프로젝트/개인/MyCryptoDiary/README|CoinPilot]] 구조를 읽다가 같은 `api` 이름이 두 위치에 있어 차이를 확인했다. 구조 전체 정리는 [[학습/공부/JS/React/Feature-Sliced Design|Feature-Sliced Design]]의 「루트 app과 src의 경계」에 연결했다.

- `app/api/upbit/ticker/route.ts`는 브라우저·curl이 `/api/upbit/ticker`로 부르는 HTTP 출입구다. 요청 파라미터·상태 코드·JSON 변환을 책임진다.
- `src/entities/coin/api/getCoins.ts`와 `src/entities/account/api/getOrCreateAccount.ts`는 import해서 쓰는 내부 함수다. 각각 업비트, Clerk·Neon이라는 앱 바깥 데이터와 통신한다는 이유로 FSD의 `api` 세그먼트에 있다.
- 서버 컴포넌트는 `src` 함수를 직접 부를 수 있지만 브라우저는 서버 파일을 직접 실행할 수 없으므로 HTTP route가 필요하다.
- 근거: MyCryptoDiary D1 업비트 경로와 D3 커밋 `ffcb94c`, PR #7.

## 참고 자료

- [Next.js — Route Handlers](https://nextjs.org/docs/app/api-reference/file-conventions/route) — `route.ts`의 메서드 export 규약 (2026-08-18 확인)
