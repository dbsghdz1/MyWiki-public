---
type: project
status: active
created: 2026-08-31
updated: 2026-09-01
related_wiki: []
---

# WristNote 개발 기록 2026-08-31 — 착수, v1 골격

홍의 착수 지시("기록하고 개발해") 후 야간 자율 작업. 목표는 [[프로젝트/개인/WristNote/README|README]]의 M1: 워치 녹음 → WCSession 전송 → 전사 → 요약 → 목록 표시의 v1 골격.

## 결정

- **스캐폴딩: Tuist 4.202.5** (로컬에 이미 설치). iOS 앱 + watchOS 앱 임베드 구조.
- **전사·요약 모두 온디바이스** — iOS 26 `SpeechAnalyzer` + `FoundationModels`. 서버·API 키 없이 v1 성립. SpeechAnalyzer 사용법은 [[작업노트/Apple/온디바이스 음성 인식과 번역|Subly 스파이크 실측]]을 재사용.
- **워치 백그라운드 녹음**: `WKBackgroundModes: [audio]` + 활성 `AVAudioSession(.record)`. 시뮬레이터로는 화면 꺼짐 지속을 판정할 수 없으므로 **실기기 M0은 별도** — 이번 작업의 검증 범위는 시뮬레이터 페어에서의 파이프라인 관통까지.

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

## 다음 (홍이 할 것)

- 실기기 M0: 워치+아이폰에 설치 → ① 화면 꺼진 채 30분 녹음 ② 전송 도착 ③ 실제 육성 한국어 전사 품질. 이 셋의 결과가 프로젝트 방향을 정한다.
- GitHub 레포 생성 여부 결정.

배운 것: [[작업노트/Apple/온디바이스 음성 인식과 번역|온디바이스 음성 인식과 번역]] · [[작업노트/Apple/WatchConnectivity와 워치 녹음|WatchConnectivity와 워치 녹음]] · [[작업노트/도구/Tuist|Tuist]]
