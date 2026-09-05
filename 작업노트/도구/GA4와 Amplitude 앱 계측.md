---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-05
updated: 2026-09-05
projects:
  - "탭탭"
---

# GA4와 Amplitude 앱 계측

앱에서 GA4를 쓰려면 Firebase Analytics SDK뿐이고, 웹처럼 자동으로 잡히는 게 없어 전부 수동으로 심는다. Amplitude를 나란히 붙이는 건 리포트를 두 벌 만들려는 게 아니라 교차검증용이다.

## 핵심 정리

- **앱용 GA4 = Firebase Analytics.** 독립 GA4 iOS SDK는 없다. 남는 선택지는 Measurement Protocol로 HTTP POST를 직접 쏘는 것인데, `first_open`·`session_start`·`user_engagement` 같은 자동 이벤트와 리텐션 리포트가 통째로 안 잡혀서 "교차검증용 두 번째 툴"이라는 목적에 못 미친다.
- **`FirebaseApp.configure()`는 `GoogleService-Info.plist`가 없으면 실패가 아니라 앱을 죽인다**(fatalError). 키를 아직 안 받은 상태에서도 앱이 돌아야 하면 `FirebaseOptions.defaultOptions()`가 `nil`인지 먼저 보고 건너뛴다. `FirebaseApp.app() != nil`이 초기화 여부 판정.
- **`GoogleService-Info.plist`를 gitignore하면 CI 빌드에는 GA4가 빠진다.** 로컬에서만 되고 TestFlight/스토어 빌드에는 안 들어간다 — CI에 시크릿으로 복원하는 스텝을 같이 넣지 않으면 조용히 계측 없는 빌드가 나간다.
- **Firebase의 화면 자동수집을 꺼야 한다.** Info.plist `FirebaseAutomaticScreenReportingEnabled = false`. 켜두면 UIViewController 기준이라 SwiftUI 앱은 전부 `UIHostingController`로 뭉개진다. Amplitude도 같은 이유로 `autocapture`에서 `.screenViews`를 빼고 `[.sessions, .appLifecycles]`만 켠다.
- **키는 Info.plist를 통해 xcconfig에서 주입한다.** `"AMPLITUDE_API_KEY": "$(AMPLITUDE_API_KEY)"` → 빌드 세팅이 비면 Info.plist에 **빈 문자열**이 남는다(키가 없어지는 게 아니다). 그래서 읽는 쪽에서 `trimmingCharacters` 후 `isEmpty`를 "키 없음"으로 처리해야 한다.
- **`screen_view`는 Firebase 예약 이벤트가 아니다** — 수동 로깅 가능하고 파라미터 키는 `screen_name`. 반면 `first_open`·`session_start`·`user_engagement`·`app_update` 등은 예약이라 못 쓴다.
- **자유 입력(검색어·제목)은 파라미터로 보내지 않는다.** 개인정보이기도 하고 GA4 파라미터 값은 100자에서 잘려서 "긴 검색어"와 "잘린 검색어"가 같은 값으로 뭉개진다. 길이 구간만 보낸다. 개수도 원값 대신 구간 — 유저 프로퍼티는 카디널리티가 낮아야 그룹핑이 된다(GA4 유저 프로퍼티는 계정당 25개 한도).
- **`Bool`은 툴마다 다르게 보낸다.** GA4는 `NSString`/`NSNumber`만 받고 리포트에서 0/1보다 `"true"`/`"false"`가 읽기 쉽다. Amplitude는 JSON이라 boolean 그대로.
- **계측은 뷰가 아니라 리듀서의 성공 액션에 심는다.** 버튼 탭에 심으면 이후 단계에서 실패한 것까지 전환으로 세어진다(탭탭에서 `link_save`를 저장 버튼에 심었다면 메타데이터 추출 실패가 저장으로 잡혔을 것). TCA에서는 `@Dependency(\.analytics)`로 받고, `.run` 클로저 안에서 쓸 땐 `.run { [analytics] send in }`으로 캡처한다.
- **TCA `testValue`는 `unimplemented`가 아니라 no-op으로 둔다.** 계측은 부수효과라 관례대로 `unimplemented`를 넣으면 계측을 심을 때마다 상관없는 기존 테스트가 깨진다.
- **화면이 아닌 리듀서에 `screen_view`를 심으면 두 번 찍힌다.** 탭탭에서 `CategoryListFeature`는 독립 화면이 아니라 홈 안의 섹션이라 홈 진입 한 번에 `screen_view home`과 `screen_view my_category`가 같이 나왔다. 실제 화면은 `MyCategoryCollectionFeature`였다 — **리듀서 이름만 보고 화면이라고 단정하지 말고 `State`가 어디에 안겨 있는지 확인한다.**
- **키가 없어도 검증할 수 있다.** 콘솔 프로바이더를 하나 더 붙여 `os.Logger`로 찍고 시뮬레이터에서 확인한다. GA4 DebugView는 반영이 늦어서 "심는 시점이 맞는가"를 보기엔 로컬 로그가 훨씬 빠르다.

