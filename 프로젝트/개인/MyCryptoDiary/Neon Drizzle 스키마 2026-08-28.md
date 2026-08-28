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
