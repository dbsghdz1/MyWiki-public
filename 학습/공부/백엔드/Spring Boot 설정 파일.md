---
type: study
area: 백엔드
audience: me
status: active
created: 2026-09-11
updated: 2026-09-11
projects:
  - "소프트웨어 마에스트로"
---

# Spring Boot 설정 파일

한 줄 요약 — `application.yaml`은 코드가 아니라 **값**을 담는 파일이다. 빌드 때 JAR 안으로 복사되고 부팅 때 Spring이 **이름으로 찾아** 읽는다. 환경마다 다른 값은 프로필 파일과 환경변수가 **위에서 덮어쓴다**.

## 핵심 정리

**경로가 곧 의미다** — `Server/src/main/resources/application.yaml`

| 조각 | 뜻 |
|---|---|
| `Server/` | 모노레포 안의 서버 모듈 — 우리 레포만의 이름 |
| `src/main/` | 제품 코드 (↔ `src/test/`는 테스트에서만 쓰인다) |
| `kotlin/` | 컴파일되는 코드 |
| `resources/` | **컴파일 안 되는 파일** — 빌드 때 그대로 클래스패스로 복사돼 JAR에 들어간다 (Gradle `processResources`) |
| `application.yaml` | Spring Boot가 **파일 이름으로** 자동으로 찾아 읽는 설정 |

Maven·Gradle 표준 레이아웃이라 Java·Kotlin 프로젝트면 어디서나 같은 자리다.

**값이 이기는 순서** (아래로 갈수록 우선)

1. `application.yaml` — 모든 환경 공통 기본값
2. `application-{프로필}.yaml` — `SPRING_PROFILES_ACTIVE=dev`면 `application-dev.yaml`이 1을 덮는다. **차이만 적는다**
3. OS 환경변수
4. 명령줄 인자 (`--gemini.model=...`)

**환경변수는 두 방식으로 끼어든다**
- **끌어오기** — 파일 안 `${GEMINI_API_KEY:}` 자리표시자. `:` 뒤가 기본값이고, 여기선 비어 있으니 없으면 빈 문자열
- **덮어쓰기** — 자리표시자가 없어도 환경변수 `GEMINI_MODEL`이 `gemini.model`을 덮는다. 점→밑줄, 대문자로 바꾼 이름이 같은 키로 취급된다(relaxed binding)

**코드는 prefix로 받는다** — `@ConfigurationProperties(prefix = "gemini")`가 붙은 `GeminiProperties(model, apiKey, …)`가 `gemini:` 아래 값을 필드에 채운다. 그래서 코드에는 모델명 문자열이 없다.

**파일에 적을까, 환경변수로 뺄까** — 기준은 *"이 레포를 지금 공개해도 되나"*(12-Factor). 모델명·타임아웃·공개 issuer URL은 파일에, 키·DB 비밀번호·버킷 이름은 `${...}`로 빼서 배포 환경이 주입한다.

> **면접 30초** — "`resources`는 컴파일되지 않는 파일이 클래스패스로 복사되는 자리이고, `application.yaml`은 Spring Boot가 부팅 때 자동으로 읽는 설정입니다. 공통값은 여기, 환경별 차이는 `application-{profile}.yaml`로 덮고, 비밀은 `${ENV}` 자리표시자로 환경변수에서 받습니다. 우선순위는 명령줄 > 환경변수 > 프로필 파일 > 기본 파일이고, 코드는 `@ConfigurationProperties`로 묶어 받습니다."

## 기록

### 2026-09-11 — application.yaml은 뭘 의미하나
- 맥락: 소프트웨어 마에스트로 랜딩(Landing_Page)의 `GEMINI_MODEL`을 `gemini-3.1-flash-lite`로 바꾸다가, 본 서비스 서버는 그 변수를 안 쓰고 `application.yaml`에 모델을 적는다는 걸 보고 물었다
- 배운 것:
  - **같은 「Gemini 모델 설정」이 두 레포에서 자리가 다르다.** 랜딩(Vite·Vercel 함수)은 `.env`의 `GEMINI_MODEL`을 `process.env`로 읽고, 서버(Spring Boot)는 `application.yaml`의 `gemini.model`을 `GeminiProperties`로 받는다 — 랜딩 `.env`를 고쳐도 서버 모델은 그대로다
  - 서버에서 **모델은 리터럴**(`gemini-3.6-flash`), **키는 자리표시자**(`${GEMINI_API_KEY:}`)다. compose도 `GEMINI_API_KEY`만 넘긴다 — 모델은 비밀이 아니라 코드와 함께 리뷰받을 값이라서
  - 프로필 파일은 **차이만** 적는다 — `application-local.yaml`은 localhost MySQL, `application-dev.yaml`은 DB 접속·S3뿐이고 나머지는 기본 파일에서 온다
- 근거: `Server/src/main/resources/application.yaml` `gemini:` 블록 · `GeminiProperties.kt` `@ConfigurationProperties(prefix = "gemini")` · `deploy/docker-compose.yaml` `SPRING_PROFILES_ACTIVE: dev` · 랜딩 `Project/api/analyze-receipt.ts` `process.env.GEMINI_MODEL`

## 더 알아보면 좋은 것
- 급할 때 배포 환경에 `GEMINI_MODEL`만 넣어도 서버 모델이 덮인다(문서 기준, 우리 서버에서 실측 안 함). 대신 레포에 적힌 값과 실제 값이 어긋나 추적이 어려워진다 — 정석은 파일을 고쳐 PR
- `@Value("${...}")`와 `@ConfigurationProperties`의 차이 — 하나씩 꽂기 vs 묶음으로 받고 부팅 때 검증

## 참고 자료
- [Spring Boot — Externalized Configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html) — 값 우선순위 목록, 프로필 파일, `${name:default}`, 환경변수 이름 규칙의 원전 (2026-09-11 확인)
- [Maven — Introduction to the Standard Directory Layout](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html) — `src/main/resources`는 "target 클래스패스로 복사되는 구조" (2026-09-11 확인)
- [Gradle — The Java Plugin](https://docs.gradle.org/current/userguide/java_plugin.html) — 프로젝트 레이아웃과 `processResources` 태스크 (2026-09-11 확인)
- [The Twelve-Factor App — III. Config](https://12factor.net/config) — "지금 당장 오픈소스로 공개해도 자격증명이 안 새는가"라는 판정 기준 (2026-09-11 확인)
