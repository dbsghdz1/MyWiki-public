---
type: project
status: active
aliases:
  - My Crypto Diary
  - 마이 크립토 다이어리
  - CoinPilot
created: 2026-07-18
updated: 2026-08-29
slack_channel: my-crypto-diary
repos:
  - "github.com/dbsghdz1/MyCryptoDiary"
related_wiki:
  - "[[_wiki/React TypeScript 제품 개발]]"
---

# CoinPilot (MyCryptoDiary)

**CoinPilot**은 가상 1,000만원으로 시작하는 모의투자 거래소 + 매매일기 + 유저 랭킹 앱이다. 2026-08-29에 서비스 표시명을 `CoinPilot`로 확정했고, 기존 링크와 코드 저장소 연결을 보존하기 위해 프로젝트 작업공간·저장소 이름은 `MyCryptoDiary`를 유지한다. 2026-08-15 전환 전에는 암호자산 거래 기록을 돌아보고 누적 손익, 거래 수, 승률 같은 지표를 확인하는 개인 앱이었다. 현재 설명은 2026-07-24에 수집한 사용자 학습 노트 [[_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes|MyCryptoDiary Day 1–4 학습 노트]]를 기준으로 한다.

## 현재 확인된 구현

- Next.js App Router 기반 라우팅
- 다크 모드 대시보드 UI
- 거래 일기와 마켓 코인의 mock data 및 TypeScript 타입
- 거래 목록, 요약 지표, 정렬
- 거래 상세 페이지와 새 기록 페이지 이동
- `DiarySummary`, `DiaryList` 컴포넌트 분리

2026-08-14에 저장소를 직접 확인한 결과: 스택은 Next.js 16 · React 19 · Tailwind 4이고 상태관리·데이터패칭 라이브러리는 없다(순수 React). TS/TSX 16개 파일 약 614줄, 마지막 커밋 2026-08-06(디자인 토큰·글래스 유틸·홈 카드 분리). **실시간 관련 코드는 아직 없으며**, `LiveMarketCard`는 제목만 "실시간 차트"이고 내용은 하드코딩 배열 5개다.

아래 목록은 Day 1–4 학습 노트에서 확인한 상태이며 실제 저장소나 실행 화면으로 검증한 것은 아니다. [[_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes|MyCryptoDiary Day 1–4 학습 노트]]

## 프로젝트에서 관리할 것

- 해결하려는 사용자 문제와 앱의 성공 기준
- MVP 범위와 제외 범위
- 화면·데이터 모델·기술 선택에 관한 결정
- 기능 실험과 사용자 피드백
- 구현 진행 상황과 다음 작업

## 위키로 보낼 것

여러 제품에서 다시 쓸 수 있는 React, TypeScript, Next.js, Tailwind CSS의 원리와 디버깅 지식은 [[_wiki/React TypeScript 제품 개발|React·TypeScript로 제품 만들기]]에 종합한다. MyCryptoDiary에만 해당하는 요구사항과 진행 기록은 이 프로젝트 폴더에 남긴다.

## 우선순위 갱신 (2026-08-29)

**CoinPilot을 최우선으로 올렸다.** D3 Clerk 인증과 최초 계좌 생성을 2026-08-29에 완료했고, 다음은 D4 매수 거래 엔진이다. 08-21~25 공백은 우선순위 위반이 아니라 의도된 배치였다 — 이 프로젝트는 학습 모드(직접 타이핑)라 자투리 시간에 진행할 수 없다. 2주 계획 기한(~08-28)은 폐기됐지만 D3~D12 순서는 유지한다.

## 우선순위 결정 (2026-08-14)

**포트폴리오를 최우선**으로 한다. iOS → TypeScript·React 직무 전환용이며 **2026-11-14 지원**(당근 윈터테크)이 목표다. (**2026-08-16 갱신**: 채용 조사 결과 지원 D-day가 `2026-11-14`로 3개월 앞당겨졌다 — 취업 운영 참고. 아래 실시간 UI 서사는 모의투자 2주 계획이 끝나는 08-28 이후 Phase 2에서 얹는다.) W1~W2(2026-08-15 ~ 08-28)는 **모의투자 거래소 구축**이 핵심이고, WebSocket·실시간 시세 UI는 08-28 이후 Phase 2에서 얹는다. 상세는 [[프로젝트/개인/MyCryptoDiary/실시간 데이터 UI 계획 2026-08-14|실시간 데이터 UI 계획]].

## 보상형 광고 결정 (2026-08-15)

