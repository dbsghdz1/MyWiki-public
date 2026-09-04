---
type: project
status: active
created: 2026-08-31
updated: 2026-09-05
related_wiki: []
---

# WristNote 개발 기록 2026-08-31 — 착수, v1 골격

홍의 착수 지시("기록하고 개발해") 후 야간 자율 작업. 목표는 [[프로젝트/개인/WristNote/README|README]]의 M1: 워치 녹음 → WCSession 전송 → 전사 → 요약 → 목록 표시의 v1 골격.

## 결정

- **스캐폴딩: Tuist 4.202.5** (로컬에 이미 설치). iOS 앱 + watchOS 앱 임베드 구조.
- **전사·요약 모두 온디바이스** — iOS 26 `SpeechAnalyzer` + `FoundationModels`. 서버·API 키 없이 v1 성립. SpeechAnalyzer 사용법은 [[작업노트/Apple/온디바이스 음성 인식과 번역|Subly 스파이크 실측]]을 재사용.
- **워치 백그라운드 녹음**: `UIBackgroundModes: [audio]` + 활성 `AVAudioSession(.record)`. 시뮬레이터로는 화면 꺼짐 지속을 판정할 수 없으므로 **실기기 M0은 별도** — 이번 작업의 검증 범위는 시뮬레이터 페어에서의 파이프라인 관통까지.

## 진행 (2026-08-31 밤 ~ 09-01)

1. **스캐폴딩** — Tuist `Project.swift`(iOS `WristNote` + watchOS `WristNoteWatch` 임베드). 루트 판정·빈 Config.swift 함정 → [[작업노트/도구/Tuist|Tuist]].
2. **API 확정** — 웹 대신 로컬 SDK `.swiftinterface`(iPhoneSimulator26.5.sdk)에서 `SpeechAnalyzer.analyzeSequence(from:)`·`SpeechTranscriber(locale:preset:)`·`AssetInventory.reserve/assetInstallationRequest`·`LanguageModelSession.respond(to:generating:)`·`@Generable/@Guide` 시그니처 확인.
3. **구현** — 워치: `WatchRecorder`(AAC 16kHz mono 32kbps) + `WatchSession`(`transferFile`, 성공 시 로컬 삭제) + `RecorderView`. 아이폰: `PhoneSession`(수신 즉시 이동) → `MeetingStore`(JSON 영속, 전사→요약 파이프라인, 재시작 시 미완료 재개) → `TranscriptionService`(SpeechAnalyzer → SFSpeechRecognizer 온디바이스 → 서버 순 폴백) → `SummaryService`(FoundationModels, 3000자 청크) → 목록·상세·공유.
4. **빌드** — `xcodebuild` iOS 시뮬레이터 빌드 성공(워치 앱 임베드 포함).
5. **아이폰 파이프라인 검증** — TTS 한국어 회의 음성(34초)을 `WRISTNOTE_IMPORT`로 투입.
   - `SpeechAnalyzer`: `supportedLocales` 빈 배열 → `not subscribed to transcription.ko` 실패.
   - `SFSpeechRecognizer` 폴백: 권한 프롬프트(TCC.db 직접 삽입으로 통과) → `kLSRErrorDomain 300` — 온디바이스·서버·영어 전부 동일. **시뮬레이터 전사 불가 확정.**
   - `WRISTNOTE_TRANSCRIPT` 주입 훅 추가 → **FoundationModels 요약 성공**: 제목 "새로운 앱 출시 일정 및 마케팅 예산 결정", 결정사항 3건, 액션아이템 "지현: 회의록 정리" — 전부 정확.
6. **워치 검증** — `simctl pair`로 Series 11 + iPhone 17 Pro 페어. `WRISTNOTE_AUTOTEST=1`로 8.6초 녹음 → 25KB 파일 → `transferFile` → 워치 `didFinish error=nil`. **아이폰 `didReceive` 무호출** — Apple 문서상 시뮬레이터 미지원 → [[작업노트/Apple/WatchConnectivity와 워치 녹음|작업노트]].
7. **한국어 전사 품질 실측(맥 호스트)** — 스크래치 SPM CLI로 같은 `SpeechAnalyzer` ko_KR 모델 실행. 0.35초에 전사되지만 "회의"→"매일" 류 오류가 핵심 명사에 집중. 이 전사를 요약에 넣으니 **모델이 보정 없이 그대로 요약** → 전사 품질이 병목. → [[작업노트/Apple/온디바이스 음성 인식과 번역|작업노트]].
8. **커밋** — `f8c4bd3` "WristNote v1 골격". GitHub 미생성.

## 2026-09-02 — 실기기 M0 → 1.0 제출 (홍 온라인, 대화형)

