---
type: study
area: JS
audience: me
status: active
created: 2026-08-18
updated: 2026-09-02
aliases: [JS 문법, 객체 리터럴, 구조 분해, destructuring]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# JavaScript 기초 문법

Swift를 쓰다 JS로 올 때 **이미 아는 것이 절반 이상**이다(제어문·함수·async/await·클래스). 실제로 걸리는 건 몇 개뿐이고, 그중 압도적으로 자주 쓰이는 둘이 **객체 리터럴**과 **구조 분해**다. 문법책을 앞에서부터 읽는 대신 "Swift에 없는 것"만 골라 익히고 프로젝트로 돌아온다.

## 핵심 정리

### 객체 리터럴 `{ }` — JS의 절반

`{ }`는 그 자리에서 객체를 만드는 문법이다. 클래스·struct 선언이 필요 없다.

```js
const obj = { error: 'markets is required', status: 400 };
obj.error       // 점 표기 — 이름이 유효한 식별자일 때
obj['error']    // 대괄호 표기 — 키에 공백·하이픈이 있거나 키가 런타임에 정해질 때
const key = 'error';
obj[key]        // 변수로 접근하려면 대괄호여야 한다
```

- Swift에 대응물이 없다. 가장 가까운 건 딕셔너리 `["error": "..."]`인데, **JS 객체는 딕셔너리이자 struct**다 — 둘이 하나로 통합돼 있다. 그래서 JSON과 모양이 똑같고 `JSON.stringify(obj)`가 그대로 `{"error":"..."}`가 된다.
- 속성은 **언제든 추가·삭제**할 수 있다. `obj.color = 'black'`, `delete obj.a`.
- **객체는 참조 타입.** 내용이 같아도 다른 객체면 `===`가 `false`다. 같은 참조일 때만 `true`.
  ```js
  ({ name: 'apple' }) === ({ name: 'apple' })   // false
  ```
- 함수에 옵션을 넘길 때 Swift처럼 파라미터를 여러 개 두는 대신 **객체 하나로 묶어 넘기는 게 JS 관례**다: `fetch(url, { next: { revalidate: 5 } })`.

### 구조 분해(destructuring) — 객체에서 필요한 것만 꺼내기

```js
const url = new URL(request.url);
const searchParams = url.searchParams;    // 이 두 줄이
const { searchParams } = new URL(request.url);   // 이 한 줄과 같다
```

"객체에서 그 이름의 속성을 꺼내 같은 이름의 변수로 만든다". 자주 쓰는 변형:

```js
const { a, b } = obj;                  // 여러 개
const { p: foo } = { p: 42 };          // 이름 바꿔 받기 → foo = 42
const { a = 10 } = {};                 // 기본값 (속성이 undefined일 때만 적용, null이면 적용 안 됨)
const { a, ...others } = { a:1, b:2 }; // 나머지 모으기 → others = { b: 2 }
function userId({ id }) { return id; } // 함수 파라미터에서 바로 분해
const [first, , third] = [1, 2, 3];    // 배열도 됨 (자리로 매칭, 건너뛰기 가능)
```

`import { getTickers } from '...'`의 중괄호도 같은 발상(모듈에서 이름 골라 꺼내기)이지만 문법상 별개다.

### truthy / falsy

`if (!markets)`가 성립하는 근거. **falsy는 6개뿐**이고 나머지는 전부 truthy다.

```
false · 0 · "" · null · undefined · NaN
```

`!`는 값을 boolean으로 바꿔 뒤집는다. `searchParams.get()`의 반환은 `string | null`이지 boolean이 아니다 — `!`가 변환하는 것이다. Swift의 `x == nil`보다 넓게 잡아서, 쿼리스트링이 `?markets=`처럼 값 없이 온 경우(빈 문자열)까지 한 번에 막아준다.

### `bigint` 나눗셈은 항상 내림 — 올림은 직접 만든다

```js
7n / 2n === 3n   // 3.5가 아니라 3. 나머지는 그냥 버려진다
```

`Math.ceil`은 못 쓴다. `number`용이라 `bigint`에 안 먹고, 변환해서 쓰면 정밀도가 깨져 애초에 `bigint`를 쓴 이유가 사라진다. 규칙은 **나머지가 0이면 그대로, 0이 아니면 +1**이고 쓰는 방법이 둘이다.

```js
// A. 나머지를 직접 본다 — 읽으면 규칙이 그대로 보인다
if (a % b === 0n) return a / b;
return a / b + 1n;

// B. 분자를 미리 밀어 올린다 — 한 줄이지만 왜 맞는지는 증명해야 안다
return (a + b - 1n) / b;
```

**B에서 `b - 1`인 이유**: "딱 모자란 만큼만 채워주는 값"이다. 나머지가 **0**이면 `b-1`을 더해도 다음 칸(`b`)에 **1 모자라** 몫이 안 변하고, 나머지가 **1이라도** 있으면 다음 칸에 **도달해** 몫이 1 오른다. `b`를 더하면 항상 넘치고(`(a+b)/b = a/b + 1`), `b-2`면 나머지 1일 때 모자란다. `b-1`만 정확히 경계에 걸친다.

