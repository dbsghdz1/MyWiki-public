---
audience: ai
분야: Flutter
---

# Google ML Kit 텍스트 인식 (google_mlkit_text_recognition)

## 핵심 정리

- **라틴 외 스크립트 모델은 앱이 직접 싣는다.** 플러그인 `android/build.gradle`이 `compileOnly("com.google.mlkit:text-recognition-korean:16.0.1")`라 링크는 되고 실행에서 `NoClassDefFoundError: com.google.mlkit.vision.text.korean.KoreanTextRecognizerOptions$Builder`로 죽는다. 앱 `android/app/build.gradle.kts`에 `implementation("com.google.mlkit:text-recognition-korean:16.0.1")`, iOS `Podfile`의 `target 'Runner'` 안에 `pod 'GoogleMLKit/TextRecognitionKorean', '~> 9.0.0'`(Podfile.lock의 GoogleMLKit 버전과 맞춘다)
- **iOS 최소 15.5.** `IPHONEOS_DEPLOYMENT_TARGET`(pbxproj 3곳)과 Podfile `platform :ios, '15.5'` 둘 다. 플러그인이 SwiftPM을 지원하지 않아 SwiftPM 프로젝트에도 `ios/Podfile`이 새로 생긴다 — 첫 빌드가 `Framework 'Pods_Runner' not found`로 죽으면 `cd ios && pod install` 뒤 재빌드
- **iOS arm64 시뮬레이터 미지원**(MLKitVision 프레임워크에 슬라이스 없음). Flutter가 `EXCLUDED_ARCHS`로 arm64를 빼고 x86_64로 빌드하며, Apple Silicon 시뮬레이터에는 설치가 거부된다(「해당 앱을 이 iOS 버전에서 사용하려면 개발자가 업데이트해야 합니다」). iOS 검증은 실기기
- 좌표: `TextElement.boundingBox`는 **EXIF 방향을 적용한 바로 선 이미지** 기준. `image` 패키지로 픽셀을 만질 때는 `bakeOrientation` 뒤에 그린다
- 주민번호 `숫자6-숫자7 형태의 가짜 번호`은 한 요소로 오기도, 하이픈 앞뒤로 쪼개져 오기도 한다 — 줄 안의 요소를 공백 없이 이어 붙여 정규식, 요소 하나에 앞자리까지 있으면 글자 수 비례로 자른다(신분증 숫자는 고정폭에 가깝다)

## 기록

### 2026-09-13 — 보험찾개냥 신분증 온디바이스 마스킹 spike (SSH-558 #139)

- 위 두 함정(한국어 모델 미탑재 크래시·arm64 시뮬레이터)을 실측. Android 에뮬레이터에 PIL로 만든 합성 주민등록증을 `adb push` + `MEDIA_SCANNER_SCAN_FILE`로 넣고 10a 픽 → 뒷 7자리만 검은 박스. 코드: `Client/lib/data/services/masking/`, 커밋 `51a7812`(의존성)

### 2026-09-16 — arm64 시뮬레이터는 **고칠 수 있다** (Podfile 헬퍼가 이미 들어 있다)

- 맥락: 보험찾개냥 SSH-568 PR에 붙일 iOS 스크린샷을 시뮬레이터로 찍으려다 또 막혔다. 08-31 항목의 「iOS 검증은 실기기」를 그대로 믿고 끝내지 않고 원인을 끝까지 팠다.
- 배운 것:
  - **원인 확정법**: fat framework의 슬라이스를 `lipo -thin <arch>`로 떼어 `otool -l | grep -A3 LC_BUILD_VERSION`을 본다. **`platform 2` = iOS 기기, `platform 7` = iOS 시뮬레이터.** MLKitVision·MLKitCommon·MLImage는 `x86_64(7) + arm64(2)`라 **arm64 시뮬레이터 슬라이스가 없다** — "arm64가 있다"는 `lipo -info`만 보면 오판한다.
  - 그래서 Flutter가 `Generated.xcconfig`에 `EXCLUDED_ARCHS[sdk=iphonesimulator*]=i386 arm64`를 박고 x86_64로 빌드하며, 애플 실리콘 시뮬레이터가 설치를 거부한다(`Failed to find matching arch` · 「해당 앱을 이 iOS 버전에서 사용하려면 개발자가 업데이트해야 합니다」).
  - **공식 우회책이 플러그인에 들어 있다** — `google_mlkit_commons 0.13.0`의 `ios/scripts/apple_silicon_simulator.rb`(+`patch_arm64_simulator.py`). Podfile에 `require File.expand_path('.symlinks/plugins/google_mlkit_commons/ios/scripts/apple_silicon_simulator', __dir__)` 후 `post_install`에서 `mlkit_apple_silicon_simulator_patch(installer)`를 부르면, 빌드마다 arm64 슬라이스의 `LC_BUILD_VERSION.platform`을 대상에 맞게 바꾸고 `EXCLUDED_ARCHS`를 걷어낸다. 시뮬레이터·실기기 양쪽이 같은 `pod install`로 돈다. 상류 이슈: Google issuetracker 178965151 · 플러그인 이슈 #861.
  - **Rosetta 대안은 무겁다** — Xcode 26의 기본 시뮬레이터 런타임은 arm64 전용이다(`simctl list runtimes --json`의 `supportedArchitectures: ['arm64']`). x86_64를 돌리려면 런타임을 Universal 변종으로 다시 받아야 하고, Apple이 Rosetta 종료를 예고했다.
- 근거: `Pods/MLKitVision/Frameworks/MLKitVision.framework` 슬라이스 실측 · `ios/.symlinks/plugins/google_mlkit_commons -> (로컬 경로)`에 스크립트 존재 확인 · 앱 바이너리는 `platform 7`뿐이었다. **Podfile 적용은 빌드 설정 변경이라 티켓·PR이 필요해 홍 판단 대기.**
