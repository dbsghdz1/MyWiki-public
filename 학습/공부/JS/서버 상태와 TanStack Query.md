---
type: study
area: JS
audience: me
status: active
created: 2026-09-08
updated: 2026-09-12
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

### `dehydrate` — 서버의 캐시를 **말려서** 브라우저로 보낸다

캐시는 **한 프로그램의 메모리**에 산다. Node 서버와 브라우저는 다른 프로그램이라 메모리를 건넬 수 없고, 네트워크로는 **글자만** 간다. 그래서 캐시를 글자로 만들 수 있는 평범한 객체로 바꾸는 게 `dehydrate`(건조), 그걸 받아 브라우저 캐시에 되살리는 게 `hydrate`(물 붓기)다.

| 단계 | 어디서 | 코드 | 결과 |
|---|---|---|---|
| 채우기 | Node | `prefetchQuery({ queryKey: ["관악구"], queryFn })` | 서버 캐시에 데이터 |
| 말리기 | Node | `dehydrate(queryClient)` | `{ queries: [{ queryKey, state: { data, dataUpdatedAt, status }, dehydratedAt }] }` |
| 운반 | HTML | `<script>window.__DEHYDRATED_STATE__ = …</script>` | 글자 |
| 물 붓기 | 브라우저 | `<HydrationBoundary state={…}>` | 브라우저 캐시에 **같은 키로** 복원 → `useQuery(["관악구"])`가 요청 없이 꺼낸다 |

- **이름이 같은 두 하이드레이션.** React의 `hydrateRoot`는 **HTML(DOM)**을 이어받고, `HydrationBoundary`는 **데이터(캐시)**를 이어받는다. 한 페이지에서 둘 다 일어난다
- **`window.__PHARMACIES__`와의 차이는 받는 쪽이다.** 전에는 배열만 보내고 손으로 만든 전역에서 읽었다. 이제는 **캐시 한 칸 통째(키 + 데이터 + 받은 시각)**를 보내서 `useQuery`가 "이미 가져왔다"고 믿는다 — 컴포넌트는 서버에서 시작하든 아니든 같은 코드다. **데이터가 HTML과 JSON 두 벌로 가는 건 그대로다**
- **받은 시각이 같이 간다** → `staleTime`은 **서버가 가져온 순간부터** 센다. 기본값 0이면 브라우저가 받자마자 stale이라 **곧바로 다시 요청한다.** SSR에서는 `staleTime`을 0보다 크게 둔다
- **기본은 성공한 쿼리만 말린다**(`status === 'success'`). 실패한 쿼리는 안 보내서 브라우저가 다시 시도한다
- **브라우저 캐시가 더 새것이면 덮어쓰지 않는다**(`dataUpdatedAt` 비교)
- **서버 `QueryClient`는 요청마다 새로 만든다.** 파일 맨 위에 하나 두면 다른 사용자의 데이터가 섞인다
- **`JSON.stringify`를 `<script>`에 그대로 넣으면 XSS 구멍이다.** 데이터에 `</script>`가 있으면 태그를 탈출한다 → `<` 문자를 유니코드 이스케이프로 바꾸거나(지금 `server/main.js`의 `.replace(/</g, …)`) `devalue`·`serialize-javascript`를 쓴다

### 낙관적 업데이트 — 성공을 가정하고 먼저 그린다

`mutate()`는 요청 → 응답 → `invalidateQueries` → refetch로 왕복이 두 번이라, 그동안 화면은 아무 일도 안 일어난 것처럼 보인다. **낙관적 업데이트는 성공을 가정하고 UI를 먼저 바꾼 뒤 실패하면 되돌린다.** '낙관'의 대가가 **롤백 책임**이다. 방법이 둘이고, 갈리는 축은 **캐시를 건드리느냐**다.

**방법 A — `variables`로 UI에서만.** 캐시는 그대로 두고 렌더링할 때 임시 항목을 하나 더 그린다.

```tsx
{todoQuery.data.map(t => <li key={t.id}>{t.text}</li>)}
{isPending && <li style={{ opacity: 0.5 }}>{variables}</li>}
```

`variables`는 `mutate(x)`에 넘긴 그 `x`다. **롤백 코드가 아예 필요 없다** — 되돌릴 상태를 만든 적이 없으니까. `isPending`이 꺼지면 임시 항목은 그냥 사라진다. 실패해도 `variables`는 남아 있어서 재시도 버튼(`mutate(variables)`)을 그릴 수 있다. 단점은 **그 컴포넌트에서만 보인다**는 것.