"광고 보면 가상 원화 충전"은 **mock provider로만 구현**한다. 일반 AdSense 유닛에 보상을 붙이면 계정 정지 사유이고, 정식 경로인 H5 Games Ads는 승인된 AdSense 계정이 선행 조건인 데다 사실상 게임 전용이다. 트래픽 0인 현재 승인 가능한 웹 보상형 네트워크는 없다. 포트폴리오 가치는 광고 태그가 아니라 **provider 추상화 + 서버 소유 잔액 + nonce·멱등성 설계**에서 나오며, 이 기능은 실시간 시세 UI보다 후순위다. 근거와 인터페이스 설계는 [[프로젝트/개인/MyCryptoDiary/보상형 광고 조사 2026-08-15|보상형 광고 조사]].

## 모의투자 전환 결정 (2026-08-15)

"매매일기"에서 **가상 1,000만원 모의투자 거래소 + 매매일기 + 유저 랭킹**으로 제품을 바꿨다. 일기는 없애지 않고 `orders.reason`(매수 이유) / `orders.review`(복기) 컬럼으로 흡수한다. 배포한다(→ Neon Postgres + Clerk, SQLite 폐기). 2주 계획(2026-08-15 ~ 08-28, 월 휴무, 12 개발일)은 로컬 `(로컬 경로)`가 원본이고, 프로젝트 저장소 `CLAUDE.md`가 학습 모드·리뷰 절차·위키 연동 규칙을 담는다.

- **핵심 규칙**: 랭킹은 자산이 아니라 **수익률** = (평가액 − 총투입원금) / 총투입원금. 광고 충전액은 `totalDeposited`에 가산해 충전해도 수익률이 오르지 않게 한다. **현금화·유저 간 양도 기능은 절대 넣지 않는다**(보상형 광고 정책).
- **아키텍처**: Feature-Sliced Design 2.1을 규격 그대로 배운다(`_app`/`_pages`/widgets/features/entities/shared, Steiger 린터로 검증). 순수 계산 함수는 `model/` 세그먼트에 두고 UI와 Server Action이 같은 함수를 쓴다.
- **테스트**: Vitest, 순수 계산 함수만, D5에 도입. 기댓값은 손으로 먼저 계산. 컴포넌트·E2E는 안 한다.
- **실시간 시세와의 관계**: 2주 계획 안에서 시세는 서버 fetch(`revalidate`) + 폴링이고 **WebSocket은 이 범위 밖으로 미뤘다**. [[프로젝트/개인/MyCryptoDiary/실시간 데이터 UI 계획 2026-08-14|실시간 데이터 UI 계획]]의 WebSocket·리렌더 최적화 서사는 거래소가 돌아간 뒤에 얹는다.
- **작업 방식**: 사용자가 React/TS를 배우면서 직접 타이핑한다. 순전히 기계적인 작업(파일 이동 등)만 에이전트가 한다. 커밋은 아주 작게, **작업마다 브랜치 → PR** (2026-08-15 추가 규칙).

### 진행 상황

