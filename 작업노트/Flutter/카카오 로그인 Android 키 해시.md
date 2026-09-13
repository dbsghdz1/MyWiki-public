---
type: study
area: Flutter
audience: ai
status: active
created: 2026-09-13
updated: 2026-09-13
projects:
  - "보험찾개냥"
---

# 카카오 로그인 Android 키 해시

## 핵심 정리

- **카카오 동의 화면에서 「Continue」가 아무 반응이 없으면 앱 쪽 설정 두 가지 중 하나다.** 웹뷰가 아니라 Chrome 커스텀 탭이라 오류 토스트도 로그도 앱에 안 온다.
  1. **manifest placeholder가 비었다** — `AndroidManifest.xml`의 `kakao${kakaoNativeAppKey}` scheme은 `build.gradle.kts`가 **환경변수 `KAKAO_NATIVE_APP_KEY`**로 채운다. `--dart-define`만 주면 Dart 쪽 `KakaoSdk.init`은 되고 리다이렉트 scheme은 `kakao`만 남아 콜백이 앱으로 안 돌아온다. 빌드 명령 앞에 `KAKAO_NATIVE_APP_KEY=... flutter build apk ...`로 둘 다 준다
  2. **키 해시 불일치** — 카카오 콘솔에 등록된 해시와 APK 서명 키가 다르면 `KakaoSdk` 로그에 `misconfigured: Android keyHash validation failed`. 앱은 그냥 로그인 화면으로 돌아온다
- **디버그 키스토어가 두 개다.** `(로컬 경로)`(Android Studio 기본)와 `(로컬 경로)`(flutter-env). 콘솔에 등록된 것은 **flutter-env 쪽**(`/8Cqgh+qVoRwbFhnyE0uSpckKU0=`). 기본 경로 것(`5gre9S8AMQCbYEw16bT3jOvKTXo=`)은 미등록이라 Gradle이 그걸 집으면 위 2번이 난다. 빌드 때 `ANDROID_USER_HOME=$HOME/Desktop/flutter-env/.android ANDROID_SDK_HOME=$HOME/Desktop/flutter-env ANDROID_PREFS_ROOT=$HOME/Desktop/flutter-env`를 같이 준다
- 해시 확인: `keytool -exportcert -alias androiddebugkey -keystore <keystore> -storepass android | openssl sha1 -binary | openssl base64`. **앱 안에서는 `KakaoSdk.platformInfo.origin`**이 지금 APK의 해시를 준다(`KakaoSdk.origin`은 없다) — 어느 키로 서명됐는지 콘솔과 대조할 때 이게 제일 빠르다
- 에뮬레이터 첫 Chrome 실행은 「Use without an account」를 눌러야 커스텀 탭이 뜬다. 동의 화면 Continue는 텍스트 셀렉터가 안 잡혀 좌표 탭(540,2083 @1080×2400)

## 기록

### 2026-09-13 — 보험찾개냥 전체 플로우 QA, 로컬 서버 × Android 에뮬레이터

- 홍 "여기서 continue가 안돼"(스크린샷). 처음엔 placeholder(1번)를 고쳐 Continue가 눌리게 됐고, 그다음 `keyHash validation failed`(2번). `flutter doctor -v`가 가리키는 SDK와 별개로 Gradle의 디버그 키스토어는 `(로컬 경로)`를 먼저 보기 때문에, 홍이 "안드로이드 키 해시를 말하는거야?" 하고 등록 해시를 알려준 뒤 두 키스토어 해시를 각각 뽑아 flutter-env 쪽이 등록된 것임을 확인, 그 환경변수로 재빌드·재설치하니 로그인 200. 홍 요청("aside에서 확인해볼래?")으로 `KakaoSdk.platformInfo.origin`을 임시 로그로 찍어 APK 해시가 등록값과 같음을 앱 안에서도 확인했다(임시 로그는 커밋 안 함)