**방법 B — `onMutate`로 캐시를 직접 고친다.** 네 부분이 각각 이유가 있다.

```tsx
onMutate: async (newTodo, context) => {
  await context.client.cancelQueries({ queryKey: ['todos'] })      // ①
  const previousTodos = context.client.getQueryData(['todos'])      // ②
  context.client.setQueryData(['todos'], old => [...old, newTodo])
  return { previousTodos }                                          // ③
},
onError: (err, newTodo, onMutateResult, context) =>
  context.client.setQueryData(['todos'], onMutateResult.previousTodos),
onSettled: (data, error, variables, onMutateResult, context) =>
  context.client.invalidateQueries({ queryKey: ['todos'] }),        // ④
```

- **① `cancelQueries`가 없으면 깜빡인다.** 내가 `setQueryData`로 고치기 **직전에 이미 날아간 refetch**가 있으면, 그 응답이 내 수정 **뒤에** 도착해 옛날 서버 데이터로 덮어쓴다. 방금 추가한 항목이 사라졌다 다시 나타난다. 취소가 끝난 뒤에 써야 하므로 `await`
- **②③ 롤백 스냅샷은 클로저가 아니라 반환값으로 넘긴다.** `onMutate`의 반환값이 `onError`/`onSettled`의 세 번째 인자로 전달된다. **mutation은 동시에 여러 개 뜰 수 있어서** 각자 자기 스냅샷을 자기 실행에 묶어야 한다 — 컴포넌트 변수에 담으면 두 번째가 첫 번째 것을 덮어쓴다
- **④ 결국 서버를 진실로 삼는다.** 낙관적으로 그린 값은 추측이다(서버가 붙이는 `id`·`createdAt`·정렬 순서를 내가 모른다). `return`으로 프로미스를 **돌려줘야** refetch가 끝날 때까지 mutation이 `pending`으로 남아 "완료됐다 다시 로딩" 깜빡임이 없다
- **동시 mutation에서는 ④가 오히려 깨진다.** 첫 mutation의 invalidate가 아직 진행 중인 두 번째의 낙관적 값을 옛 데이터로 덮는다. TkDodo는 `if (queryClient.isMutating() === 1) invalidateQueries(...)`로 **다른 mutation이 없을 때만** invalidate하라고 한다(`onSettled` 실행 중에는 자기 자신이 세어지므로 `1`)

**콜백 인자 순서가 v5 중간에 바뀌었다.** 예전엔 `onMutate`의 반환값 이름이 `context`였는데, 지금은 **`onMutateResult`**이고 `context`는 **마지막 인자**(`{ client, meta, mutationKey }`)다. 그래서 `useQueryClient()` 없이 `context.client`로 쓴다. 인터넷 예제가 `onError: (err, vars, context) => context.previousTodos` 꼴이면 옛 시그니처다.

```
onMutate  (variables, context)
onSuccess (data, variables, onMutateResult, context)
onError   (error, variables, onMutateResult, context)
onSettled (data, error, variables, onMutateResult, context)
```

**언제 어느 쪽** — 낙관적 결과를 **한 군데서만** 보여주면 되면 A(코드가 적고 롤백이 없다), **여러 화면이 같이 반응해야** 하면 B. 공식 문서 결론도 같다.

## 기록

### 2026-09-09 — useEffect 버전과 나란히 놓고 요청 수를 셌다 (야생학습 사다리 2-B)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[학습/야생학습/약국맵 사다리 2 — useEffect와 useQuery 2026-09-09|사다리 세션 2·2-B]]. 같은 약국 목록 API를 두 벌로 구현해 Network 줄 수를 비교
- 배운 것: 위 「핵심 정리」 전부가 이 세션에서 나왔다. 특히 **`useQuery`가 기본값에서는 `useEffect`보다 요청을 더 많이 보낸다**(포커스마다) — 그게 손해가 아니라 **약국 영업 상태처럼 변하는 데이터에는 맞는 기본값**이고, 빈도는 `staleTime`으로 내가 정한다
- 계측 함정: Network 탭의 **`Preserve log`가 켜져 있으면** 새로고침해도 줄이 안 지워져서 "요청이 계속 쌓인다"로 보인다. **줄 수를 세는 실험에서는 꺼야 한다**
- 근거: `pharmacy-map` 커밋 `238d87f`