| 일차 | 날짜 | 상태 |
|---|---|---|
| D1 FSD 재구조화 + 업비트 실데이터 | 8/15~20 (4 작업일) | **완료** — DoD 3개 통과. 이전 서술: **부분 완료** — FSD 이사·Steiger 위반 0 (브랜치 `refactor/fsd-structure`, 커밋 `a053007`·`e7751b8`). 업비트 연동은 D2 앞에 붙임. 상세 [[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|D1 작업 기록]] |
| D2 Neon + Drizzle 스키마 | 8/18 착수 · 8/26~28 | **완료 + 리뷰 반영** — Neon에 6테이블·FK 5개·복합 PK·enum·UNIQUE 반영, 5명 seed 최초·재실행 및 Studio 읽기 확인. 브랜치 `feat/db-schema`, 커밋 `1cfe07c`·`8eadff3`·`f7ccda4`, PR [#5](https://github.com/dbsghdz1/MyCryptoDiary/pull/5). 리뷰 반영은 PR [#6](https://github.com/dbsghdz1/MyCryptoDiary/pull/6)(드라이버 교체·`shared/db` 분리·seed 정합성·AGENTS.md). 상세 [[프로젝트/개인/MyCryptoDiary/Neon Drizzle 스키마 2026-08-28|D2 작업 기록]] |
| D3 Clerk 인증 | 8/28~29 | **완료** — Clerk 앱·키, proxy·Provider, 로그인·회원가입, 헤더 상태, Clerk ID 기반 계좌 lazy create, 홈 외 UI 보호. 브랜치 `feat/clerk-auth`, 커밋 `338e062`·`3001f51`·`7e22168`·`ffcb94c`·`6f47e54`, PR [#7](https://github.com/dbsghdz1/MyCryptoDiary/pull/7). 서비스 표시명 `CoinPilot` 확정(`7100212`). 상세 [[프로젝트/개인/MyCryptoDiary/Clerk 인증 D3 2026-08-29|D3 작업 기록]] |

> D4~D12 행은 계획 확정 후 추가 (계획 루틴이 이 표에서 일간 슬롯을 뽑는다)

## 배운 것

- [[공부/JS/Feature-Sliced Design|Feature-Sliced Design]] — 계층·슬라이스·세그먼트, `index.ts` public API, Next.js `app/`와 FSD `_app`의 분리, Steiger 0.6.0의 `_` 접두사 버그
- [[공부/JS/JavaScript 모듈 시스템|JavaScript 모듈 시스템]] — `export { default as X } from` 재수출, JS 모듈엔 접근제어자가 없어 슬라이스 경계는 언어가 아니라 린터가 지킨다
- [[공부/JS/JavaScript 기초 문법|JavaScript 기초 문법]] — 객체 리터럴과 구조 분해가 JS의 절반, `if (!x)`가 성립하는 truthy/falsy 근거
- [[공부/JS/JavaScript 런타임|JavaScript 런타임]] — Node는 V8 + OS 기능이라 서버가 될 수 있고, 그래서 API 키를 서버에만 둔다
- [[공부/JS/TypeScript 타입 시스템|TypeScript 타입 시스템]] — 타입은 컴파일하면 사라지므로 `res.ok` 검사를 손으로 넣는다
- [[공부/JS/Next.js 서버와 캐싱|Next.js 서버와 캐싱]] — 서버 컴포넌트가 직접 `await` 하고, `revalidate`는 데이터가 변하는 속도로 정한다. **`NEXT_PUBLIC_`은 허가가 아니라 브라우저 번들에 박으라는 명령이다**. proxy는 페이지가 아니라 **요청마다** 돌고 `matcher: []`는 아무것도 실행하지 않는다
- [[공부/CS/네트워크|네트워크]] — 서버는 포트를 잡은 프로그램일 뿐, `localhost`는 루프백이고 `*:3000`은 같은 와이파이에 열려 있다
- [[공부/JS/외부 API 데이터 모델링|외부 API 데이터 모델링]] — 응답 타입을 두 겹으로, 숫자로 들고 다니다 그릴 때만 포맷, 코인 이름은 업비트에서 받는다
- [[공부/CS/데이터베이스|데이터베이스]] — 스키마·DTO·Model의 경계, PK·FK·복합 PK, Drizzle `numeric`/`bigint`, 파생 랭킹 스냅샷

## 다음에 정할 것

- 누구의 어떤 거래 기록 문제를 해결하는가?
- 기존 메모장, 스프레드시트, 거래소 기록과 무엇이 다른가?
- 첫 사용자가 반드시 완료해야 하는 하나의 핵심 흐름은 무엇인가?
- mock data 다음으로 어떤 데이터를 실제 저장할 것인가?

## 코드 저장소

- 로컬: `(로컬 경로)` — Next.js 16, React 19, Tailwind CSS 4 (2026-07-29 경로 확인)
- GitHub: `dbsghdz1/MyCryptoDiary`

## 관련 자료

- 보존 원본: [[_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes|MyCryptoDiary Day 1–4 학습 노트]] — Day 1–4 구현 목록과 개념 정리
- 다음 학습 예고(노트 기준): 현물/선물 segmented control, 코인 목록 컴포넌트, 실시간 가격 상태 업데이트, Currency Detail 동적 페이지
- **학습 계획**: [[프로젝트/개인/MyCryptoDiary/학습 계획 2026-08-28|학습 계획 (2026-08-28 전면 개정)]] — 야생학습 9요소 진단(병목 = 연습·습관·에너지), 블록 루프, D3~D12 일정. **세션 시작 시 이 문서를 먼저 읽는다**
- 작업 기록: [[프로젝트/개인/MyCryptoDiary/실시간 데이터 UI 계획 2026-08-14|실시간 데이터 UI 계획]] · [[프로젝트/개인/MyCryptoDiary/보상형 광고 조사 2026-08-15|보상형 광고 조사]] · [[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|모의투자 전환 D1]] · [[프로젝트/개인/MyCryptoDiary/Neon Drizzle 스키마 2026-08-28|Neon·Drizzle 스키마 D2]] · [[프로젝트/개인/MyCryptoDiary/Clerk 인증 D3 2026-08-29|Clerk 인증 D3]]
- 학습 위키: [[_wiki/React TypeScript 제품 개발|React·TypeScript로 제품 만들기]]
