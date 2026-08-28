---
type: project
status: active
aliases:
  - MyCryptoDiary D2
  - Neon Drizzle 스키마
created: 2026-08-28
updated: 2026-08-28
related_wiki:
  - "[[공부/CS/데이터베이스]]"
---

# MyCryptoDiary D2 — Neon + Drizzle 스키마 (2026-08-18, 26~28)

[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] 모의투자 전환의 두 번째 작업. 로컬 목 데이터 다음 단계로 Neon PostgreSQL과 Drizzle을 연결하고, 거래·랭킹의 원본 및 파생 데이터를 담는 스키마를 실제 원격 DB에 반영했다.

## 한 것

- `@neondatabase/serverless` + Drizzle ORM/Kit, `dotenv-cli`, `tsx` 구성
- `DATABASE_URL` 런타임 검증과 `server-only` 경계를 둔 lazy singleton `getDB`
- `users` · `accounts` · `holdings` · `orders` · `ad_rewards` · `rank_snapshots` 6테이블
- FK 5개와 cascade, `holdings(user_id, market)` 복합 PK, `order_side` enum, `provider_txn_id` UNIQUE
- 원화 확정액은 PostgreSQL `bigint` ↔ TypeScript `bigint`, 코인 수량·단가·수익률은 `numeric` ↔ 문자열로 모델링
- 가짜 사용자·계좌·랭킹 5개를 넣는 재실행 가능한 seed

