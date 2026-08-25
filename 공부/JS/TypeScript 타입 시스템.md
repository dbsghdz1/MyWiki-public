---
type: study
area: 언어·프레임워크
audience: me
status: active
created: 2026-08-18
updated: 2026-08-18
aliases: [타입 소거, type erasure, 컴파일 타임 런타임]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# TypeScript 타입 시스템

TypeScript의 타입은 **실행 전에 실수를 잡는 도구이지, 실행 중에 지켜주는 보호막이 아니다.** 컴파일하면 타입은 전부 지워지고 순수 JavaScript만 남는다. Swift는 타입이 런타임까지 살아남지만 TS는 그렇지 않다는 것이 모든 차이의 출발점이다.

## 핵심 정리

### 두 시점

| | 언제 | 누가 | 이 프로젝트에선 |
|---|---|---|---|
| **컴파일 타임** | 코드를 실행 가능한 형태로 바꿀 때 | `tsc` | `npx tsc --noEmit`, `npm run build` |
| **런타임** | 프로그램이 실제로 돌 때 | node / 브라우저 | `npm run dev`로 띄운 서버, 유저가 페이지 열 때 |

### 타입은 컴파일하면 사라진다 (type erasure)

```ts
// demo.ts — 내가 쓴 것
type Coin = { market: string; price: number };
function printCoin(coin: Coin): string { return `${coin.market}: ${coin.price}원`; }
const btc: Coin = { market: 'KRW-BTC', price: 89300000 };
```
```js
// demo.js — 실제로 node가 실행하는 것
function printCoin(coin) { return `${coin.market}: ${coin.price}원`; }
const btc = { market: 'KRW-BTC', price: 89300000 };
```

`type Coin` 블록이 통째로 없어지고 `: Coin`, `: string` 표기도 전부 지워진다. node와 브라우저는 JavaScript만 실행할 수 있고, TypeScript를 이해하는 건 `tsc`뿐이다. 즉 **타입은 나와 tsc 사이의 약속**이지 실행되는 코드의 일부가 아니다.

### 그래서 런타임엔 아무도 막지 않는다

```ts
const fromServer: unknown = { error: 'too many requests' };  // 서버가 준 JSON이라 치자
const coin = fromServer as Coin;   // "이건 Coin이야" 우기기 → tsc 에러 없음
coin.price       // undefined
coin.price * 2   // NaN   → 화면에 "NaN원"이 뜬다
```

- **TS의 `as`는 Swift의 `as?`가 아니라 `as!`에 가깝다.** 검사 없이 우기는 것이고, 틀려도 크래시조차 없이 조용히 `undefined`가 흘러간다. 그런 의미에선 더 위험하다.
- Swift는 타입 메타데이터가 런타임에 남아 `is`/`as?`/리플렉션이 되지만 TS엔 그런 게 없다.

### 결론 — 외부에서 들어오는 것은 손으로 검사한다

API 응답·유저 입력처럼 **경계를 넘어 들어오는 데이터**는 타입 표기를 아무리 잘해도 보장되지 않는다.

```ts
if (!res.ok) throw new Error(`Upbit ticker ${res.status}`);  // 손으로 검사
return res.json();   // Promise<any>. TS는 우리가 적은 반환 타입을 그냥 믿는다
```

`fetch`는 404·500을 받아도 **에러를 던지지 않는다**("서버가 답을 줬으니 성공"). 네트워크가 끊겼을 때만 throw 한다. 그래서 `res.ok`(상태 200~299) 검사가 반드시 필요하고, 없으면 429 에러 JSON이 `UpbitTicker[]`인 척 UI까지 흘러간다. (더 강하게 막으려면 zod 같은 런타임 검증 라이브러리로 스키마를 검사한다 — 지금 범위 밖.)

### `import type`

```ts
import type { UpbitTicker } from './types';
```

타입만 가져올 때 쓴다. "컴파일 때만 필요하다"고 명시하는 것이라 **빌드 결과에서 이 import 줄 자체가 지워진다**. 값(함수·상수)을 가져올 땐 그냥 `import`.

## 기록

### 2026-08-18 — 업비트 client를 쓰다가

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1, `shared/api/upbit/client.ts`에서 `Promise<UpbitTicker[]>`를 반환하며 "타입 지정이 런타임/컴파일에 무슨 차이가 있는지" 막힘 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]])
- 배운 것: 위 핵심 정리 전체. 특히 `tsc`로 실제 컴파일해 `.ts` → `.js`를 나란히 놓고 타입이 사라지는 걸 눈으로 확인한 것이 결정적이었다.
- 파생: 외부 API 응답 타입을 **두 겹으로 두는 이유**도 여기서 나온다. 1겹 `UpbitTicker`는 업비트가 준 모양 그대로(snake_case) — 여기서 camelCase로 바꿔 적으면 `JSON.parse` 결과와 키가 안 맞아 값이 `undefined`가 된다. Swift `CodingKeys`가 해주던 매핑을 TS에선 **변환 함수를 손으로 써서** 2겹(camelCase, 우리 모델)으로 옮긴다. 타입 표기는 매핑을 해주지 않는다.
- 근거: 커밋 `c761e7a`, `src/shared/api/upbit/{types,client}.ts`

## 참고 자료

- [TypeScript Handbook — Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) — 타입 표기의 기본, `any`/`unknown` (2026-08-18 확인)
- [TypeScript Handbook — Type Assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions) — `as`가 런타임 검사를 하지 않는다는 원문 (2026-08-18 확인)
- [MDN — fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Window/fetch) — HTTP 에러 상태에서 reject 하지 않는다는 동작 (2026-08-18 확인)
