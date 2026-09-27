---
type: project
title: "CoinPilot (MyCryptoDiary)"
summary: "가상자산 모의투자와 매매 회고를 결합한 웹 서비스의 작업 진입점"
status: active
aliases:
  - My Crypto Diary
  - 마이 크립토 다이어리
  - CoinPilot
created: 2026-07-18
updated: 2026-09-28
slack_channel: 20-개인-my-crypto-diary
repos:
  - "github.com/dbsghdz1/MyCryptoDiary"
related_wiki:
  - "React TypeScript 제품 개발"
---

# CoinPilot (MyCryptoDiary)

## 현재 카드
- **단계**: 운영 — 재개(2026-09-24 홍 결정) · D5 착수는 2026-10-13 주(10월 중순, 2026-09-28 확정)
- **현재**: D4 매수 엔진까지 머지된 상태(마지막 커밋 `1d7fa69`)에서 재개한다 — 어디서 막혔고 어떻게 풀었는지는 [D4 회고](D4%20%ED%9A%8C%EA%B3%A0%202026-09-04.md)
- **다음 판정**: D5 완료 시 — 매도·손익이 돌면 D4~D5에서 푼 기술 문제를 글로 정리한다
- **지금 할 일**: D5(매도·손익·Vitest) 계획에 D4 회고 「고칠 것」(새 개념 목록 세기·블록별 검증 명령)을 적는다 — 10-13 주 착수 전
- **하지 않을 일**: 랭킹·소셜 등 새 기능 확장


