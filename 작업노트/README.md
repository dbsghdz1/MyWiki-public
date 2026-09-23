---
type: hub
title: "작업노트"
summary: "다음 작업의 AI를 위한 재현 가능한 작업 경험 허브"
status: active
aliases: [AI 노트, 작업 경험, worknotes]
created: 2026-08-28
updated: 2026-09-23
---

# 작업노트

**AI(에이전트)가 작업하며 알게 된 것을 다음 작업의 AI를 위해 남기는 영역이다.** 홍은 다시 읽지 않는다. 2026-08-28에 `학습/공부/`에서 분리했다 — 공부는 홍이 이해하는 것, 여기는 AI가 검색하는 것.

**판정 질문: "이 프로젝트를 다시 만질 때 필요한가?"** — 그렇다면 여기에 쓴다. "왜 그런지 궁금해서 파고들었나?"에 해당하는 개념·원리는 [[학습/공부/README|공부]]로 보낸다.

## 좋은 노트의 조건

- **검색되는 것이 전부다.** API명·에러 메시지·파일명·커밋 해시·증상을 **그대로** 남긴다. 요약해서 고유명사를 지우면 안 된다 — 그게 검색 키다.
- 내용은 **이미 밟은 지뢰, 비자명한 API 동작, 재현된 원인**이다. 안 읽으면 같은 버그를 다시 만든다.
- 세션 시작 시 AI가 이 영역을 찾는 방법:

```bash
# 경로에 공백이 있으므로 --null/-0 필수
grep -rl --null "^audience: ai" "$VAULT/작업노트" | xargs -0 grep -l "<프로젝트명>"
```

## 원칙

1. **주제별 파일, 날짜별 항목 append.** 같은 주제로 경험이 쌓이며 한 파일이 두꺼워지는 것이 목적이다. 날짜별 파일은 만들지 않는다.
2. **양방향 연결.** 항목은 맥락이 된 프로젝트를 링크하고, 그 프로젝트 README의 `## 배운 것` 섹션은 이 주제를 링크한다. 작업 기록 파일에도 `배운 것:` 한 줄. 한쪽만 있으면 미완성이다.
    - 맥락이 프로젝트가 아닐 수 있다 — 볼트 운영·비용·인프라 주제는 그 맥락 문서(`운영/…`, `금융/…`)를 링크한다. 이때도 역링크 의무는 그대로다.
    - 비공개 영역을 맥락으로 링크할 때는 **금액·개인 사정을 본문에 그대로 옮기지 않는다.** 필요하면 `<!-- private:start -->`로 감싼다 — 작업노트는 공개 영역이라 그대로 public 저장소로 나간다.
3. **작업에 닿아 있어야 한다.** 근거는 커밋·파일·재현한 현상·실제 결정이다. 작업과 무관한 일반론을 옮겨 적지 않는다.
4. **`_wiki` 승격.** 한 주제가 외부 출처 여럿을 종합할 만큼 커지면 `_wiki/`에 주제 허브를 만들고 여기서 링크한다. 작업노트 파일은 그대로 남긴다.

## 파일 규칙

- 경로: **`작업노트/<분야>/<주제>.md`**. 분야는 `Apple` · `AppStore` · `도구` · `Flutter` 넷이고, 아래 「주제 목록」의 절 이름과 1:1로 맞춘다. 주제명은 검색 가능한 명사구.
- **어느 분야인지 애매하면 폴더를 새로 만들지 말고 [[작업노트/미분류|미분류]]에 항목만 append한다.** 같은 성격이 3개 모이면 그때 분야를 만들고, 만들면 주제 목록에 절을 함께 추가한다.
- frontmatter:

```yaml
---
type: study
area: Apple | AppStore | 도구 | Flutter
audience: ai
status: active | stale
created: YYYY-MM-DD
updated: YYYY-MM-DD
projects:
  - "[[프로젝트/개인/<이름>/README|<이름>]]"
---
```

- 본문 구조:

```markdown
# <주제>

한 줄 요약 — 현재까지의 결론.

## 핵심 정리
누적된 이해를 갱신하는 살아있는 요약. 기록이 쌓이면 여기를 고쳐 쓴다.

## 기록
### YYYY-MM-DD — 제목
- 맥락: [[프로젝트/개인/<이름>/README|<이름>]]에서 무엇을 하다가
- 배운 것: 결론 먼저 1~3불릿 (API명·에러 메시지·커밋 해시 그대로)
- 근거: 커밋 / 파일 / 재현 / 외부 링크
```

- [[작업노트/미분류|작업노트/미분류.md]]: 주제가 아직 애매한 항목을 임시로 두는 곳. "작업노트 미분류 정리해줘"로 주제 파일에 옮기고 원 항목은 삭제한다 (append-only 아님).
- 프로젝트 README의 `## 배운 것` 섹션: 첫 기록 시 만든다. `- [[작업노트/<분야>/<주제>|<주제>]] — 이 프로젝트에서 배운 요점 한 줄` 형식.

## 주제 목록

허브 규칙에 따라 실제 파일과 항상 일치시킨다. 절 이름은 하위 폴더와 1:1. 각 줄은 `링크 — 한 줄 요약 (갱신일)`.