`b = 3`으로 손으로 확인:

| a | a + 2 | ÷3 | 나머지 |
|---|---|---|---|
| 6 | 8 | 2 | 0 → 안 오름 |
| 7 | 9 | 3 | 1 → 오름 |
| 9 | 11 | 3 | 0 → 안 오름 |

**어느 쪽을 쓰나**: 돈 계산처럼 읽는 사람이 정확성을 눈으로 검증해야 하는 코드는 **A**. B는 짧지만 리뷰어가 왜 맞는지 증명해야 하고, 실수하면(`b+1`을 쓰거나 `b`를 빠뜨리거나) 타입 검사에 안 걸리고 조용히 틀린다.

### 그 밖에 Swift와 다른 것

| 개념 | 요점 |
|---|---|
| `===` vs `==` | **항상 `===`.** `==`는 타입을 멋대로 변환해 `0 == ''`가 `true`가 된다 |
| `new` | 클래스 인스턴스를 만들 땐 반드시 `new URL(...)`. Swift처럼 생략 불가. 반면 `NextResponse.json(...)`처럼 점 찍고 부르는 건 static 메서드라 `new`가 없다 |
| `extends` | **상속**이다. Swift `class A: B`에 해당. Swift의 `extension`(기존 타입에 메서드 덧붙이기)과는 전혀 다르고, JS엔 `extension`에 해당하는 문법이 없다 |
| `?.` `??` | Swift와 사실상 동일 |
| 배열 메서드 | `map` `filter` `find` `reduce` — Swift와 이름·개념이 거의 같다. `join(',')`은 Swift `joined(separator:)`, `split(',')`은 그 반대 |
| `this`·프로토타입 | React 함수형 컴포넌트만 쓰면 거의 안 만난다. **지금은 건너뛴다** |

## 기록

### 2026-09-02 — 수수료 올림 계산 (MyCryptoDiary D4)

맥락: 매수 수수료 0.05%를 `bigint`로 계산하는 함수(`entities/order/model/fee.ts`). 반올림 규칙을 "유저가 내는 건 올림"으로 정해둬서 올림이 필요했다.

처음엔 B(한 줄) 방식으로 갔는데 **세 번 연속 틀렸다** — `(a + b/2) / b`(반올림), `(a + b + 1) / b`(항상 +1), `(a + 1) / b`(`b`가 빠져 올림이 아예 안 일어남). 셋 다 `tsc`를 통과했고 **숫자를 넣어봐야만 드러났다.**

그래서 A(`if` + `%`)로 바꿨다. 읽으면 규칙이 그대로 보이고 결과는 동일하다. 경계값 10개(1, 1999, 2000, 2001, 3999, 4000 …)를 손으로 계산해 검산했고, 이 기댓값이 D5 테스트가 된다.

배운 것은 트릭 자체보다 **"타입이 통과해도 숫자는 틀릴 수 있다"**는 쪽이다. 금액 계산은 타입 검사가 아니라 손계산이 검증한다.

### 2026-08-18 — 업비트 route handler를 쓰다가

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1 업비트 연동 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]]). `app/api/upbit/ticker/route.ts`를 쓰면서 문법이 막혔다.
- 배운 것:
  - `const { searchParams } = new URL(request.url)` — **이미 쓰고 있었는데 구조 분해인 줄 몰랐다.** 뜻을 알고 나니 `import { }`와 옵션 객체 전달이 한 덩어리로 이해됨.
  - `NextResponse.json({ error: 'markets is required' }, { status: 400 })`의 `{ }`가 객체 리터럴. 인자 두 개가 각각 "본문"과 "옵션"이고, 옵션을 객체로 넘기는 게 JS 관례라는 것.
  - `searchParams.get()`은 boolean이 아니라 `string | null`. `if (!markets)`의 `!`가 truthy/falsy 변환을 하는 것이었다.
  - `.d.ts`를 열어 `export declare class NextResponse extends Response`를 읽고 `extends`를 Swift `extension`으로 오해 → 상속임을 정정.
- 실제로 막힌 원인(문법과 무관): 코드가 `searchParams.get('market')`인데 curl은 `?markets=`로 보내 이름이 어긋나 400이 났다. **쿼리 파라미터 이름은 코드와 호출부가 정확히 같아야 한다.**
- 근거: 커밋 `2551b96`, `app/api/upbit/ticker/route.ts`

## 참고 자료

- [MDN — Working with objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects) — 객체 리터럴, 점/대괄호 접근, 속성 추가·삭제, 참조 타입 (2026-08-18 확인)
- [MDN — Destructuring assignment](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) — 이름 바꾸기·기본값·중첩·rest·함수 파라미터 분해 (2026-08-18 확인)
- [MDN — JavaScript 모듈 가이드](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) — [[학습/공부/JS/JavaScript 모듈 시스템|모듈 시스템]] 쪽과 이어짐 (2026-08-16 확인)