커밋은 `1cfe07c`(도구·설정), `8eadff3`(client·schema), `f7ccda4`(seed). PR [#5](https://github.com/dbsghdz1/MyCryptoDiary/pull/5).

## 결정

- `accounts.user_id`는 PK이자 `users.clerk_user_id` FK다. 사용자당 계좌 하나를 DB가 강제한다.
- `holdings`는 같은 사용자가 여러 시장을 보유할 수 있으므로 `user_id` 하나가 아니라 `(user_id, market)` 조합을 PK로 둔다.
- `orders.id`는 같은 사용자가 같은 코인을 반복 거래할 수 있어 DB 생성 UUID를 쓴다. Clerk ID처럼 외부 시스템이 소유한 값에는 임의 UUID 기본값을 만들지 않는다.
- `realized_pnl`은 매수의 "해당 없음"을 NULL, 본전 매도의 0과 구분한다.
- `rank_snapshots`는 원본이 아니라 `accounts + holdings + 현재 시세`에서 다시 만들 수 있는 최신 랭킹 캐시다. 사용자당 최신 행 하나를 upsert하는 구조라 `user_id`가 PK다.
- BUY의 reason 필수 CHECK와 양수 범위 CHECK, 조회 인덱스는 실제 쓰기·조회가 생기는 D4·D6·D8에서 쿼리와 함께 확정한다.

## 검증

- TypeScript, ESLint, Steiger 통과
- DB 반영 전 생성 SQL에서 6테이블·FK 5개·enum·복합 PK·UNIQUE와 삭제 구문 없음 확인
- `drizzle-kit push`로 Neon 반영
- seed 최초 실행과 재실행 모두 성공; `users`·`accounts`·`rank_snapshots` 각 5행을 Drizzle Studio에서 확인

## 배운 것

- [[공부/CS/데이터베이스|데이터베이스]] — 스키마와 DTO·Model의 차이, PK·FK·복합 PK, 공유 기본키, `numeric`과 `bigint`, 파생 스냅샷

## 다음

D3 Clerk 인증: `proxy.ts`, 로그인·회원가입, 인증 사용자 ID와 `users.clerk_user_id` 연결, 최초 진입 시 1,000만원 계좌 lazy create.

## PR #5 리뷰 결과 (2026-08-28)

`mattpocock-skills:code-review` 2축(Standards·Spec), fixed point `585b51c`. **작업 기록의 「결정」 절이 이미 답한 지적은 제외**했다 — `rank_snapshots` PK=user_id(최신 캐시라 의도적), CHECK 제약·조회 인덱스 연기(D4·D6·D8에서 쿼리와 함께 확정), 단가·수익률의 `numeric` 채택이 그것이다.

**A. `neon-http` 드라이버는 트랜잭션을 지원하지 않는다 (D4 착수 전 필수)**
`client.ts`가 쓰는 `drizzle-orm/neon-http`는 `session.js:152`에서 `throw new Error("No transactions support in neon-http driver")`. **직접 확인함.** 계획의 핵심 학습 포인트 2가 "현금 차감 + 보유 증가 + 주문 기록이 원자적"이고 D4 DoD가 여기 걸려 있는데, `db.transaction()`을 부르는 순간 런타임 에러가 난다. HTTP는 인터랙티브 트랜잭션을 못 하는 게 원인이라 설정으로 못 푼다 — `drizzle-orm/neon-serverless`(WebSocket + `Pool`)로 교체해야 한다(이미 `node_modules`에 있음). D2 DoD는 읽기/쓰기까지라 통과했지만 **D4 첫 줄에서 막히는 구조**다.

**B. `shared/api/index.ts`가 D1 결정을 반전시켰다**
현재 이 배럴이 `getDB`와 upbit 3함수·3타입을 함께 내보낸다. D1 결정("public API는 `shared/api/upbit/index.ts` 하나. 전부 모으면 `db`만 쓰려던 파일이 upbit까지 끌고 온다")의 정반대이고, `client.ts` 첫 줄 `import 'server-only'` 때문에 피해 방향이 하나 더 생겼다 — **이 배럴을 타면 server-only가 딸려온다.** 현재 `'use client'` 파일이 0개(직접 확인)라 빌드가 통과할 뿐이고, `getCoins` 체인(`LiveMarketCard`·`WatchlistCard`)에 클라이언트 컴포넌트가 하나 붙으면 그때 터진다. 백엔드별로 문을 나누면(`@/shared/api/upbit`, `@/shared/api/db`) 둘 다 해소된다.

**C. seed의 `rank_snapshots`가 `accounts`와 모순된다**
5계좌 전부 현금·원금 1,000만이고 `holdings`가 비어 있어 실제 수익률은 전원 0%인데, 스냅샷에는 +20%~−15%가 박혀 있다(직접 조회 확인). D8 랭킹 잡이 처음 돌면 화면이 전부 0%로 평탄해진다. `onConflictDoNothing`이라 재시드해도 갱신되지 않는다. 랭킹 UI를 미리 보려는 의도였다면 `holdings`까지 함께 시드해야 앞뒤가 맞는다.

**D. 반올림 규칙이 어디에도 없다**
`price`·`quantity`는 `numeric(30,8)`인데 `amount`·`fee`는 `bigint`다. `price × quantity`는 소수인데 `amount`는 정수라 **절삭/반올림 방향이 정의돼 있지 않다.** D5가 "소수점 절삭 방향, 수수료 반올림(원 단위 정수 유지)"을 테스트하라고 하므로, 명세가 없으면 테스트가 구현을 베끼게 된다(계획이 경고한 바로 그 실패). 부수 효과로 0.05% 수수료가 소액 주문에서 `0`으로 내려 무료 거래가 된다. D4에서 계산 함수를 만들 때 규칙을 먼저 글로 정하고 시작할 것.

**E. 마이그레이션 산출물이 없다**
`drizzle.config.ts`에 `out: './drizzle'`을 두고도 `push`만 써서 `drizzle/`이 비어 있다. D2 학습 포인트에 "마이그레이션"이 있는데 이력이 남지 않았다. 혼자 개발하는 동안 `push`로 가는 것은 합리적이나, 배포 후에는 스키마 변경 이력이 필요해지므로 **언제 `generate`+`migrate`로 넘어갈지**를 정해 두는 편이 좋다.

**판단 필요 (연기 가능)**: `DATABASE_URL` 가드 + `neon()` + `drizzle()`가 `client.ts`·`seed.ts`·`drizzle.config.ts` 3곳에 복제돼 있다. `seed.ts`가 `getDB()`를 못 쓰는 이유는 `server-only`가 tsx에서 깨지기 때문이라 근거는 있으나, `server-only` 없는 `createDB()`를 분리하고 배럴에서만 막는 선택지가 있다. / `orders`가 체결 컬럼과 일기 컬럼(`reason`·`review`·`reviewedAt`)을 한 테이블에 담아 일기 편집 사유로 `orders`가 바뀐다 — 다만 이건 "조인 없이 한 행에서 조회"라는 8/15 설계 의도의 결과라 트레이드오프가 이미 선택된 것이다.
