---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-27
updated: 2026-08-27
projects:
  - "소프트웨어마에스트로"
---

# dio multipart

Flutter dio로 multipart를 보낼 때의 두 함정 — JSON 파트의 content-type, 그리고 파트는 1회용이라는 것.

## 핵심 정리

- **JSON 파트는 content-type을 명시해야 서버가 읽는다** — `MultipartFile.fromString(jsonEncode(body), contentType: DioMediaType('application', 'json'))`. 안 넣으면 파트가 text/plain으로 나가고, Spring `@RequestPart`가 JSON 컨버터를 못 골라 415가 난다. curl로 재현할 때도 `-F "request=@file.json;type=application/json"`처럼 `;type=`이 필요하다.
- **`MultipartFile`은 finalize 1회용이다.** dio가 전송하면서 이미 finalize하므로, 테스트에서 `RequestOptions.data as FormData`로 잡아 보낸 파트를 되읽으려고 `part.finalize()`를 부르면 `Bad state: The MultipartFile has already been finalized`가 난다. **`part.clone().finalize()`로 읽는다** — fromString/fromBytes로 만든 파트는 데이터를 들고 있어 clone이 된다.

## 기록

### 2026-08-27 — 진료내역 저장 API를 multipart로 전환하며

- 맥락: 소프트웨어마에스트로 SSH-289 — `POST /api/v1/treatment-records`를 `request` JSON 파트 + `receiptImage` 파일 파트로 전송하도록 `TreatmentService.createTreatmentRecord`를 바꿈
- 배운 것: 위 핵심 정리 두 가지
- 근거: `Client/lib/data/services/api/treatment_service.dart`, `Client/test/data/repositories/treatment/treatment_repository_remote_test.dart`(clone 주석), PR #83. finalize 에러는 테스트 실행에서 실측
