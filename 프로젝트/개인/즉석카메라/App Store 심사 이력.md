---
type: project
status: active
created: 2026-08-27
updated: 2026-09-02
---

# Fadeo App Store 심사 이력

appId `6805274191` · bundle `com.hong.fadeo`

| 날짜 | 버전(빌드) | 상태 | 비고 |
|---|---|---|---|
| 2026-08-27 | 1.0 (3) | 제출 | 첫 제출. **무료** |
| **2026-08-31** | 1.0 (3) | **출시됨** | 22:12 UTC. KR·US 동시 |
| 2026-09-02 | 1.1 (4) | 제출 | 필름 6종·물성 렌더·콜드 스타트·ASO 개편. **유료 전환(₩1,100) 예정** |

## 1.0 (3) 제출 기록 — 세 번 만에 들어갔다

1. **1차 실패 — 연령 등급 신필수 항목.** 2025 개편으로 `ageRatingDeclaration`에 필수 항목이 늘었는데 rating.json이 구양식. **API가 부족한 항목을 한 번에 다 알려주지 않는다** — 1차에서 `userGeneratedContent`·`advertising`·`healthOrWellnessTopics`·`messagingAndChat`, 2차에서 `gunsOrOtherWeapons`·`lootBox`·`parentalControls`·`ageAssurance`를 요구했다.
2. **2차 실패 — 같은 지점, 2라운드 항목.** 위 4개 추가 후 통과.
3. **3차 — 스크린샷·빌드 연결까지 성공, 최종 `submit_for_review`에서 `is not in valid state`.** 원인은 ASC 웹에서만 되는 두 가지가 미완이어서: **가격 미설정 + App Privacy 미게시.** 홍이 웹에서 설정하고 직접 제출 완료.

부산물: en-US 스크린샷이 6장(1장 중복) — deliver 이중 업로드 경합. 제출 후엔 잠겨서 1.0.1에서 정리.

빌드 이력: 1(초기) → 2(6개 언어) → **3(전체 화면 카메라·실사 배출·성능 수정 포함, 심사 대상)**

## 다음 제출 때 기억할 것

- `fastlane ios submit`은 `FADEO_BUILD=<빌드번호>` 필요
- 심사 연락처 4파일은 gitignore — 지워졌으면 재작성 (전화 +82 10-8696-2118)
- 개인정보/지원 URL: 노션 `Privacy-Policy-3c85…` / `Support-3c85…`

## 1.1 (4) 제출 기록 — 버전이 번들에 안 들어갔다

빌드는 통과했는데 **업로드가 죽었다.**

```
[altool] ERROR: Invalid Pre-Release Train.
The train version '1.0' is closed for new build submissions (90186)
```

`Project.swift`의 `MARKETING_VERSION`을 1.1로 올리고 `tuist generate`까지 했는데도 IPA의 `CFBundleShortVersionString`이 1.0이었다. **tuist `extendingDefault` 기본 Info.plist가 "1.0"을 리터럴로 박는다** — 상세는 [[작업노트/Apple/Xcode 빌드와 번들 구성|Xcode 빌드와 번들 구성]]. `"CFBundleShortVersionString": "$(MARKETING_VERSION)"`을 직접 넣어 해결.

메타데이터·스크린샷은 그 전에 이미 1.1로 올라가 있었다(deliver가 바이너리보다 먼저 돈다).

## 1.1에서 고친 것

- **콜드 스타트** — 첫 프레임 327ms → 253~283ms. `startRunning()` 200ms는 못 줄이고 시작 시각을 `App.init`으로 당겼다
- **로컬라이제이션 버그** — 상세 화면 버튼 4개가 모든 언어에서 한국어로 나갔다. 1.0에 그대로 출시된 상태였다
- **필름 6종** — 흑백·나이트 추가, 물성 4겹(할레이션·입자·번짐·얼룩)
- **ASO 개편** — 이름·부제·키워드 (아래)

## 스토어 메타데이터 변경 (1.1)

| | 1.0 | 1.1 |
|---|---|---|
| 이름(ko) | `Fadeo` | `Fadeo 페이디오 - 즉석 필름 카메라` (22/30) |
| 부제(ko) | `10분을 기다리는 즉석카메라` | `10분을 기다려 현상되는 사진` (16/30) |
| 키워드(ko) | 54자 | 88자 — `인화` 제거, `일회용카메라`·`암실`·`현상소` 추가 |
| 이름(en) | `Fadeo` | `Fadeo - Instant Film Camera` |
| 부제(en) | `The camera that makes you wait` | `Photos that develop in 10 min` |

근거: [[작업노트/AppStore/앱 이름과 검색 노출|앱 이름과 검색 노출]] — **자기 이름으로 검색해도 200위 안에 없었다.**

## 다음 제출 때 기억할 것

- `fastlane ios submit`은 `FADEO_BUILD=<빌드번호>` 필요 (기본값을 없앴다 — 하드코딩하면 또 틀린다)
- `app_version`은 `Project.swift`에서 읽는다 (Fastfile `marketing_version`)
- 심사 연락처 4파일은 gitignore — 지워졌으면 재작성
- **가격은 ASC 웹에서만 된다** (1.0 때 `submit_for_review`가 `is not in valid state`로 막힌 원인이 가격 미설정 + App Privacy 미게시였다)
- 스크린샷은 `fastlane/make_screenshots.sh` — 시뮬레이터를 지우고 시작한다(안 그러면 상태 표시줄에 이전 앱 복귀 표시가 박힌다)
