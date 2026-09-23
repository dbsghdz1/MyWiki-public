---
type: study
area: JS
audience: me
status: active
created: 2026-09-23
updated: 2026-09-23
aliases: [useEffect, StrictMode, cleanup]
projects:
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# useEffect와 StrictMode

`useEffect`는 화면을 그린 뒤에 할 일이고, `return`하는 함수는 「실행할 일」이 아니라 「치울 일」이다. 개발 모드의 StrictMode가 그걸 한 번 더 돌려 검사한다.

## 핵심 정리

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

### 2026-09-09 — cleanup 자리에 fetch를 넣어 「개발에서만 되는 버그」를 만들었다 (실습 2)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[프로젝트/개인/약국맵/실습 2 — useEffect와 useQuery 2026-09-09|실습 세션 2]]. 버튼 없이 자동 요청으로 바꾸며
- 배운 것:
  - `useEffect(() => { return () => { fetch(...) } }, [])` — **fetch가 cleanup에 들어갔는데 화면엔 데이터가 떴다.** StrictMode의 `setup → cleanup → setup` 덕에 cleanup이 한 번 불렸기 때문. **배포하면 화면이 영영 빈다**
  - `[]`를 빼면 2줄이 됐다. 3줄로 안 간 건 **두 번째 응답이 첫 번째와 똑같은 문자열**이라 React가 재렌더를 건너뛴 것뿐 — 응답에 시각이 섞였으면 무한 루프였다
  - `queryFn: fetch(url).then(...)`으로 또 같은 실수를 했다. **`onClick={fn()}`과 같은 모양** — "필요할 때 부르겠다"는 자리엔 언제나 함수를 넘긴다
- 근거: `pharmacy-map` 커밋 `238d87f`

- 이 노트는 2026-09-23 [[학습/공부/JS/React/React 컴포넌트와 JSX|React 컴포넌트와 JSX]]가 150줄을 넘어 useEffect·StrictMode 절과 09-09 기록을 떼어 만들었다.
