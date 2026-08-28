---
type: study
area: Flutter
audience: ai
status: active
created: 2026-08-29
updated: 2026-08-29
projects:
  - "보험찾개냥"
---

# flutter run 프로세스와 앱 수명

`flutter run`(debug) 호스트가 죽어도 시뮬레이터의 앱은 계속 돈다 — 하지만 로그·VM Service는 호스트와 함께 사라져서, 로그가 필요하면 재실행이 답이다.

## 핵심 정리

- **호스트와 앱은 수명이 다르다.** `flutter run` 프로세스를 죽여도(SIGTERM) 시뮬레이터 앱 프로세스는 살아서 화면·상태를 유지한다. `xcrun simctl spawn booted launchctl list | grep <bundle id>`로 확인.
- **대신 잃는 것** — ① `print`/`debugPrint` 출력: flutter run stdout으로만 나오고, **macOS unified log(`log show --predicate 'process == "Runner"'`)에는 안 잡힌다**(실측 — Runner 라인에 flutter: 출력 없음). ② VM Service: `http://127.0.0.1:<port>/<token>/` 포워딩이 호스트와 함께 죽어 `flutter attach --debug-url`도 connection refused, mDNS 재발견(`flutter attach`)도 "Waiting for a connection"에서 멈춘다.
- **재실행해도 앱 데이터는 유지된다** — flutter_secure_storage(Keychain) 세션은 재빌드·재설치에도 남아 로그인을 다시 안 해도 된다.
- 세션 도중 로그 스트림이 필요하면 처음부터 `flutter run 2>&1 | tee <log>`를 오래 사는 프로세스로 잡아 두는 것이 유일한 안전책이다.

## 기록

### 2026-08-29 — 실서버 E2E 중 flutter run이 두 번 죽고 알게 된 것

- 맥락: 보험찾개냥 dev 실서버 E2E — 백그라운드 `flutter run`이 외부 요인으로 중단됐는데 앱은 온보딩 화면 그대로 살아 있었다.
- 배운 것:
  - 앱 생존 확인은 `simctl spawn booted launchctl list`. 스크린샷(`simctl io booted screenshot`)도 앱 상태를 그대로 보여준다.
  - 죽은 호스트의 로그를 살릴 방법은 없다 — unified log에 Flutter print가 없고, VM Service 포트도 죽는다. `flutter attach`는 mDNS·`--debug-url` 둘 다 실패했다.
  - 재실행 비용은 낮다 — Keychain 세션이 남아 로그인 없이 이어졌다.
- 근거: 보험찾개냥 세션 2026-08-29 재현 2회(iPhone 17 Pro 시뮬레이터, Flutter 3.44.8). `log show --last 3m --predicate 'process == "Runner"'`에 UIKit 이벤트만 있고 flutter: 라인 0건.
