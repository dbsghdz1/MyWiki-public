---
type: hub
title: "Flutter 작업노트"
summary: "Flutter 위젯과 제스처, 접근성, 플러그인, 모바일 테스트와 배포"
status: active
created: 2026-11-19
updated: 2026-11-19
---

# Flutter 작업노트

[전체 분야](../README.md#주제-목록)

- [Flutter iOS TestFlight 업로드](Flutter%20iOS%20TestFlight%20%EC%97%85%EB%A1%9C%EB%93%9C.md) — fastlane 없이 **API 키 하나로** 남의 팀에 올린다: `flutter build ios --no-codesign` → `xcodebuild archive`/`-exportArchive`(`destination=upload`)에 `-authenticationKey*`. **로컬 배포 인증서 0개여도 클라우드 서명으로 통과**. `--dart-define`은 `Generated.xcconfig` `DART_DEFINES`(base64)로 검증, `grep -q`+`pipefail`은 거짓 실패 · **빌드 번호는 내 기록이 아니라 ASC에 묻는다** — `bundle version must be higher than … '3'`이면 그 값+1 · **xcconfig에서 키를 `^KEY=`로 뽑으면 빈 키로도 `flutter build ios`가 exit 0** — `^KEY *=` + 길이 단언, worktree엔 gitignore된 `Kakao.xcconfig`가 없다 (2026-09-18)
- [위젯 테스트와 ListView 지연 빌드](%EC%9C%84%EC%A0%AF%20%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%99%80%20ListView%20%EC%A7%80%EC%97%B0%20%EB%B9%8C%EB%93%9C.md) — 기본 뷰포트 800×600에서 `ListView` 아래쪽 자식은 **빌드조차 안 돼** `find.text`가 못 찾는다 → `tester.view.physicalSize`로 세로를 키우고 `addTearDown(tester.view.reset)`. 생성자에서 load하는 ViewModel + Fake는 `pump()` 한 번이면 로드 뒤 (2026-09-12)
- [Google ML Kit 텍스트 인식](Google%20ML%20Kit%20%ED%85%8D%EC%8A%A4%ED%8A%B8%20%EC%9D%B8%EC%8B%9D.md) — 한국어 모델은 앱이 직접 싣는다(플러그인이 `compileOnly`라 런타임 `NoClassDefFoundError`), iOS 15.5. **arm64 시뮬레이터는 `google_mlkit_commons` Podfile 헬퍼로 고친다**(원인은 ML Kit podspec, 판정은 `lipo -info`가 아니라 `otool` `LC_BUILD_VERSION platform 7`). 실물 검출률이 낮으면 검출기 튜닝 대신 사람이 박스를 옮기게 한다 (2026-09-17)
- [드래그 편집 UI의 좌표계와 경계](%EB%93%9C%EB%9E%98%EA%B7%B8%20%ED%8E%B8%EC%A7%91%20UI%EC%9D%98%20%EC%A2%8C%ED%91%9C%EA%B3%84%EC%99%80%20%EA%B2%BD%EA%B3%84.md) — **화면 단위 상수를 원본 좌표로 옮기면 배율의 역수만큼 커져** 하한이 상한을 넘고, `num.clamp`는 그때 `ArgumentError`다. **제스처 콜백의 예외는 삼켜져** 크래시가 아니라 「이 핸들만 안 움직임」으로 나타난다. `Clip.none`은 그리기만 허용하고 히트 테스트는 부모 밖으로 안 간다 (2026-09-17)
- [카카오 로그인 Android 키 해시](%EC%B9%B4%EC%B9%B4%EC%98%A4%20%EB%A1%9C%EA%B7%B8%EC%9D%B8%20Android%20%ED%82%A4%20%ED%95%B4%EC%8B%9C.md) — 동의 화면 Continue 무반응은 manifest placeholder(환경변수) 또는 키 해시 불일치. 디버그 키스토어가 둘(`(로컬 경로)` vs flutter-env), 등록된 건 flutter-env 쪽. `KakaoSdk.platformInfo.origin`으로 APK 해시 확인 (2026-09-13)
- [Semantics와 GestureDetector tap 액션](Semantics%EC%99%80%20GestureDetector%20tap%20%EC%95%A1%EC%85%98.md) — up·cancel 핸들러만 있어도 tap 액션이 실린다, tap은 Semantics가 소유 (2026-09-13)
- [머티리얼 날짜 피커와 ko 로케일](%EB%A8%B8%ED%8B%B0%EB%A6%AC%EC%96%BC%20%EB%82%A0%EC%A7%9C%20%ED%94%BC%EC%BB%A4%EC%99%80%20ko%20%EB%A1%9C%EC%BC%80%EC%9D%BC.md) — `showDatePicker` 입력 모드의 안내(`dateHelpText`=`yyyy.mm.dd`)와 파서(`parseCompactDate`→`DateFormat.yMd('ko').parseStrict`, `1995. 3. 2.`만 허용)가 **어긋나 안내대로 치면 전부 거부**된다. 로케일 문제라 iOS·Android 동일 — 직접 입력은 우리 `TextField`로 받고 달력은 `calendarOnly`로 연다 (2026-09-03)
- [flutter analyze와 분석 서버](flutter%20analyze%EC%99%80%20%EB%B6%84%EC%84%9D%20%EC%84%9C%EB%B2%84.md) — `flutter analyze`가 `FormatException`·`analysis server exited with code 255`로 죽어도 **`dart analyze`는 같은 규칙으로 돈다**. 소스 한 줄도 안 가리키고 죽으면 도구 문제다(한글 경로에서 실측). **analyze와 format은 다른 관문** — analyze 무경고여도 CI 포맷 검사(`dart format --output=none --set-exit-if-changed .`)에서 떨어지니 올리기 전 `dart format .` (2026-09-16)
- [flutter run 프로세스와 앱 수명](flutter%20run%20%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80%20%EC%95%B1%20%EC%88%98%EB%AA%85.md) — `flutter run` 호스트가 죽어도 시뮬레이터 앱은 살지만 **print 로그와 VM Service는 못 살린다**(unified log에도 안 잡힘, attach 불가) — 로그가 필요하면 재실행이 답이고 Keychain 세션은 유지된다 · **안쪽 화면 스크린샷은 `flutter run --route=/<경로>`로 그 화면에서 시작** — 전역 redirect가 없고 인자가 없는 경로면 세션 없이 뜬다 (2026-09-16)
- [제스처 아레나와 스크롤 경쟁](%EC%A0%9C%EC%8A%A4%EC%B2%98%20%EC%95%84%EB%A0%88%EB%82%98%EC%99%80%20%EC%8A%A4%ED%81%AC%EB%A1%A4%20%EA%B2%BD%EC%9F%81.md) — 스크롤러 안 pan은 세로를 빼앗긴다 → `addAllowedPointer`에서 즉시 `resolve(accepted)`로 선점. `Container`에 `alignment`를 주면 부모를 채워 바깥 `Align`이 무력화되고 탭 표면이 행 전체가 된다 (2026-08-27)
- [iOS 접근성 트리 공백](iOS%20%EC%A0%91%EA%B7%BC%EC%84%B1%20%ED%8A%B8%EB%A6%AC%20%EA%B3%B5%EB%B0%B1.md) — 특정 화면(병원 검색 오버레이 계열)에서 접근성 트리가 통째로 빈다. XCTest 도구·VoiceOver 둘 다 못 읽는다 — 자동화가 한 화면에서만 실패하면 셀렉터 전에 `maestro hierarchy`부터 (2026-08-27) · 보험 등록 화면도 트리 공백 — iOS 시뮬레이터를 좌표로 모는 법 (2026-09-16)
