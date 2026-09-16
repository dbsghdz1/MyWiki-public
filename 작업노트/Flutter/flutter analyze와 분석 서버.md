---
type: study
area: Flutter
audience: ai
status: active
created: 2026-09-02
updated: 2026-09-16
projects:
  - "보험찾개냥"
---

# flutter analyze와 분석 서버

`flutter analyze`가 분석 서버 크래시로 죽어도 **`dart analyze`는 같은 규칙으로 멀쩡히 돈다** — 정적 분석이 불가능해진 게 아니라 그 래퍼만 죽은 것이다.

## 핵심 정리

- `flutter analyze`는 **LSP 분석 서버를 띄우고 JSON으로 대화하는 래퍼**다. 그 채널이 깨지면 `FormatException: Unexpected end of input` + `analysis server exited with code 255`로 죽는다. 스택은 전부 `analysis_server` 내부라 **내 코드와 아무 관련이 없어 보인다** — 실제로 관련이 없다.
- **`dart analyze`로 갈아탄다.** 같은 `analysis_options.yaml`을 읽고 같은 진단(린트 포함)을 낸다. CI가 `flutter analyze`를 쓰더라도 진단 판정은 이걸로 충분하다.
- **다만 포맷은 analyze가 안 본다.** 보험찾개냥 CI(`client-ci.yml`)에는 「포맷 검사」가 별도 단계로 있고 `dart format --output=none --set-exit-if-changed .`로 돈다. **analyze 무경고여도 여기서 떨어진다** — 줄바꿈 위치 하나면 충분하다. 올리기 전에 `dart format .`을 함께 돌린다.
- 판별법: `flutter analyze`가 **소스 한 줄도 안 가리키고** 죽으면 도구 문제, 파일·줄이 나오면 내 코드 문제.

## 기록

### 2026-09-16 — analyze는 통과했는데 CI가 포맷에서 떨어졌다 (보험찾개냥 SSH-539)

- 맥락: 화면 2개를 구현하고 `dart analyze lib test` 무경고 · `flutter test` 1200개 통과를 확인한 뒤 올렸는데, CI 「Flutter 빌드·테스트」가 **1분 3초 만에** 실패했다.
- 원인: 잡은 것은 테스트가 아니라 **포맷 검사** — `Changed lib/ui/support/support_content.dart` 외 2건, `Formatted 461 files (3 changed)`. 내가 손으로 쓴 줄바꿈이 `dart format`의 판단과 달랐다(긴 문자열 상수, 인자 3개짜리 호출).
- 배운 것: **analyze와 format은 다른 관문이다.** 로컬 검증 목록에 `dart format .`을 넣지 않으면 CI가 대신 잡아 주는데, 그 왕복이 한 번에 몇 분이다.


### 2026-09-02 — 경로에 한글이 든 프로젝트에서 재현

- 맥락: 소프트웨어마에스트로 SSH-441 — `LoginOutcome` 도메인 추가 후 analyze
- 증상: `flutter analyze`가 `FormatException: Unexpected end of input (at character 393)`로 죽었다. 끊긴 자리가 **워크스페이스 경로를 담은 초기화 메시지**(`...%A1/Boheomgaenyang/Client/"}],"capabilities":...`)였고, 그 경로에는 한글(`소마`)이 들어 있다. 인코딩된 경로가 들어간 initialize 요청이 잘려 서버가 파싱에 실패한 모양이다
- 해법: `dart analyze` — 정상 동작했고 실제 오류 8건(타입 불일치·미사용 import)을 그대로 잡아 줬다. 그날 작업 내내 이걸로 갈음했다
- 근거: PR #105 작업 중 실측. 같은 트리에서 `flutter test`는 정상이었다 — **테스트 러너는 분석 서버를 안 쓴다**
