---
type: study
area: 언어·프레임워크
status: active
created: 2026-08-18
updated: 2026-08-18
projects:
  - "소프트웨어마에스트로"
---

# JPA와 Spring Data

DB 행을 객체로 바꾸는 층에서 생기는 문제들. **매핑 실패의 폭발 반경**과 **예외가 어디까지 번역되는가**가 지금까지 배운 두 축이다.

## 핵심 정리

- **매핑 단계 예외는 행 단위로 격리되지 않는다.** 잘못된 행 하나가 같은 쿼리로 읽힌 멀쩡한 행까지 통째로 죽인다. 그래서 "잘못된 데이터가 들어올 수 있는가"보다 **"들어오면 몇 개가 죽는가"**를 먼저 본다.
- **Spring의 `DataAccessException` 계층은 완전하지 않다.** 번역하지 못한 벤더 오류는 `UncategorizedSQLException`으로 올라온다. 예외 타입으로 분기하거나 단언할 때 이걸 전제해야 한다.

## 기록

### 2026-08-18 — `@Enumerated(STRING)`의 폭발 반경은 행이 아니라 쿼리 전체다

- 맥락: 보험찾개냥 SSH-293 필요서류 추천 API. `document_rule`의 `claim_type` 등을 `@Enumerated(EnumType.STRING)`으로 읽는 첫 코드였고, AI 리뷰가 "SQL로 새 값 하나만 넣어도 그 보험사 조회가 500"이라고 지적했다.
- 배운 것:
  - enum에 없는 문자열을 만나면 Hibernate는 **결과를 엔티티로 만드는 단계**에서 던진다. 조회는 이미 성공했고 변환에서 죽는 것이라, **그 행만 빠지는 게 아니라 같은 조회 전체가 실패한다.** 이 API는 보험사의 룰 여러 개를 한 번에 읽으므로 폭발 반경이 그 보험사 전부였다.
  - 방어 위치가 두 갈래다. **읽을 때** 모르는 값을 흘려보내면(컨버터로 null 처리 등) 500은 막지만, 잘못 넣은 행이 **조용히 무시되어** 운영자가 자기가 넣은 데이터가 왜 안 보이는지 알 수 없다. **쓸 때** 막으면(DB `CHECK`) 넣는 순간 에러가 나서 원인이 그 자리에서 드러난다.
  - **틀린 데이터가 실패해야 할 시점은 "쓸 때"다.** 읽기 방어는 사고를 미루고 사람에게서 원인을 숨긴다. 대신 값을 늘릴 때 코드와 마이그레이션을 같이 고쳐야 하는 비용을 받아들인다.
- 근거: 커밋 `0cdeacf`(V3 CHECK 제약 + `DocumentRuleEnumConstraintTest`), PR #22. DB 쪽 문법은 [[공부/CS/데이터베이스|데이터베이스]].

### 2026-08-18 — Spring이 번역하지 못하는 SQL 예외가 있다

- 맥락: 위 CHECK 제약이 실제로 막는지 테스트를 짜면서, `DataIntegrityViolationException`을 기대했는데 실패했다.
- 배운 것:
  - Spring은 벤더별 `SQLException`을 `DataAccessException` 계층으로 번역해 주지만 **전부는 아니다.** MySQL의 CHECK 위반(오류 3819)은 매핑이 없어 `UncategorizedSQLException`(= "분류하지 못했다")으로 온다. UNIQUE 위반이 `DuplicateKeyException`으로 깔끔하게 오는 것과 대조적이다.
  - 그래서 제약 위반을 예외 타입으로 구분하려 할 때 **DB마다·제약 종류마다 다르다는 것을 전제**해야 한다. 상위 타입(`DataAccessException`)으로 받고 원인 메시지를 보는 쪽이 안전하다 — 대신 그러면 단언이 헐거워지므로 [[공부/개발방법/테스트 코드|테스트 코드]]의 주의가 함께 필요하다.
- 근거: `DocumentRuleEnumConstraintTest`. 첫 시도가 `AssertionFailedError`로 떨어지고 원인이 `UncategorizedSQLException`이었던 실행 로그.

## 참고 자료

- [Spring Framework Reference — DAO Support](https://docs.spring.io/spring-framework/reference/data-access/dao.html) — "Spring provides a convenient translation from technology-specific exceptions, such as SQLException to its own exception class hierarchy, which has DataAccessException as the root exception" (2026-08-18 확인)
