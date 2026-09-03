---
type: project
status: active
created: 2026-08-27
updated: 2026-09-04
---

# Fadeo App Store 심사 이력

appId `6805274191` · bundle `com.hong.fadeo`

| 날짜 | 버전(빌드) | 상태 | 비고 |
|---|---|---|---|
| 2026-08-27 | 1.0 (3) | 제출 | 첫 제출. **무료** |
| **2026-08-31** | 1.0 (3) | **출시됨** | 22:12 UTC. KR·US 동시 |
| 2026-09-02 | 1.1 (4) | 제출 | 필름 6종·물성 렌더·콜드 스타트·ASO 개편 |
| **2026-09-03** | 1.1 (4) | **출시됨** | 심사 통과 |
| 2026-09-04 | 1.2 (5) | 제출 | 필름 장전·잠금화면 라이브 액티비티·홈 위젯·인앱 리뷰 요청. **스토어 기본 언어 ko → en-US** |

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

## 1.2 (5) — 스토어 기본 언어가 한국어였다 ★

**증상**: 일본·중국·프랑스·독일·스페인·브라질 스토어에서 앱 이름이 `Fadeo 페이디오 - 즉석 필름 카메라`로, 설명도 한국어로 나가고 있었다.

**진단법** — 스토어프론트별로 lookup을 돌려 어느 언어로 떨어지는지 본다. 한국어·영어 둘 다 없는 지역이 무엇을 받는지가 곧 기본 언어다.

```bash
for c in kr us fr de jp cn es br in; do
  curl -s "https://itunes.apple.com/lookup?id=<appId>&country=$c" |
    python3 -c "import sys,json; r=json.load(sys.stdin)['results'][0]; print('$c', r['trackName'])"
done
```

실측 결과 미국·인도(영어 스토어프론트)만 영어였고 나머지는 전부 한국어 → `primaryLocale = ko` 확정.

**고치는 법**: `PATCH /v1/apps/{id}` 에 `attributes.primaryLocale = "en-US"`.

⚠️ **한 번에 안 바뀐 것처럼 보인다.** PATCH가 200을 주는데 바로 GET 하면 여전히 `ko`다. **심사 대기 버전(PREPARE_FOR_SUBMISSION)이 하나도 없으면 반영되지 않는다** — 새 `appStoreVersions`를 만들고 나서 다시 읽으면 `en-US`로 보인다. `appInfos`에 `primaryLocale`을 PATCH 하는 건 다른 길인 줄 알았는데 `'primaryLocale' is not an attribute`(409)로 막힌다. 앱 리소스 쪽이 맞다.

**앱 안(번들)은 별개고, 이미 맞게 돼 있었다** — `CFBundleDevelopmentRegion: en`, `.lproj` 6개(en·ko·ja·es·zh-Hans·zh-Hant) 전부 포함. 카탈로그 61개 키를 6개 언어로 감사했고 누락은 `%@`·`%@/%@` 둘뿐(번역할 글자가 없는 포맷 문자열).

## 1.2 제출 메모

- 빌드 업로드 직후 `submit`을 돌리면 `Build number: 5 does not exist`로 죽는다. Apple 처리에 1~2분 걸린다 — `/v1/builds?filter[app]=…&sort=-uploadedDate`로 `processingState`가 `VALID`가 될 때까지 기다렸다가 제출한다.
- **위젯 익스텐션이 처음 들어간 릴리스**였고, 수동 서명(`CODE_SIGN_STYLE=Manual` + 타깃별 `PROVISIONING_PROFILE_SPECIFIER`)으로 아카이브·export·업로드가 전부 통과했다. Xcode에 Apple ID 세션이 없어도 된다.
