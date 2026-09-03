---
type: project
status: shipped
aliases:
  - WristNote
  - 워치 회의 녹음
created: 2026-08-31
updated: 2026-09-03
repos:
  - "~/Desktop/개인 앱/WristNote — GitHub 미생성"
related_wiki: []
---

# WristNote (코드명) — 애플워치로 회의 녹음하고 AI가 정리해주는 앱

**손목에서 녹음 시작 → 워치가 회의를 녹음 → 아이폰으로 자동 전송 → 온디바이스 전사 → AI 요약이 내 앱에 도착.** 2026-08-31 홍 발의, 같은 날 착수 지시("기록하고 개발해"). **2026-09-03 App Store 1.0 출시(READY_FOR_SALE, 리젝 0회)** ([[프로젝트/개인/WristNote/App Store 심사 이력|심사 이력]]). 착수(08-31)부터 출시까지 약 2.5일.

**1.1 방향 (2026-09-03 홍 논의)**: 차별점은 "무제한·구독 없음·서버 없음"(경쟁 앱들은 서버 STT라 분 제한·구독이 구조적) + 주제 그래프. 1.1 = 주제 추출(`topics` 스키마)+칩+주제 페이지 + **마크다운 폴더 내보내기(옵시디언 볼트, 옵시디언 위키링크(주제명 이중대괄호) 포함)** + 워치 컴플리케이션. 1.2 = 인앱 그래프 뷰(Canvas force-directed) + 노션(사용자 토큰 방식, 서버 없이).

> [!note] 슬롯 배분과의 관계
> 개인 허브의 `paused` 원칙(2026-11-14 지원 마감 역산)과 별개로 홍이 착수를 직접 지시했다(2026-08-31). AI의 사전 평가는 "아이디어 기록 + 스파이크만, 본격 착수는 11-14 이후"였으나 홍의 결정으로 착수. 다른 active 프로젝트(Fadeo·한능검·Zappy 런치)와 슬롯을 나눈다.

## 왜 이 앱인가 — 웨지

"회의 녹음 + AI 정리"는 레드오션이다(클로바노트 무료·한국어 STT 최강, Otter, 애플도 iOS 18.1부터 통화 녹음·전사). 차별점은 **"애플워치로"**:

- **Plaud가 증명한 수요** — 폰 안 꺼내고 회의를 녹음하는 용도로 $159 전용 하드웨어를 팔아 성장 중. 손목에 이미 있는 워치가 그 하드웨어를 대체한다.
- **마찰 제거** — 폰을 꺼내 테이블에 올리는 대신 손목에서 바로 시작. 회의 시작 30초 안에 녹음이 켜진다.
- **애플 기본 음성 메모(워치)는 녹음까지만** — 전사·요약·정리가 없다. "녹음→전사→요약→도착" 자동 파이프라인 전체가 제품이다.

## 파이프라인 (v1 설계)

```
워치 (SwiftUI, watchOS 26)          아이폰 (SwiftUI, iOS 26)
┌─────────────────────┐            ┌──────────────────────────┐
│ AVAudioRecorder      │  WCSession │ 파일 수신 → 저장           │
│ AAC m4a 16kHz mono   │──────────▶│ SpeechAnalyzer 전사(온디바이스)│
│ 백그라운드 녹음 유지    │ transferFile│ FoundationModels 요약(온디바이스)│
└─────────────────────┘            │ 회의 목록·요약 화면          │
                                   └──────────────────────────┘
```

- 전사·요약 모두 **온디바이스·무료** (iOS 26 `SpeechAnalyzer` + `FoundationModels`) — 서버·API 키 없이 v1이 성립한다. SpeechAnalyzer는 [[프로젝트/개인/Subly/README|Subly]] 스파이크에서 이미 실측(영어, 110단어에 오류 1건) — [[작업노트/Apple/온디바이스 음성 인식과 번역|온디바이스 음성 인식과 번역]]. **한국어 전사 품질은 미검증.**
- 30분 회의 ≈ AAC 모노 10~15MB → `WCSession.transferFile` 백그라운드 전송 부담 없음.

## 리스크 — 2026-09-01 v1 골격 검증 후

