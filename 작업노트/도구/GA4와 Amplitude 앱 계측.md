---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-05
updated: 2026-09-10
projects:
  - "탭탭"
---

# GA4와 Amplitude 앱 계측

앱에서 GA4를 쓰려면 Firebase Analytics SDK뿐이고, 웹처럼 자동으로 잡히는 게 없어 전부 수동으로 심는다. Amplitude를 나란히 붙이는 건 리포트를 두 벌 만들려는 게 아니라 교차검증용이다.

## 핵심 정리

- **앱용 GA4 = Firebase Analytics.** 독립 GA4 iOS SDK는 없다. 남는 선택지는 Measurement Protocol로 HTTP POST를 직접 쏘는 것인데, `first_open`·`session_start`·`user_engagement` 같은 자동 이벤트와 리텐션 리포트가 통째로 안 잡혀서 "교차검증용 두 번째 툴"이라는 목적에 못 미친다.
- **`FirebaseApp.configure()`는 `GoogleService-Info.plist`가 없으면 실패가 아니라 앱을 죽인다**(fatalError). 키를 아직 안 받은 상태에서도 앱이 돌아야 하면 `FirebaseOptions.defaultOptions()`가 `nil`인지 먼저 보고 건너뛴다. `FirebaseApp.app() != nil`이 초기화 여부 판정.
- **계측을 넣은 것과 사용자가 추적되는 것은 다른 사건이다.** 탭탭은 계측 PR을 2026-09-09에 머지했는데 스토어 1.2.1은 **그 전날(09-08)에 업로드**된 빌드라, 「계측 다 넣었다」고 기록해둔 동안 실제 사용자는 **한 명도 추적되지 않고 있었다**. 판정은 코드가 아니라 **출시된 빌드에 그 커밋이 들어 있는가**로 한다 — `git branch -r --contains <계측커밋>`이 `main`/출시 태그를 포함하는지 본다.
- **계측을 추가하는 릴리즈는 App Store Connect의 「앱 개인정보」 선언을 같이 고쳐야 한다.** Amplitude·Firebase Analytics는 **사용 데이터(제품 상호작용)**와 **식별자(기기 ID — Amplitude device ID · Firebase app instance ID)**를 수집한다. 「데이터를 수집하지 않음」으로 둔 채 올리면 **사실과 다른 신고**가 된다. 반면 **추적(Tracking)은 「아니오」**가 맞다 — 광고 네트워크에 연결하거나 데이터 브로커에 넘기지 않으면 ATT 대상이 아니다(어트리뷰션 툴을 붙이는 순간 달라진다). 이건 법적 선언이라 에이전트가 대신 채우지 않는다.
- **프로바이더 하나만 빼려면 그 키/설정 파일을 번들에서 빼면 된다.** `AnalyticsConfiguration.fromMainBundle()`이 `bundle.path(forResource:"GoogleService-Info", ofType:"plist")`로 판정하므로, plist를 `Resources/`에서 치우면 GA4가 조용히 빠지고 나머지는 그대로 돈다(실측: `분석 프로바이더 시작: Console, Amplitude`). **«키가 없으면 프로바이더가 조용히 빠지게» 설계한 것이 릴리즈 범위를 자르는 스위치로도 쓰인다.**
- **Debug 번들 ID 불일치는 Release에서 사라진다.** plist는 스토어 번들 ID(`com.Nbs.dev.ADA.app`)로 발급받는데 Debug는 `com.Nbs.dev.app`이라 `I-COR000008` 경고가 뜬다. **Release 빌드는 번들 ID가 일치하므로 경고가 없다** — 즉 이 경고는 개발 빌드 전용 문제이고, 진짜 쟁점은 「개발 트래픽이 운영 속성에 섞이는가」다.
- **Firebase를 정적 링크로 붙이면 앱 타깃에 `-ObjC`가 필요하다. 없으면 GA4가 «켜진 것처럼 보이면서» 업로드가 0건이다.** `AnalyticsKit`이 `.staticFramework`라 GoogleUtilities의 ObjC 카테고리(`GULNSData+zlib`)가 최종 링크에 안 실렸고, 그 결과 `-[APMDatabase insertBundle:isRealtime:error:]`가 `+[NSData gul_dataByGzippingData:error:]: unrecognized selector sent to class`로 죽었다. **ObjC 카테고리는 링크 시점에 미해결 심볼을 만들지 않아서, 정적 라이브러리에서 그 오브젝트 파일 자체가 안 끌려온다** — `-ObjC`가 그 라이브러리의 ObjC 오브젝트를 전부 싣게 하는 플래그다. 증상이 지독한 이유는 **초기화도 되고(`분석 프로바이더 시작: Console, GA4, Amplitude`) 이벤트도 찍히는데(`screen_view (_vs)` debug 마킹) 큐잉 직전에만 죽어서** 로그를 대충 보면 정상으로 읽히는 것이다. 판정은 두 가지로 한다 — 로그에 `I-ACS030000 Exception on worker queue`가 있는가, 그리고 **`nm -a <앱>.debug.dylib | grep GULNSData`가 비어 있는가**. 고친 뒤에는 `Bundle added to the upload queue` → `Uploading data. Host: app-analytics-services.com` → `Successful upload ... Code: 204`가 이어서 나온다.
- **`GoogleService-Info.plist`를 gitignore하면 CI 빌드에는 GA4가 빠진다.** 로컬에서만 되고 TestFlight/스토어 빌드에는 안 들어간다 — CI에 시크릿으로 복원하는 스텝을 같이 넣지 않으면 조용히 계측 없는 빌드가 나간다.
- **Firebase의 화면 자동수집을 꺼야 한다.** Info.plist `FirebaseAutomaticScreenReportingEnabled = false`. 켜두면 UIViewController 기준이라 SwiftUI 앱은 전부 `UIHostingController`로 뭉개진다. Amplitude도 같은 이유로 `autocapture`에서 `.screenViews`를 빼고 `[.sessions, .appLifecycles]`만 켠다.
- **키는 Info.plist를 통해 xcconfig에서 주입한다.** `"AMPLITUDE_API_KEY": "$(AMPLITUDE_API_KEY)"` → 빌드 세팅이 비면 Info.plist에 **빈 문자열**이 남는다(키가 없어지는 게 아니다). 그래서 읽는 쪽에서 `trimmingCharacters` 후 `isEmpty`를 "키 없음"으로 처리해야 한다.
- **`screen_view`는 Firebase 예약 이벤트가 아니다** — 수동 로깅 가능하고 파라미터 키는 `screen_name`. 반면 `first_open`·`session_start`·`user_engagement`·`app_update` 등은 예약이라 못 쓴다.
- **자유 입력(검색어·제목)은 파라미터로 보내지 않는다.** 개인정보이기도 하고 GA4 파라미터 값은 100자에서 잘려서 "긴 검색어"와 "잘린 검색어"가 같은 값으로 뭉개진다. 길이 구간만 보낸다. 개수도 원값 대신 구간 — 유저 프로퍼티는 카디널리티가 낮아야 그룹핑이 된다(GA4 유저 프로퍼티는 계정당 25개 한도).
- **`Bool`은 툴마다 다르게 보낸다.** GA4는 `NSString`/`NSNumber`만 받고 리포트에서 0/1보다 `"true"`/`"false"`가 읽기 쉽다. Amplitude는 JSON이라 boolean 그대로.
- **계측은 뷰가 아니라 리듀서의 성공 액션에 심는다.** 버튼 탭에 심으면 이후 단계에서 실패한 것까지 전환으로 세어진다(탭탭에서 `link_save`를 저장 버튼에 심었다면 메타데이터 추출 실패가 저장으로 잡혔을 것). TCA에서는 `@Dependency(\.analytics)`로 받고, `.run` 클로저 안에서 쓸 땐 `.run { [analytics] send in }`으로 캡처한다.
- **"성공 액션"으로는 부족하다 — `case` 진입부가 아니라 성공한 `do`/`try` 다음 줄이어야 한다.** `case` 첫 줄의 `analytics.track(...)`은 그 아래 `guard ... else { return .none }`도, `.run`의 `catch`도 지나지 않는다. 탭탭에서 실제로 `memo_save`가 빈 메모(early return)에도 찍혔고, `link_move_category`는 `moveLinks`가 throw해도 찍혔다(`await send(.moveDone)`이 `do/catch` **밖**에 있었다). 고치는 방법은 track을 `.run { [analytics] _ in try …; analytics.track(…) }`처럼 throwing 호출 **뒤로** 옮기는 것뿐이다.
- **track이 붙은 액션을 아무도 `send`하지 않으면 지표는 에러 없이 그냥 0이다.** 탭탭 `DeleteLinkFeature`는 `case .deleteDone`에 `analytics.track(ConversionEvent.linkDeleted)`가 멀쩡히 있었는데 성공 경로가 `.delegate(.route(.back))`으로 바로 빠져서 `.deleteDone`을 보내는 곳이 하나도 없었다 — 컴파일도 되고 삭제도 되는데 전환만 통째로 안 찍힌다. **"이벤트를 다 심었나"는 정의 대비 발화 지점으로 세면 놓친다. 발화 지점이 실제로 `send`되는지까지 봐야 한다.**
- **한 시트가 여러 대상을 처리하면 이벤트도 대상별로 갈라야 한다.** 탭탭 `HighlightEditFeature`는 `State.Context`가 `.comment`/`.highlight` 둘인데 삭제 확인에서 항상 `highlightDeleted`를 보냈다 — 메모 삭제 수는 0, 하이라이트 삭제 수는 부풀었다. **실제 삭제를 하는 쪽(부모 `SummaryFeature`)으로 track을 옮기면** 컨텍스트 분기와 저장 성공 확인이 한 자리에서 해결된다.
- **실패를 "0건 결과"로 내려보내면 대시보드에서 영원히 못 가른다.** 탭탭 검색은 catch에서 `.searchResponse(response: [], totalCount: 0)`을 보냈고, `search_submit`의 `has_result`는 `resultCount > 0`이라 SwiftData 에러와 정상 무결과가 같은 값으로 찍혔다. **실패는 `totalCount: nil`처럼 "값 없음"으로 보내고 그 분기에서는 track하지 않는다.**
- **화면 상태(`state.articles.count`)로 유저 프로퍼티를 세지 않는다.** 그 배열이 항상 최신이라는 보장이 없다 — 탭탭은 `safariInfo == false`면 `onAppear`에서 `fetchLinks()`를 안 타서 링크가 여러 개인데도 `saved_link_count`가 1로 갈 수 있었다. 저장 성공 뒤 `fetchLinksCount(predicate: nil)`로 저장소에서 다시 센다.
- **TCA `testValue`는 `unimplemented`가 아니라 no-op으로 둔다.** 계측은 부수효과라 관례대로 `unimplemented`를 넣으면 계측을 심을 때마다 상관없는 기존 테스트가 깨진다.
- **화면이 아닌 리듀서에 `screen_view`를 심으면 두 번 찍힌다.** 탭탭에서 `CategoryListFeature`는 독립 화면이 아니라 홈 안의 섹션이라 홈 진입 한 번에 `screen_view home`과 `screen_view my_category`가 같이 나왔다. 실제 화면은 `MyCategoryCollectionFeature`였다 — **리듀서 이름만 보고 화면이라고 단정하지 말고 `State`가 어디에 안겨 있는지 확인한다.**
- **키가 없어도 검증할 수 있다.** 콘솔 프로바이더를 하나 더 붙여 `os.Logger`로 찍고 시뮬레이터에서 확인한다. GA4 DebugView는 반영이 늦어서 "심는 시점이 맞는가"를 보기엔 로컬 로그가 훨씬 빠르다.

