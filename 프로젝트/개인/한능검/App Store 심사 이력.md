---
type: project
status: active
created: 2026-08-28
updated: 2026-09-04
---

# App Store 심사 이력 — 한능검 정복

| | |
|---|---|
| 앱 이름 | **한능검 정복** (홍이 ASC에서 생성) |
| App ID | `6806144570` |
| Bundle ID | `com.hong.hangeom` |
| 팀 | `WN2B884S76` |

## 2.0.3 (예정) — ASO 메타데이터 교체 (홍 결정 2026-09-04, 2.0.2 심사 중이라 대기)

"한국사" 검색 색인 누락([[작업노트/AppStore/앱 이름과 검색 노출|앱 이름과 검색 노출]] 09-04 기록)을 고치는 메타데이터 전용 변경. 2.0.2 심사를 취소하지 않고 다음 버전에 싣기로 했다.

| 항목 (ko) | 2.0.2 (현재) | 2.0.3 |
|---|---|---|
| 이름 | 한능검 정복 | **한능검 정복 - 한국사 기출** (15자) |
| 부제 | 심화·기본 최신 기출 전 문항 · 해설·노트 | **한국사능력검정시험 심화·기본 전 문항 해설** (23자) |
| 키워드 | 한능검,한국사능력검정시험,한국사,기출문제,심화,기본,3급,4급,공무원,공시,수능한국사,오답노트,요약노트,기출,역사 (63자) | **기출문제,3급,4급,9급,공무원,공시,수능한국사,오답노트,요약노트,역사,국사,문제집,모의고사,해설,cbt,시험** (61자, 이름·부제와 중복 제거) |

- **`web/fastlane/metadata/ko/{name,subtitle,keywords}.txt`는 09-04에 이미 위 값으로 바꿔뒀다** (레포가 git이 아니라 커밋 없음). 프로모션 문구·설명·en-US는 그대로.
- 2.0.3 릴리즈 때 할 일: ① 2.0.2 READY_FOR_SALE 확인 ② `ios/App` `MARKETING_VERSION 2.0.2 → 2.0.3`, `CURRENT_PROJECT_VERSION 8 → 9` ③ **`fastlane/Fastfile`의 `app_version: "2.0.2"` 두 곳(`meta`·`deliver_all`)을 `2.0.3`으로** ④ `fastlane ios release` (deliver가 metadata 폴더의 새 이름·부제·키워드를 올린다; 스크린샷 중복 지뢰는 기존 플레이북대로) ⑤ 승인 며칠 뒤 `itunes.apple.com/search?term=한국사&country=kr&entity=software`에서 6806144570 순위 재측정.
- 주의: metadata 파일이 이미 새 값이므로 **2.0.2가 심사 중인 동안 `fastlane ios meta`를 돌리면 안 된다**(잠긴 버전에 새 이름을 밀어 넣으려다 실패하거나, 개발자 거절 상태면 2.0.2에 섞여 들어간다). `resubmit`은 `skip_metadata: true`라 안전.
- pbxproj에 `MARKETING_VERSION = 2.1.0 / build 9`인 설정 블록(1F35F02E·9E3D2499)이 별도로 있다 — 어느 타깃인지 릴리즈 전에 확인.

## 2.0.2 (build 8) — 2026-09-03 15:07 최종 재제출 (WAITING_FOR_REVIEW)

홍 실기기 확인 "가로가 별로" → 원인은 세로 고정. **iPadOS 26에선 `UIRequiresFullScreen`이 폐기**돼 세로 전용 앱이 가로에서 떠 있는 창으로 나온다 — 플래그 제거 + iPad 전 방향 허용(리사이즈 가능 창, 단일 컬럼이라 성립). iPhone은 세로 유지. iPad 회귀 118단계 재통과, build 8. 이중 업로드 6번째(9장) → 취소→정리→재제출. 최종: build 8 · 4세트 × 6장.

## (build 7) — 15:54 제출 → 가로 지원 위해 자진 취소 — 영어 페이지 + iPad

홍 지시 연타: 유료 전환 직접 완료(₩6,600) → "iPad용도 개발해". build 6 심사를 취소하고 iPad를 실어 build 7으로.
- **iPad 지원**: `TARGETED_DEVICE_FAMILY "1,2"` + **세로 고정 + UIRequiresFullScreen**(문제집 UX·멀티태스킹 요건 회피). 레이아웃은 기존 720px 중앙 컬럼이 그대로 성립.
- **iPad에서 잡은 실버그**: 터치 핸들러가 720px `.wrap`에 있어 **좌우 거터에서 시작한 스와이프가 죽었다** → `.wrap.q`를 전체 폭으로 펴고 내용은 패딩 중앙 정렬. 열린 강의 iframe은 여전히 터치 사각지대(구조상 수용) — 플로우는 캡슐 탭(맨 위로) 후 상단에서 스와이프.
- **iPad 시뮬 함정**: iPad Pro 시뮬레이터가 가로로 부팅돼 세로 앱과 좌표축이 어긋남 — Simulator 창 열고 ⌘← 회전 후 주행. iPad 회귀 118단계 통과.
- 스토어 컷: `storeshots.py`를 크기 무관으로(세로 기준 폰트 스케일), iPad 2064×2752 세트(i01~i06)를 ko·en 각각 — deliver가 해상도로 APP_IPAD_PRO_3GEN_129에 매핑.
- 이중 업로드 5번째(10장 제거) → 재제출. 최종: build 7 · ko 6+6 · en-US 6+6.