### Apple
- [[작업노트/Apple/AppKit 오프스크린 렌더와 argument domain|AppKit 오프스크린 렌더와 argument domain]] — 창 없는 `NSView`는 **시스템 외관(다크)을 물려받아** 밝은 배경에서 라벨이 흰색으로 사라진다 → `view.appearance` 고정(`performAsCurrentDrawingAppearance`는 무효). drawingHandler 안에서 뷰를 그리면 벡터 확대. `-AppleLanguages`·`-onboarded NO` 같은 argument domain으로 상태 강제, **비ASCII 배열은 따옴표 필수** (2026-09-01)
- [[작업노트/Apple/Capacitor 웹앱 iOS 래핑|Capacitor 웹앱 iOS 래핑]] — 상태 표시줄은 `setOverlaysWebView(false)`(검은 띠 남음)가 아니라 **`true` + CSS `env(safe-area-inset-top)`**가 정답이고, `viewport-fit=cover`가 없으면 safe-area 값이 전부 0이라 CSS만 봐서는 원인을 못 찾는다. **`dist/`는 통째로 앱 번들에 들어가므로** 데이터를 넣어두면 오프라인 동작 = App Store 4.2 방어가 된다 (2026-08-27)
- [[작업노트/Apple/CoreImage 색 파이프라인|CoreImage 색 파이프라인]] — 색조 이동은 **±10% 안쪽**(R 0.45배로 죽이면 전체가 파래진다), 블랙 리프트는 `bias`가 아니라 `CIToneCurve` S커브, 시간 연출은 속성마다 구간·지수를 다르게, **색은 UI 맥락(프레임·배경) 없이 판정하면 안 된다** **`CIImage`는 EXIF 방향을 무시한다**, 표시 크기로 먼저 디코드(ImageIO 썸네일) (2026-08-25)
- [[작업노트/Apple/온디바이스 음성 인식과 번역|온디바이스 음성 인식과 번역]] — `SpeechAnalyzer`는 앱이 모델을 받지만 **`Translation`은 못 받는다**(SwiftUI `.translationTask` 경유 필수). macOS 26은 **오디오 전용 SCK 스트림**이 되고, ad-hoc 서명은 빌드마다 TCC를 초기화한다. API 가용성은 SDK `.swiftinterface`로 확인. **iOS 시뮬레이터는 음성 인식이 통째로 안 된다** — `supportedLocales` 빈 배열, `SFSpeechRecognizer`도 `kLSRErrorDomain 300`; 반면 `FoundationModels` 요약은 시뮬레이터에서 되고 한국어 품질 좋음. 요약 모델은 전사 오류를 못 고친다 (2026-09-01)
- [[작업노트/Apple/WatchConnectivity와 워치 녹음|WatchConnectivity와 워치 녹음]] — **시뮬레이터는 `transferFile`을 배달하지 않는다**(Apple 문서 명시), 실기기는 몇 초 안에 배달. **실기기 워치 녹음 세 겹**: ① `stop()` 직후 전송은 잘린 파일(모든 길이가 24,588B) → `didFinishRecording` 뒤로 ② 실기기 `AVAudioRecorder` AAC는 패킷을 안 씀(`fileSize=28` 고정, `currentTime`은 전진) → µ-law 16kHz ③ watchOS `record()`는 권한 프롬프트를 안 띄움(4,096B 고정) → `requestRecordPermission`. 백그라운드 오디오 키는 `UIBackgroundModes`(`WKBackgroundModes: audio`는 업로드 90362 거절) (2026-09-02)
- [[작업노트/Apple/메인 스레드를 막는 것들|메인 스레드를 막는 것들]] — `setActive`·햅틱 `start`·atomic 쓰기·세션 구성·`CIContext` 생성은 이름만 봐서는 비싼 줄 모른다, **블록을 없애면 우연에 기대던 코드가 깨진다** (2026-08-27)
- [[작업노트/Apple/카메라 캡처와 실시간 필터|카메라 캡처와 실시간 필터]] — 촬영음은 한국·일본 기기에서 공식 API로도 못 끄고 **억지로 켜면 앱이 죽는다**, 진짜 무음은 `AVCapturePhotoOutput`을 안 쓰는 것, `AVCaptureVideoPreviewLayer`에는 필터를 못 건다 (2026-08-26)
- [[작업노트/Apple/시뮬레이터로 UI 검증하기|시뮬레이터로 UI 검증하기]] — `simctl`에 **탭이 없고** 카메라도 없다, 권한 다이얼로그는 `erase`만 듣고 그 고착이 **오진을 만든다**, 답은 앱 안의 `#if DEBUG` 데모 모드 (2026-08-25)
- [[작업노트/Apple/SwiftUI|SwiftUI]] — `.plain` 버튼 히트 영역은 라벨 불투명 픽셀뿐, `.blur`는 사방 가장자리, **`.onExitCommand`는 포커스 없는 오버레이에 아예 안 온다**(컴파일은 되고 런타임에 무동작), **`Text("\(n)개")`는 천 단위 구분 기호가 붙고 String으로 넘기면 안 붙는다**, **다크의 모달은 배경을 낮추는 게 아니라 카드를 올려서 띄운다**, 화면 밖 요소는 여백 대신 `.clipped()`, **워치 경과 시간은 내 `Timer` tick 말고 `Text(timerInterval:)`** (손목 내림·감광 중 tick이 멈춰 건너뛰며 올라간다) (2026-09-05)
- [[작업노트/Apple/Swift와 Objective-C 브리징|Swift와 Objective-C 브리징]] — `@objc` 셀렉터는 Swift 이름+인자 레이블에서 생성(`mouseEntered(with:)` → `mouseEnteredWith:`), 프레임워크 콜백과 어긋나면 조용히 미호출. `responds(to:)`·`strings`로 검증 (2026-08-18)
- [[작업노트/Apple/WidgetKit과 AppIntents|WidgetKit과 AppIntents]] — AppIntents 문구는 컴파일 타임 리터럴만 허용, 런타임 L10n과 공존 불가 → xcstrings + knownRegions. `AppIntentConfiguration`은 macOS 14+ (2026-08-19)
- [[작업노트/Apple/SceneKit과 3D 에셋 파이프라인|SceneKit과 3D 에셋 파이프라인]] — **다시 export하는 것은 생김새를 바꾸는 일**. 숨긴 오브젝트는 변환이 평가 안 됨, usdz 애니는 이미 씬 타임베이스, 가시성 애니는 못 믿음, 조각된 셰이프 키 ≫ 노드 배율 (2026-08-20)
- [[작업노트/Apple/macOS 메뉴바와 샌드박스|macOS 메뉴바와 샌드박스]] — 샌드박스는 AX 전면 차단이지만 **IORegistry 조회는 그대로 허용**(배터리 사이클 수), 메뉴바 아이템 식별은 CGWindowList + 화면 기록 권한(재실행 필요) (2026-08-19)
- [[작업노트/Apple/StoreKit 2 권리 확인|StoreKit 2 권리 확인]] — `currentEntitlements`의 빈 결과는 "미구매"가 아니라 "아직 모름" — 올리는 건 유효 거래로, 내리는 건 `revocationDate` 관측으로만. 대칭으로 짜면 유료 기능·위젯이 깜빡인다 (2026-08-21)
- [[작업노트/Apple/Xcode 빌드와 번들 구성|Xcode 빌드와 번들 구성]] — 동기화 폴더에서 파일 빼기는 pbxproj `membershipExceptions`(예외=제외), 기능을 끄면 **릴리즈 최적화가 문자열째 걷어내므로** `strings` 대조가 검증이 된다 (2026-08-20)
- [[작업노트/Apple/Swift 동시성과 UIKit 메인 스레드|Swift 동시성과 UIKit 메인 스레드]] — `await` 뒤의 코드는 **마지막 실행자**에서 돈다. non-isolated VM의 async에서 화면 전환하면 백그라운드 UIKit → `setViewControllers` NSAssertion SIGABRT. VM을 `@MainActor`로 하거나 `MainActor.run` (2026-08-22)
- [[작업노트/Apple/HealthKit 수면 데이터 조회|HealthKit 수면 데이터 조회]] — "지난밤"은 달력 날짜로 못 자른다: 자정 기준 윈도우는 자정~취침 사이에 직전 밤을 놓친다. 36h lookback + 3h 공백 세션 분리 + 합집합, 읽기 거부는 빈 결과로만 온다 (2026-08-22)
- [[작업노트/Apple/UIKit 내비게이션 바 숨김과 pop 제스처|UIKit 내비게이션 바 숨김과 pop 제스처]] — 바를 숨기면 pop 제스처가 죽는 건 설계(delegate가 back 버튼 유무로 판단). 서브클래스에서 delegate를 맡고 `count > 1 && transitionCoordinator == nil`, 바 숨김은 한 곳에서만, 헤더 밑 스크롤 뷰는 `.never` (2026-08-22)
- [[작업노트/Apple/블루투스 기기 배터리 읽기|블루투스 기기 배터리 읽기]] — 맥 주변기기 잔량은 **경로가 둘**이다: 클래식 BT는 IORegistry `BatteryPercent`, **BLE는 거기 아예 없고** GATT `0x180F`를 CoreBluetooth로 직접 읽어야 한다. "Control Center엔 보이는데 내 코드엔 안 보인다"가 그 신호. TCC 프롬프트는 `CBCentralManager` 생성 시점에 뜨므로 시점 통제가 가능 (2026-08-22)
- [[작업노트/Apple/macOS 템플릿 아이콘 그리기|macOS 템플릿 아이콘 그리기]] — `isTemplate`은 **알파만 남기고 색을 버린다**. 채움 위에 그리려면 `.clear` knockout, 경계가 얼굴을 지나면 split face, '위에 얹힘'은 틈으로만 표현된다. 컬러의 하이라이트·결은 실루엣을 조각내므로 빼는 게 기본 (2026-08-25)

