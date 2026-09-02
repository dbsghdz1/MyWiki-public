---
type: study
area: Flutter
audience: ai
status: active
created: 2026-09-02
updated: 2026-09-02
projects:
  - "보험찾개냥"
---

# flutter analyze와 분석 서버

`flutter analyze`가 분석 서버 크래시로 죽어도 **`dart analyze`는 같은 규칙으로 멀쩡히 돈다** — 정적 분석이 불가능해진 게 아니라 그 래퍼만 죽은 것이다.

## 핵심 정리

- `flutter analyze`는 **LSP 분석 서버를 띄우고 JSON으로 대화하는 래퍼**다. 그 채널이 깨지면 `FormatException: Unexpected end of input` + `analysis server exited with code 255`로 죽는다. 스택은 전부 `analysis_server` 내부라 **내 코드와 아무 관련이 없어 보인다** — 실제로 관련이 없다.
- **`dart analyze`로 갈아탄다.** 같은 `analysis_options.yaml`을 읽고 같은 진단(린트 포함)을 낸다. CI가 `flutter analyze`를 쓰더라도 로컬 판정은 이걸로 충분하다.
- 판별법: `flutter analyze`가 **소스 한 줄도 안 가리키고** 죽으면 도구 문제, 파일·줄이 나오면 내 코드 문제.

## 기록

### 2026-09-02 — 경로에 한글이 든 프로젝트에서 재현

- 맥락: 소프트웨어마에스트로 SSH-441 — `LoginOutcome` 도메인 추가 후 analyze
- 증상: `flutter analyze`가 `FormatException: Unexpected end of input (at character 393)`로 죽었다. 끊긴 자리가 **워크스페이스 경로를 담은 초기화 메시지**(`...%A1/Boheomgaenyang/Client/"}],"capabilities":...`)였고, 그 경로에는 한글(`소마`)이 들어 있다. 인코딩된 경로가 들어간 initialize 요청이 잘려 서버가 파싱에 실패한 모양이다
- 해법: `dart analyze` — 정상 동작했고 실제 오류 8건(타입 불일치·미사용 import)을 그대로 잡아 줬다. 그날 작업 내내 이걸로 갈음했다
- 근거: PR #105 작업 중 실측. 같은 트리에서 `flutter test`는 정상이었다 — **테스트 러너는 분석 서버를 안 쓴다**
