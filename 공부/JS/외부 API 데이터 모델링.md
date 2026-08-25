---
type: study
area: 개발 방법
audience: me
status: active
created: 2026-08-20
updated: 2026-08-20
aliases: [DTO, 응답 타입 두 겹, 데이터 모델링]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# 외부 API 데이터 모델링

남의 API 응답을 화면까지 가져올 때 **어디까지 그들의 모양을 유지하고, 어디서 우리 모양으로 바꾸고, 어디서 문자열로 포맷할지**를 정하는 문제. 경계를 흐리면 외부 API가 바뀔 때 UI 전체가 흔들리고, 포맷된 문자열을 들고 다니면 그 문자열을 다시 뜯어보는 코드가 생긴다.

## 핵심 정리

### 1. 응답 타입은 두 겹으로

```
UpbitTicker   업비트가 준 모양 그대로 (snake_case, 필요한 필드만)
     ↓ toCoin(ticker, koreanName)   ← 변환 함수. 여기가 유일한 경계
Coin          우리 모양 (camelCase, 도메인 이름)
```

- **1겹에서 이름을 바꾸면 안 된다.** TS엔 Swift `CodingKeys`가 없어서 `JSON.parse` 결과는 서버가 준 키 그대로다. `trade_price`를 `tradePrice`로 적어두면 값이 조용히 `undefined`가 된다 → [[공부/JS/TypeScript 타입 시스템|타입은 런타임에 아무것도 검사하지 않는다]].
- 1겹 파일에 snake_case가 보이는 건 오히려 **"이건 외부 응답 그대로"라는 신호**다.
- 얻는 것: 외부가 필드명을 바꾸거나 다른 제공자를 붙일 때 **변환 함수 한 곳만** 고친다. UI는 외부 스키마를 모른다.
- **2겹은 소비자가 생길 때 만든다.** 소비자가 없는데 미리 만들면 조기 추상화다.

### 2. 데이터는 숫자로 들고 다니고, 포맷은 그리는 순간에만

기존 목 데이터가 `price: '92,481,000원'`, `change: '+2.43%'`처럼 **이미 포맷된 문자열**이었더니 등락 색 판정이 이렇게 돼 있었다:

```tsx
coin.change.startsWith('-') ? 'text-fall' : 'text-rise'   // 문자열 첫 글자를 본다
```

숫자로 들고 있으면 `coin.changeRate < 0`으로 끝난다. **포맷된 값을 저장하면 그 문자열을 다시 파싱하는 코드가 반드시 생긴다.**

- 변환(예: 비율 → 퍼센트의 ×100)은 **한 군데에서만**. 여기선 `formatPercent` 안.
- 포맷 함수는 순수 함수라 `lib`/`model` 세그먼트에 두고 UI 밖에 있어야 테스트가 가능하다.

### 3. 파생할 수 있는 건 저장하지 않는다

`'KRW-BTC'` 하나에서 심볼(`BTC`)과 쌍 표기(`BTC/KRW`)가 나온다. 필드로 저장하지 말고 함수로 뽑는다. 단 이 함수는 **도메인 지식**(마켓 코드 형식)이라 범용 `shared/lib`가 아니라 `entities/coin`에 둔다.

### 4. 메타데이터를 하드코딩하면 조용히 틀려진다

코인 한글 이름·심볼·배지색을 상수 배열로 갖고 있다가 업비트 `/market/all`에서 받아오게 바꿨다. 그 직후 화면에 **"리플"이 아니라 "엑스알피(리플)"**로 떴다 — 업비트가 공식 명칭을 바꾼 것이고, 하드코딩했다면 틀린 이름이 계속 떠 있었을 것이다. 상장폐지·신규상장도 마찬가지다.

**남길 것은 "무엇을 보여줄지"라는 제품 결정뿐이다.** 그건 외부 API가 알 수 없다.

```ts
export const MARKET_BOARD_MARKETS = ['KRW-BTC', 'KRW-ETH', 'KRW-SOL', 'KRW-XRP', 'KRW-DOGE'];
```

### 5. 두 소스를 합칠 때 — 열쇠와 순서

시세(ticker)와 이름(markets)은 다른 엔드포인트라 **공통 열쇠**(`market`)로 이어붙인다.

```ts
const meta = marketList.find((m) => m.market === ticker.market);
toCoin(ticker, meta?.korean_name ?? ticker.market);   // 못 찾아도 화면이 안 깨지게
```

**바깥 루프는 우리 목록**이어야 한다. 외부 API가 요청한 순서대로 준다는 보장이 없기 때문에, 응답 배열을 그대로 돌면 화면 순서가 흔들린다.

```tsx
{MARKET_BOARD_MARKETS.map((market) => {
  const coin = coins.find((c) => c.market === market);
  ...
})}
```

### 6. 필드 이름은 값의 **의미**와 맞아야 한다

`acc_trade_price_24h`(24시간 누적 **거래대금**, 원)를 받아 `Coin.tradeVolume24h`로 이름 붙였다가 리뷰에서 잡혔다. **Volume은 거래량(수량)의 이름**이고, 업비트에는 `acc_trade_volume_24h`라는 진짜 수량 필드가 따로 있다. 값은 맞는데 이름이 **반대 개념**이라, 나중에 수량이 필요해지는 순간 충돌한다.