## (build 6) — 2026-09-03 제출 → iPad 포함 위해 자진 취소

홍 지시: 가격은 직접(₩6,600, ASC 웹), 주 언어 영어 + 스크린샷에 유튜브. 내용:
- **en-US 로케일 신설** — 이름 "한능검 정복 – Korean History", 영문 설명·키워드·릴리스 노트, 영문 헤드라인 스크린샷 6컷(`storeshots.py` compose(SHOTS_EN)).
- **스크린샷에 강의 플레이어** — 래퍼에 중립 포스터(탭 전 유튜브 미로드)를 넣어 강사 초상 없이 플레이어 UI를 노출(s3). 초상권·5.2 리스크 회피 + 문항 로딩 개선 부수효과.
- 절차 지뢰: 출시된 버전은 편집 불가 → `POST /v1/appStoreVersions`로 2.0.2 레코드 선생성 → 편집 가능 appInfo가 생김 → `add-locale`(appInfo id! appInfoLocalization id 아님) → release(build 6) → cancel→dedupe(ko 1장)→resubmit. **`set-primary en-US`는 "모든 버전에 en-US 스크린샷 필요" 409** — 이미 출시된 과거 버전엔 넣을 수 없으므로 2.0.2 출시 후 재시도.
- asc.swift에 generic `patch` 추가.

## 2.0.1 (build 5) — 2026-09-02 18:25 제출 → **당일 통과·출시** (READY_FOR_SALE)

2.0.0 통과 확인 직후 당일 후속. 내용: 앱 내 강의 재생(https 래퍼 — 임베드 오류 153 해결), 채점 시 자료 단서 강조(태그 매칭), 화면 전환·엣지 뒤로가기(iOS 곡선), 신호색 톤 다운, disabled 터치 사각지대 수정, 스토어 스크린샷 6컷 재촬영(최종 색감).
**deliver 스크린샷 이중 업로드 4번째 재현** → cancel → dedupe(4장) → `HANGEOM_BUILD=5 resubmit`. 이 왕복이 고정 비용이 됐다 — 다음 릴리스 땐 release 후 자동으로 screenshots 검증→dedupe→resubmit까지 스크립트로 묶을 것.
확정: 2.0.1 WAITING_FOR_REVIEW · build 5 · ko 6장.

## 2.0.0 (build 4) — 2026-09-02 제출 → 당일 통과·출시 (READY_FOR_SALE)

홍 지시 "이제 제출해". 거의 전부 새로 만든 릴리스: 총 1,849문항(심화 1,149·기본 700), 전 문항 해설, 사진 선지 107문항 복원(자유이용 사진 395장 + 출처 표기), 주제별 풀기·모의고사·연표·약한 주제, 온보딩·설정, 노션형+리퀴드 재설계. 스크린샷 6컷 재생성(`tools/storeshots.py`, s6=사진 문항).

- 04:32 `fastlane ios release` — 아카이브 5분, 업로드, 메타·스크린샷, 04:37 제출
- **또 스크린샷 이중 업로드**(01~04 두 장씩, 10장) — 플레이북 그대로: `cancel-review` → `dedupe-screenshots`(4장 제거) → `HANGEOM_BUILD=4 fastlane ios resubmit` → 04:39 재제출. deliver 업로드 경합은 세 번째 재현이라 이제 **제출 직후 `asc screenshots` 검증이 고정 절차**다.
- 확정 상태: 2.0.0 WAITING_FOR_REVIEW · build 4(2026-09-02 04:35 KST 업로드분) attached · ko 스크린샷 6.

## 1.1 (build 3) — 2026-09-01 제출 → **READY_FOR_SALE** (2026-09-01 확인)

> 2026-09-01 밤 ASC API로 지원 URL을 읽다가 확인 — 제출 당일 통과. 1.0(08-30 제출)에 이어 두 번째 무리젝.

1.0이 **READY_FOR_SALE**로 통과한 직후 제출. 기본 모드(640문항)·오답노트·요약 노트(60 topic)·좌우 스와이프·홈 개편(D-day·현황·이어풀기)을 한 번에 실었다.

