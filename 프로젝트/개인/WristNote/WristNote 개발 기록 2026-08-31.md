---
type: project
status: active
created: 2026-08-31
updated: 2026-08-31
related_wiki: []
---

# WristNote 개발 기록 2026-08-31 — 착수, v1 골격

홍의 착수 지시("기록하고 개발해") 후 야간 자율 작업. 목표는 [[프로젝트/개인/WristNote/README|README]]의 M1: 워치 녹음 → WCSession 전송 → 전사 → 요약 → 목록 표시의 v1 골격.

## 결정

- **스캐폴딩: Tuist 4.202.5** (로컬에 이미 설치). iOS 앱 + watchOS 앱 임베드 구조.
- **전사·요약 모두 온디바이스** — iOS 26 `SpeechAnalyzer` + `FoundationModels`. 서버·API 키 없이 v1 성립. SpeechAnalyzer 사용법은 [[작업노트/Apple/온디바이스 음성 인식과 번역|Subly 스파이크 실측]]을 재사용.
- **워치 백그라운드 녹음**: `WKBackgroundModes: [audio]` + 활성 `AVAudioSession(.record)`. 시뮬레이터로는 화면 꺼짐 지속을 판정할 수 없으므로 **실기기 M0은 별도** — 이번 작업의 검증 범위는 시뮬레이터 페어에서의 파이프라인 관통까지.

## 진행

(작업 진행하며 append)