### AppStore
- [[작업노트/AppStore/인앱 구매 등록 API|인앱 구매 등록 API]] — ASC API로 IAP를 만들면 `MISSING_METADATA`로 시작하고 **다섯 조각**(생성·로컬라이제이션·가격·**지역**·심사 스크린샷)을 다 채워야 `READY_TO_SUBMIT`이 된다. 빠진 것을 알려주지 않는다. 관계 이름은 `inAppPurchaseV2`, 가격 스케줄에 `data.id`를 넣으면 409 (2026-09-09)
- [[작업노트/AppStore/스토어 스크린샷 중복 업로드|스토어 스크린샷 중복 업로드]] — `overwrite_screenshots: true`인데도 deliver가 두 벌을 올리는 일이 있고, **심사 제출 뒤엔 삭제가 막힌다**(409 "Can't Delete Screenshot After Submit for review"). 라이브 제품 페이지는 새 버전으로만 고친다. 방어는 업로드→dedupe→제출로 쪼개는 것 (2026-09-09)
- [[작업노트/AppStore/App Store 성장 도구|App Store 성장 도구]] — **Mac 전용 앱은 Apple Ads·인앱 이벤트·커스텀 제품 페이지가 전부 막힌다**(전부 iPhone/iPad 전용). 남는 것은 피처링 노미네이션(최소 3주 전)·프로모션 텍스트 (2026-08-19)
- [[작업노트/AppStore/App Store Connect 분석 리포트 API|App Store Connect 분석 리포트 API]] — 앱마다 `analyticsReportRequests`를 POST해야 데이터가 쌓이기 시작한다(`ONGOING`·`ONE_TIME_SNAPSHOT` 별개). 요청 직후 리포트 정의 156개는 보이지만 **instances는 한동안 `total: 0`**. 급하면 `salesReports`인데 `vendorNumber`는 API로 못 얻는다. 추세의 '단위'엔 업데이트가 섞여 있다 (2026-09-01)
- [[작업노트/AppStore/TestFlight 내부 그룹과 수출 규정|TestFlight 내부 그룹과 수출 규정]] — 내부 그룹엔 빌드를 못 붙인다(자동 배포), 수출 규정 질문은 `ITSAppUsesNonExemptEncryption`로 없앤다
- [[작업노트/AppStore/App Store Server Notifications|App Store Server Notifications]] — 비소모성도 프로덕션 ONE_TIME_CHARGE 수신, 실패 시 1h→12h→24h→48h→72h 재시도(72h 안에 고치면 자동 도착), 전송 이력·테스트는 App Store Server API(ASC 팀 키로 가능) (2026-08-19)
- [[작업노트/AppStore/유료 앱 판매 알림|유료 앱 판매 알림]] — **유료 다운로드 앱은 App Store Server Notifications가 안 온다**(IAP·구독 전용). 대안은 `salesReports` DAILY를 Vercel Cron(07:00 KST)으로 당겨 다음 날 Slack 요약. 거래 0건이면 404, 유료 판매는 `Customer Price > 0`으로 가른다. 수동 실행은 대시보드 Run 버튼뿐 (2026-09-03)
- [[작업노트/AppStore/미국 세금 양식 W-8BEN|미국 세금 양식 W-8BEN]] — **ASC 웹 세금 양식에는 Line 10(Article) 칸이 없다.** 조약 조항을 지정하려면 IRS 원본 PDF를 작성해 Finance Support 링크에 올리고 **파일명을 Case ID 8자리로** + **메일 회신까지** 해야 접수된다. 갱신 결과는 ASC 화면에 절대 반영되지 않는다. 한미조세조약 로열티는 Article 14(일반 15%, 저작권 10%) (2026-08-25)
- [[작업노트/AppStore/Google Play 개발자 계정|Google Play 개발자 계정]] — **Android 첫 출시의 병목은 코드가 아니라 계정 유형이다.** 개인 계정은 신규 앱마다 **테스터 12명 × 14일 연속 비공개 테스트** 필수, 조직(사업자) 계정은 면제 — 개인사업자로도 만들 수 있다. 단 조직 계정은 **D-U-N-S(최대 30일) + 도메인 연결 웹사이트**를 요구한다. **개인사업자에게 D-U-N-S는 구글 전용** — 애플은 개인사업자를 Organization으로 안 받고 Individual엔 애초에 불필요하다 (2026-08-26)
- [[작업노트/AppStore/앱 이름 속 Apple 상표 (5.2.5)|앱 이름 속 Apple 상표 (5.2.5)]] — 5.2.5 리젝은 스토어 등재명이 아니라 **기기에 표시되는 앱 이름**을 본다. `CFBundleDisplayName`이 없으면 조용히 `CFBundleName = $(PRODUCT_NAME)` = **타깃명**이 노출돼 `TapTapMac`처럼 Apple 제품명이 샌다. 빌드 설정에 이름을 넣어도 Info.plist 매핑이 없으면 에러 없이 버려진다 — 제출 전 아카이브 plist를 읽어 확인한다 (2026-09-11)
- [[작업노트/AppStore/앱 이름과 검색 노출|앱 이름과 검색 노출]] — **짧은 조어 브랜드명은 흔한 단어에 먹힌다.** `Fadeo`는 `Video`에 퍼지 매칭돼 **자기 이름 검색 200위 밖**(KR·US). 이름 짓기 전 iTunes Search API로 *정확 일치가 없을 때 무엇이 대신 나오는지*를 봐야 한다. 키워드는 단어 뜻이 아니라 **그 검색어가 속한 시장** 기준 — `인화`는 찍스·스냅스 같은 사진인화 서비스 시장이라 카메라 앱이 못 잡는다 (2026-09-02)
- [[작업노트/AppStore/앱 내 계정 삭제 요건|앱 내 계정 삭제 요건]] — 계정 생성 앱은 **앱 안에서 계정 삭제**가 필수(2022-06-30~)이고 **Sign in with Apple을 쓰면 삭제 시 REST API로 토큰 해지**까지가 요건이다. 해지하려면 서버가 애플 refresh 토큰을 갖고 있어야 하므로 ID 토큰만 검증하는 구조는 선행 설계가 필요 (2026-09-04)
- [[작업노트/AppStore/실물을 흉내 내는 앱의 상표 위험|실물을 흉내 내는 앱의 상표 위험]] — 폴라로이드 흰 테두리는 **classic border logo** 등록 상표이고 재판 계류 중, 실질 위험은 소송이 아니라 **App Store 신고 한 건** (2026-08-26)

