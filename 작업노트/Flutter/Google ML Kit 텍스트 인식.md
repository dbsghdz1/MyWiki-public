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