```bash
xcrun simctl spawn booted log stream --level debug --predicate 'subsystem == "TapTap"'
# 📊 screen_view { screen_name=home }
```

- 시뮬레이터에서 UserDefaults 플래그를 바꿔 온보딩 이후 화면부터 확인하려면 앱을 먼저 종료하고 `xcrun simctl spawn <udid> defaults write <bundleID> <key> -bool true`. 앱 컨테이너의 plist를 PlistBuddy로 직접 고치는 방식은 cfprefsd 캐시 때문에 안 먹었다.

- **`git add -A <디렉터리>`는 빌드 산출물을 통째로 삼킨다.** 로컬 검증용으로 `-derivedDataPath ./dd`를 쓰면 그 폴더가 레포 안에 생기는데, `.gitignore`에 없으면 `git add -A TapTap`이 **11,948개 파일을 커밋에 넣는다**(탭탭 실측 — PR을 열고 나서야 "12,069 files changed"로 발견했다). 파일 목록을 보지 않고 `-A`를 쓰지 않는다. 이미 들어갔으면 **마지막 정상 커밋으로 `git reset --soft` → 경로를 골라 다시 커밋**이 가장 깔끔하다(`filter-branch`는 인덱스에 그 산출물의 변경이 남아 있으면 "Cannot rewrite branches: You have unstaged changes"로 시작조차 안 된다). 그 다음 `.gitignore`에 넣는다.
- **퍼널 이벤트는 버튼 탭이 아니라 화면 진입에서 찍는다.** 온보딩 단계 이벤트를 "다음" 버튼 탭에 심으면 **그 단계를 통과한 사람만** 세어져서 이탈률이 구조적으로 안 보인다(탭탭 실측: 1~7단계 중 5개가 탭 시점에 붙어 있었다). 리듀서에 `onAppear` 액션이 없으면 액션을 만들고 뷰에 `.onAppear { store.send(.onAppear) }`를 붙여서라도 진입에 심는다. 반대로 **건너뛰기는 탭에 심는 게 맞다** — 그건 행동이지 노출이 아니다.
- **확장·위젯처럼 수명이 짧은 프로세스에는 SDK를 두지 않는다.** 사파리 확장 프로세스는 몇 백 밀리초만 살아 있어 Amplitude의 30초 flush 타이머가 한 번도 안 돈다 — SDK를 붙여도 이벤트는 그대로 버려진다. **앱 그룹 `UserDefaults`에 쌓고 앱이 켜지거나 포그라운드로 올라올 때 대신 보내는 것**이 안전하다(탭탭: `ExtensionAnalyticsQueue` 최대 200건 → `NbsApp.deliverPendingExtensionEvents`). 이때 확장에서 일어난 실제 시각을 `occurred_at`으로 같이 실어야 나중에 순서가 복원된다.
- **앱 그룹 큐를 거치면 값의 타입이 문자열로 뭉개진다.** JS가 `String(count)`로 넘기고 `NSExtensionContext` → `[String: String]` → `UserDefaults`를 지나는 동안 boolean·숫자가 전부 문자열이 된다. Amplitude는 JSON boolean을 그대로 받는데 `"true"` 문자열이 가면 대시보드에서 앱 이벤트와 확장 이벤트의 같은 파라미터가 다른 타입으로 갈린다. **전송 직전(탭탭 `ExtensionEvent.typedValue(forKey:rawValue:)`)에 키별로 원래 타입으로 되돌린다.**
- **큐를 언제 비웠는지도 파라미터로 남기려면 호출자에게서 받아야 한다.** 탭탭은 `delivered_by`를 `"app_launch"` 상수로 박아둬서 `didFinishLaunching`과 `scenePhase == .active` 두 경로가 구분되지 않았다. `deliverPendingExtensionEvents(trigger:)`처럼 계기를 인자로 받는다.
- **앱 그룹 `UserDefaults`의 read-modify-write는 프로세스 간 원자적이지 않다.** `append`(read → set)와 `drain`(read → removeObject) 사이에 트랜잭션이 없어서, `drain`이 읽은 뒤 확장이 append하면 `removeObject`가 그것까지 지운다. `NSLock`이나 직렬 큐는 프로세스 안에서만 유효해서 소용없다 — 제대로 고치려면 앱 그룹 공용 파일 락(`NSFileCoordinator`)이나 SQLite다. 탭탭은 앱이 포그라운드인 동안 사파리 확장이 도는 창이 좁아 알려진 한계로 남겨뒀다.
- **확장 콘텐츠 스크립트에 브라우저 분석 SDK를 넣으면 남의 사이트를 수집하게 된다.** 콘텐츠 스크립트는 사용자가 방문하는 **모든 페이지**에서 도는데, 거기에 자동 수집이나 세션 리플레이가 붙으면 그 페이지 내용·URL이 우리 프로젝트로 넘어온다(개인정보·앱 심사 양쪽 문제). JS에서는 이벤트 이름과 색·길이 같은 값만 `browser.runtime.sendMessage` → `sendNativeMessage`로 네이티브에 넘기고, 전송은 네이티브 SDK가 한다.
- **시뮬레이터 검증 빌드를 `CODE_SIGNING_ALLOWED=NO`로 만들면 앱이 시작하자마자 죽는다.** entitlement가 안 붙어 App Group 컨테이너 조회가 `nil`이 되고, 탭탭의 `Core/AppGroupContainer.swift`는 거기서 `fatalError`를 던진다(2초 만에 종료). **그 크래시 때문에 Amplitude의 30초 flush 타이머가 한 번도 못 돌아 "이벤트가 안 올라간다"로 보인다.** 크래시 리포트(`(로컬 경로)`)의 triggered thread는 메인 런루프만 보여줘 원인이 안 나오고, **`xcrun simctl spawn booted log stream --predicate 'process == "TapTap"'`의 마지막 줄**에 `Fatal error: Failed to find App Group container`가 있다. 검증 빌드는 `CODE_SIGN_IDENTITY="-" CODE_SIGNING_REQUIRED=NO`로 만든다(서명은 하되 프로파일은 요구하지 않음).
- **Amplitude-Swift의 전송은 로그만으로 판정하지 않는다.** `logLevel: .debug`면 subsystem `Amplitude`로 `Log: Start flushing N events`가 찍히지만(기본 `flushIntervalMillis=30_000`, `flushQueueSize=30`), 성공 응답은 별도 줄로 안 나온다. **판정은 앱 컨테이너의 `Library/Application Support/amplitude/<키>-$default_instance.events.index/` 디렉터리가 비었는지로 한다** — `PersistentStorageResponseHandler`가 200에서 `storage.remove(eventBlock:)`으로 파일을 지우고, 실패하면 남겨 재시도한다(400·413 같은 영구 실패에서도 지우므로 최종 확인은 Amplitude 대시보드).
- **원격 설정 캐시가 키 유효성의 방증이다.** `Library/Preferences/com.amplitude.remoteconfig.cache.$default_instance.plist`에 sessionReplay 샘플링 등 서버 응답이 들어 있으면 **그 API 키로 Amplitude 서버와 통신이 됐다는 뜻**이다 — 네트워크·키 문제를 이벤트 전송과 분리해서 볼 수 있다.