| 리스크 | 판정 |
|---|---|
| **워치 화면 꺼짐·손목 내림 상태에서 30~60분 녹음 지속** | **미판정 — 1.0은 이 검증 없이 제출.** `UIBackgroundModes: audio` + 활성 `.record` 세션으로 구현 |
| **실기기 워치 녹음** | **✅ 09-02 해결 (세 겹)** — ① `stop()` 직후 전송은 잘린 파일 → `didFinishRecording` 뒤로 ② 실기기 `AVAudioRecorder` AAC는 무출력 → 코덱 사다리, **µ-law 16kHz(16KB/s)** 로 동작 ③ watchOS는 `record()`가 권한 프롬프트를 안 띄움 → `requestRecordPermission` 명시. [[작업노트/Apple/WatchConnectivity와 워치 녹음|작업노트]] |
| 워치→아이폰 `transferFile` | **✅ 실기기에서 몇 초 안에 배달** (시뮬레이터는 미지원). WCSession 활성화는 앱 `init`에서 |
| 워치 배터리 소모 (1시간 녹음) | 미측정 — 실기기 |
| **한국어 전사 품질 (SpeechAnalyzer ko)** | **⚠ 합성 음성 기준 나쁨** — macOS 호스트 실측(같은 모델): 34초를 0.35초에 전사하지만 "회의"→"매일", "안건"→"하면", "예산"→"예상", "신규 앱의 출시"→"싱U M의 소시". 숫자·이름·요일은 정확. **09-02 실기기 육성 전사는 동작 확인**(홍 "좋았어") — 정량 품질은 아직 안 잼 |
| FoundationModels 한국어 요약 | **✅ 시뮬레이터에서 동작, 품질 좋음** — 정확한 전사를 주면 제목·요약·결정사항·액션아이템(담당자 포함)을 정확히 뽑음. **단, 오염된 전사는 보정 못 하고 그대로 요약** |
| iOS 시뮬레이터 전사 | **불가** — `SpeechAnalyzer`·`SFSpeechRecognizer` 모두 동작 안 함(`kLSRErrorDomain 300`). 전사 주입 훅으로 우회 |
| 법적 | 한국은 **본인이 참여한 대화 녹음은 합법**(통신비밀보호법은 타인 간 대화만 금지). 앱 문구를 "내가 참석한 회의"로 명확히 |

## 마일스톤

- **M1 (v1 골격)** — ✅ 2026-09-01 완료(커밋 `f8c4bd3`). 워치 녹음 → 전송 → 전사 → 요약 → 목록·상세 UI. 시뮬레이터에서 검증 가능한 부분(워치 녹음, 요약, UI)은 통과, 나머지는 실기기.
- **M0 (실기기 스파이크)** — ✅ 부분 완료 09-02: ② `transferFile` 배달 ✅ ③ 육성 전사 동작 ✅(품질 정량은 미측정). **① 30분+ 백그라운드 녹음 지속·④ 배터리는 아직** — 1.0 심사 중에 홍이 실제 회의 1건으로 검증할 것
- **1.0 제출** — ✅ 2026-09-02 02:26 WAITING_FOR_REVIEW(build 1). 이름 WristNote 유지(en-US는 선점 때문에 "WristNote: Watch Meeting Notes"), 미니멀 아이콘, 무료, 탭바(녹음·설정)·날짜별 섹션·권한/Apple Intelligence 상태 안내·개인정보 처리방침 내장. → [[프로젝트/개인/WristNote/App Store 심사 이력|심사 이력]]
- 1.1 후보 — 청크 요약 실측, 워치 컴플리케이션(1탭 시작), 녹음 용량(µ-law 16KB/s → AVAudioEngine 소프트웨어 AAC), 콜드 스타트 반응 지연, 한국어 전사 정량 측정

## 코드 저장소

- 로컬: `(로컬 경로)` — Tuist 기반 (iOS 앱 + watchOS 앱 임베드). 빌드·검증 훅은 저장소 README, 배포는 `fastlane/`(archive·upload·release·submit·resubmit) + appstore-release 스킬
- GitHub: 미생성 (홍 승인 후)
- 앱 아이콘: `scripts/icon-gen.swift`(1024·알파 없음), 스크린샷 데모: `WRISTNOTE_DEMO=1`·`WRISTNOTE_OPEN=first`·`WRISTNOTE_TAB=settings`

## 배운 것

- [[작업노트/Apple/온디바이스 음성 인식과 번역|온디바이스 음성 인식과 번역]] — iOS 시뮬레이터는 음성 인식이 통째로 안 되고, FoundationModels는 된다. 요약 모델은 전사 오류를 못 고친다
- [[작업노트/Apple/WatchConnectivity와 워치 녹음|WatchConnectivity와 워치 녹음]] — 시뮬레이터는 `transferFile`을 배달하지 않는다(워치 성공 콜백은 옴)
- [[작업노트/도구/Tuist|Tuist]] — 루트 판정·빈 Config.swift 함정, watchOS 임베드

## 작업 기록

- [[프로젝트/개인/WristNote/WristNote 개발 기록 2026-08-31|WristNote 개발 기록 2026-08-31]] — 착수, v1 골격, 실기기 M0, 1.0 제출까지
- [[프로젝트/개인/WristNote/App Store 심사 이력|App Store 심사 이력]] — 1.0 (build 1) 2026-09-02 제출