| 업비트 필드 | 뜻 | 우리 이름 |
|---|---|---|
| `acc_trade_price_24h` | 거래**대금** (원) | `tradeValue24h` |
| `acc_trade_volume_24h` | 거래**량** (코인 수량) | `tradeVolume24h` |

1겹→2겹 변환은 이름을 새로 짓는 자리이므로 **원본 필드가 실제로 무엇인지 확인하고 짓는다**. 짐작으로 지으면 도메인 용어가 오염되고, 타입은 둘 다 `number`라 컴파일러가 못 잡는다. (Fowler의 *Mysterious Name*)

### 7. 외부 API는 실패한다 — 실패 경계를 어디에 둘지

`if (!res.ok) throw`만 두고 아무도 잡지 않으면, 외부 API가 429 하나를 주는 순간 **그 값을 쓰는 화면 전체가 500**이 된다. 서버 컴포넌트는 컴포넌트 함수 안에서 `await` 하기 때문에 예외가 렌더 트리를 타고 위로 올라가기 때문이다.

- 실패를 **위젯 단위로 가둬야** 카드 하나가 못 그려져도 나머지 화면이 산다.
- 상태 코드 검사는 여전히 필수다 — [[공부/JS/TypeScript 타입 시스템|fetch는 404·500에 reject 하지 않는다]].
- 에러 메시지는 **어느 API가 왜 터졌는지** 알 수 있게 규칙을 통일한다. 접두사가 함수마다 제각각이면 로그에서 추적이 안 된다.

### 8. 표시용 숫자와 계산용 숫자의 경계

시세를 화면에 **보여주기만** 할 땐 `number`(부동소수점)로 충분하다. 하지만 그 값이 **잔고를 더하고 빼는 계산에 들어가는 순간** 오차가 쌓여 돈이 안 맞는다. MyCryptoDiary는 원화를 `bigint`(원 단위 정수)로 두기로 했으므로, "표시용 float"가 체결가로 넘어가지 않게 **경계를 코드에 명시**해야 한다. 지금은 허용이지만 D4(매수)에서 위반이 된다.

### 9. 캐시 주기 = 데이터가 변하는 속도

ticker 5초 · 일봉 60초 · 마켓 목록 3600초. 임의의 숫자가 아니라 각 리소스가 실제로 바뀌는 주기에서 나온다. → [[공부/JS/Next.js 서버와 캐싱|Next.js 서버와 캐싱]]

## 기록

### 2026-08-20 — 홈을 업비트 실시세로 (D1 마무리)

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]]) — `market-board`·`watchlist`를 목 데이터에서 실데이터로.
- 배운 것: 위 핵심 정리 전체.
- **눈으로 안 잡히는 버그를 처음 만남**: `formatPercent`에서 부호를 `rate >= 0 ? '+' : '-'`로 붙였더니 음수가 `--2.43%`가 됐다. `toFixed`가 음수 부호를 이미 포함하기 때문. **상승장에서만 보면 영원히 못 잡는다.** 계획서가 D5에 유닛 테스트를 넣는 이유를 미리 본 셈. 같은 부류로 `formatKRW(amount) + '원'`처럼 이미 포함된 것을 또 붙이는 실수도 했다 → `50,000,000원원`.
- 검증: 화면 5종을 업비트 원본과 대조해 가격·등락률·거래대금 일치 확인. 미세한 차이는 `revalidate: 5` 캐시가 실제로 동작한다는 증거였다.
- 근거: 커밋 `76aa5bd`(entities/coin), `8c27df9`(포맷), `c900afd`(market-board), `1fa0732`(watchlist)

### 2026-08-20 (2) — 코드 리뷰에서 나온 것

- 맥락: D1 PR #4에 Standards/Spec 두 축 리뷰를 돌린 결과 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]])
- 배운 것: 위 6·7·8번. 특히 **`tradeVolume24h` 오명명은 두 축 중 Spec 리뷰만 잡았다** — 코드 규칙(Standards)으로는 멀쩡하고 **원본 API 문서와 대조해야만** 보이는 종류라, 리뷰를 두 축으로 나눈 것이 실제로 값을 했다.
- 두 축이 함께 잡은 것(신호가 강한 것): 외부 API 실패 시 화면 전체가 죽는 구조, 같은 상황(`getCoins` 결과 비어 있음)에 대한 가드가 한 위젯엔 있고 다른 위젯엔 없는 불일치, 입력 무검증(`count=abc` → `NaN`이 외부로 전송).
- 근거: PR #4 리뷰, `src/entities/coin/model/types.ts`, `src/shared/api/upbit/client.ts`

## 참고 자료

- [업비트 — 시세 Ticker 조회](https://docs.upbit.com/kr/reference/ticker%ED%98%84%EC%9E%AC%EA%B0%80-%EC%A0%95%EB%B3%B4) — `trade_price`·`signed_change_rate`·`acc_trade_price_24h` 필드 정의 (2026-08-20 확인)
- [MDN — Array.prototype.find](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find) — 두 소스를 열쇠로 합칠 때 (2026-08-20 확인)
