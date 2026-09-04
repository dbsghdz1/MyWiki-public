---
type: hub
title: "React·TypeScript로 제품 만들기"
summary: "React·TypeScript 학습을 실제 제품 개발과 연결하는 지식 허브"
status: active
aliases:
  - React TypeScript Product Development
  - React TS 제품 개발
created: 2026-07-14
updated: 2026-08-18
sources:
  - "2026-07-14-roadmap-sh-react-roadmap"
  - "2026-07-15-roadmap-sh-javascript-roadmap"
  - "2026-07-23-typescript-handbook-variable-declarations"
  - "[[_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes]]"
---

# React·TypeScript로 제품 만들기

React와 TypeScript를 따로 암기하는 대신 실제 제품을 설계하고 구현하면서 얻은 재사용 가능한 지식을 축적하는 최상위 허브다. 공식 문서와 외부 글, 구현 기록, 디버깅 경험을 연결해 “무엇인가”뿐 아니라 “언제, 왜, 어떻게 사용하는가”까지 정리한다.

> [!note] 시작 상태
> 첫 기준 자료로 roadmap.sh의 React Developer Roadmap을 수집했다. 로드맵은 학습 범위를 찾는 지도이며 모든 도구를 순서대로 익혀야 하는 의무 목록으로 취급하지 않는다. roadmap.sh React 로드맵

## 목표

- React와 TypeScript의 핵심 원리를 실제 코드와 제품 결정에 연결한다.
- 같은 문제를 반복해서 검색하지 않도록 오류, 해결 과정, 선택의 이유를 축적한다.
- 개인 앱을 작은 실험장으로 사용해 학습 결과를 검증한다.
- 면접에서 개념을 설명하는 능력과 제품에서 올바르게 적용하는 능력을 함께 기른다.

## 지식 지도

| 영역 | 축적할 내용 | 페이지 생성 기준 |
|---|---|---|
| React 컴포넌트 설계 | 책임 분리, 합성, 렌더링 경계, 재사용 판단 | 서로 다른 출처와 구현 사례가 함께 생길 때 |
| TypeScript 타입 모델링 | 도메인 모델, union, generic, narrowing, 외부 입력 검증 | 실제 모델링 결정이나 반복되는 오류가 생길 때 |
| 상태와 데이터 흐름 | 로컬·서버 상태, 비동기 흐름, 캐시, 폼 | 대안을 비교하거나 제품 사례가 생길 때 |
| 테스트와 디버깅 | 테스트 전략, 재현 절차, 실패 원인, 회귀 방지 | 재사용 가능한 조사·해결 패턴이 생길 때 |
| 성능과 사용자 경험 | 렌더링, 번들, 네트워크, 접근성, 체감 성능 | 측정값 또는 사용자 관찰이 있을 때 |
| 개발 도구와 배포 | 빌드 도구, lint, CI, 관측, 배포 | 설정 선택의 이유와 결과를 비교할 수 있을 때 |
| 개인 앱 사례 | 요구사항, 가설, 기술 결정, 실험 결과 | 앱 아이디어나 구현이 구체화될 때 |

각 영역은 처음부터 빈 개별 페이지로 만들지 않는다. 관련 자료와 경험이 쌓이기 전에는 이 표와 기존 허브에 통합하고, 여러 출처를 압축할 독립적인 주제가 형성되면 분리한다.

## 현재 학습 기준선

> [!note] 실제 학습 기록은 공부 영역에 있다 (2026-08-18)
> 이 허브의 지도에 대응하는 1인칭 학습 노트가 MyCryptoDiary 작업에서 나오고 있다 — [[학습/공부/JS/JavaScript 기초 문법|JavaScript 기초 문법]] · [[학습/공부/JS/JavaScript 런타임|JavaScript 런타임]] · [[학습/공부/JS/JavaScript 모듈 시스템|JavaScript 모듈 시스템]] · [[학습/공부/JS/TypeScript 타입 시스템|TypeScript 타입 시스템]] · [[학습/공부/JS/Next.js 서버와 캐싱|Next.js 서버와 캐싱]] · [[학습/공부/JS/Feature-Sliced Design|Feature-Sliced Design]]. 새 이해는 그쪽 `## 기록`에 쌓고, 이 허브는 외부 로드맵 기준선과 지식 지도만 유지한다.


