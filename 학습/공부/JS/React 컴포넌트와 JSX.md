---
type: study
area: JS
audience: me
status: active
created: 2026-09-08
updated: 2026-09-09
projects:
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# React 컴포넌트와 JSX

**JSX는 HTML이 아니라 JS다.** 태그처럼 생겼지만 빌드할 때 함수 호출로 바뀌고, `{ }` 안은 전부 JS 표현식이다. 여기서 오는 오해가 초보 버그의 대부분이다.

## 핵심 정리

### JSX는 함수 호출로 컴파일된다

```jsx
<button onClick={handleClick}>불러오기</button>
```
↓
```js
jsx("button", { onClick: handleClick, children: "불러오기" })
```

**속성이 JS 객체의 키가 된다.** 그래서 HTML 속성 이름 규칙이 아니라 JS 이름 규칙(카멜 표기)을 따른다.

| HTML | JSX | 왜 |
|---|---|---|
| `onclick` | `onClick` | 객체 키라서 카멜 표기 |
| `class` | `className` | `class`는 JS 예약어 |

### 소문자 = HTML 요소, 대문자 = 내 컴포넌트

React가 태그 이름의 **첫 글자 대소문자**로 갈린다. `<button>`은 브라우저에 넘기고, `<App />`은 내가 만든 함수를 찾는다. **컴포넌트 이름을 반드시 대문자로 시작해야 하는 이유**가 이것.

확인법: 개발자도구 **Elements** 탭에 남는 건 실제 DOM뿐이다. `<button>`은 보이지만 `<App>`은 없다 — 컴포넌트는 HTML을 만들어내는 함수일 뿐 결과물만 DOM에 남는다.

### `{ }` 안은 렌더링 시점에 계산된다

이게 `onClick` 함정의 정체다.

```jsx
onClick={console.log("안녕")}   // 렌더될 때 실행됨 → 결과 undefined가 저장 → 클릭해도 무반응
onClick={() => console.log("안녕")}  // 함수라는 "값"이 저장 → 클릭할 때 실행
```

`() =>`는 **지금 실행하지 말고 함수로 싸두라**는 포장지다. React 공식 문서도 같은 말을 한다 — *"Functions passed to event handlers must be passed, not called."*

**위험해지는 경우**: `onClick={setCount(count + 1)}` → 렌더 중 상태 변경 → 재렌더 → 또 실행 → **무한 루프**.

TS는 이걸 정확히 말해준다:
```
Type 'void' is not assignable to type 'MouseEventHandler<HTMLButtonElement>'
```
`is not assignable to` **앞 = 내가 준 것**, **뒤 = 필요한 것**. 이 두 단어만 읽어도 대부분 풀린다.

### `useState` — 이름은 내가 짓는다

```js
const [name, setName] = useState("초기값");
```

`useState`는 `[현재값, 바꾸는 함수]` **두 칸짜리 배열**을 돌려주고 구조 분해로 받는 것뿐이다. `setState`라는 이름은 React가 주는 게 아니다.

**화면을 직접 고치지 않는다**는 게 핵심. `element.textContent = ...` 없이 상태만 바꾸면 React가 컴포넌트 함수를 다시 호출해 화면을 다시 그린다. *"어떻게 고칠지"가 아니라 "지금 상태면 화면이 어떤 모양인지"만 적는 것* = 선언적.

### `return`이 JS 세상과 JSX 세상의 경계다

```jsx
function App() {
  const [name, setName] = useState("");   // ← JS 세상
  return (                                 // ← 경계
    <>
      ...                                  // ← JSX 세상. 여기 쓴 건 전부 "화면에 그릴 것"
    </>
  );
}
```

`return`을 `<>` 안에 넣으면 **`return`이라는 글자가 화면 텍스트가 되고**, 함수는 아무것도 반환하지 않는다. React는 `undefined`를 받으면 **에러 없이 조용히 아무것도 안 그린다.**

> **화면이 통째로 비면 콘솔보다 `return`을 먼저 본다.** 단서가 안 남는 종류의 버그다.

`<>...</>`(프래그먼트)는 아무것도 그리지 않고 여러 요소를 묶기만 하는 빈 태그.

### 문서는 어디서 보나

**비율로는 MDN이 9할이다.** React가 만들어내는 건 진짜 HTML 요소라, "이 요소가 뭐 하는 물건인가"는 전부 MDN이다. React 문서를 볼 일은 **적는 법**(`onClick`, `className`)과 훅뿐이다.

### `useEffect` — 화면을 그린 **뒤에** 할 일

```js
useEffect(() => {
  // setup: 화면이 뜬 뒤 실행할 것
  return () => {
    // cleanup: 치울 것. setup을 되돌리는 코드만 온다
  };
}, [의존성]);
```

**의존성 배열 세 가지** (react.dev 레퍼런스의 표 그대로):

| 쓰는 법 | 언제 실행 |
|---|---|
| `useEffect(fn)` — 배열 없음 | **매 렌더마다** |
| `useEffect(fn, [])` — 빈 배열 | **처음 한 번만** |
| `useEffect(fn, [a, b])` | 처음 + `a`나 `b`가 바뀔 때 |