9. **실기기 빌드** — 홍이 Xcode로 아이폰·워치에 설치(워치 개발자 모드는 설정 → 개인정보 보호 및 보안, 안 보이면 USB 연결 후 Devices 창에서 신뢰). 워치 녹음·`transferFile`·아이폰 `didReceive` **전부 동작** — 시뮬레이터 한계였던 전송이 실기기에선 몇 초 안에 배달.
10. **전사 "빈 텍스트" 실패 → 세 겹 원인** — `devicectl device copy from`으로 아이폰 컨테이너를 꺼내 보니 파일이 24,588바이트·0.06초. ① `stop()` 직후 전송(`9bcdaff`로 didFinish 뒤로) → ② 그래도 `fileSize=28` 고정: 실기기 AAC 인코더 무출력, 48kHz도 동일(`fc903de`) → 코덱 사다리 IMA4('fmt?' 실패)→**µ-law**(`44cd002`) → ③ 그래도 `fileSize=4096`: 마이크 권한 미요청 — watchOS는 `record()`가 프롬프트를 안 띄움 → `requestRecordPermission`(`61aae30`). 그 뒤 홍 "좋았어 확인했어". → [[작업노트/Apple/WatchConnectivity와 워치 녹음|작업노트]]
11. **UI** — 탭바(녹음·설정), 날짜별 섹션(오늘/어제/8월 30일 일요일), 설정 탭: 권한·Apple Intelligence 상태 점 + 안내 + 설정 앱 열기, 전사 언어, 저장 공간, 개인정보 처리방침·녹음 유의사항 내장. 제목에서 "회의" 제거, 날짜·시간은 ko_KR 고정(UI가 한국어 전용). 서버 전사 폴백 제거(처리방침 "서버로 안 나감"을 코드가 지키게). (`b3e4121`·`b6abeec`·`84c49a3`)
12. **온디바이스 AI 대안 정리** — FoundationModels 유지 권고(무료·용량 0), 대안은 MLX/llama.cpp 로컬 모델(1~3GB, 구형 기기), 전사 대안 WhisperKit. 병목은 요약이 아니라 전사.
13. **1.0 제출** — 홍 결정: 이름 WristNote 유지, 아이콘 미니멀, URL은 Notion(홍이 ASC에 직접 입력), 심사 먼저. 아이콘 생성(`scripts/icon-gen.swift`), Tuist에 팀·버전, fastlane 파이프라인(Fadeo 것 복제 + Xcode 26 export 수정), 메타데이터 ko·en-US, 데모 데이터 훅으로 스크린샷 5장(iPhone 6.9″ 3 + Watch 2). 지뢰 4개(`WKBackgroundModes` 90362 → `UIBackgroundModes`, Fastfile def 스코프, en-US 이름 선점 → `asc add-locale`, 스크린샷 이중 업로드 → cancel·dedupe·resubmit). **02:26 WAITING_FOR_REVIEW.** → [[프로젝트/개인/WristNote/App Store 심사 이력|심사 이력]]

## 2026-09-03 — 1.0 출시 확인 → 1.1.0 개발 (홍 지시 "한번에 1.1, 1.2 둘다")

14. **1.0 READY_FOR_SALE** — 제출 하루 만에 승인, 리젝 0회, 자동 출시.
15. **1.1.0 (build 2) 구현 완료** — 커밋 `2833e05`:
    - **주제**: 요약 스키마에 `topics` 추가(3~7개 명사구), 1.0 회의는 `extractTopics`로 백필. 표기 흔들림은 `topicKey`(trim+lowercase)로 병합.
    - **주제 탭**: Canvas force-directed 그래프(Fruchterman–Reingold 250회, 결정적 원형 초기 배치, 노드 크기=회의 수, 엣지=동시 등장) + 주제 목록 + 주제 페이지(백링크). List 안 숨김 NavigationLink는 셰브런을 그려서 탭은 `onTapGesture(location:)`+`navigationDestination(item:)`으로.
    - **옵시디언 내보내기**: 폴더 security-scoped bookmark 저장, 요약 완료 시 자동 저장 + 상세 화면 수동 버튼. frontmatter(created·duration·source) + 주제 이중대괄호 위키링크 + 요약·결정·액션(`- [ ]`)·전사.
    - **노션**: 사용자 integration 토큰(키체인) + 부모 페이지 URL→32hex ID 파싱, `POST /v1/pages` 직접 호출(rich_text 2000자 제한 분할, to_do 블록). 서버 없음.
    - **워치 컴플리케이션**: WidgetKit appExtension 타깃(`com.hong.wristnote.watchkitapp.widget`, accessoryCircular·Corner·Inline) → `widgetURL(wristnote://record)` → 워치 앱 `onOpenURL`에서 즉시 녹음 시작.
    - 시뮬레이터 검증: 그래프·칩·내보내기 섹션 스크린샷 확인. 릴리즈 노트 ko·en 작성.