roadmap.sh는 React 학습 전에 JavaScript 초급 주제를 익히고, Vite를 이용한 환경 구성에서 함수형 컴포넌트, JSX, props와 state, 조건부 렌더링, 합성으로 진행하는 경로를 제시한다. 이어 기본·공통 Hooks와 렌더링 개념을 다룬 뒤 상태 관리, 라우팅, API 호출, 테스트, TypeScript와 검증, 폼으로 범위를 넓힌다. 순서는 엄격하지 않다고 명시한다. roadmap.sh React 로드맵

JavaScript 로드맵은 프레임워크보다 먼저 함수, 연산자, 자료구조 등 언어의 공통 기반을 익혀야 한다고 설명한다. 초급 주제로 변수와 scope, primitive와 object, 타입 변환과 동등성, 반복문, 연산자, 제어 흐름, 함수, 배열, 예외 처리, Fetch 등을 배치한다. lexical scope와 closure, DOM API, strict mode, `this`, promise와 `async/await`, ES Modules는 중급으로 분류한다. roadmap.sh JavaScript 로드맵

### React 학습 전에 확인할 JavaScript

아래 우선순위는 JavaScript 로드맵을 React 제품 개발 관점에서 재구성한 **종합**이다.

| 우선순위 | JavaScript 주제 | React에서 연결되는 지점 |
|---|---|---|
| 먼저 | `let`·`const`, scope, hoisting | 컴포넌트와 Hook 내부 값의 생명주기 이해 |
| 먼저 | primitive, object, array, 타입 변환, `===` | props·state 비교와 데이터 가공의 기반 |
| 먼저 | 함수, parameter, arrow function, lexical scope, closure | 컴포넌트, event handler, Hook callback 이해 |
| 먼저 | array 반복과 변환, 조건문, 연산자 | 목록 렌더링과 조건부 UI 작성 |
| 먼저 | exception, promise, `async/await`, Fetch | API 요청과 loading·error 흐름 구현 |
| 먼저 | ES Modules | 컴포넌트와 기능 모듈 구성 |
| 필요할 때 | DOM API, event loop, `this`, prototype | ref·effect·비동기 동작 또는 레거시 코드 분석 |
| 나중 | iterator·generator, class, memory management | 성능 문제나 라이브러리 내부 동작을 파고들 때 |

이 표는 “모두 마친 뒤 React를 시작한다”는 관문이 아니다. 먼저 항목을 짧게 점검하고 React 기능을 만들다가 빈틈이 드러나면 해당 JavaScript 주제로 돌아오는 왕복 경로다.

### 변수 선언: var·let·const와 구조 분해

위 표의 “먼저: `let`·`const`, scope, hoisting” 항목을 TypeScript 공식 핸드북으로 채운 첫 페이지다. 핸드북이 직접 말하는 규칙은 다음과 같다. TS 핸드북 변수 선언

| 항목 | `var` | `let` | `const` |
|---|---|---|---|
| 스코프 | 함수 스코프 | 블록 스코프 | 블록 스코프 |
| 같은 스코프 재선언 | 허용(버그 유발) | 오류 | 오류 |
| 선언 전 접근 | `undefined` | 오류(TDZ) | 오류(TDZ) |
| 루프 클로저 캡처 | 모든 콜백이 최종값 공유 | 반복마다 새 환경 | 반복마다 새 환경 |
| 재할당 | 가능 | 가능 | 불가(내부 프로퍼티는 수정 가능) |

