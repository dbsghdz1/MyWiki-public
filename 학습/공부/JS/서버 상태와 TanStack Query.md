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

# 서버 상태와 TanStack Query

서버에서 가져온 데이터는 **화면이 소유한 상태가 아니라 서버에서 빌려 온 것**이다. `useState`로 들고 있으면 그 전제가 깨진다.

## 학습 계획

**미션:** 서버에서 가져온 데이터를 「화면이 소유한 상태」가 아니라 「서버에서 빌려 온 것」으로 다룰 줄 안다. 도구 이름 없이 *왜 그렇게 다뤄야 하는지* 설명할 수 있으면 끝이다.
할 수 있게 되는 것 — ① 같은 화면을 `useEffect`+fetch와 `useQuery` 두 벌로 만들고 **요청 수가 달라지는 이유**를 실측 근거로 설명한다 ② `staleTime`·`queryKey`·`gcTime`이 각각 무엇을 정하는지 **기본값과 함께**, 그리고 **언제 바꿔야 하는지** 말한다 ③ 응답 타입을 직접 정의해 `useQuery` 결과가 **로딩 중에는 `undefined`로 좁혀지는 지점**을 타입으로 설명한다. 범위 밖 — SSR·하이드레이션([[학습/야생학습/README|사다리]] 3·6에서).

- [ ] **Important Defaults** — 기본값 4개(`staleTime` 0 · `gcTime` 5분 · 자동 배경 재요청 트리거 3개). 완료 기준: 탭 전환 시 요청이 가는 이유를 기본값으로 설명
- [ ] **Query Keys** — 배열인 이유, 결정론적 해시, queryFn의 변수가 키에 들어가야 하는 이유. 완료 기준: 키를 잘못 잡으면 무슨 일이 나는지 예로 설명
- [ ] **Practical React Query** — server state의 정의, `staleTime`을 언제 만지는지. 완료 기준: "앱이 소유하지 않는다"를 자기 말로
- [ ] 타입 — `data`가 `T | undefined`인 이유와 좁히는 법

## 용어집

수업은 이 말로만 한다: `server state` · `queryKey` · `queryFn` · `fresh ↔ stale` · `staleTime` · `gcTime` · `background refetch` · `refetchOnWindowFocus`

## 핵심 정리

### 요청 수로 본 차이 (2026-09-09 실측, 같은 화면·같은 API)

| | 첫 로드 | 탭 갔다 돌아오기 |
|---|---|---|
| `useEffect` + `fetch`, deps `[]` | **2줄** (StrictMode가 개발 모드에서 두 번 돌림) | **0줄** |
| `useQuery` 기본값 | **1줄** | **+1줄** (포커스마다) |
| `useQuery` + `staleTime: 60000` | 1줄 | **0줄** |

**세 줄이 이 라이브러리를 요약한다.**

### 왜 `useEffect`는 2줄인데 `useQuery`는 1줄인가 — 중복 제거

같은 `queryKey`로 요청이 **겹치면 하나로 합친다.** StrictMode가 컴포넌트를 두 번 마운트해도 두 번째는 *"이미 그 주소로 가는 중"*인 요청에 올라탄다. 손으로 짠 `useEffect`에서는 막을 수 없던 중복이 **애초에 생기지 않는다.**

### `staleTime`은 「얼마나 오래 신선한 걸로 칠지」다

기본값 **0** = 받자마자 stale. stale인 쿼리는 **① 그 쿼리를 쓰는 화면이 새로 뜰 때 ② 창이 다시 포커스될 때 ③ 네트워크 재연결 때** 배경에서 다시 가져온다. 그래서 기본값 그대로면 **에디터 갔다 브라우저로 올 때마다 요청이 하나씩 는다.**

**단, `staleTime`은 새로고침을 막지 못한다.** 캐시는 메모리에만 있어서 페이지를 다시 열면 통째로 사라진다. `staleTime`이 줄이는 건 **같은 페이지 수명 안에서의 재요청**이다.

### `queryFn`은 **함수**여야 한다

```js
queryFn: fetch(url).then(...)        // 🔴 렌더 시점에 실행되고 Promise가 들어간다
queryFn: () => fetch(url).then(...)  // ✅ 재요청할 때마다 다시 부를 수 있는 함수
```

JSX의 `onClick={fn()}`과 **완전히 같은 실수**다. 라이브러리가 "필요할 때 부르겠다"고 하면 넘길 것은 언제나 함수다.

### 서버 데이터를 `useState`로 들고 있지 않는다

`queryFn`은 값을 **`return`**하고, 화면은 **`query.data`**를 본다. `queryFn` 안에서 `setState`를 부르면 캐시와 화면 상태가 둘로 갈라져 이 라이브러리를 쓰는 의미가 없어진다. 덤으로 `isLoading`이 생긴다 — 손으로 짠 `useEffect` 버전에는 **"아직 안 온 것"과 "빈 것"을 구분할 방법이 아예 없었다.**

## 기록

### 2026-09-09 — useEffect 버전과 나란히 놓고 요청 수를 셌다 (야생학습 사다리 2-B)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[학습/야생학습/약국맵 사다리 2 — useEffect와 useQuery 2026-09-09|사다리 세션 2·2-B]]. 같은 약국 목록 API를 두 벌로 구현해 Network 줄 수를 비교
- 배운 것: 위 「핵심 정리」 전부가 이 세션에서 나왔다. 특히 **`useQuery`가 기본값에서는 `useEffect`보다 요청을 더 많이 보낸다**(포커스마다) — 그게 손해가 아니라 **약국 영업 상태처럼 변하는 데이터에는 맞는 기본값**이고, 빈도는 `staleTime`으로 내가 정한다
- 계측 함정: Network 탭의 **`Preserve log`가 켜져 있으면** 새로고침해도 줄이 안 지워져서 "요청이 계속 쌓인다"로 보인다. **줄 수를 세는 실험에서는 꺼야 한다**
- 근거: `pharmacy-map` 커밋 `238d87f`


## 참고 자료

- [TanStack Query — Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults) — 기본값의 정본. *"by default consider cached data as stale"*, *"inactive queries are garbage collected after 5 minutes"* (2026-09-08 확인)
- [TanStack Query — Query Keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys) — *"query keys act as dependencies for your query functions"*. 객체 키 순서는 무관하지만 배열 항목 순서는 유의미 (2026-09-08 확인)
- [TkDodo — Practical React Query](https://tkdodo.eu/blog/practical-react-query) — 메인테이너 본인 글. *"server state … your app does not own it"*, `staleTime` 기본 0 (2026-09-08 확인)
