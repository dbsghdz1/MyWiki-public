---
type: study
area: JS
audience: me
status: active
created: 2026-09-08
updated: 2026-09-08
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

(아직 없음 — 첫 세션 이후 채운다)

## 기록

## 참고 자료

- [TanStack Query — Important Defaults](https://tanstack.com/query/latest/docs/framework/react/guides/important-defaults) — 기본값의 정본. *"by default consider cached data as stale"*, *"inactive queries are garbage collected after 5 minutes"* (2026-09-08 확인)
- [TanStack Query — Query Keys](https://tanstack.com/query/latest/docs/framework/react/guides/query-keys) — *"query keys act as dependencies for your query functions"*. 객체 키 순서는 무관하지만 배열 항목 순서는 유의미 (2026-09-08 확인)
- [TkDodo — Practical React Query](https://tkdodo.eu/blog/practical-react-query) — 메인테이너 본인 글. *"server state … your app does not own it"*, `staleTime` 기본 0 (2026-09-08 확인)