- 권장 순서는 **기본 `const`, 재할당이 필요할 때만 `let`**이다(최소 권한의 원칙). TS 핸드북 변수 선언
- `const`는 재할당만 막고 불변을 보장하지 않는다. 객체 멤버까지 잠그려면 `readonly`를 쓴다. TS 핸드북 변수 선언
- 구조 분해는 배열·튜플·객체 모두 지원하고, 이름 바꾸기(`{ a: newName }`), 기본값(`{ b = 1001 }`), 나머지(`...rest`)를 조합할 수 있다. 깊은 중첩은 가독성 때문에 피하라고 권고한다. TS 핸드북 변수 선언
- 전개(spread)는 **얕은 복사**이며 객체 전개는 뒤에 오는 프로퍼티가 앞을 덮어쓴다. 클래스 인스턴스를 전개하면 메서드를 잃는다. TS 핸드북 변수 선언

다음은 이 규칙을 React 제품 코드와 연결한 **종합**이다.

- `useState`가 돌려주는 `[value, setValue]`는 배열(튜플) 구조 분해이고, props를 받는 `function Card({ title, onClick })`은 기본값까지 조합할 수 있는 객체 구조 분해다. 핸드북의 구조 분해 규칙이 React 코드의 첫 줄부터 쓰인다.
- `setItems([...items, newItem])`, `setUser({ ...user, name })` 같은 불변 갱신 패턴은 전개의 “얕은 복사 + 뒤가 앞을 덮어씀” 규칙 그대로다. 얕은 복사이므로 중첩 객체는 한 단계씩 다시 전개해야 한다.
- `var`의 루프 캡처 함정(콜백이 최종값을 공유)은 React에서도 stale closure 형태로 재현된다. `let`·`const`가 반복마다 새 환경을 만든다는 사실이 event handler와 `useEffect` 클로저를 이해하는 기반이 된다.
- MyCryptoDiary 코드에서 `var`가 보이면 교체 대상이고, ESLint의 `prefer-const`는 핸드북의 권장을 자동화한 것이다.

핸드북의 “spread는 얕은 복사” 규칙은 실제 구현에서 이미 사용되었다. MyCryptoDiary Day 1–4에서 `sort()`가 원본 배열을 변경하기 때문에 props를 직접 바꾸지 않으려고 `[...diaries].sort(...)`로 복사 후 정렬했다. 두 출처가 같은 규칙을 원리(핸드북)와 적용 사례(학습 노트)로 뒷받침한다. TS 핸드북 변수 선언 [[_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes|MyCryptoDiary Day 1–4 학습 노트]]

### SwiftUI 경험에서 React로

BarStack으로 SwiftUI를 써 본 경험을 React 학습의 지렛대로 쓴다. Day 1–4 학습 노트가 정리한 대응 관계다. 역할이 비슷하다는 뜻이며 내부 구현이 같은 것은 아니다. [[_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes|MyCryptoDiary Day 1–4 학습 노트]]

| React·TypeScript | Swift·SwiftUI | 주의할 차이 |
|---|---|---|
| object type / union type | `struct` / `enum` 비슷한 제한 값 | union은 값의 집합이지 케이스별 연관값이 없다 |
| 함수 컴포넌트, props | `View`, initializer 파라미터 | props는 읽기 전용 입력이다 |
| `useState`의 `[value, setValue]` | `@State` | setter 함수 호출이 재렌더링을 일으킨다 |
| `map()`으로 JSX 목록 + `key` | `ForEach(..., id:)` | `key`는 UI에 보이지 않는 내부 식별자다 |
| `onClick` 화살표 함수 | `Button` action closure | — |
| Next.js `Link` | `NavigationLink` | 파일 기반 라우팅(`app/diary/[id]`)과 결합된다 |
| `sort()` 뒤 spread 복사 | `sorted()`가 새 배열 반환 | JS `sort()`는 **원본을 변경**한다 — Swift와 반대 |

노트가 남긴 재사용 가능한 구조 원칙(종합 표시): Next.js App Router 컴포넌트는 기본이 Server Component이고 `useState`·`onClick`이 필요한 파일만 `'use client'`로 경계를 만든다. Mock 데이터는 `data/`로 분리해 두면 API 응답으로 교체하기 쉽고 컴포넌트 테스트가 쉬워진다 — 이는 MyCryptoDiary만이 아니라 이후 React 프로젝트의 기본 배치로 삼는다. [[_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes|MyCryptoDiary Day 1–4 학습 노트]]

### 제품 중심 학습 순서

