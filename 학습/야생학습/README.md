---
type: hub
title: "야생학습"
summary: "실제 작업을 계기로 손으로 배우는 세션의 허브"
status: active
created: 2026-08-30
updated: 2026-09-05
---

# 야생학습

**실제 작업이 던진 문제를 계기로, 커리큘럼 없이 손으로 부딪혀 배우는 기록이다.** 학교학습이 커리큘럼 순서대로 배운다면, 야생학습은 **문제가 학습 순서를 정한다** — 필요한 것을 필요한 순간에, 예측하고 실행해서 배운다.

## 언제 하나

에이전트가 작업 중 신호를 포착해 제안한다 (규칙 원본: [[학습/README|학습]] 허브의 발동 규칙). 홍이 승낙하면 그 자리에서 10~30분 진행한다. 작업 흐름을 끊지 않는 것이 우선이므로, 급하지 않으면 그날 작업 마무리 뒤로 미룬다 (공부 README의 "작업 중 규칙"과 동일).

## 어떻게 기록하나

- 파일: **`학습/야생학습/<주제> YYYY-MM-DD.md`** — 세션 단위 기록. 주제가 반복되면 주제 파일로 합친다.
- 내용: ① 계기가 된 작업(프로젝트 링크) ② 해본 것(코드·명령 그대로) ③ **예측 vs 실제** ④ 결론 한 줄.
- 세션에서 나온 개념 정리가 다시 읽을 가치가 있으면 [[학습/공부/README|공부]] 주제 파일로 승격하고, 여기엔 과정이 남는다.
- 새 파일을 만들면 아래 목록에 등재한다.

## 예정 세션 — 프론트 사다리 (2026-09-05)

**계기**: 당근 FE 인턴 공고 스택(React·TS·**TanStack Query·Vite·Fastify·`react-dom/server` Streaming SSR**)과 [[프로젝트/개인/MyCryptoDiary/D4 회고 2026-09-04|MyCryptoDiary D4 회고]](한 일차에 새 개념 10개 → 3x). 새 제품 7일 MVP에 들어갈 때 이 개념들이 "처음 보는 것"이 아니게 만든다. 배경은 취업 운영 2026-09-05 (2차).

**규칙**: 세션당 새 개념 **≤3** · 홍이 전부 타이핑 · 에이전트는 로드맵 + 찾는 법(공식 문서 절·`node_modules/<pkg>/dist` 타입·grep)만, 빈칸 금지 · **예측 → 실행 → 기록** · 검증 명령을 시작 전에 정한다 · 끝에 한 줄 설명. 작업 폴더 `(로컬 경로)`(버리는 코드, git 불필요). 막히면 메모만 하고 세션 끝에 정리. 막히는 세션은 둘로 쪼갠다. 슬롯은 **소마 작업 시작 전 아침 30~45분, 화~토**(계획 우선순위 표). 측정: 주당 실행 횟수 · 예측 적중/누락 · 에너지 1~10.

| #   | 세션                                                                     | 새 개념                                      | 예측 질문                           | 검증                               |
| --- | ---------------------------------------------------------------------- | ----------------------------------------- | ------------------------------- | -------------------------------- |
| 1   | Vite+React+TS 빈 앱에서 같은 목록을 `useEffect`+fetch와 TanStack `useQuery` 두 벌로 | queryKey · staleTime · 캐시                 | "탭 A→B→A 왕복 시 요청은 몇 번?"         | DevTools Network 요청 수            |
| 2   | `refetchInterval` 폴링 + `staleTime` 조절                                  | refetchInterval · isFetching vs isLoading | "staleTime 10초면 재마운트 때 요청이 가나?" | Network + 화면 상태                  |
| 3   | Fastify + `renderToPipeableStream` 20줄 서버 + `hydrateRoot` 클라           | 서버 엔트리/클라 엔트리 · 하이드레이션                    | "JS 끄고도 글자가 보이나?"               | `curl` 응답 HTML에 텍스트, 브라우저 JS 비활성 |
| 4   | 느린 컴포넌트를 `Suspense`로 감싸 셸 먼저 보내기                                       | Suspense 경계 · 스트리밍 청크                     | "셸이 몇 ms에 도착하나?"                | `curl --no-buffer` 청크 순서         |
| 5   | 하이드레이션 불일치 일부러 내기(`Date.now()` 렌더)                                     | 서버/클라 출력 동일성                              | "경고가 뜨나, 화면은 어느 쪽 값?"           | 콘솔 경고 재현 → 수정                    |
| 6   | 서버 `prefetchQuery` → `dehydrate` → 클라 `HydrationBoundary`              | 서버 상태 직렬화                                 | "클라에서 같은 쿼리 요청이 다시 가나?"         | Network 요청 0건                    |
| 7   | 1초 폴링 목록에서 React Profiler로 렌더 수 세기, `memo` 전/후                         | Profiler · memo · 리렌더 범위                  | "행 10개 중 값 바뀐 1개만 렌더되나?"        | Profiler 커밋 수 before/after       |
| 8   | (선택) vanilla-extract 30분 — Tailwind와 무엇이 다른가                           | 빌드 타임 CSS · 타입 안전 스타일                     | "런타임에 CSS가 생성되나?"               | 빌드 산출물                           |
| 9   | (선택) iOS WKWebView 껍데기에 3번 페이지 띄우기                                     | 웹뷰 ↔ 웹 경계                                 | "Referer·localStorage가 웹과 같은가?" | 시뮬레이터                            |

- [ ] 1 — useEffect fetch vs TanStack useQuery
- [ ] 2 — refetchInterval 폴링과 staleTime
- [ ] 3 — Fastify + renderToPipeableStream + hydrateRoot 최소 SSR
- [ ] 4 — Suspense 스트리밍: 셸 먼저
- [ ] 5 — 하이드레이션 불일치 재현
- [ ] 6 — TanStack Query SSR prefetch → dehydrate → HydrationBoundary
- [ ] 7 — Profiler로 리렌더 세기, memo 전/후
- [ ] 8 — (선택) vanilla-extract 맛보기
- [ ] 9 — (선택) WKWebView 껍데기

일간 루틴은 이 체크리스트의 **첫 미체크 항목**을 `- [ ] 야생학습(아침 30~45분): <세션 제목>`으로 넣는다. 세션이 끝나면 체크하고 `학습/야생학습/<주제> YYYY-MM-DD.md`에 기록해 아래 목록에 등재한다. 개념이 남으면 [[학습/공부/README|공부]] `JS/` 주제 파일로 승격(실제로 나온 것만, 참고 자료는 URL 확인 후).

## 기록 목록

(아직 없음 — 첫 야생학습 세션에서 시작)