> [!NOTE]
> 2026-09-05 보류 — **잠시 중지된 프로젝트다 (중단·폐기가 아니다)**
>
> 본인 판단: *"내 수준에선 너무 어려워."* 원인은 제품·코드 결함이 아니라 **수준 대비 난이도**다 — [D4 회고](D4%20%ED%9A%8C%EA%B3%A0%202026-09-04.md)에 적힌 대로 한 일차에 새 개념이 열 개 들어왔고(고정소수점·`bigint`·Server Action·신뢰 경계·트랜잭션·upsert·Drizzle 어휘…), 그중 React/UI 개념은 없었다. 어려움은 08-15 모의투자 거래소 전환이 만든 **풀스택 범위**에서 왔다.
> **보존 상태**: D1 FSD 재구조화 · D2 Neon+Drizzle 스키마(PR #5·#6) · D3 Clerk 인증(PR #7) · D4 매수 엔진(12블록, 커밋 9개) — 전부 머지된 채로 멈춰 있고 그 지점에서 재개할 수 있다. 서버 가격 재조회·정수 금액 모델·DB 트랜잭션·공격 입력 검증은 그대로 남아 있다.
> **재개 조건**: ~~위 현재 카드의 다음 판정~~ → 2026-09-24 재개 결정으로 닫혔다(D5 착수 10-13 주). 재개한다면 D4 회고의 「고칠 것」(일차 시작 전 새 개념 목록 세기, 막힌 종류 분류, 블록별 검증 명령)을 먼저 적용한다.
> 이 슬롯은 [실습](../%EC%95%BD%EA%B5%AD%EB%A7%B5/%ED%95%99%EC%8A%B5%20%EB%A1%9C%EB%93%9C%EB%A7%B5.md)로 바뀌었다.

**CoinPilot**은 가상 1,000만원으로 시작하는 모의투자 거래소 + 매매일기 + 유저 랭킹 앱이다. 2026-08-29에 서비스 표시명을 `CoinPilot`로 확정했고, 기존 링크와 코드 저장소 연결을 보존하기 위해 프로젝트 작업공간·저장소 이름은 `MyCryptoDiary`를 유지한다. 2026-08-15 전환 전에는 암호자산 거래 기록을 돌아보고 누적 손익, 거래 수, 승률 같은 지표를 확인하는 개인 앱이었다. 현재 설명은 2026-07-24에 수집한 사용자 학습 노트 [MyCryptoDiary Day 1–4 학습 노트](../../../_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes.md)를 기준으로 한다.

## 현재 확인된 구현

- Next.js App Router 기반 라우팅
- 다크 모드 대시보드 UI
- 거래 일기와 마켓 코인의 mock data 및 TypeScript 타입
- 거래 목록, 요약 지표, 정렬
- 거래 상세 페이지와 새 기록 페이지 이동
- `DiarySummary`, `DiaryList` 컴포넌트 분리

2026-08-14에 저장소를 직접 확인한 결과: 스택은 Next.js 16 · React 19 · Tailwind 4이고 상태관리·데이터패칭 라이브러리는 없다(순수 React). TS/TSX 16개 파일 약 614줄, 마지막 커밋 2026-08-06(디자인 토큰·글래스 유틸·홈 카드 분리). **실시간 관련 코드는 아직 없으며**, `LiveMarketCard`는 제목만 "실시간 차트"이고 내용은 하드코딩 배열 5개다.

아래 목록은 Day 1–4 학습 노트에서 확인한 상태이며 실제 저장소나 실행 화면으로 검증한 것은 아니다. [MyCryptoDiary Day 1–4 학습 노트](../../../_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes.md)

## 프로젝트에서 관리할 것

- 해결하려는 사용자 문제와 앱의 성공 기준
- MVP 범위와 제외 범위
- 화면·데이터 모델·기술 선택에 관한 결정
- 기능 실험과 사용자 피드백
- 구현 진행 상황과 다음 작업

## 위키로 보낼 것

여러 제품에서 다시 쓸 수 있는 React, TypeScript, Next.js, Tailwind CSS의 원리와 디버깅 지식은 [React·TypeScript로 제품 만들기](../../../_wiki/React%20TypeScript%20%EC%A0%9C%ED%92%88%20%EA%B0%9C%EB%B0%9C.md)에 종합한다. MyCryptoDiary에만 해당하는 요구사항과 진행 기록은 이 프로젝트 폴더에 남긴다.

## 우선순위 갱신 (2026-08-29)

**CoinPilot을 최우선으로 올렸다.** D3 Clerk 인증과 최초 계좌 생성을 2026-08-29에 완료했고, 다음은 D4 매수 거래 엔진이다. 08-21~25 공백은 우선순위 위반이 아니라 의도된 배치였다 — 이 프로젝트는 학습 모드(직접 타이핑)라 자투리 시간에 진행할 수 없다. 2주 계획 기한(~08-28)은 폐기됐지만 D3~D12 순서는 유지한다.

## 우선순위 결정 (2026-08-14)

iOS에서 TypeScript·React로 옮겨가며 만드는 **주력 프로젝트**다. W1~W2(2026-08-15 ~ 08-28)는 **모의투자 거래소 구축**이 핵심이고, WebSocket·실시간 시세 UI는 08-28 이후 Phase 2에서 얹는다. 상세는 [실시간 데이터 UI 계획](%EC%8B%A4%EC%8B%9C%EA%B0%84%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20UI%20%EA%B3%84%ED%9A%8D%202026-08-14.md).

## 보상형 광고 결정 (2026-08-15)

"광고 보면 가상 원화 충전"은 **mock provider로만 구현**한다. 일반 AdSense 유닛에 보상을 붙이면 계정 정지 사유이고, 정식 경로인 H5 Games Ads는 승인된 AdSense 계정이 선행 조건인 데다 사실상 게임 전용이다. 트래픽 0인 현재 승인 가능한 웹 보상형 네트워크는 없다. 설계할 거리는 광고 태그가 아니라 **provider 추상화 + 서버 소유 잔액 + nonce·멱등성 설계**에 있으며, 이 기능은 실시간 시세 UI보다 후순위다. 근거와 인터페이스 설계는 [보상형 광고 조사](%EB%B3%B4%EC%83%81%ED%98%95%20%EA%B4%91%EA%B3%A0%20%EC%A1%B0%EC%82%AC%202026-08-15.md).

## 반올림 규칙 (2026-08-30 확정, D4 착수 전)

코인 가격·수량은 `numeric`(소수)이고 원화 금액은 `bigint`(정수)라 곱하면 반드시 자를 곳이 생긴다. **오차가 생길 때 시스템이 손해 보지 않는 쪽으로 자른다** — 유저가 내는 것은 올리고, 유저가 받는 것은 내린다. 반복 거래로 잔고가 음수로 새지 않게 하기 위해서다.

| 값 | 방향 |
|---|---|
| 주문금액 (차감할 현금) | **올림** |
| 수수료 | **올림** |
| 매수 수량 | **내림** |

**최소 수수료 1원은 따로 둔 규칙이 아니라 올림에서 자동으로 따라온다** — 0보다 큰 금액을 올림하면 최소가 1원이다. 1,000원 주문의 수수료 0.5원은 1원이 된다.

이 규칙은 D5에서 테스트로 고정한다. **정한 뒤에는 바꾸지 않는다** — 규칙이 없으면 테스트가 구현을 그대로 베끼게 되고(계획서가 경고한 실패 모양), 나중에 바꾸면 이미 쌓인 주문 기록과 어긋난다.

## 모의투자 전환 결정 (2026-08-15)

"매매일기"에서 **가상 1,000만원 모의투자 거래소 + 매매일기 + 유저 랭킹**으로 제품을 바꿨다. 일기는 없애지 않고 `orders.reason`(매수 이유) / `orders.review`(복기) 컬럼으로 흡수한다. 배포한다(→ Neon Postgres + Clerk, SQLite 폐기). 2주 계획(2026-08-15 ~ 08-28, 월 휴무, 12 개발일)은 로컬 `(로컬 경로)`가 원본이고, 프로젝트 저장소 `CLAUDE.md`가 학습 모드·리뷰 절차·위키 연동 규칙을 담는다.

- **핵심 규칙**: 랭킹은 자산이 아니라 **수익률** = (평가액 − 총투입원금) / 총투입원금. 광고 충전액은 `totalDeposited`에 가산해 충전해도 수익률이 오르지 않게 한다. **현금화·유저 간 양도 기능은 절대 넣지 않는다**(보상형 광고 정책).
- **아키텍처**: Feature-Sliced Design 2.1을 규격 그대로 배운다(`_app`/`_pages`/widgets/features/entities/shared, Steiger 린터로 검증). 순수 계산 함수는 `model/` 세그먼트에 두고 UI와 Server Action이 같은 함수를 쓴다.
- **테스트**: Vitest, 순수 계산 함수만, D5에 도입. 기댓값은 손으로 먼저 계산. 컴포넌트·E2E는 안 한다.
- **실시간 시세와의 관계**: 2주 계획 안에서 시세는 서버 fetch(`revalidate`) + 폴링이고 **WebSocket은 이 범위 밖으로 미뤘다**. [실시간 데이터 UI 계획](%EC%8B%A4%EC%8B%9C%EA%B0%84%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20UI%20%EA%B3%84%ED%9A%8D%202026-08-14.md)의 WebSocket·리렌더 최적화 작업은 거래소가 돌아간 뒤에 얹는다.
- **작업 방식**: 사용자가 React/TS를 배우면서 직접 타이핑한다. 순전히 기계적인 작업(파일 이동 등)만 에이전트가 한다. 커밋은 아주 작게, **작업마다 브랜치 → PR** (2026-08-15 추가 규칙).

### 진행 상황

| 일차 | 날짜 | 상태 |
|---|---|---|
| D1 FSD 재구조화 + 업비트 실데이터 | 8/15~20 (4 작업일) | **완료** — DoD 3개 통과. 이전 서술: **부분 완료** — FSD 이사·Steiger 위반 0 (브랜치 `refactor/fsd-structure`, 커밋 `a053007`·`e7751b8`). 업비트 연동은 D2 앞에 붙임. 상세 [D1 작업 기록](%EB%AA%A8%EC%9D%98%ED%88%AC%EC%9E%90%20%EC%A0%84%ED%99%98%20D1%202026-08-16.md) |
| D2 Neon + Drizzle 스키마 | 8/18 착수 · 8/26~28 | **완료 + 리뷰 반영** — Neon에 6테이블·FK 5개·복합 PK·enum·UNIQUE 반영, 5명 seed 최초·재실행 및 Studio 읽기 확인. 브랜치 `feat/db-schema`, 커밋 `1cfe07c`·`8eadff3`·`f7ccda4`, PR [#5](https://github.com/dbsghdz1/MyCryptoDiary/pull/5). 리뷰 반영은 PR [#6](https://github.com/dbsghdz1/MyCryptoDiary/pull/6)(드라이버 교체·`shared/db` 분리·seed 정합성·AGENTS.md). 상세 [D2 작업 기록](Neon%20Drizzle%20%EC%8A%A4%ED%82%A4%EB%A7%88%202026-08-28.md) |
| D3 Clerk 인증 | 8/28~29 | **완료** — Clerk 앱·키, proxy·Provider, 로그인·회원가입, 헤더 상태, Clerk ID 기반 계좌 lazy create, 홈 외 UI 보호. 브랜치 `feat/clerk-auth`, 커밋 `338e062`·`3001f51`·`7e22168`·`ffcb94c`·`6f47e54`, PR [#7](https://github.com/dbsghdz1/MyCryptoDiary/pull/7). 서비스 표시명 `CoinPilot` 확정(`7100212`). 상세 [D3 작업 기록](Clerk%20%EC%9D%B8%EC%A6%9D%20D3%202026-08-29.md) |

| D4 거래 엔진 ① 매수 | 8/30~ | **완료** — 서버 가격 재조회(신뢰 경계)·잔고 검증·트랜잭션(현금↓·보유↑·주문기록)·입력 검증·매수 폼. 12블록, 커밋 9개. 회고 [D4 회고](D4%20%ED%9A%8C%EA%B3%A0%202026-09-04.md) 상세 [D4 작업 기록](%EA%B1%B0%EB%9E%98%20%EC%97%94%EC%A7%84%20D4%202026-09-02.md) |

> ~~D5~D12 행은 계획 확정 후 추가 (계획 루틴이 이 표에서 일간 슬롯을 뽑는다)~~ **2026-09-05 보류 — 보류 중에는 D5~D12 행을 추가하지 않는다.** 일간 루틴은 이 프로젝트의 슬롯을 만들지 않는다(보류 프로젝트 제외 규칙, 계획). **2026-09-28 — 10-13 주 착수 전에 D5 행을 추가한다**(10-12까지는 슬롯 없음).

## 배운 것

- [Feature-Sliced Design](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/React/Feature-Sliced%20Design.md) — 계층·슬라이스·세그먼트, `index.ts` public API, Next.js 루트 `app/`(URL·예약 파일)과 `src/`(제품 구현)의 경계, `app/api`와 `src/**/api`의 차이
- [JavaScript 모듈 시스템](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/%EC%96%B8%EC%96%B4/JavaScript%20%EB%AA%A8%EB%93%88%20%EC%8B%9C%EC%8A%A4%ED%85%9C.md) — `export { default as X } from` 재수출, JS 모듈엔 접근제어자가 없어 슬라이스 경계는 언어가 아니라 린터가 지킨다
- [컴퓨터의 수 표현](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/CS/%EC%BB%B4%ED%93%A8%ED%84%B0%EA%B5%AC%EC%A1%B0/%EC%BB%B4%ED%93%A8%ED%84%B0%EC%9D%98%20%EC%88%98%20%ED%91%9C%ED%98%84.md) — 이진법에서 십진 소수는 무한소수라 float은 돈에 못 쓴다. 단위를 쪼개 정수로 다루고(고정소수점), 스케일링은 문자열로 한다
- [JavaScript 기초 문법](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/%EC%96%B8%EC%96%B4/JavaScript%20%EA%B8%B0%EC%B4%88%20%EB%AC%B8%EB%B2%95.md) — 객체 리터럴과 구조 분해가 JS의 절반, truthy/falsy, **`bigint` 나눗셈은 항상 내림이라 올림은 직접 만든다**
- [고차함수와 배열 메서드](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/%EC%96%B8%EC%96%B4/%EA%B3%A0%EC%B0%A8%ED%95%A8%EC%88%98%EC%99%80%20%EB%B0%B0%EC%97%B4%20%EB%A9%94%EC%84%9C%EB%93%9C.md) — `while`로 0을 채우다 버그 둘을 만든 자리가 `padEnd` 한 번으로 사라졌다: 짧아서가 아니라 부품이 적어서 (D4 블록 5a)
- [JavaScript 런타임](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/%EC%96%B8%EC%96%B4/JavaScript%20%EB%9F%B0%ED%83%80%EC%9E%84.md) — Node는 V8 + OS 기능이라 서버가 될 수 있고, 그래서 API 키를 서버에만 둔다
- [TypeScript 타입 시스템](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/%EC%96%B8%EC%96%B4/TypeScript%20%ED%83%80%EC%9E%85%20%EC%8B%9C%EC%8A%A4%ED%85%9C.md) — 타입은 컴파일하면 사라지므로 `res.ok` 검사를 손으로 넣는다
- [Next.js 서버와 캐싱](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/Next.js/Next.js%20%EC%84%9C%EB%B2%84%EC%99%80%20%EC%BA%90%EC%8B%B1.md) — 서버 컴포넌트·Route Handler, `app/api` HTTP 출입구와 `src/**/api` 내부 함수, `revalidate`, 환경변수 인라인, proxy의 요청 단위 실행
- [포트와 localhost](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/CS/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/%ED%8F%AC%ED%8A%B8%EC%99%80%20localhost.md) — 서버는 포트를 잡은 프로그램일 뿐, `localhost`는 루프백이고 `*:3000`은 같은 와이파이에 열려 있다
- [외부 API 데이터 모델링](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/React/%EC%99%B8%EB%B6%80%20API%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20%EB%AA%A8%EB%8D%B8%EB%A7%81.md) — 응답 타입을 두 겹으로, 숫자로 들고 다니다 그릴 때만 포맷, 코인 이름은 업비트에서 받는다
- [키와 스키마 설계](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/CS/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4/%ED%82%A4%EC%99%80%20%EC%8A%A4%ED%82%A4%EB%A7%88%20%EC%84%A4%EA%B3%84.md) — 스키마·DTO·Model의 경계, PK·FK·복합 PK, Drizzle `numeric`/`bigint`, 파생 랭킹 스냅샷
- [Route Handler와 내부 API](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/Next.js/Route%20Handler%EC%99%80%20%EB%82%B4%EB%B6%80%20API.md) — 같은 로직을 함수(`src/**/api`)와 HTTP(`app/api/**/route.ts`) 두 가지로 노출한다. 서버 컴포넌트는 함수를 직접 부르고 브라우저는 HTTP로 부탁한다
- [proxy 미들웨어와 matcher](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/Next.js/proxy%20%EB%AF%B8%EB%93%A4%EC%9B%A8%EC%96%B4%EC%99%80%20matcher.md) — proxy(구 middleware)는 페이지마다가 아니라 요청마다 돈다. matcher 빈 배열은 「아무 데서도 안 돈다」이고 빌드 출력의 `ƒ Proxy (Middleware)` 줄이 실행 증거다
- [빌드 타임과 런타임](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/Next.js/%EB%B9%8C%EB%93%9C%20%ED%83%80%EC%9E%84%EA%B3%BC%20%EB%9F%B0%ED%83%80%EC%9E%84.md) — 빌드 때 번들에 새겨지는 것(타입·Tailwind 클래스·`NEXT_PUBLIC_`)과 실행 때 결정되는 것을 나누면 환경변수 유출과 클래스명 조립 금지가 같은 이야기다
- [bigint와 정수 연산](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/JS/%EC%96%B8%EC%96%B4/bigint%EC%99%80%20%EC%A0%95%EC%88%98%20%EC%97%B0%EC%82%B0.md) — `bigint`는 `number`와 섞이지 않고 나눗셈은 항상 내림이다. 변환은 경계에서 한 번만 한다
- [TCP/IP 4계층과 캡슐화](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/CS/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/TCP-IP%204%EA%B3%84%EC%B8%B5%EA%B3%BC%20%EC%BA%A1%EC%8A%90%ED%99%94.md) — 계층의 차이는 「얼마나 멀리 책임지나」다. 그래서 IP는 끝까지 안 바뀌고 MAC은 홉마다 바뀐다
- [계층화의 설계 근거](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/CS/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/%EA%B3%84%EC%B8%B5%ED%99%94%EC%9D%98%20%EC%84%A4%EA%B3%84%20%EA%B7%BC%EA%B1%B0.md) — 링크 L가지 × 앱 A가지를 L+A로 줄이려고 나눴고, 대가는 헤더 오버헤드·기능 중복·정보 은닉이다

## 다음에 정할 것

- 누구의 어떤 거래 기록 문제를 해결하는가?
- 기존 메모장, 스프레드시트, 거래소 기록과 무엇이 다른가?
- 첫 사용자가 반드시 완료해야 하는 하나의 핵심 흐름은 무엇인가?
- mock data 다음으로 어떤 데이터를 실제 저장할 것인가?

## 코드 저장소

- 스택: Next.js 16, React 19, Tailwind CSS 4
- GitHub: `dbsghdz1/MyCryptoDiary`

## 관련 자료

- 보존 원본: [MyCryptoDiary Day 1–4 학습 노트](../../../_wiki/Sources/2026/07/2026-07-24-mycryptodiary-day1-4-learning-notes.md) — Day 1–4 구현 목록과 개념 정리
- 다음 학습 예고(노트 기준): 현물/선물 segmented control, 코인 목록 컴포넌트, 실시간 가격 상태 업데이트, Currency Detail 동적 페이지
- **학습 계획**: [학습 계획 (2026-08-28 전면 개정)](%ED%95%99%EC%8A%B5%20%EA%B3%84%ED%9A%8D%202026-08-28.md) — 9요소 진단(병목 = 연습·습관·에너지), 블록 루프, D3~D12 일정. **세션 시작 시 이 문서를 먼저 읽는다**
- 작업 기록: [실시간 데이터 UI 계획](%EC%8B%A4%EC%8B%9C%EA%B0%84%20%EB%8D%B0%EC%9D%B4%ED%84%B0%20UI%20%EA%B3%84%ED%9A%8D%202026-08-14.md) · [보상형 광고 조사](%EB%B3%B4%EC%83%81%ED%98%95%20%EA%B4%91%EA%B3%A0%20%EC%A1%B0%EC%82%AC%202026-08-15.md) · [모의투자 전환 D1](%EB%AA%A8%EC%9D%98%ED%88%AC%EC%9E%90%20%EC%A0%84%ED%99%98%20D1%202026-08-16.md) · [Neon·Drizzle 스키마 D2](Neon%20Drizzle%20%EC%8A%A4%ED%82%A4%EB%A7%88%202026-08-28.md) · [Clerk 인증 D3](Clerk%20%EC%9D%B8%EC%A6%9D%20D3%202026-08-29.md) · [거래 엔진 D4](%EA%B1%B0%EB%9E%98%20%EC%97%94%EC%A7%84%20D4%202026-09-02.md) · [D4 회고](D4%20%ED%9A%8C%EA%B3%A0%202026-09-04.md)
- 학습 위키: [React·TypeScript로 제품 만들기](../../../_wiki/React%20TypeScript%20%EC%A0%9C%ED%92%88%20%EA%B0%9C%EB%B0%9C.md)
