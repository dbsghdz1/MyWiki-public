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
- **OCR 화질을 올리겠다고 `image_picker`의 `imageQuality`를 빼면 마스킹이 통째로 깨진다** — 옵션이 없으면 안드로이드 `ImageResizer`가 아예 안 돌아 HEIF 원본이 그대로 오는데(SSH-402 주석), 박스를 그리는 `image` 패키지는 HEIC/HEIF를 디코드하지 못한다. 화질을 올리려면 재인코딩이 유지되는지 먼저 확인한다

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
- 근거: `Pods/MLKitVision/Frameworks/MLKitVision.framework` 슬라이스 실측 · `ios/.symlinks/plugins/google_mlkit_commons -> (로컬 경로)`에 스크립트 존재 확인 · 앱 바이너리는 `platform 7`뿐이었다. **Podfile 적용은 빌드 설정 변경이라 티켓·PR이 필요해 홍 판단 대기.** → SSH-569(PR #147)로 머지됨(9/16).

### 2026-09-17 — 실물 검출률이 낮다 → 자동 검출을 버리지 않고 **사람이 박스를 옮기게** 한다 (SSH-574 #149)

- 맥락: 보험찾개냥 SSH-558이 남긴 실물 검증에서 **실제 주민등록증 검출률이 낮았다**(합성 이미지는 됐다). 대안으로 클라우드 OCR(Gemini)을 검토했다가 접었다 — 미마스킹 원본을 단말 밖으로 내보내는 순간 온디바이스를 고른 이유가 사라진다.
- 배운 것:
  - **인식 정확도 문제를 UI로 푸는 길이 있다.** OCR이 「초기 위치」만 주고 사용자가 드래그로 맞추면 검출률이 병목이 아니게 된다. 검출기를 더 튜닝하는 것보다 싸고 성공률이 100%다.
  - **원본을 화면에 띄우는 순간 EXIF 방향이 세 번째 좌표계로 끼어든다.** 지금까지는 「ML Kit `boundingBox`」와 「`bakeOrientation` 뒤 픽셀」 둘만 맞으면 됐다 — 확인 시트가 *칠해진* 파일(EXIF 비워짐)을 보여줬기 때문이다. 원본 위에 박스를 얹으려면 **화면에 그리는 좌표**가 셋째로 붙는다. 해법은 픽 직후 `bakeOrientation`을 적용한 JPEG를 한 장 만들어 OCR·표시·칠하기가 **전부 그 파일만 쓰게** 하는 것 — 「Flutter가 EXIF를 적용해 그리는가」를 플랫폼별로 검증할 필요가 사라진다.
  - **마스킹 박스를 사용자가 그리게 하면 서버 게이트와 충돌할 수 있다.** 명세 v1.4 4-3 게이트는 `숫자6-`까지 보여야 통과하고 **아무 패턴도 없으면 거부**한다(신분증이 아닌 사진 판정). 자동 검출은 뒷자리만 잡아 앞자리가 늘 남았는데, 수동 박스는 앞 6자리까지 덮을 수 있다. 수동은 코드로 못 막으니 문구로 안내하고 부채로 적었다.
- 근거: spec `docs/spec/SSH-574/spec.md` · 코드 `Client/lib/data/services/masking/` · 커밋 `d185e19` · Draft PR #149(구현 전, 1차 리뷰 대기)