16. **위젯 서명·포털 검증 (09-03 오후, 홍 "위젯작업이 안된걸루 알아 확인해봐")** — 확인 결과 코드·서명 모두 완료 상태였다:
    - Apple Developer 포털 실측(aside): 검증 전엔 App ID 2개뿐(`com.hong.wristnote`·`.watchkitapp`), **위젯 App ID 미등록**이 맞았다.
    - `fastlane ios archive`(1.1.0 build 2) 성공 → IPA에 `WristNoteWatchWidget.appex` + App Store 프로파일(`WN2B884S76.com.hong.wristnote.watchkitapp.widget`) 포함, 포털 재확인으로 **App ID 자동 등록 확정** (appstore-release 스킬의 `-allowProvisioningUpdates`+API 키 규칙이 기존 앱의 새 익스텐션 타깃에도 적용됨을 실측).
    - 주의: 이 레포는 Gemfile이 없어 `bundle exec fastlane`은 "Could not locate Gemfile"로 죽는다 — `fastlane` 직접 실행.
    - 남은 것은 아래 실기기 검증뿐.

## 2026-09-05 — 워치 타이머 UI 수정 (1.1.0 심사 중)

17. **경과 시간이 건너뛴다** — 홍 보고: "시간초가 갑자기 올라가 7초 이렇게". 1초 `Timer` tick으로 `elapsed`를 갱신하던 표시를 `Text(timerInterval: startedAt...Date.distantFuture, countsDown: false)`로 교체. 손목 내림·Always-On 감광 중엔 tick이 멈춰 화면을 다시 볼 때 밀린 초가 한 번에 반영되던 것. `@Published var elapsed`도 제거(매초 `objectWillChange`가 나가면 뷰가 어차피 초당 재렌더). 곁가지로 코덱 사다리 성장 판정의 `wall == 2` / `wall == 5` 등호를 부등호 + 기준 +3초로 바꿨다 — tick이 밀리면 판정이 아예 안 돌던 구멍. 커밋 `7e557ec`, 워치 시뮬레이터 자동 테스트로 확인. **1.1.0 build 2(심사 중)에는 안 들어갔다 — 다음 빌드에 실린다.** → [[작업노트/Apple/SwiftUI|작업노트]]

18. **1.1.1 (build 3) 제출** — 홍 "심사 끝났어 심사올려"(1.1.0 READY_FOR_SALE 확인). 버전 1.1.0(2) → 1.1.1(3), 타이머 수정 + 컴플리케이션 URL 스킴 등록(`CFBundleURLTypes: wristnote` — 미등록이라 `simctl openurl`이 LSApplicationWorkspaceErrorDomain 115로 실패했다; WidgetKit 직배달 경로는 별개라 컴플리케이션이 실제로 깨졌는지는 미확정), 워치 스크린샷 2장 재촬영(타이머 표기 변경 반영), 릴리즈 노트 ko·en. `fastlane ios release` 성공 → WAITING_FOR_REVIEW. **스크린샷 이중 업로드 3회 연속 재발 — 정리는 홍이 `scripts/fix-screenshots-resubmit.sh` 실행.** 커밋 `5d43e5b`. → [[프로젝트/개인/WristNote/App Store 심사 이력|심사 이력]]

## 다음 (홍이 할 것)

- **`scripts/fix-screenshots-resubmit.sh` 실행** — 1.1.1 스크린샷 이중 업로드 정리 + 재제출 (자동 모드가 `asc cancel-review`를 차단)
- **실기기에서 1.1.0 확인**: ① 컴플리케이션 추가(시계 화면 편집) → 탭 → 바로 녹음 ② 설정 → 마크다운 폴더에 옵시디언 볼트 지정 → 회의 후 볼트에 파일·그래프 연결 확인 ③ 노션 토큰 연결 → 보내기 ④ **30분+ 백그라운드 녹음(아직 미검증)**
- 확인되면 "제출해" 한마디 — `fastlane ios release`로 1.1.0 (build 2) 올린다 (스크린샷에 주제 그래프 추가 예정).
- GitHub 레포 생성 여부 결정.

배운 것: [[작업노트/Apple/온디바이스 음성 인식과 번역|온디바이스 음성 인식과 번역]] · [[작업노트/Apple/WatchConnectivity와 워치 녹음|WatchConnectivity와 워치 녹음]] · [[작업노트/도구/Tuist|Tuist]] · [[작업노트/Apple/SwiftUI|SwiftUI]]