### 2026-09-12 — `dehydrate`가 뭔지 물었다 (야생학습 사다리 6 진행 중)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] 사다리 6. `window.__PHARMACIES__`를 `prefetchQuery` → `dehydrate` → `HydrationBoundary`로 바꾸는 코드를 쓰던 중 *"dehydrate 개념이 뭐야"*
- 배운 것: 위 「`dehydrate`」 절. 핵심은 **캐시는 메모리라 못 건너가고 글자만 건너간다** — 그래서 말렸다가 붓는다
- 정정: [[학습/야생학습/약국맵 사다리 3-B·3-C — SSR과 하이드레이션 2026-09-09|3-C 기록]]의 *"HTML·JSON 두 벌 중복을 없애는 게 사다리 6"*은 **반만 맞다.** 두 벌로 가는 건 그대로고, 없어지는 건 **손으로 만든 전역과 그 전역을 읽는 별도 코드**다
- 근거: 설치된 `@tanstack/query-core` 5.102.8 `src/hydration.ts` — `defaultShouldDehydrateQuery`가 `status === 'success'`, `hydrate`가 `state.dataUpdatedAt > query.state.dataUpdatedAt`일 때만 덮어씀. 작업 파일 `server/main.js` · `src/main.tsx`(미커밋)

### 2026-09-12 — 낙관적 업데이트 문서를 읽고 개념을 물었다

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] 작업 중 Optimistic Updates 문서를 붙여넣고 *"개념 설명해줘"*. **지금 약국맵은 조회만 해서 `useMutation`이 한 군데도 없다** — 적용 대상이 아직 없는 상태에서 개념만 먼저 잡았다
- 배운 것: 위 「낙관적 업데이트」 절. 핵심 두 가지 — **캐시를 안 건드리면 롤백도 필요 없다**(방법 A), **`cancelQueries`는 이미 날아간 refetch가 내 수정을 덮는 경쟁을 막는 것**이지 예의상 붙이는 게 아니다
- 근거: 설치된 `@tanstack/react-query` 5.102.8의 `query-core/build/modern/hydration-Bjs0MSgg.d.ts` `MutationOptions` — `onError`/`onSettled`의 세 번째 인자가 `onMutateResult`, 네 번째가 `MutationFunctionContext = { client, meta, mutationKey }`로 확인됨. 즐겨찾기·메모 같은 쓰기 기능을 붙이는 순간 쓸 자리가 생긴다

## 참고 자료

- [TanStack Query — Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults) — 기본값의 정본. *"by default consider cached data as stale"*, *"inactive queries are garbage collected after 5 minutes"* (2026-09-08 확인)
- [TanStack Query — Query Keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys) — *"query keys act as dependencies for your query functions"*. 객체 키 순서는 무관하지만 배열 항목 순서는 유의미 (2026-09-08 확인)
- [TkDodo — Practical React Query](https://tkdodo.eu/blog/practical-react-query) — 메인테이너 본인 글. *"server state … your app does not own it"*, `staleTime` 기본 0 (2026-09-08 확인)
- [TanStack Query — Server Rendering & Hydration](https://tanstack.com/query/latest/docs/framework/react/guides/ssr) — `prefetch` → `dehydrate` → `HydrationBoundary` 흐름의 정본. *"set some default staleTime above 0 to avoid refetching immediately on the client"*, 요청마다 새 `QueryClient`, `JSON.stringify`의 XSS 경고 (2026-09-12 확인)
- [TanStack Query — Advanced Server Rendering](https://tanstack.com/query/latest/docs/framework/react/guides/advanced-ssr) — 같은 흐름을 Server Components에서. *"HydrationBoundary is a Client Component, so hydration will happen there"* (2026-09-12 확인)
- [TanStack Query — Optimistic Updates](https://tanstack.com/query/latest/docs/framework/react/guides/optimistic-updates) — 두 방법의 정본. *"This is the simpler variant, as it doesn't interact with the cache directly"*, *"Cancel any outgoing refetches (so they don't overwrite our optimistic update)"*, 한 군데면 `variables`·여러 곳이면 캐시 (2026-09-12 확인)
- [TkDodo — Concurrent Optimistic Updates in React Query](https://tkdodo.eu/blog/concurrent-optimistic-updates-in-react-query) — 동시 mutation에서 첫 invalidate가 두 번째의 낙관적 값을 덮는 문제와 `isMutating() === 1` 해법 (2026-09-12 확인)