### 도구
- [[작업노트/도구/macOS 파일 접근 권한과 휴지통|macOS 파일 접근 권한과 휴지통]] — **launchd 작업은 `(로컬 경로)`을 못 읽는다**(exit 127 `can't open input file`) → 실행본을 `(로컬 경로)`에 둔다(2026-09-23). Claude Code 셸에서 `(로컬 경로)` 읽기가 `Operation not permitted`면 **터미널 앱의 Full Disk Access 부재**다. `sudo`·root osascript도 같이 막히고, Finder를 AppleScript로 시켜도 커널 로그의 deny 주체는 osascript. 그 상태의 "권한 없음"은 진단 근거가 아니다. Finder "being used by another task"는 `killall Finder`로 해소 (2026-09-22)
- [[작업노트/도구/Orca 설정 변경|Orca 설정 변경]] — 설정은 `orca-data.json`의 `settings`지만 **앱 실행 중 파일 편집은 메모리 값으로 덮어써진다**. Orca 안에서 연 세션(`TERM_PROGRAM=Orca`)은 앱을 못 끄므로 `orca computer`로 Settings UI를 조작 — **React 버튼은 `--element-index`가 안 먹고 `--x --y` 좌표 클릭**, scroll 대신 섹션 접기, index는 매번 stale. UI Zoom 110%·터미널 16px가 홍 기준 (2026-09-22)
- [[작업노트/도구/스택 PR과 Squash Merge|스택 PR과 Squash Merge]] — 앞 PR squash 뒤 뒤 PR은 **`CONFLICTING`일 때만** `rebase --onto` + force push. `CLEAN`이면 그대로 머지해도 squash는 자기 파일만 담는다(PR 화면에만 앞 변경이 섞임). **잘라낼 기준점은 끝 커밋 ancestor 검사로 판정하지 말 것** — 중간 커밋에서 딴 브랜치를 놓친다. auto mode는 `--force-with-lease`도 `[Git Destructive]`로 막는다 (2026-09-15)
- [[작업노트/도구/GitHub 브랜치 rename과 열린 PR|GitHub 브랜치 rename과 열린 PR]] — 브랜치 rename API는 **그 브랜치를 head로 둔 열린 PR을 `head_ref_deleted`로 닫고 reopen도 안 된다**(base로 쓰인 PR은 따라온다). 이름을 바꿔야 하면 새 이름 push → 새 PR → 옛 PR 닫기 (2026-09-12)
- [[작업노트/도구/Aside 브라우저 폼 자동 입력|Aside 브라우저 폼 자동 입력]] — **로그인 계정의 웹 폼은 `aside repl`로 직접 채울 수 있다**(`aside exec`는 OpenAI 402로 죽음). React 폼 지뢰 셋 — **react-select는 control 클릭으로 안 열리고** `input[role=combobox]`에 `focus()`+`ArrowDown`이어야 하며 **옵션 검색을 전역으로 하면 다른 select의 메뉴를 눌러 엉뚱한 필드에 값이 들어간다**, **`locator.fill()`은 `90000`→`90,000`처럼 값을 포맷하는 입력에서 예외를 던져 루프를 통째로 끊는다**(네이티브 setter + `input` 이벤트로), **contenteditable은 클릭해도 activeElement가 BODY**라 Range로 캐럿을 직접 놔야 `keyboard.type`이 먹는다. **캡처에 쓸 때는 따로 조심** — `aside repl`은 **호출마다 세션이 새로 뜨므로**(pwd·탭 목록이 갈린다) 탭 열기·조작·스크린샷을 **한 호출에서** 끝내야 하고, 나누면 `attachActiveBrowserTab()`이 엉뚱한 탭을 잡는다. 저장 경로가 세션 디렉터리 밖이면 `escapes the session directory`이고, `setViewportSize`·`screenshot({clip})`·요소 `.screenshot()`은 아예 없다 — 전체를 찍고 셸에서 자른다 (2026-09-16)
- [[작업노트/도구/Claude 아티팩트 공개 배포|Claude 아티팩트 공개 배포]] — **아티팩트는 뷰어에게 claude.ai 로그인을 요구하므로 계정 없는 사람에게 못 보여준다.** 정적 호스팅으로 옮길 땐 **아티팩트가 publish 시 자동으로 씌우던 `<meta name="viewport">`를 직접 붙여야** 한다 — 빠뜨리면 폰에서 980px 가상 뷰포트로 축소돼 «화면 절반 폭 기둥»이 된다. `assets` capability가 없으면 미디어는 `data:` URI뿐이고 **base64 33% 팽창으로 실질 예산은 12MB**. 삭제 API는 없음. 곁가지로 **부모 `z-index`가 만든 쌓임 맥락이 자식 클릭을 막는 패턴**과 **append 직후 클래스 변경 시 transition이 생략되는 문제**(`void el.offsetWidth`) (2026-09-10)
- [[작업노트/도구/CodeRabbit 리뷰 자동 반영|CodeRabbit 리뷰 자동 반영]] — 코멘트 첫 줄이 등급(`_🟠 Major_`)이고 `cr-indicator-types`·`🤖 Prompt for AI Agents` 블록이 숨어 있다. **«아직 안 고친 것» 판정은 REST에 필드가 없어 GraphQL `reviewThreads.isResolved`뿐**이고, **«마지막 코멘트가 나»로 보면 안 된다** — CodeRabbit이 답글에 다시 답글을 달아 의도적으로 남긴 지적이 매번 자동수정 대상이 된다. 실측 분포는 Major 10·Minor 13, `profile: chill`이라 **Critical은 전례 없음** (2026-09-09)
- [[작업노트/도구/MacBook 상시 가동|MacBook 상시 가동]] — **전원 연결 시 `sudo pmset -c sleep 0` + 외부 모니터 꽂고 덮개 닫기면 맥미니처럼 상시 가동된다.** Claude Code 세션이 거는 `caffeinate -i -t 300`(부모 `claude`)을 상시 잠자기 방지로 착각하지 말 것. FileVault On이면 재부팅 뒤 자동 로그인이 안 돼 로그인 세션 기반 자동화가 멈춘다. 배터리는 macOS 26.4+ 충전 한도 80% (2026-09-12)
- [[작업노트/도구/소셜 게시 API와 자동화 정책|소셜 게시 API와 자동화 정책]] — **Instagram·Threads는 공식 API로 무료 자동 게시, X는 건당 과금(링크 포함 $0.20), Reddit은 초안까지만.** Threads는 본인 tester 토큰이면 App Review가 필요 없다. 자동 답글·DM은 네 곳 모두 "상대가 먼저 반응"한 경우만, X의 AI 답글은 사전 승인 사항. 서버 RAM 1GB라 헤드리스 브라우저 게시는 불가 (2026-09-12)
- [[작업노트/도구/KBO 하이라이트 클립 수급과 릴스 컷|KBO 하이라이트 클립 수급과 릴스 컷]] — **KBO 경기 영상은 40초 미만·비상업이면 SNS 2차 창작 허용**(티빙 2024~2026). 클립은 `youtube.com/@TVINGSPORTS/videos`의 경기별 하이라이트(경기 뒤 30~60분, 제목 `경기|Game|Match` 세 변형)를 `uvx yt-dlp@latest --js-runtimes node:<mise shim>`로 — **JS 런타임이 없으면 구간 다운로드가 403 → `ffmpeg exited with code 8`**, `--download-sections`는 **같은 파일명이면 새 구간을 무시**한다. **Oracle 서버는 `Video unavailable`(IP 차단)이라 Mac에서만**, launchd는 `(로컬 경로)`을 못 읽어 실행본을 `(로컬 경로)`로. 장면은 자동 자막(조각 시각 `tOffsetMs`) 키워드 + ebur128 음량으로 좁히되 **기록 잡담 «시즌 11홈런»·조사 «와»를 거르고**, 캐스터 «홈런» 콜은 타격 5~10초 뒤라 16초 앞에서 시작. 문안 사실은 기사와 `claude -p` 두 번 대조 (2026-09-23)
- [[작업노트/도구/Instagram 콘텐츠 발행 API|Instagram 콘텐츠 발행 API]] — **게시물 자동 발행은 된다.** `graph.instagram.com`(Instagram Login) 3단계: `/media` 컨테이너 → `status_code=FINISHED` 폴링 → `/media_publish`. **multipart가 없어 `image_url`(공개 HTTPS)만 받으므로 imgbb 중계가 필수**, **FB 페이지는 불필요**(그건 `graph.facebook.com` 경로), 한도는 24시간 100건·캐러셀 1건. 한도와 별개로 «API access blocked» 임시 차단이 있어 발행 간격 600초. **되는 건 발행뿐 — 프로필(소개글·링크)·스토리 하이라이트는 쓰기 API가 없고, 무작위 DM도 Messaging API로는 불가**(사용자가 먼저 건 대화 + 24시간 창). **릴스도 `video_url`만 — Instagram Login 토큰은 resumable upload를 400 code 100으로 거절**해 litterbox 임시 호스팅으로 넘긴다. **게시된 캡션은 API로 못 고친다**(`comment_enabled`만). 계정명이 `yagu.3cut`로 바뀌어 워터마크 `HANDLE` 수정 (2026-09-23)
- [[작업노트/도구/Instagram 발행 계정·앱·토큰 셋업|Instagram 발행 계정·앱·토큰 셋업]] — **계정·앱·키를 0에서 만드는 순서**: FB 계정+페이지 → IG 프로페셔널 전환 → Meta 개발자 앱 → [Instagram]→[API 설정]에서 `instagram_business_basic`+`instagram_business_content_publish` 권한·장기 토큰(60일) → `me?fields=user_id` → ImgBB 키 → `.env`(`IG_USER_ID`·`IG_ACCESS_TOKEN`·`IMGBB_API_KEY`). **만료된 토큰은 refresh 불가·재발급**, `403 (code 4)`는 실패 아님, **Threads 자동 게시는 계정 영구 삭제 사례(2026-06-12)** → 수동만. 출처 AISW 강의 PDF 2026-07-05 (2026-09-22)
- [[작업노트/도구/공공데이터포털 오픈API|공공데이터포털 오픈API]] — **키 없이 먼저 찔러본다.** 더미 키의 `SERVICE_KEY_IS_NOT_REGISTERED_ERROR`는 «경로가 맞다»는 뜻이고 `해당 오픈API 서비스가 없거나 폐기됨`은 «경로 오타»다. **`apis.data.go.kr`는 CORS로 안 막힌다** — Origin을 반사하므로 브라우저에서 직접 호출된다(프록시 근거는 키 노출·캐싱). 에러는 200이 아니라 **403 + XML**. FullData 오퍼레이션이 있으면 일 1,000회 한도가 무의미해진다. **키가 `undefined`여도 응답은 똑같이 「등록되지 않은 서비스키」**이므로 `KEY?.length`부터 확인할 것. Fastify 프록시 최소 구성과 `_type=json` 응답 모양(`response.body.items.item[]`) 포함 (2026-09-09)
- [[작업노트/도구/한국 지도 API 선택|한국 지도 API 선택]] — **네이버 지도 API는 신규로 못 쓴다**(2025-05-22 신청 종료·07-01 무료 종료). 기본값은 카카오맵(개인 일 20만, 단 2026-07-21부터 '첫 활성화 앱'에만 무료), 공공 대안은 VWorld. 지도 라이브러리는 전부 브라우저 전용이라 SSR에서 분리 필요. **디자인 제약도 여기 있다 — 카카오맵은 베이스맵 스타일을 바꿀 수 없고**(custom style API 없음, `MapTypeId` 전환뿐) **kakao 로고·축척바는 `setCopyrightPosition`으로 위치만 고르고 삭제 불가**, **상호 라벨·오렌지 POI 아이콘이 타일에 박혀 있어 반투명 흰 pill 핀은 실물에서 사라진다**. `MarkerClusterer`는 `Marker` 대상이라 HTML `CustomOverlay` 핀에는 안 붙는다 (2026-09-16)
- [[작업노트/도구/LLM 문항 생성 검증|LLM 문항 생성 검증]] — **검증기가 통과시켰다는 게 품질의 근거가 못 된다.** 복제 검사 4자 샘플링은 사이 구간을 놓쳐 배포된 심화 7문항이 뒤늦게 드러났고(전수는 shingle 집합으로), 블라인드 사본은 **선지를 섞지 않으면 무력화**된다(기본 세트2는 정답 60/60이 1번). 실질 중복은 자료 Jaccard가 아니라 **같은 주제+정답 문장 유사도**로 잡는다 (2026-09-05)
- [[작업노트/도구/MyWiki 구조 lint와 pre-commit 훅|MyWiki 구조 lint와 pre-commit 훅]] — 커밋이 막혔을 때: **훅은 클론마다 `git config core.hooksPath .githooks`로 켠다**, index는 훅이 생성·stage하니 손대지 않는다, log는 새 4줄 형식만·아카이브는 가장 오래된 항목을 같은 커밋의 `log-YYYY-MM.md`로. 메시지별 조치 표 (2026-09-05)
- [[작업노트/도구/Figma MCP|Figma MCP]] — **`use_figma` 스크립트는 한 트랜잭션이라 마지막 줄이 던지면 앞 mutation 전부 롤백**. `query()` 셀렉터 값의 `/`(`icon/chevron-right`)는 파서 에러이고 **공백(`[name=3D slots]`)도 안 잡힌다** — `findAllWithCriteria`·`children.find`로. 오버레이 화면은 기존 Scrim·시트를 `clone()`. **플러그인 환경 폰트 목록은 파일이 쓰는 폰트와 다르다**(시안 전체가 Pretendard인데 `does not exist`), 대시는 `strokeDashPattern`이 아니라 **`dashPattern`**, `layoutPositioning='ABSOLUTE'`는 **부모가 auto-layout일 때만**, **`get_screenshot`은 그림자 패딩을 포함해 돌려주므로 합성 좌표에서 `(original-frame)/2`를 빼야** 한다 (2026-09-16)
- [[작업노트/도구/공공 문화유산 이미지 조달|공공 문화유산 이미지 조달]] — 국가유산청 1유형·museum.go.kr 1유형·Commons API로 라이선스 기계 확인. **표준영정은 화가 저작권이라 불가** (2026-09-01)
- [[작업노트/도구/Tuist|Tuist]] — 루트 판정은 `.git` 또는 `Tuist/`(없으면 `Couldn't locate the root directory`), **빈 `Tuist/Config.swift`는 `manifest is corrupted … not valid JSON`으로 죽는다**. watchOS 앱 임베드는 iOS 타깃 `dependencies: [.target(name:)]` 하나로 끝. **Firebase 같은 바이너리 xcframework SPM도 `.external`로 그냥 붙고 `productTypes`는 안 건드는 게 안전**, `Tests` 타깃엔 전용 스킴이 안 생긴다 (2026-09-05)
- [[작업노트/도구/GA4와 Amplitude 앱 계측|GA4와 Amplitude 앱 계측]] — **앱용 GA4는 Firebase Analytics SDK가 유일**하고 `FirebaseApp.configure()`는 plist가 없으면 **앱을 죽인다**(fatalError). **plist를 gitignore하면 CI 빌드에 GA4가 빠진다.** 화면 자동수집(`FirebaseAutomaticScreenReportingEnabled`)은 SwiftUI에서 `UIHostingController`로 뭉개지니 끈다. 계측은 뷰가 아니라 **리듀서의 성공 액션**에, 그것도 `case` 진입부가 아니라 **성공한 `try` 다음 줄**에 — `guard`와 `catch`를 안 지나면 실패가 전환으로 세어진다. **track이 붙은 액션을 아무도 `send`하지 않으면 지표는 에러 없이 0이다** (2026-09-08)
- [[작업노트/도구/LLM 콘텐츠 생산 파이프라인|LLM 콘텐츠 생산 파이프라인]] — 에이전트로 해설 1,742·노트 154 하루 생산. 원문 한정·복제 검증기·표본 검수 구조. **자료를 600자에서 잘랐더니 에이전트가 정답을 의심** — 재료는 자르지 말고 답지를 진실로 (2026-09-01)
- [[작업노트/도구/Notion MCP|Notion MCP]] — **전용 삭제 도구는 없지만 본문의 `<page>`/`<database>` 태그를 `update_content`+`allow_deleting_content`로 빼면 하위 페이지·연결 뷰가 삭제된다**(플래그 없이 먼저 보내면 삭제 대상 목록을 알려줌). DB 행은 안 되니 보관 페이지 + `notion-move-pages`(100개까지). 옮기기만 한 차트 뷰는 무료 차트 쿼터를 계속 차지한다. `notion-search`의 sort/filters는 개인 플랜에서 거부, `formulaCode://`는 fetch 불가(수식은 덮어쓰기만). `ALTER COLUMN … SET SELECT`에 기존 옵션을 다시 나열하면 ID 보존. 마켓플레이스 템플릿 DB는 relation 한쪽이 끊겨 오니 풀리는 쪽에서만 연결 (2026-09-01)
- [[작업노트/도구/Claude 클라우드 루틴 운영|Claude 클라우드 루틴 운영]] — **루틴 SUCCEEDED ≠ 작업 수행됨.** CCR 샌드박스에선 detached HEAD가 진짜 최신이고 `refs/remotes/origin/main`은 낡은 캐시일 수 있다 — 샌드박스의 merge-base 실패를 "이력 단절"로 믿으면 루틴이 push 없이 self-abort한다(일간 08-29 미생성 건). 진단은 `RemoteTrigger list_runs`→`get_run_log` (2026-08-30)
- [[작업노트/도구/Ghostty 설정|Ghostty 설정]] — macOS config는 두 군데(`(로컬 경로)`·App Support `config.ghostty`, `Cmd+,`는 후자), 테마명은 `Catppuccin Mocha`처럼 대문자+공백. **cmux 세션은 `GHOSTTY_RESOURCES_DIR`가 cmux 것이라** 진짜 Ghostty.app 검증도 엉뚱한 테마 폴더를 본다 — `env -u`로 벗겨야 함 (2026-08-30)
- [[작업노트/도구/historyexam 자료실 수집|historyexam 자료실 수집]] — 한능검 기출 PDF 수집 절차: 목록 `fn_goDetail` ID → `POST /pst/view.do`(GET은 ERROR) → `FileDown.do?atch_file_id=`. **자료실은 최근 회차만 보관하고 옛 글은 ID 프로브로도 안 열린다** — 새 회차 게시 즉시 받아야 한다 (2026-08-30)
- [[작업노트/도구/Maestro|Maestro]] — 모바일 UI 자동화. 텍스트 셀렉터는 접근성 트리 의존(Flutter 텍스트필드 힌트는 안 잡힘), **좌표 탭은 요소를 안 기다려** 전환 중 허공을 때리고도 COMPLETED — 좌표 앞엔 반드시 대기. 실패 아티팩트는 `(로컬 경로)` (2026-08-27)
- [[작업노트/도구/Obsidian git 동기화|Obsidian git 동기화]] — obsidian-git 자동 백업은 **`autoCommitOnlyStaged`가 켜져 있으면 무동작**(Obsidian 편집은 stage를 안 만든다), 알림도 없다. 클라우드 루틴은 원격만 보므로 push 안 된 `[x]`는 `[ ]`로 취급돼 이월되고, 로컬 dirty 파일은 모든 세션의 `pull --rebase`를 막는다 (2026-08-27)
- [[작업노트/도구/fastlane 로컬 실행 환경|fastlane 로컬 실행 환경]] — 로컬 배포가 깨지는 건 대개 fastlane이 아니라 **`bundle exec`이 심은 `RUBYOPT`가 xcodebuild 자식까지 새는 것**. 시스템 ruby 2.6이 Homebrew ruby 4의 bundler를 로드하다 죽는데, **stderr는 gym 로그를 안 거쳐서** 원인 불명 build error로 보인다. `env -u RUBYOPT`로 직접 아카이브해보면 1분 만에 판별된다 (2026-08-26)
- [[작업노트/도구/fastlane 배포 알림|fastlane 배포 알림]] — 배포 알림이 안 오는 건 대개 실패가 아니라 **안 불린 것**. `notify_discord`는 `DISCORD_WEBHOOK_URL`이 없으면 조용히 `return`하고(webhook은 gitignore된 `fastlane/.env`에만 있어 **CI는 원래부터 무음**), 나중에 만든 `platform :macos` lane에는 호출 자체가 빠져 있었다. 확인은 fastlane summary에 `curl -H` 스텝이 있는지로 (2026-09-01)
- [[작업노트/도구/macOS 앱 QA 자동화|macOS 앱 QA 자동화]] — 시뮬레이터가 없는 맥 앱을 `CGEvent`·`screencapture -l`로 조작·캡처해 QA한다. **`mouseEventClickState`를 1로 안 넣으면 클릭이 통째로 무시**되는데 호버는 되므로 앱 버그로 오진하기 쉽고, 판정은 눈이 아니라 픽셀 rgb로 해야 오진이 사라진다 (2026-08-26)
- [[작업노트/도구/Vercel 배포|Vercel 배포]] — Git 연동은 레포 루트 기준, 서브폴더 사이트는 Root Directory 필수(빠지면 통째로 404), 배포 URL은 보호 때문에 401/302라 alias로 점검 (2026-08-19)
- [[작업노트/도구/문서 포맷 파싱|문서 포맷 파싱]] — `.pptx`는 zip+OOXML, `.key`는 zip+IWA(snappy 압축 protobuf) — 앱 없이 텍스트 추출 가능. PDF 메타데이터는 날짜 추정 근거 (2026-08-19)
- [[작업노트/도구/Claude Code 설정과 훅|Claude Code 설정과 훅]] — **설정이 안 먹으면 문법보다 로딩 시점을 의심한다.** `paths: "**/*"`는 항상 로드가 아니라 조건부(빼야 세션 시작 로드), path-scoped는 Read에만, `allowed-tools`는 제한이 아니라 사전승인, 복합 명령은 조각별 매칭, 새 `settings.json`은 그 세션에서 안 먹음. **스킬 라우팅은 기본값과 다른 것만 적는다**(스킬 설명 복제는 no-op), **플러그인 정리는 `claude plugin uninstall`**로(캐시만 지우면 유령 항목), `disable-model-invocation: true`는 버그가 아니라 «사람이 연다»는 설계. **앱을 지워도 그 앱의 플러그인 훅은 남는다** — `hook error … gk: No such file` 같은 «없는 경로» 훅 에러는 `(로컬 경로)`에서 찾아 `claude plugin uninstall` (2026-09-22)
- [[작업노트/도구/HTML을 이미지로 렌더링|HTML을 이미지로 렌더링]] — 크롬·ImageMagick 없이 **Swift + WKWebView 스냅샷**으로 HTML을 정확한 크기 PNG로. `document.fonts.ready` 뒤에 찍어야 폰트가 붙고, `zoom`을 쓰면 `getBoundingClientRect`도 배율로 돌아온다. 고정 캔버스 넘침은 `scrollHeight`로 못 잡는다. Pillow 경로: Jua는 앱 저장소 폰트가 원본(gstatic TTF는 폴백), **Dia.app은 헤드리스 안 됨**, 4:5는 9:16 축소가 아니라 별도 컴팩트 분기, ffmpeg `xfade` 체인으로 슬라이드쇼 (2026-09-04)
- [[작업노트/도구/Claude Code 사용량과 한도|Claude Code 사용량과 한도]] — **한도는 하나가 아니다** — Fable 5는 전체 한도와 별개의 자체 쿼터라 `/model`로 Opus 5에 내려오면 풀린다(플랜 문제 아님). 사용량은 `(로컬 경로)`로 실측(중복 제거 키 `(message.id, requestId)`), 비용 절반 이상이 **캐시 읽기**(1h 쓰기 2×·읽기 0.1×) (2026-08-22)
- [[작업노트/도구/셸 초기화와 터미널 통합|셸 초기화와 터미널 통합]] — **`.zshrc`에 쓴 것이 최종이 아니다.** 터미널 앱(cmux)의 셸 통합이 rc가 다 돈 뒤 **첫 프롬프트 `precmd` 훅**에서 내 `claude()`를 자기 래퍼로 되돌린다 — 훅이 자기를 제거하니 "source하면 되는데 새 탭이면 또 안 됨". `type -w`는 함수 존재만, 본문은 `functions`로 (2026-08-24)
- [[작업노트/도구/커넥터 권한과 데이터 경계|커넥터 권한과 데이터 경계]] — **커넥터가 보여주는 범위 ≠ 내 것.** `list_calendars`는 구독한 남의 캘린더까지 같은 모양으로 준다 — 자동화가 "전부 조회"하면 남의 일정이 내 계획에 들어온다. 소유 판정은 `primary`/`accessRole`·ID 화이트리스트로, 내용 추론으로 하지 않는다. 규칙이 두 곳에 있으면 둘 다 고친다 (2026-08-25)
- [[작업노트/도구/공공데이터 라이선스|공공데이터 라이선스]] — **공공누리 유형은 시작이지 끝이 아니다.** 그 위에 저작권법 §24조의2(국가가 업무상 작성·공표 → 허락 없이 이용 가능, 상업적 제한 없음)가 있어 **유형 표시가 법률상 자유이용을 줄이지 못한다.** 국가기관 저작물의 4유형은 오히려 단서 — *데이터 안에 제3자 저작물이 섞였다*는 신호다. 봐야 할 건 딱지가 아니라 **누가 그 콘텐츠 권리를 실제로 갖는가** (2026-08-26)
- [[작업노트/도구/Hermes cron 운영|Hermes cron 운영]] — **Hermes cron이 조용히 죽는 원인은 대부분 무료 LLM 쿼터**(Ollama 주간 한도는 롤링 — 매일 도는 120b 잡이 예산을 재소진해 429 고착). 브리핑류 잡은 `hermes cron edit --script <name>.sh --no-agent`로 제로 토큰 전환이 유일한 영구 해결. 에이전트의 낡은 메모리·channel_prompts는 분석 자체를 오염시킨다(존재하지 않는 경로를 grep하다 이미 끝난 결정을 재논의) (2026-08-29) · **스크립트가 위키 경로를 하드코딩하므로 볼트 폴더 재편 시 같은 턴에 수정** — 09-02 재편 뒤 07:20 감시 오경보 사흘 (2026-09-04)
- [[작업노트/도구/유튜브 임베드 판정|유튜브 임베드 판정]] — **oEmbed 200은 임베드 허용이 아니다**(사실상 존재 판정). 근거는 `/embed/<id>` 안의 `embedded_player_response` → `videoFlags.playableInEmbed`. 그리고 **"전부 OK"는 검증이 아니다** — 항상 OK만 뱉는 판정기와 구별되지 않으니 차단 대조군을 먼저 확보한다 · **`timedtext` 자막은 서버에서 200-빈본문으로 막혔고** 재생목록은 100개가 벽이라 채널 내 검색으로 우회한다 (2026-08-27)
- [[작업노트/도구/dio multipart|dio multipart]] — JSON 파트는 `DioMediaType('application','json')`을 명시해야 Spring `@RequestPart`가 읽는다(기본 text/plain→415). 보낸 파트 되읽기는 finalize 1회용이라 **`clone().finalize()`** (2026-08-27)
- [[작업노트/도구/build_runner 부분 빌드|build_runner 부분 빌드]] — **`--build-filter`가 필터 밖의 기존 `.g.dart`를 지운다.** 부분 빌드 직후 무관 DTO에서 `uri_has_not_been_generated`가 쏟아지면 전체 build가 복구다 (2026-08-27)
- [[작업노트/도구/gstack|gstack]] — Garry Tan의 Claude Code 스킬 팩. **`./setup`이 전역 `(로컬 경로)`를 건드린다** — 기본값은 prefix 없는 `/review`·`/ship`이라 `--prefix`로 격리하고, `settings.json`에 Stop 훅(`gstack-timeline-stop`)을 묻지 않고 심는다. **Aside가 깔려 있으면 자동으로 primary 브라우저**, 번들 Chromium 274MB는 `GSTACK_SKIP_PLAYWRIGHT=1`로 생략 가능 (2026-09-16)
- [[작업노트/도구/Jira MCP|Jira MCP]] — Atlassian Rovo로 Jira 읽기·쓰기. **하위 작업(Subtask)은 스프린트에 연결할 수 없다** — 상위 작업의 스프린트를 그대로 따르므로 옮기려면 상위를 옮겨야 한다(상태 전환은 하위 작업도 된다). 스프린트 필드는 `customfield_10020`에 **id 정수**, 전환 id는 `getTransitionsForJiraIssue`로 확인(SSH 완료 = 41). 쓰기 도구는 분류기에 한 번 튕길 수 있으니 마지막에 JQL로 실제 상태를 다시 읽는다 (2026-09-16)