| 항목 | 내용 |
|---|---|
| 부제 | `심화·기본 기출 1,742문항 · 요약 노트` (24/30자) |
| 키워드 | 기본·4급·요약노트 추가 (63/100자) |
| 스크린샷 | **새 UI 5장** — Pro Max 시뮬레이터에서 Maestro(`maestro/store.yaml`)로 원본을 찍고 `tools/storeshots.py`가 다크 프레임+헤드라인 합성. 강의 카드는 접힌 상태로만(초상권) |
| 심사 메모 | 요약 노트가 자체 재서술 저작물임을 추가 |
| 파이프라인 | `fastlane ios release` 한 번에 아카이브→업로드→메타→제출 성공 |

### 밟은 지뢰 — 이중 업로드 후 정리 경로

deliver가 또 스크린샷을 **두 번 올려 10장**이 됐다(플레이북 경고 그대로). 제출 후엔 삭제가 잠기므로 `asc cancel-review` → 정리 → 재제출로 갔는데, **취소하면 버전 상태가 PREPARE_FOR_SUBMISSION이 아니라 `DEVELOPER_REJECTED`가 된다.** `asc screenshots`·`dedupe-screenshots`가 PREPARE만 편집 가능으로 보고 있어 **"removed: 0"으로 조용히 무동작**했다. 도구의 상태 가드에 DEVELOPER_REJECTED를 추가해 재컴파일 → 5장 제거 → 새 `resubmit` lane(스크린샷·메타·바이너리 skip, 제출만)으로 재제출. 스킬 플레이북에 반영.

## 1.0 (build 2) — 2026-08-30 아이폰 전용 전환

**아이패드 스크린샷 요구를 피하려고 iPad 지원을 뺐다** (홍 결정 2026-08-30). `TARGETED_DEVICE_FAMILY = "1,2"` → `1` (Debug·Release 2곳), 빌드 넘버 1→2. IPA의 `UIDeviceFamily [1]`·`CFBundleVersion 2`를 업로드 전에 확인했다. 아이폰 전용이어도 아이패드에선 호환 모드로 실행된다. build 2 업로드 → VALID → 버전 1.0에 연결 완료. 남은 것은 build 1 때와 동일(아래 셋).

## 1.0 (build 1) — 2026-08-28 준비 중

### 진행 상태

| 단계 | 상태 |
|---|---|
| 아카이브·IPA | ✅ `fastlane/build/App.ipa` (1.3MB) |
| 바이너리 업로드 | ✅ build 1 · **VALID** → **build 2로 교체 (2026-08-30)** |
| 버전 연결 | ✅ `asc attach-build` |
| 메타데이터(ko) | ✅ 이름·부제·설명·키워드·심사메모 |
| 스크린샷 | ✅ 5장 (1320×2868) — **중복 제거 후 5장 확정** |
| **지원/개인정보 URL** | ⏳ 홍이 Notion으로 작성 예정 |
| **가격 ₩4,400** | ⏳ ASC 웹 |
| **App Privacy** | ⏳ ASC 웹 (전부 "수집 안 함") |
| 심사 제출 | ⏳ 위 셋 완료 후 `fastlane ios submit` |

### 이름이 바뀌었다

제안은 `한국사 정복 - 한능검 기출`이었는데 홍이 ASC에서 **`한능검 정복`**으로 생성했다. 메타데이터를 거기 맞추고, 빠진 "한국사" 검색어는 **부제(`한국사 심화 기출 1,102문항`)와 키워드**로 채웠다.

### 밟은 지뢰 넷 (전부 스킬 플레이북에 반영)

1. **`build_app`이 `invalid byte sequence in UTF-8`로 죽었다** — 진짜 원인은 **Xcode 26부터 export method가 `app-store` → `app-store-connect`로 바뀐 것**. gym이 옛 값을 보내 실패했고, 그 에러를 처리하던 코드가 **한글 경로 때문에 UTF-8 예외**로 죽어 원인이 가려졌다. `export_options: { method: "app-store-connect" }`로 해결.
2. **설명의 괘선(`─`)을 ASC가 거부** — *"Description can't contain the following character(s): ─"*. ASCII 하이픈으로 교체. (`■`는 통과)
3. **스크린샷이 이중 업로드됐다** — 로그에 `Successfully uploaded all screenshots`가 2번 찍혔고 실제로 **10장**이 올라갔다. 플레이북이 경고한 그대로라 **제출 전 API로 장수를 검증**해 5장으로 정리했다.
4. **`submit_for_review: false`면 deliver가 빌드를 연결하지 않는다** — `build_number`를 줘도 붙지 않는다. `asc attach-build`로 직접 연결.

### 도구 개선

`asc`에 명령 넷을 추가했다 — `builds` · `attach-build` · `screenshots` · `dedupe-screenshots`. 3번 지뢰(이중 업로드)를 API로 잡으려면 목록·삭제가 필요한데 기존 도구엔 없었다.