```bash
xcrun simctl spawn booted log stream --level debug --predicate 'subsystem == "TapTap"'
# 📊 screen_view { screen_name=home }
```

- 시뮬레이터에서 UserDefaults 플래그를 바꿔 온보딩 이후 화면부터 확인하려면 앱을 먼저 종료하고 `xcrun simctl spawn <udid> defaults write <bundleID> <key> -bool true`. 앱 컨테이너의 plist를 PlistBuddy로 직접 고치는 방식은 cfprefsd 캐시 때문에 안 먹었다.

## 이벤트 설계 (멘토링에서 온 규칙)

원칙 출처는 전수열 멘토링.

1. **UI 종속 이벤트와 전환 이벤트를 타입부터 나눈다.** 전환은 UI를 갈아엎어도 이름·의미가 그대로여야 한다. 섞어두면 UI를 바꿀 때마다 전환 지표가 끊겨 "개선했더니 떨어진 것"인지 "안 찍히는 것"인지 구분이 안 된다.
2. **이벤트 파라미터는 이벤트에, 유저 프로퍼티는 사람에 귀속.** 둘이 같이 쌓여야 "링크 20개 이상 모은 유저의 검색 사용률" 같은 그룹핑이 된다.
3. **정의의 원천을 한 곳에.** 팀이 전원 개발자면 타입(enum + `AnalyticsEvent` 변환)으로 정의하고 문서는 그걸 옮겨 적는다. 이름 오타는 크래시가 아니라 리포트에서 이벤트가 조용히 둘로 갈리는 식으로 나타나므로 **이름·파라미터 키를 단위 테스트로 박아둔다**.

**어트리뷰션 툴(AppsFlyer/Airbridge)은 별개다.** 웹은 UTM으로 광고→유입이 이어지지만 앱은 앱스토어를 거치며 연결이 끊긴다. 유료 광고를 집행하기 전에 붙여야 한다.

## 기록

### 2026-09-05 — 탭탭 iOS에 AnalyticsKit 모듈 신설

- 맥락: 탭탭 홍보 착수를 앞두고 지표를 심었다. Tuist + TCA 구조에 `Projects/AnalyticsKit`(staticFramework)을 만들어 GA4(Firebase 12.18.0)·Amplitude(Amplitude-Swift 1.18.8)·콘솔 세 프로바이더로 팬아웃.
- 배운 것: 위 「핵심 정리」 전부. 특히 ① `FirebaseApp.configure()`의 fatalError ② gitignore된 plist 때문에 CI 빌드가 GA4 없이 나가는 것 ③ `CategoryListFeature`가 화면이 아니라 홈 섹션이라 `screen_view`가 두 번 찍힌 것.
- 근거: 브랜치 `feat/analytics-ga4-amplitude` (`origin/develop` `7b507de` 기준), `Docs/analytics-events.md`. 시뮬레이터 로그로 `screen_view { screen_name=home }`·`has_onboarded=true`·`device_shell=phone` 실제 발화 확인, `AnalyticsKit` 스킴 테스트 12건 통과. 키(Firebase plist·`AMPLITUDE_API_KEY`)는 아직 없어 콘솔 프로바이더만 활성.
- 관련: [[작업노트/도구/Tuist|Tuist]] — Firebase SPM을 Tuist external로 붙이는 부분.