## 이벤트 설계 (멘토링에서 온 규칙)

원칙 출처는 전수열 멘토링.

1. **UI 종속 이벤트와 전환 이벤트를 타입부터 나눈다.** 전환은 UI를 갈아엎어도 이름·의미가 그대로여야 한다. 섞어두면 UI를 바꿀 때마다 전환 지표가 끊겨 "개선했더니 떨어진 것"인지 "안 찍히는 것"인지 구분이 안 된다.
2. **이벤트 파라미터는 이벤트에, 유저 프로퍼티는 사람에 귀속.** 둘이 같이 쌓여야 "링크 20개 이상 모은 유저의 검색 사용률" 같은 그룹핑이 된다.
3. **정의의 원천을 한 곳에.** 팀이 전원 개발자면 타입(enum + `AnalyticsEvent` 변환)으로 정의하고 문서는 그걸 옮겨 적는다. 이름 오타는 크래시가 아니라 리포트에서 이벤트가 조용히 둘로 갈리는 식으로 나타나므로 **이름·파라미터 키를 단위 테스트로 박아둔다**.

**어트리뷰션 툴(AppsFlyer/Airbridge)은 별개다.** 웹은 UTM으로 광고→유입이 이어지지만 앱은 앱스토어를 거치며 연결이 끊긴다. 유료 광고를 집행하기 전에 붙여야 한다.

