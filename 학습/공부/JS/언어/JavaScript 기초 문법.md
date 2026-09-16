---
type: study
area: JS
audience: me
status: active
created: 2026-08-18
updated: 2026-09-16
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

### 고차함수 — 그리고 "짧은 코드"가 언제 읽기 쉬운가

**고차함수는 함수를 인자로 받거나 함수를 돌려주는 함수다.**

```js
[1,2,3].map(x => x * 2)   // map은 고차함수 — 함수를 받는다
"1.5".split('.')          // 문자열을 받는다 → 그냥 메서드
"5".padEnd(8, '0')        // 숫자·문자열을 받는다 → 그냥 메서드
```

`map`·`filter`·`reduce`·`sort`·`forEach`가 고차함수고, `split`·`padEnd`·`slice`는 아니다. 구분이 중요한 이유는 **가독성 문제의 성격이 다르기 때문**이다 — 고차함수는 콜백을 읽어야 하지만 일반 메서드는 읽을 콜백이 없다.

**판단 기준: 무엇을(what) 말하는가, 어떻게(how)를 늘어놓는가.**

```js
// how — 절차를 나열한다. 맞는지 알려면 머릿속으로 실행해야 한다
while (count < scale) { s += '0'; count += 1; }

// what — 의도가 이름에 있다
s.padEnd(scale, '0')
```

짧아서 좋은 게 아니라 **부품이 적어서** 좋다. 위 `while` 버전에는 실제로 버그가 둘 있었다(무한 루프 — `count`를 안 늘림 / `count` 시작값을 0으로 둬서 이미 들어온 소수부 자릿수를 무시). `padEnd`에는 그 버그가 들어갈 자리가 없다.

**반대로 체인이 길어지면 읽기 나빠진다.**

```js
data.filter(x => x.a).map(x => x.b).reduce((s, x) => s + x.c, 0)
```

세 단계를 한 줄에 밀어넣으면 각 단계가 무엇을 만드는지 안 보인다. 중간에 이름을 붙여 끊는 게 낫다. `reduce`에 복잡한 누적기가 들어갈 때도 `for` 루프가 더 명확할 수 있다 — **억지로 함수형으로 쓰는 것이 손해인 경우가 있다.**

**남는 함정**: 짧은 코드는 그 어휘를 아는 사람에게만 읽기 쉽다. 다만 `split`·`padEnd`·`map` 정도는 JS에서 모두가 아는 어휘라 익혀두는 쪽이 이득이다.

### 그 밖에 Swift와 다른 것

| 개념 | 요점 |
|---|---|
| `===` vs `==` | **항상 `===`.** `==`는 타입을 멋대로 변환해 `0 == ''`가 `true`가 된다 |
| `new` | 클래스 인스턴스를 만들 땐 반드시 `new URL(...)`. Swift처럼 생략 불가. 반면 `NextResponse.json(...)`처럼 점 찍고 부르는 건 static 메서드라 `new`가 없다 |
| `extends` | **상속**이다. Swift `class A: B`에 해당. Swift의 `extension`(기존 타입에 메서드 덧붙이기)과는 전혀 다르고, JS엔 `extension`에 해당하는 문법이 없다 |
| `?.` `??` | Swift와 사실상 동일 |
| 배열 메서드 | `map` `filter` `find` `reduce` — Swift와 이름·개념이 거의 같다. `join(',')`은 Swift `joined(separator:)`, `split(',')`은 그 반대 |
| `this`·프로토타입 | React 함수형 컴포넌트만 쓰면 거의 안 만난다. **지금은 건너뛴다** |

`bigint` 나눗셈·`number`와의 경계는 [[학습/공부/JS/언어/bigint와 정수 연산|bigint와 정수 연산]]으로 분리했다 (2026-09-16).

## 기록

### 2026-09-09 — `SyntaxError`는 원인보다 뒤에서 터진다

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[프로젝트/개인/약국맵/실습 3-A — Fastify 프록시 2026-09-09|실습 3-A]]. `async () = {`처럼 화살표의 `>`를 빠뜨렸다
- 배운 것:
  - 에러는 **6번 줄(`return { ok: true };`)**을 가리켰지만 범인은 **5번 줄**이었다. 파서가 `() = {`를 "대입인가 보다" 하고 넘어갔다가 `return`을 만나서야 멈춘 것
  - **`SyntaxError`가 뜨면 가리킨 줄과 그 위 한두 줄을 같이 본다.** 런타임 에러(`TypeError` 등)는 반대로 가리킨 자리가 대체로 범인이다
  - 에러가 잡히는 층이 다르다: **읽다가**(`SyntaxError` — 실행조차 안 됨) / **실행 중**(`ERR_ASSERTION` 같은 값 위반) / **원격 서버가**(잘못된 값을 보냈을 때). 어느 층에서 났는지가 어디를 봐야 하는지를 정한다
- 근거: `pharmacy-map` `9cad2bd` 작업 중 재현

### 2026-09-04 — 소수 문자열을 정수로 (MyCryptoDiary D4 블록 5a)

맥락: `"0.001"` → `100000n` 변환 함수(`shared/lib/toScaleBigInt.ts`). `parseFloat`을 쓰면 안 되는 이유는 [[학습/공부/CS/컴퓨터의 수 표현|수 표현]] 참조.

`for` + `flag` + `count`로 30줄을 쓴 뒤 `split`·`padEnd`로 **6줄**로 줄였다. 그 과정에서 얻은 것:

- **`split('.')`은 배열을 돌려준다.** `value.split('.')[0]`처럼 바로 `[0]`을 붙이면 그 자리에서 소수부가 버려지고, 이후 `[0]`·`[1]`은 **문자열의 글자**를 꺼내게 된다(`"1"[1]` → `undefined`). 배열 인덱싱과 문자열 인덱싱이 문법이 같아서 헷갈린다.
- **점이 없으면 `split` 결과에 1번 칸이 없다.** `?? ''`로 막는다.
- 인자로 받은 `scale`을 쓰지 않고 `8`을 하드코딩하면 **시그니처가 거짓말**을 한다. `tsc`는 잡지 못한다.

그리고 "짧은 코드 = 읽기 어려움"이 아니라는 것(위 절). 다만 **`split`/`padEnd`는 고차함수가 아니다** — 용어를 정정했다.

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