### Flutter
- [[작업노트/Flutter/Flutter iOS TestFlight 업로드|Flutter iOS TestFlight 업로드]] — fastlane 없이 **API 키 하나로** 남의 팀에 올린다: `flutter build ios --no-codesign` → `xcodebuild archive`/`-exportArchive`(`destination=upload`)에 `-authenticationKey*`. **로컬 배포 인증서 0개여도 클라우드 서명으로 통과**. `--dart-define`은 `Generated.xcconfig` `DART_DEFINES`(base64)로 검증, `grep -q`+`pipefail`은 거짓 실패 (2026-09-15)
- [[작업노트/Flutter/위젯 테스트와 ListView 지연 빌드|위젯 테스트와 ListView 지연 빌드]] — 기본 뷰포트 800×600에서 `ListView` 아래쪽 자식은 **빌드조차 안 돼** `find.text`가 못 찾는다 → `tester.view.physicalSize`로 세로를 키우고 `addTearDown(tester.view.reset)`. 생성자에서 load하는 ViewModel + Fake는 `pump()` 한 번이면 로드 뒤 (2026-09-12)
- [[작업노트/Flutter/Google ML Kit 텍스트 인식|Google ML Kit 텍스트 인식]] — 한국어 모델은 앱이 직접 싣는다, iOS 15.5·arm64 시뮬레이터 불가
- [[작업노트/Flutter/드래그 편집 UI의 좌표계와 경계|드래그 편집 UI의 좌표계와 경계]] — **화면 단위 상수를 원본 좌표로 옮기면 배율의 역수만큼 커져** 하한이 상한을 넘고, `num.clamp`는 그때 `ArgumentError`다. **제스처 콜백의 예외는 삼켜져** 크래시가 아니라 「이 핸들만 안 움직임」으로 나타난다. `Clip.none`은 그리기만 허용하고 히트 테스트는 부모 밖으로 안 간다 (2026-09-17)
- [[작업노트/Flutter/카카오 로그인 Android 키 해시|카카오 로그인 Android 키 해시]] — 동의 화면 Continue 무반응은 manifest placeholder(환경변수) 또는 키 해시 불일치. 디버그 키스토어가 둘(`(로컬 경로)` vs flutter-env), 등록된 건 flutter-env 쪽. `KakaoSdk.platformInfo.origin`으로 APK 해시 확인
- [[작업노트/Flutter/Semantics와 GestureDetector tap 액션|Semantics와 GestureDetector tap 액션]] — up·cancel 핸들러만 있어도 tap 액션이 실린다, tap은 Semantics가 소유
- [[작업노트/Flutter/머티리얼 날짜 피커와 ko 로케일|머티리얼 날짜 피커와 ko 로케일]] — `showDatePicker` 입력 모드의 안내(`dateHelpText`=`yyyy.mm.dd`)와 파서(`parseCompactDate`→`DateFormat.yMd('ko').parseStrict`, `1995. 3. 2.`만 허용)가 **어긋나 안내대로 치면 전부 거부**된다. 로케일 문제라 iOS·Android 동일 — 직접 입력은 우리 `TextField`로 받고 달력은 `calendarOnly`로 연다 (2026-09-03)
- [[작업노트/Flutter/flutter analyze와 분석 서버|flutter analyze와 분석 서버]] — `flutter analyze`가 `FormatException`·`analysis server exited with code 255`로 죽어도 **`dart analyze`는 같은 규칙으로 돈다**. 소스 한 줄도 안 가리키고 죽으면 도구 문제다 (한글 경로에서 실측, 2026-09-02)
- [[작업노트/Flutter/flutter run 프로세스와 앱 수명|flutter run 프로세스와 앱 수명]] — `flutter run` 호스트가 죽어도 시뮬레이터 앱은 살지만 **print 로그와 VM Service는 못 살린다**(unified log에도 안 잡힘, attach 불가) — 로그가 필요하면 재실행이 답이고 Keychain 세션은 유지된다 (2026-08-29)
- [[작업노트/Flutter/제스처 아레나와 스크롤 경쟁|제스처 아레나와 스크롤 경쟁]] — 스크롤러 안 pan은 세로를 빼앗긴다 → `addAllowedPointer`에서 즉시 `resolve(accepted)`로 선점. `Container`에 `alignment`를 주면 부모를 채워 바깥 `Align`이 무력화되고 탭 표면이 행 전체가 된다 (2026-08-27)
- [[작업노트/Flutter/iOS 접근성 트리 공백|iOS 접근성 트리 공백]] — 특정 화면(병원 검색 오버레이 계열)에서 접근성 트리가 통째로 빈다. XCTest 도구·VoiceOver 둘 다 못 읽는다 — 자동화가 한 화면에서만 실패하면 셀렉터 전에 `maestro hierarchy`부터 (2026-08-27)

## 자주 쓰는 요청

- `이거 작업노트에 기록해줘` — 지금 대화의 작업 경험을 주제 파일에 append + 프로젝트 README `배운 것` 링크
- `<프로젝트> 작업 정리해줘` — Sync 중 배운 것을 자동 추출해 작업노트에 반영
- `작업노트 미분류 정리해줘` — 미분류 항목을 주제 파일로 이동