## 기록

### 2026-09-10 — Amplitude만 담아 1.2.2 업로드: 「계측 머지」와 「사용자 추적」 사이의 구멍 (탭탭)

- 맥락: 탭탭에서 홍 «amplitude 들어간거 업데이트하자 유저 추적하게». 확인해보니 **스토어의 1.2.1에는 계측이 아예 없었다** — 계측 PR #148은 09-09 머지, 1.2.1 업로드는 09-08이었다. 범위는 홍의 지시로 **#149까지 + Amplitude만**(GA4 수정 PR #150 제외).
- 배운 것: 위 「핵심 정리」에 넣은 네 항목. 특히 —
  - **GA4를 빼는 방법이 코드 수정이 아니라 plist 제거였다.** `-ObjC` 없이 GA4가 들어가면 이벤트를 하나도 못 올리는 채로 출시되므로, plist를 레포 밖(`(로컬 경로)`)으로 옮겨 프로바이더를 뺐다. 빌드해서 `분석 프로바이더 시작: Console, Amplitude`로 실측 확인.
  - **탭탭 `appstore` 레인은 업로드까지만 한다**(`submit_for_review: false`). 그리고 `latest_testflight_build_number(version:) + 1`을 계산해 **xcconfig의 `MARKETING_VERSION`·`CURRENT_PROJECT_VERSION`을 직접 써넣는다** — 버전 올림 커밋이 필요 없고, `*.xcconfig`가 gitignore라 그 변경은 git에 남지도 않는다.
  - **업로드 직후에는 빌드가 버전 레코드에 안 붙는다.** Apple 처리에 수 분 걸리고, 그동안 `asc state`는 «build: ❌ not attached»만 보여준다. **처리 중인지 판단하려면 `asc builds`**(연결 여부와 무관하게 업로드된 빌드를 보여준다)를 보고, 나타나면 `asc attach-build <versionId> <buildId>`로 붙인다.
