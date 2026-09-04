---
type: project
title: "DayTune"
summary: "수면 데이터를 바탕으로 하루 계획을 돕는 iOS 앱의 작업 진입점"
status: paused
aliases:
  - 데이튠
created: 2026-08-09
updated: 2026-08-22
slack_channel: daytune
repos:
  - "github.com/dbsghdz1/DayTune"
related_wiki:
  - "[[_wiki/AI 디자인 스킬]]"
---

# DayTune

지난밤 수면(Apple Health)을 바탕으로 "오늘 하루를 어떻게 보낼지"를 추천하는 iOS 앱. 데이터 나열보다 행동 추천이 먼저이고, 차분한 톤에 의학적 단정을 피하는 것이 제품 원칙이다 (레포 AGENTS.md).

> [!note] 2026-08-14 보류 — **잠시 중지된 프로젝트다 (중단·폐기가 아니다)**
> 취업 지원을 위해 MyCryptoDiary에 집중하기로 하면서 이 프로젝트는 보류로 전환했다. **제품이나 코드에 문제가 있어서가 아니라 슬롯 배분 결정이다** — MVP 화면은 전부 merge된 상태로 멈춰 있고, 그대로 이어서 재개할 수 있다.
> **재개 조건**: 지원 마감 `2026-11-14` 경과 후 재개 검토. **지원 D-day는 2026-08-16에 12월 → `2026-11-14`로 앞당겨졌다**(당근 윈터테크 마감). 근거: 계획 우선순위 표.

## 현재 상태 (2026-08-09 기준)

- **MVP 화면 전부 merge 완료**: 건강 앱 연결 → 수면 분석 중 → Plan Home(PR #11) → 오늘의 계획(PR #13) → 추천 상세(PR #17). 수면 결측 시 데이터 없음 화면으로 분기하고 직접 입력 시트·샘플 계획으로 홈 진입(PR #15). 설정 화면(PR #19), 영문 로컬라이제이션 116키(PR #21), 앱 아이콘(PR #23)까지 모두 develop에 머지됨.
- **디자인 확정**: Figma 파일 "MyCryptoDiary"(fileKey `HwlPMPo43BU1y0DXQtNlzx` — 파일명만 재활용, [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary 프로젝트]]와 무관) 안의 "DayTune 2.0" 페이지가 마스터. 13개 화면을 오토레이아웃+컴포넌트로 재건축했다.

## 기술 구조

- Clean Architecture + MVVM-C, UIKit + SnapKit, Tuist, iOS 18+. ViewModel은 UIKit 미포함, 화면 전환은 Coordinator 전담.
- 디자인시스템: `AppColor.Brand`(다크 브랜드 팔레트), `DTGradientCTAButton`/`DTBevelButton`/`DTSparkleView`/`DTGlow` 공용 컴포넌트.
- HealthKit: `HealthKitHealthDataRepository`가 수면(단계 합산)·HRV·안정심박 + 30일 베이스라인을 읽는다. 데이터 없으면 nil로 두고 `TodayStateAnalyzer`가 missingMetrics로 우아하게 강등 처리.
- 디버그: `-daytune.showAnalyzing` 런치 아규먼트로 분석 플로우 직접 진입.

## 주요 결정

- 온보딩 4페이지 크로스페이드 → 데이터 투명성 카드를 갖춘 단일 "건강 앱 연결" 화면으로 축약 (2026-08-05, Figma 디자인 기준).
- 추천 카피(`RecommendationEngine`)는 도메인 레이어에서 `String(localized:)`로 한국어화 — 16개 추천 × 제목/본문. en 번역은 미작성.
- 홈은 분석 실패/데이터 없음 시에도 빈 입력 기준의 보수적 계획을 보여준다 (죽은 화면 없음).

## 작업 기록

- 2026-08-22 (보류 중 소규모 수정, 미커밋 워킹트리) — `DTGradientCTAButton` 입체감 완화(그라디언트 명도차·베벨·그림자 절반), 건강 앱 연결 권한 허용 직후 SIGABRT 수정(`HealthConnectionViewModel`, `MainActor.run`). 시뮬레이터 iPhone 17 Pro에서 새 설치→허용→수면 데이터 없음 화면까지 검증. 추가: 추천 행 내부 여백 확대(`DTRecommendationRow`), 수면 조회를 36h + 최근 세션 집계로 교체(`latestSleepSession`, 테스트 6개). **Tuist 프로젝트라 새 파일 추가 후 `tuist generate` 필요**(안 하면 테스트 타깃에 안 들어감). `DTNavigationController` 신설로 바 숨김·pop 제스처 일원화(7개 VC 토글 제거)

- [[프로젝트/개인/DayTune/DayTune 개발 기록 2026-08-09|DayTune 개발 기록 2026-08-09]] — Figma 재건축 + 코어 플로우 구현

## 배운 것

- [[작업노트/Apple/Swift 동시성과 UIKit 메인 스레드|Swift 동시성과 UIKit 메인 스레드]] — 건강 앱 권한 허용 직후 크래시: HealthKit 콜백이 백그라운드로 돌아와 `setViewControllers`가 메인 밖에서 호출됨. `MainActor.run`으로 수정 (2026-08-22)
- [[작업노트/Apple/HealthKit 수면 데이터 조회|HealthKit 수면 데이터 조회]] — 새벽 2시에 "수면 데이터 없음": 자정 기준 윈도우가 직전 밤을 놓침. 36h + 최근 세션만 집계로 수정 (2026-08-22)
- [[작업노트/Apple/UIKit 내비게이션 바 숨김과 pop 제스처|UIKit 내비게이션 바 숨김과 pop 제스처]] — 바 숨김으로 죽은 엣지 스와이프 pop·전환 중 상단 튐을 `DTNavigationController` 한 곳으로 해결 (2026-08-22)

## 다음 작업 (재개 시)

- **TestFlight 업로드 재개**: fastlane 구성·서명·아카이브 검증 완료(PR #25, DayTune.ipa 생성됨). 블로커는 **ASC 앱 레코드** — 2026-08-12·08-15 두 차례 확인했지만 API(`asc apps`)에 `com.hong.daytune`이 안 잡힘. 사용자가 ASC 웹에서 "DayTune" 이름으로 생성 시도했으나 미반영(이름 선점 또는 다른 팀 선택 추정). 재개 시 절차: ASC 웹 우상단에서 **Zappy·BarStack이 보이는 팀(WN2B884S76)** 선택 확인 → 앱 추가(iOS, 이름 "DayTune: 수면 기반 하루 계획" 등 대체 이름, 번들 ID 드롭다운 `com.hong.daytune`, SKU `daytune`) → `asc apps`로 확인 → `fastlane ios beta`
- 실기기 테스트 (Apple Watch 수면 데이터로 실데이터 경로 검증)