아래 순서는 원본 항목을 이 위키의 목적에 맞게 재구성한 **종합**이다.

1. **기반 확인**: 위 JavaScript 선행 지식을 점검하고 Vite, React, TypeScript로 작은 실행 환경을 만든다.
2. **UI 표현**: 함수형 컴포넌트, JSX, props와 state, 조건부 렌더링, composition을 사용해 한 화면을 완성한다.
3. **상호작용과 흐름**: 이벤트, `useState`, `useEffect`, 목록과 key를 실제 기능에 적용한다.
4. **구조화**: custom Hooks, Context, 라우팅을 도입하되 현재 제품에서 문제가 생길 때만 외부 상태 관리 도구를 비교한다.
5. **데이터와 경계**: API 호출, TypeScript 모델링, Zod 검증, 폼 처리를 하나의 사용자 흐름에서 연결한다.
6. **신뢰성**: React Testing Library와 Vitest로 핵심 동작을 검증하고, 필요한 사용자 여정만 Playwright로 보호한다.
7. **필요 기반 확장**: UI 라이브러리, 프레임워크, 애니메이션, Suspense, Server APIs, React Native는 제품 요구가 생길 때 학습한다.

로드맵이 Zustand, Jotai, Tailwind CSS, Shadcn UI, React Query, Vitest, Playwright, TypeScript, Zod, React Hook Form 등을 개인 추천으로 표시하더라도 이를 이 위키의 기본 기술 스택으로 자동 채택하지 않는다. 각 도구는 해결할 문제가 생겼을 때 공식 문서와 직접 실험을 추가해 판단한다. roadmap.sh React 로드맵

### 레거시 React

로드맵은 Class Components를 현재 학습 대상으로 권장하지 않지만 레거시 프로젝트에서 마주칠 수 있다고 설명한다. 따라서 새 개인 앱에서는 함수형 컴포넌트를 기준으로 삼고, Class Components는 기존 코드를 읽거나 마이그레이션할 필요가 생길 때 다룬다. roadmap.sh React 로드맵

## 자료 우선순위

1. React·TypeScript 공식 문서와 표준 문서
2. 직접 작성한 코드, 오류 메시지, 실험 및 측정 결과
3. 신뢰할 수 있는 라이브러리의 공식 문서와 유지관리자 설명
4. 기술 글, 강의, 면접 자료

면접용 요약이나 블로그의 주장을 공식 문서와 같은 근거 수준으로 취급하지 않는다. 튜토리얼의 패턴이 실제 제품에도 적합한지는 요구사항과 트레이드오프를 별도로 검토한다.

## 기록할 구현 경험

구현 기록은 단순 일지보다 다음 질문에 답할 때 위키에 남긴다.

- 해결하려던 사용자 또는 개발 문제는 무엇이었는가?
- 검토한 선택지는 무엇이고 어떤 기준으로 결정했는가?
- 예상과 실제 결과가 어떻게 달랐는가?
- 다른 프로젝트에서도 재사용할 수 있는 원리는 무엇인가?
- 다음에 같은 상황이 생기면 무엇을 다르게 할 것인가?

개인 앱은 별도 학습 도메인이 아니라 이 지식을 시험하는 사례로 둔다. 현재 활성 프로젝트는 [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]이며, 프로젝트 고유 요구사항과 진행 기록은 프로젝트 폴더에서 관리한다.

## 위키 자체에 관한 메타 지식

이 vault의 축적 방식과 품질 관리 원칙은 [[_wiki/LLM Wiki|LLM Wiki]]에 정리되어 있다. 이 페이지는 학습 도메인의 진입점이고, LLM Wiki 페이지는 위키 운영법을 설명하는 메타 허브다.

## 첫 수집 후보

- 위 제품 중심 학습 순서에서 지금 시작할 단계의 React 또는 TypeScript 공식 문서 한 편
- 최근 이해하기 어려웠거나 반복해서 검색한 개념
- 직접 마주친 타입 오류나 React 동작 문제와 재현 코드
- 만들고 싶은 개인 앱의 문제 정의 또는 가장 작은 기능 가설