- 결과: **1.2.2 (2) 업로드 → 빌드 `4cbe7ba3` 연결 완료, `PREPARE_FOR_SUBMISSION`**. 릴리즈 노트는 1.2.1 내용이 남아 있어 «내부를 정비했어요»로 교체(사용자에게 보이는 변화가 없는 릴리즈다). **심사 제출은 앱 개인정보 선언 확인 전까지 하지 않았다.**
- 근거: develop `c5e9226d`(#149 + #148), 버전 레코드 `c80a4eda-a795-4730-a309-27d042fdccae`, 빌드 `4cbe7ba3-471f-43b9-8646-1eb5980d90b5`. `env -u RUBYOPT fastlane ios appstore version:1.2.2 skip_screenshots:true`([[작업노트/도구/fastlane 로컬 실행 환경|RUBYOPT 누출]] 회피). 시뮬레이터 프로바이더 실측 `Console, Amplitude`.

### 2026-09-09 — GA4를 실제로 켜다: 링커 플래그 하나가 업로드를 통째로 막고 있었다 (탭탭)

- 맥락: 탭탭에 GA4를 붙이는 마지막 단계. 코드는 9/5에 이미 다 배선돼 있었고(`hasFirebaseConfigFile`이 true면 프로바이더가 붙는 구조) **남은 건 `GoogleService-Info.plist`뿐**이었다.
- 배운 것:
  - **`-ObjC` 누락**(위 「핵심 정리」). 이번 작업에서 제일 값진 발견이다. plist만 넣으면 끝인 줄 알았는데, 넣고 로그를 끝까지 읽지 않았으면 **계측 0건인 채로 출시됐다.** 「프로바이더 시작」 로그와 「이벤트 찍힘」 로그가 둘 다 정상이라 중간에서 멈추면 성공으로 보인다.
  - **업로드 204는 «데이터가 쌓였다»의 근거가 못 된다.** GA4 속성이 연결 안 된 프로젝트도 `app-analytics-services.com`이 204를 준다. 최종 판정은 **GA4 DebugView**뿐이고, `-FIRDebugEnabled -FIRAnalyticsDebugEnabled`를 launch argument로 주면 실시간으로 뜬다.
  - **`firebase-tools`로 콘솔 작업 대부분을 대신할 수 있다.** `firebase apps:create IOS "<이름>" --bundle-id <ID>` → `firebase apps:sdkconfig IOS <appId> --out <경로>`로 plist가 바로 떨어진다. 단 **`firebase projects:create`는 계정에 Firebase 프로젝트가 하나도 없으면 실패한다** — GCP 프로젝트는 만들어지는데 `addFirebase`가 403 `PERMISSION_DENIED`이고, 원인은 그 프로젝트의 `firebase.googleapis.com`이 `DISABLED`인 것이다(serviceusage로 확인). **첫 프로젝트는 콘솔에서 만들어야 하고**, 그때 「Google 애널리틱스 사용 설정」을 반드시 켠다.
  - **Debug와 Release의 번들 ID가 다르면 plist는 한쪽만 맞는다.** 탭탭은 Debug `com.Nbs.dev.app` / Release `com.Nbs.dev.ADA.app`이라, 스토어용으로 받은 plist를 쓰면 개발 빌드에서 `I-COR000008` 경고가 뜬다. **경고일 뿐 Firebase는 그대로 동작해서 개발 트래픽이 운영 GA4로 들어간다** — 막으려면 DEBUG에서 프로바이더를 빼야 한다(미결).
  - **이름 통일은 프로바이더가 아니라 `AnalyticsEvent` 한 곳에서 보장된다.** 두 프로바이더가 `event.name`을 그대로 쓰므로 갈릴 구조가 없다. 갈리는 건 값의 표현뿐(`.bool` → GA4 `"true"` 문자열 / Amplitude native boolean, 의도적). **다만 각 SDK의 자동 이벤트는 이름이 다르다** — GA4 `first_open`·`session_start`·`user_engagement` vs Amplitude `[Amplitude] Session Start`. 두 툴을 맞대볼 때는 **우리가 심은 전환 이벤트로만** 비교하고 세션·활성 지표는 비교하지 않는다.
- 근거: PR [#150](https://github.com/TapTapTeam/taptap-ios/pull/150) 커밋 `1373d8ea`(`Target+Templates.swift`에 `OTHER_LDFLAGS = $(inherited) -ObjC`). 실측 — 수정 전 예외 3건·업로드 로그 0건·`GULNSData+zlib.o` 미링크 → 수정 후 예외 0건·`Successful upload 204`·`.o` 링크됨. GA4 DebugView에 `screen_view` 3 · `first_open` · `session_start` · `user_engagement` + 유저 속성 `has_onboarded=true` 수신 확인. Firebase 프로젝트 `taptap-57734`, iOS 앱 `com.Nbs.dev.ADA.app`, 레포 시크릿 `GOOGLE_SERVICE_INFO_PLIST` 등록. iOS·macOS 빌드와 `AnalyticsKit` 테스트 12건 통과.

### 2026-09-08 — 코드리뷰 12건이 전부 "지표가 거짓말한다"였다 (탭탭 PR #148)

- 맥락: 탭탭 계측 PR #148에 CodeRabbit 인라인 12건. 사람 리뷰어는 없었다. 브랜치 `a319c463` 코드에 전부 대조했더니 **12건 모두 실재**했고, 지적이 하나로 묶였다 — 이벤트가 "사용자가 눌렀다"에 붙어 있어 저장 실패·early return·미삭제가 전부 전환으로 세어진다. **계측을 심는 PR에서 나오는 리뷰는 대개 로직 버그가 아니라 "이 숫자를 믿어도 되나"다.**
- 배운 것: 위 「핵심 정리」에 새로 넣은 항목들. 가장 값진 것 순서로 —
  ① `case .deleteDone`에 track이 있는데 **그 액션을 send하는 곳이 없어** `link_delete`가 통째로 0이었다. 전날 "정의된 이벤트 중 미발화 0"까지 감사 스크립트로 확인했는데도 못 잡았다 — 스크립트가 *정의 대비 발화 지점*만 셌지 *그 지점이 도달 가능한지*는 안 봤다.
  ② `case` 첫 줄의 track은 `guard`도 `catch`도 안 지난다. `.run { [analytics] _ in try …; analytics.track(…) }`로 옮기는 게 유일한 해법.
  ③ 한 시트가 `.comment`/`.highlight` 둘을 처리하는데 이벤트가 하나여서 메모 삭제가 하이라이트 삭제로 집계됐다. `ConversionEvent.memoDeleted`(`memo_delete`)를 새로 정의하고 track을 실제 삭제를 하는 부모로 옮겼다.
  ④ 검색 실패를 `totalCount: 0`으로 내려보내 SwiftData 에러와 정상 무결과가 `has_result=false`로 같이 찍혔다.
- 남긴 것: `ExtensionAnalyticsQueue`의 프로세스 간 원자성(위 핵심 정리). 저장소를 갈아야 해서 PR 범위 밖으로 두고 커밋 메시지에 명시했다.
- 함께: `Docs/analytics-events.md`를 삭제했다(홍 지시). 이벤트 원천이 코드인데 표가 따로 있으면 이번처럼 이벤트를 고칠 때마다 두 곳을 맞춰야 하고 어긋난 문서가 더 헷갈린다. **대신 「이벤트 설계」의 "정의의 원천을 한 곳에"가 말하는 단위 테스트(이름·파라미터 키 고정)가 그만큼 더 중요해졌다.**
- 근거: 커밋 `da98e3e7`(수정 11건)·`648372d5`(문서 삭제), 브랜치 `feat/analytics-amplitude` push. iOS(`TapTap`, iPhone 17 Pro 시뮬레이터)·macOS(`TapTapMac`) 빌드 통과, `AnalyticsKit` 스킴 테스트 통과.

### 2026-09-07 — 추적 범위 확대: 미삽입 이벤트 전부 + 사파리 확장(JS) (탭탭)

- 맥락: 전송 확인 직후 홍의 "앱 내에서 추적할 수 있는 모든 것들을 추적하고 싶어 JS 포함". `Docs/analytics-events.md`에 ⬜(미삽입)로 남아 있던 항목들이 그대로 할 일 목록이었다.
- 배운 것: 위 「핵심 정리」 확장 관련 두 항목. 그리고 **정의만 만들어 두고 발화 지점을 안 심으면 문서의 ⬜가 그대로 남는다** — 이번에 `link_delete`·`link_move_category`·`memo_save`·`onboarding_step_view`·`category_favorite_toggle`·`link_filter_change`·`setting_row_tap`을 실제 성공 액션에 붙여 8개 ⬜를 지웠다.
- 근거: 커밋 `f56fb6c`(모듈·계측), `146fa56`(확장 브리지), 브랜치 `feat/analytics-ga4-amplitude` push. iOS·macOS 빌드 통과, 시뮬레이터 스모크 테스트에서 `분석 프로바이더 시작: Console, Amplitude` → `screen_view` 정상. AnalyticsKit은 `.shared()`에 걸려 있어 피처마다 `Project.swift`를 고칠 필요가 없었다(확장은 `.core()`만 써서 SDK가 안 딸려간다).

### 2026-09-07 — 실제 전송 검증: "안 올라간다"의 범인은 내 빌드 플래그였다 (탭탭)

- 맥락: `AMPLITUDE_API_KEY`를 `Project.xcconfig`에 넣은 뒤 홍의 "amplitude 개발하자" → 실제 전송 확인. iPhone 17 Pro 시뮬레이터에 Debug 빌드를 올려 이벤트를 쐈다.
- 배운 것: 위 「핵심 정리」 뒤쪽 세 항목.
  - 처음 세 번의 시도에서 `screen_view`까지는 콘솔에 찍히는데 flush 로그가 한 번도 안 나왔다. 이유는 **앱이 2초 만에 죽고 있었기 때문** — `CODE_SIGNING_ALLOWED=NO` 빌드라 App Group entitlement가 없었고 `AppGroupContainer`가 `fatalError`. 서명을 켜서 다시 올리자 `Start flushing 6 events`가 30초 뒤에 정상으로 찍혔다.
  - 전송 성공은 큐 디렉터리가 비는 것으로 확인했다(2회 연속 `find … -type f | wc -l` = 0).
- 근거: 미커밋 작업 트리. 로그 `[TapTap:AnalyticsKit] 분석 프로바이더 시작: Console, Amplitude` → `📊 screen_view { screen_name=home }` → `[Amplitude:Logging] Log: Start flushing 4 events` → 큐 0개. Firebase는 `GoogleService-Info.plist`가 없어 설계대로 조용히 빠졌다.

### 2026-09-05 — 탭탭 iOS에 AnalyticsKit 모듈 신설

- 맥락: 탭탭 홍보 착수를 앞두고 지표를 심었다. Tuist + TCA 구조에 `Projects/AnalyticsKit`(staticFramework)을 만들어 GA4(Firebase 12.18.0)·Amplitude(Amplitude-Swift 1.18.8)·콘솔 세 프로바이더로 팬아웃.
- 배운 것: 위 「핵심 정리」 전부. 특히 ① `FirebaseApp.configure()`의 fatalError ② gitignore된 plist 때문에 CI 빌드가 GA4 없이 나가는 것 ③ `CategoryListFeature`가 화면이 아니라 홈 섹션이라 `screen_view`가 두 번 찍힌 것.
- 근거: 브랜치 `feat/analytics-ga4-amplitude` (`origin/develop` `7b507de` 기준), `Docs/analytics-events.md`. 시뮬레이터 로그로 `screen_view { screen_name=home }`·`has_onboarded=true`·`device_shell=phone` 실제 발화 확인, `AnalyticsKit` 스킴 테스트 12건 통과. 키(Firebase plist·`AMPLITUDE_API_KEY`)는 아직 없어 콘솔 프로바이더만 활성.
- 관련: [[작업노트/도구/Tuist|Tuist]] — Firebase SPM을 Tuist external로 붙이는 부분.
