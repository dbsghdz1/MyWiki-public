---
type: study
area: 백엔드
audience: me
status: active
created: 2026-08-18
updated: 2026-09-02
projects:
  - "소프트웨어마에스트로"
---

# JPA와 Spring Data

DB 행을 객체로 바꾸는 층에서 생기는 문제들. **매핑 실패의 폭발 반경**과 **예외가 어디까지 번역되는가**가 지금까지 배운 두 축이다.

## 핵심 정리

- **매핑 단계 예외는 행 단위로 격리되지 않는다.** 잘못된 행 하나가 같은 쿼리로 읽힌 멀쩡한 행까지 통째로 죽인다. 그래서 "잘못된 데이터가 들어올 수 있는가"보다 **"들어오면 몇 개가 죽는가"**를 먼저 본다.
- **벌크 UPDATE(`@Modifying` + JPQL)는 엔티티를 지나가지 않는다.** DB로 바로 나가는 SQL이라 `@PreUpdate` 같은 생명주기 콜백도, 영속성 컨텍스트도 안 태운다 — 그래서 ① `updated_at` 같은 자동 채움 컬럼을 쿼리 안에 손으로 써야 하고 ② 같은 트랜잭션에서 앞서 읽어 둔 엔티티가 **낡은 값을 든 채 남는다**(`clearAutomatically = true`가 그걸 비운다).
- **그럼에도 벌크 UPDATE를 쓰는 이유는 "읽고 판단하고 쓰기" 사이에 잠금이 없기 때문이다.** 조건을 `WHERE`에 실으면 DB가 판정하고 **영향 행 수**가 승패를 알려준다 — 동시 요청 둘이 똑같이 "유효하다"를 읽는 창이 사라진다.
- **Spring의 `DataAccessException` 계층은 완전하지 않다.** 번역하지 못한 벤더 오류는 `UncategorizedSQLException`으로 올라온다. 예외 타입으로 분기하거나 단언할 때 이걸 전제해야 한다.

## 기록

### 2026-08-18 — `@Enumerated(STRING)`의 폭발 반경은 행이 아니라 쿼리 전체다

- 맥락: 보험찾개냥 SSH-293 필요서류 추천 API. `document_rule`의 `claim_type` 등을 `@Enumerated(EnumType.STRING)`으로 읽는 첫 코드였고, AI 리뷰가 "SQL로 새 값 하나만 넣어도 그 보험사 조회가 500"이라고 지적했다.
- 배운 것:
  - enum에 없는 문자열을 만나면 Hibernate는 **결과를 엔티티로 만드는 단계**에서 던진다. 조회는 이미 성공했고 변환에서 죽는 것이라, **그 행만 빠지는 게 아니라 같은 조회 전체가 실패한다.** 이 API는 보험사의 룰 여러 개를 한 번에 읽으므로 폭발 반경이 그 보험사 전부였다.
  - 방어 위치가 두 갈래다. **읽을 때** 모르는 값을 흘려보내면(컨버터로 null 처리 등) 500은 막지만, 잘못 넣은 행이 **조용히 무시되어** 운영자가 자기가 넣은 데이터가 왜 안 보이는지 알 수 없다. **쓸 때** 막으면(DB `CHECK`) 넣는 순간 에러가 나서 원인이 그 자리에서 드러난다.
  - **틀린 데이터가 실패해야 할 시점은 "쓸 때"다.** 읽기 방어는 사고를 미루고 사람에게서 원인을 숨긴다. 대신 값을 늘릴 때 코드와 마이그레이션을 같이 고쳐야 하는 비용을 받아들인다.
- 근거: 커밋 `0cdeacf`(V3 CHECK 제약 + `DocumentRuleEnumConstraintTest`), PR #22. DB 쪽 문법은 [[학습/공부/CS/데이터베이스|데이터베이스]].

### 2026-08-18 — Spring이 번역하지 못하는 SQL 예외가 있다

- 맥락: 위 CHECK 제약이 실제로 막는지 테스트를 짜면서, `DataIntegrityViolationException`을 기대했는데 실패했다.
- 배운 것:
  - Spring은 벤더별 `SQLException`을 `DataAccessException` 계층으로 번역해 주지만 **전부는 아니다.** MySQL의 CHECK 위반(오류 3819)은 매핑이 없어 `UncategorizedSQLException`(= "분류하지 못했다")으로 온다. UNIQUE 위반이 `DuplicateKeyException`으로 깔끔하게 오는 것과 대조적이다.
  - 그래서 제약 위반을 예외 타입으로 구분하려 할 때 **DB마다·제약 종류마다 다르다는 것을 전제**해야 한다. 상위 타입(`DataAccessException`)으로 받고 원인 메시지를 보는 쪽이 안전하다 — 대신 그러면 단언이 헐거워지므로 [[학습/공부/개발방법/테스트 코드|테스트 코드]]의 주의가 함께 필요하다.
- 근거: `DocumentRuleEnumConstraintTest`. 첫 시도가 `AssertionFailedError`로 떨어지고 원인이 `UncategorizedSQLException`이었던 실행 로그.

### 2026-09-02 — 경합을 조건부 UPDATE로 넘기면 스레드 없이 테스트된다

- 맥락: 보험찾개냥 SSH-444 로그아웃·refresh 저장소. 같은 refresh 토큰으로 회전이 동시에 들어오면 둘 다 새 토큰을 받아서는 안 된다
- 배운 것:
  - **"찾아서 유효하면 폐기한다"를 조회 + 저장으로 쓰면 그 사이에 아무 잠금이 없다.** 두 요청이 똑같이 `revoked_at IS NULL`을 읽고 둘 다 통과한다
  - `UPDATE … SET revoked_at = :now WHERE token_hash = :h AND revoked_at IS NULL AND expires_at > :now` 한 방으로 바꾸면, **행 잠금이 DB 안에서 직렬화**되어 영향 행 수가 1인 쪽만 승자다
  - **부수 효과가 테스트를 쉽게 만든다** — 같은 입력으로 두 번 부르면 첫 번째가 남긴 `revoked_at`이 두 번째를 막으므로, **스레드를 띄우지 않고도** 패자 경로가 실제로 돈다. 동시성 테스트가 타이밍에 기대지 않는다
  - 대가: 콜백을 안 태우므로 `updated_at`을 쿼리에 직접 넣어야 하고, 0이 돌아왔을 때 "없음"인지 "이미 폐기"인지는 **그때만** 따로 조회해 가른다(승자 경로에는 조회가 없다)
- 근거: `RefreshTokensIntegrationTest` 10건(실 MySQL) — 「같은 토큰으로 두 번 회전하면 뒤엣것이 진다」·「진 쪽의 새 토큰은 저장되지 않는다」. bootRun 실호출로도 401/204를 확인했다. PR #104

## 참고 자료

- [Spring Data JPA Reference — Query Methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html) — *"As the `EntityManager` might contain outdated entities after the execution of the modifying query, we do not automatically clear it … you can set the `@Modifying` annotation's `clearAutomatically` attribute to `true`."* (2026-09-02 확인)
- [Spring Framework Reference — DAO Support](https://docs.spring.io/spring-framework/reference/data-access/dao.html) — "Spring provides a convenient translation from technology-specific exceptions, such as SQLException to its own exception class hierarchy, which has DataAccessException as the root exception" (2026-08-18 확인)
