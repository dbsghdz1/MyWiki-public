---
type: study
area: 도구
audience: ai
status: active
created: 2026-10-17
updated: 2026-10-17
projects:
  - "소프트웨어마에스트로"
---

# Spring Boot 4와 Flyway

Boot 3 지식으로 Spring Boot 4 서버에 의존성을 넣으면 빌드는 되는데 런타임에만 조용히 안 도는 것들이 있고, 병렬 브랜치는 같은 Flyway 마이그레이션 번호를 잡는다. 둘 다 에러가 늦게 난다. 2026-10-17 기준 결론이고 Spring Boot 4.1.0, Kotlin 2.3.21에서 확인했다.

## 핵심 정리

- **Jackson Kotlin 모듈 좌표가 바뀌었다.** Boot 4는 Jackson 3이라 groupId가 `tools.jackson`이다. `com.fasterxml.jackson.module:jackson-module-kotlin`이 아니라 `tools.jackson.module:jackson-module-kotlin`을 쓴다.
- **`flyway-core`만 넣으면 Flyway가 조용히 실행되지 않는다.** Boot 4부터 자동설정이 `org.springframework.boot:spring-boot-starter-flyway`로 분리됐다. 부팅 로그에 Flyway 줄이 없으면 이 문제다.
- 위 둘은 빌드가 성공하고 런타임에만 조용히 죽어서 알아채기 어렵다. 관리 좌표가 의심되면 `./gradlew dependencyManagement`로 BOM을 확인한다. Boot 3 시절 좌표를 그대로 쓰지 않는다.
- **병렬 브랜치는 같은 Flyway 번호를 고른다.** 브랜치를 딸 때 `main`의 다음 번호를 잡기 때문이다. CI는 못 잡는다. 각 브랜치에서는 번호가 유일해 전부 초록이고, 머지된 뒤 `main`에서 처음 터진다.
  - 이미 적용된 버전이면 Flyway가 새 파일을 아예 안 돌려서 테이블이 조용히 안 생기거나 `Migration checksum mismatch`로 부팅이 실패한다.
  - 같은 번호 파일이 둘 다 `main`에 있으면 다음 배포가 `Found more than one migration with version N`으로 부팅에서 죽는다.
- **마이그레이션을 추가하면 머지 직전에 `main`의 최신 번호를 다시 확인한다.** 열린 PR과 브랜치도 함께 훑는다: `git ls-tree -r origin/main --name-only | grep db/migration`, 열린 브랜치마다 `git ls-tree --name-only <브랜치> Server/src/main/resources/db/migration/`. 번호를 옮기면 그 파일명을 적은 주석도 같이 고친다.
- 자주 걸리기 시작하면 타임스탬프 버전(`V20260822_1430__`)이 대안이지만, 이 규모에서는 머지 직전 사람 확인으로 충분했다.

## 기록

### 2026-10-17 — Spring Boot 4 의존성 좌표와 Flyway 스타터 (원 기록 2026-08-05)

- 맥락: 소프트웨어마에스트로 서버 스캐폴딩(SSH-241, PR #6)에서 빌드 의존성을 넣다가 알게 됐다. Claude 메모리 이관(원 기록 2026-08-05). 같은 내용이 개발 기록 2026-08-06에 인라인으로 있다.
- 배운 것: 위 핵심 정리의 Jackson 좌표와 `spring-boot-starter-flyway` 두 항목.
- 근거: SSH-241의 검증(compose up, build와 test 통과, bootRun 1.4초 부팅, `flyway_schema_history` 생성 확인).

### 2026-10-17 — 병렬 브랜치가 같은 Flyway 번호를 잡았다 (원 기록 2026-08-22)

- 맥락: 같은 프로젝트에서 SSH-293(닫힌 PR #22)과 SSH-329(PR #60)가 둘 다 `V3`를 잡았다. Claude 메모리 이관(원 기록 2026-08-22).
- 배운 것: 위 핵심 정리의 번호 충돌 항목. 이번에는 #22가 닫혀 `main`에 V3가 들어간 적이 없어 무사했다. 닫힌 브랜치 쪽을 `V4__add_document_rule_enum_check.sql`로 미리 옮겨 뒀지만 이식 시점에 또 밀릴 수 있어 그때 다시 확인하기로 했고, 실제로 2026-08-26 이식 때는 V4부터 V7이 이미 다른 마이그레이션(V4는 테이블명 복수형 통일 `V4__rename_tables_to_plural.sql`)에 쓰여 `V8__add_document_rule_enum_check.sql`이 됐다. 파일명을 참조한 주석(`DocumentRule.kt`)도 같이 고쳤다.
- 근거: 레포 `.claude/rules/server-convention.md` 「마이그레이션 번호는 머지 직전에 확정한다」, 개발 기록 2026-08-26의 이식, 개발 기록 2026-09-11의 V15(열린 PR 다섯의 `db/migration`을 `git ls-tree`로 확인해 번호 확정).
