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

# Gemini API

서버에서 Gemini API를 부르며 실측으로 확인한 함정이다. 키 형식과 모델 가용성은 계정과 시점에 따라 바뀌므로 2026-08-14 실측 기준이고, 같은 내용이 2026-10-17 서버 코드에 그대로 남아 있다(`GeminiReceiptExtractor.kt`, `application.yaml`의 `gemini.model`).

## 핵심 정리

- **API 키가 `AQ.`로 시작한다(53자).** 2026년부터 `AIza` Standard 키에서 `AQ.` Auth 키로 전환 중이고 `AIza`는 2026년 9월부터 거부한다. `AQ.A...`를 보고 잘못 복사한 코드로 의심하지 않는다.
- **AQ 키 신규 계정은 `models/{model}:generateContent`가 404다.** 메시지는 「no longer available to new users. use the Interactions API」다. 증상이 계정과 상황에 따라 다르다: 404 즉답이거나, 응답 없이 매달려 30초 타임아웃.
- **Interactions API로 간다.** `POST https://generativelanguage.googleapis.com/v1beta/interactions`이고 인증 헤더는 그대로 `x-goog-api-key`다. 요청은 snake_case로 `{model, input:[{type:"image",mime_type,data(base64)},{type:"text",text}], response_format:[{type:"text",mime_type:"application/json",schema}]}`. 응답은 `steps[]`에서 `type=="model_output"`인 step의 `content[]` 중 `type=="text"`의 `text`다.
- **구조화 출력의 INTEGER 필드에는 `minimum`과 `maximum`이 필수다.** 상한이 없으면 자릿수가 수백 자리로 폭주하는 생성 결함이 재현됐다(`gemini-3.6-flash`). 상한을 건 뒤로는 재현되지 않았다. 서버는 상한을 스키마에 선언하고 파싱 뒤에도 같은 값으로 다시 본다.
- **모델**: `gemini-3.6-flash`를 실측으로 검증했다(이미지와 스키마 응답 7~10초). `gemini-2.5-flash`와 `gemini-flash-latest`는 generateContent 전용 별칭이라 신규 계정에서는 죽어 있다.

## 기록

### 2026-10-17 — 영수증 분석 서버에서 밟은 Gemini API 함정 (원 기록 2026-08-14)

- 맥락: 소프트웨어마에스트로 영수증 분석 API(`POST /api/v1/receipts`, SSH-268)를 만들며 실측했다. 같은 API의 설계는 개발 기록 2026-08-18에 있다. Claude 메모리 이관(원 기록 2026-08-14).
- 배운 것: 위 핵심 정리 전부. 키가 `AQ.`로 시작해도 정상이고, 같은 키가 generateContent에서만 404라 키 문제로 오진하기 쉽다.
- 근거: 서버 코드 `Server/src/main/kotlin/com/boheomgaenyang/receipt/external/gemini/GeminiReceiptExtractor.kt`(주석에 Interactions API를 쓰는 이유가 있다), `docs/spec/SSH-268`. 서버 기본 모델과 폴백 체인은 다른 이야기이고 [Hermes cron 운영](Hermes%20cron%20%EC%9A%B4%EC%98%81.md)에 있다.
