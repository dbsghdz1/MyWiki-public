---
type: study
area: Flutter
audience: ai
status: active
created: 2026-08-29
updated: 2026-09-16
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
- **`--route`로 특정 화면에 바로 들어간다.** `flutter run -d <device> --route=/faq`면 앱이 그 경로에서 시작한다(go_router는 플랫폼이 준 초기 경로를 `initialLocation`보다 우선한다). **화면 스크린샷을 찍으려고 UI를 눌러 들어갈 필요가 없다** — 로그인·데이터가 필요한 안쪽 화면이라도, 그 경로에 인자·가드가 없으면 바로 뜬다. 좌표 탭·접근성 트리로 헤매던 일이 사라진다.
- **백그라운드로 돌릴 때 `| tail`을 붙이면 출력이 안 보인다** — 파이프가 버퍼링해서 프로세스가 끝나야 흘러나온다. 진행을 보려면 `> <log> 2>&1`로 파일에 직접 쓰고 `grep -q "Dart VM Service"`로 기다린다.

## 기록

### 2026-09-16 — `--route`로 안쪽 화면 스크린샷 찍기 (보험찾개냥 SSH-539)

- 맥락: 보험찾개냥 10-1 자주 묻는 질문·10-2 고객센터. 둘 다 **마이페이지 안쪽** 화면이라 정공법이면 로그인·펫 등록을 거쳐 들어가야 했다. 직전 작업(SSH-568)에서 같은 일을 Maestro 좌표 탭으로 하다 5번 실패한 적이 있다.
- 배운 것: `flutter run -d <udid> --route=/support`로 **그 화면에서 앱이 시작한다.** 라우터에 전역 redirect가 없고 그 경로가 인자를 안 받으면 세션이 없어도 뜬다. 라이트·다크는 앱을 다시 띄울 필요 없이 `xcrun simctl ui <udid> appearance dark`(안드로이드는 `adb shell cmd uimode night yes`)로 바꿔 찍는다. iOS·안드로이드 6장을 좌표 한 번 안 찍고 얻었다.
- 대가: 경로마다 앱을 다시 띄워야 한다(같은 실행 중에는 경로를 못 바꾼다). 빌드 캐시가 있으면 두 번째부터는 십몇 초다.


### 2026-08-29 — 실서버 E2E 중 flutter run이 두 번 죽고 알게 된 것

- 맥락: 보험찾개냥 dev 실서버 E2E — 백그라운드 `flutter run`이 외부 요인으로 중단됐는데 앱은 온보딩 화면 그대로 살아 있었다.
- 배운 것:
  - 앱 생존 확인은 `simctl spawn booted launchctl list`. 스크린샷(`simctl io booted screenshot`)도 앱 상태를 그대로 보여준다.
  - 죽은 호스트의 로그를 살릴 방법은 없다 — unified log에 Flutter print가 없고, VM Service 포트도 죽는다. `flutter attach`는 mDNS·`--debug-url` 둘 다 실패했다.
  - 재실행 비용은 낮다 — Keychain 세션이 남아 로그인 없이 이어졌다.
- 근거: 보험찾개냥 세션 2026-08-29 재현 2회(iPhone 17 Pro 시뮬레이터, Flutter 3.44.8). `log show --last 3m --predicate 'process == "Runner"'`에 UIKit 이벤트만 있고 flutter: 라인 0건.