**`return`하는 함수는 "실행할 일"이 아니라 "치울 일"이다.** 컴포넌트가 사라질 때, 그리고 effect를 다시 실행하기 직전에 불린다. **cleanup만 있고 setup이 없는 코드는 냄새**이고, react.dev Troubleshooting에 `🔴 Avoid` 예시로 실려 있다.

### StrictMode — 개발 모드에서만 두 번 돈다

`<StrictMode>`(**React 기능이다. Vite가 아니다** — `main.tsx`에서 직접 감싼다)가 켜져 있으면 개발 모드에서 **`setup → cleanup → setup`**을 한 번 더 돌린다. cleanup이 setup을 제대로 되돌리는지 검사하는 스트레스 테스트다.

**배포판에서는 한 번.** 그래서 **개발에서 요청이 2줄, 운영에서는 1줄**이 된다. 두 번 도는 걸 막으려 하지 말고 cleanup을 제대로 쓰는 게 정답이다.

## 기록

### 2026-09-09 — `createRoot` vs `hydrateRoot`, 그리고 JSX 없이 `createElement` 직접 쓰기

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[학습/야생학습/약국맵 사다리 3-B·3-C — SSR과 하이드레이션 2026-09-09|사다리 3-B·3-C]]
- **`createRoot`는 «이 자리는 내가 처음부터 그린다»**라서 서버가 보낸 HTML을 버린다. **`hydrateRoot`는 이미 그려진 DOM을 그대로 두고 이벤트·상태만 붙인다** — 「수분 공급」이라는 이름 그대로 마른 뼈대를 적시는 것
  - 관찰 가능한 차이: 데이터가 HTML에 이미 있는데도 `createRoot`면 `/api/pharmacies`를 **또** 부른다(Network 1줄) → `hydrateRoot`면 0줄
  - 이어받으려면 **클라이언트의 첫 렌더 결과가 서버 HTML과 같아야** 한다. 그래서 서버가 데이터를 `window.__PHARMACIES__`로 같이 심는다 — `<li>1번약국</li>`은 **글자**지 데이터가 아니라서 되꺼낼 수 없다
- **JSX 없이 쓰면 이렇게 생겼다**: `<li key={p.hpid}>{p.dutyName}</li>` = `h("li", { key: p.hpid }, p.dutyName)`. 서버 파일이 `.js`라 JSX를 못 써서 컴파일 결과를 손으로 썼다 — 09-08의 *"JSX는 함수 호출로 컴파일된다"*가 실물로 나온 자리
- 근거: `pharmacy-map` `9270889`, `src/main.tsx`·`server/main.js`

### 2026-09-09 — cleanup 자리에 fetch를 넣어 「개발에서만 되는 버그」를 만들었다 (사다리 2)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[학습/야생학습/약국맵 사다리 2 — useEffect와 useQuery 2026-09-09|사다리 세션 2]]. 버튼 없이 자동 요청으로 바꾸며
- 배운 것:
  - `useEffect(() => { return () => { fetch(...) } }, [])` — **fetch가 cleanup에 들어갔는데 화면엔 데이터가 떴다.** StrictMode의 `setup → cleanup → setup` 덕에 cleanup이 한 번 불렸기 때문. **배포하면 화면이 영영 빈다**
  - `[]`를 빼면 2줄이 됐다. 3줄로 안 간 건 **두 번째 응답이 첫 번째와 똑같은 문자열**이라 React가 재렌더를 건너뛴 것뿐 — 응답에 시각이 섞였으면 무한 루프였다
  - `queryFn: fetch(url).then(...)`으로 또 같은 실수를 했다. **`onClick={fn()}`과 같은 모양** — "필요할 때 부르겠다"는 자리엔 언제나 함수를 넘긴다
- 근거: `pharmacy-map` 커밋 `238d87f`


### 2026-09-08 — 버튼 하나를 만들며 막힌 자리 전부 (야생학습 사다리 세션 1)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[학습/야생학습/약국맵 사다리 1 — fetch와 useState 2026-09-08|사다리 세션 1]]. Vite 템플릿에서 시작해 "버튼 누르면 약국 XML을 화면에" 까지
- 배운 것:
  - `onClick={console.log("안녕")}`을 쓰고 **버튼을 누르지 않았는데 콘솔에 찍혔다.** 예측은 "안 찍힌다"였다 — `{ }`가 렌더 시점에 계산된다는 걸 몸으로 본 자리
  - `return`을 `<>` 안에 넣어 화면이 통째로 비었다. **에러가 안 났다**는 게 이 버그의 성질
  - `useState`의 두 이름이 내가 짓는 것임을 몰라 `setState`를 호출했다
  - HTML/React 구분 질문이 반복해서 나왔다 → **소문자/대문자 규칙**과 **Elements 탭 확인법**으로 정리
- 근거: `pharmacy-map` 커밋 `3c81de8`, `src/App.tsx`

## 참고 자료

- [React — Responding to Events](https://react.dev/learn/responding-to-events) — `onClick={handleClick}` vs `handleClick()` 함정을 Pitfall로 명시 (2026-09-08 확인)
- [React — `useState`](https://react.dev/reference/react/useState) — 반환 배열 두 칸, set 함수가 재렌더를 유발, 같은 값이면 재렌더를 건너뜀 (2026-09-08 확인)
