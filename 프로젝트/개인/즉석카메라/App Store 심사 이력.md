---
type: project
status: active
created: 2026-08-27
updated: 2026-08-27
---

# Fadeo App Store 심사 이력

appId `6805274191` · bundle `com.hong.fadeo`

| 날짜 | 버전(빌드) | 상태 | 비고 |
|---|---|---|---|
| 2026-08-27 | 1.0 (3) | **WAITING_FOR_REVIEW** | 첫 제출. **무료**(유료 전환은 1.1.0 StoreKit과 함께) |

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
