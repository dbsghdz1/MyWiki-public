---
type: project
status: active
aliases:
  - BarStack 심사 이력
created: 2026-07-22
updated: 2026-09-30
related_wiki: []
sources:
  - "2026-07-22-apple-app-review-2-1-information-needed-barstack"
  - "2026-07-23-apple-app-review-2-4-5-entitlements-barstack"
---

# BarStack — App Store 심사 이력

버전·빌드별 제출과 심사 결과, 그리고 각 대응을 시간순으로 기록한다. 앱: BarStack, 번들 ID `com.hong.CollectionTopBar`, 팀 `WN2B884S76`.

## 1.0 (1) — 최초 제출

- **결과**: 2026-07-22 **Guideline 2.1 - Information Needed** 수신 (기능 거절이 아니라 정보 요청) — 원문: [Apple 회신 원문](../../../_wiki/Sources/2026/07/2026-07-22-apple-app-review-2-1-information-needed-barstack.md)
- **요구 사항 7가지**: ① 실기기 스크린 레코딩(앱 실행 장면으로 시작, 민감 권한 프롬프트 포함) ② 테스트 기기·OS 목록 ③ 앱 목적·대상 사용자 ④ 설정·사용 방법 ⑤ 외부 서비스 목록 ⑥ 지역별 차이 ⑦ 규제 산업·제3자 자료 증빙
- **당시 앱 상태**: 손쉬운 사용(Accessibility) 권한 기반으로 메뉴바 아이템을 감지·클릭하는 설계였고, "Allow Access" 버튼이 샌드박스에서 무반응인 버그가 있었다.

### 2차 회신 — 2.4.5(i) Performance (2026-07-23)

- 1차 정보 회신 후 **Guideline 2.4.5(i)** 수신 — 원문: [Apple 2차 회신 원문](../../../_wiki/Sources/2026/07/2026-07-23-apple-app-review-2-4-5-entitlements-barstack.md)
- 지적: 기능과 무관한 entitlements 4개(`assets.music/movies/pictures.read-write`, `files.downloads.read-write`) — Xcode 템플릿 잔재. 메시지는 최초 바이너리 1.0 (1) 기준으로 작성된 것
- 사실 확인: 1.0 (3)은 이미 07-22 16:11 아카이브·업로드되어 "Ready for Review" 상태였고, 로컬 아카이브 검사로 entitlement가 `app-sandbox` 하나뿐임을 확인(빌드 2·3 모두). 해당 entitlements 제거는 커밋 `7325b01`
- 대응: 재업로드·Developer Reject 불필요. Resolution Center에 "1.0 (1)의 템플릿 잔재이며 현재 첨부된 1.0 (3)에는 app-sandbox만 남음" 회신

## 1.0 (2) — 준비만 하고 건너뜀 (미업로드)

- 커밋 `14d6fd0`에서 빌드 번호를 2로 올렸으나, 업로드 전에 추가 재설계(자동 그룹·프리셋 제거)가 이어져 실제로 업로드하지 않았다.
- 이 단계까지의 대응: BarStack 브랜딩 통일(`7325b01`), 샌드박스 권한 흐름 수정, 불필요 entitlement 제거, **샌드박스가 AX 감지를 차단함을 실측 확인 후 권한 제로 재설계**(`1763b29`), 보이는 `‹` 경계 핸들(`f044ebc`).

## 1.0 (3) — 승인·출시 ✅

- **결과**: 심사 통과, App Store 출시됨 (2026-07-25 사용자 확인 기준. 승인 일시와 Apple 통지 원문은 아직 수집하지 않음 — 원문을 Inbox에 넣으면 이 항목에 연결한다)
- 아래는 제출 당시(2026-07-22)의 기록이다.

- 커밋 `d17d338`에서 빌드 번호 3 확정. 최종 반영 사항:
  - 실행 앱 자동 그룹 제거(`92e4ef9`), 프리셋 등 껍데기 기능 전면 제거(`39eb36c`)
  - 거짓 개인정보 문구 수정(프리셋·Accessibility 언급 삭제)
  - 최종 앱: 권한 0, 네트워크 0, 5개 설정 탭, 전 기능 실동작
- **회신 계획**: Resolution Center에 영문 답변 + 실기기 녹화 첨부, 3~7번 내용을 App Review Information → Notes에 복사. 답변 요지: 권한·계정·결제·UGC 전무, 외부 서비스 없음, 지역 차이는 UI 언어(en/ko/ja/es)뿐, 규제 해당 없음. 전문: `(로컬 경로)`
- **함께 교체**: 스크린샷을 새 UI 기준으로 재제작(Figma 템플릿 한/영 각 6장, 2880×1800) — 원문 하단의 2.3.3 경고(스크린샷은 실제 앱 화면) 대응

## 1.1 (4) — 승인·출시 ✅

- **제출**: 2026-07-29 (사용자 제출 확인). 아카이브는 07-29 생성, 제출 전 codesign 검사로 entitlement가 `app-sandbox` 하나뿐임을 확인.
- **결과**: **승인 → App Store 출시** (2026-07-30 사용자 확인 기준. 지적 사항 없이 1회 통과. Apple 통지 원문 미수집 — Inbox에 넣으면 연결한다)
- **포함 내용**: 첫 실행 온보딩(신규 설치만, 기존 사용자는 자동 건너뜀) · 숨긴 아이콘 잠깐 보기 버튼 · 호버로 펼치기(기본 꺼짐) · 항목 표시 유지 정책(명시적 펼침은 접기 전까지 유지 — 이에 따라 접힘 시간 피커 제거) · 접기 20회 후 1회 리뷰 요청 · 새 앱 아이콘(‹ 핸들 히어로, 투명 여백) · 팝오버 헤더 정리(이름+상태)
- 관련 코드 커밋: `8e147fe`~`9753592`. 결정 경위는 [1.1 계획](1.1%20%EA%B3%84%ED%9A%8D%EA%B3%BC%20%EC%B6%9C%EC%8B%9C%20%EC%A7%81%ED%9B%84%20%EC%9E%91%EC%97%85.md) 참고. 제출 문구: `(로컬 경로)`

## 1.1.1 (6) — 승인·출시 ✅

- **제출**: fastlane `mac release`로 아카이브 → 업로드 → 메타데이터 → 심사 제출 자동화(첫 도입, Zappy 레인 복제). 첫 실행은 새 버전 리뷰 정보에 연락처(이름·이메일·전화)가 비어 `contactFirstName…` 필수 오류로 실패 → `review_information/` 4개 파일 채우고(git 제외) 재실행 → **Successfully submitted**. `asc state`: **WAITING_FOR_REVIEW**, build 6 VALID, en-US 스크린샷 4장(기존 승인본 유지).
- **포함 내용**: 팝오버 "숨긴 아이콘" 카드(실제 글리프 캡처, 옵트인 화면 기록 권한) · 즉시 접힘/펼침(슬라이드 애니메이션 제거) · 팝오버에서 로그인 시 실행 제거 · 팝오버 배경 regularMaterial. **Pro IAP·트리거는 코드에 있으나 사이드바에서 숨김**(`SidebarDestination.proSurfacesEnabled = false`) — 사용자 결정으로 다음 버전에.
- **메타데이터**: 스토어 설명이 1.0 시절(프리셋·자동 그룹·트리거·손쉬운 사용 권한)로 남아 있어 실제 기능에 맞게 재작성. 릴리즈 노트·심사 노트에 화면 기록 권한 용도("메뉴바 아이콘 자체만 캡처, 화면은 캡처 안 함")와 테스트 절차 명시. 로케일은 ASC에 en-US만 존재.
- **결과**: **승인 → App Store 출시** (2026-08-17 사용자 확인, `asc state` READY_FOR_SALE. 지적 사항 없이 1회 통과 — 화면 기록 권한 설명이 심사 노트로 충분했음)
- 관련 커밋: `fdff799`~`e62793c` (경위: [개발 기록 2026-08-16](BarStack%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-16.md)).

## 상태 요약

| 빌드 | 제출 여부 | 결과 |
|---|---|---|
| 1.0 (1) | 제출 | 2.1 Information Needed (07-22) → 정보 회신 후 2.4.5(i) entitlements 지적 (07-23) |
| 1.0 (2) | 미제출 | 재설계 계속으로 건너뜀 |
| 1.0 (3) | 제출됨 (07-22 업로드) | **승인 → App Store 출시** (2026-07-25 사용자 확인) |
| 1.1 (4) | 제출됨 (07-29) | **승인 → 출시** (07-30, 1회 통과) |
| 1.1.1 (6) | 제출됨 (08-17 00:03) | **승인 → 출시** (08-17, 1회 통과) |
| 1.1.2 (7) | 제출됨 (09-09) | **승인 → 출시** (09-10 01:44 KST, 1회 통과 — 2026-09-23 lint에서 확인) |
| 1.1.3 (8) | 제출됨 (09-30 11:44 KST) | 심사 대기 (WAITING_FOR_REVIEW) |

다음 갱신 시점: 다음 버전 제출 시. 상세한 재설계 경위는 [개발 기록](BarStack%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-07-22.md), 다음 버전 계획은 [로드맵](BarStack%20%EB%A1%9C%EB%93%9C%EB%A7%B5%EA%B3%BC%20%EC%9C%A0%EB%A3%8C%ED%99%94%20%EA%B3%84%ED%9A%8D.md) 참고.

## 1.1.3 (8) — 2026-09-30 제출

- **결과**: 제출 직후 `asc state 6792326039` → `WAITING_FOR_REVIEW`, build 8 VALID, en-US 스크린샷 4장(중복 없음). 자동 출시.
- **담은 것**: ① macOS 27에서 Compact가 아무것도 숨기지 못하던 문제 — 27은 화면 폭 절반 이상인 상태 아이템을 빼 버려 10,000pt ‹ 핸들이 사라졌다. 27에서만 핸들을 좁은 화면 폭/2−64로 줄이고 길이 0 스페이서 6개로 폭을 메운다. 27에서 숨긴 아이콘 목록은 끔. ② **1.1.2에 출시된 버그** — 접힘 상태에서 목록이 항상 0개(`c880818`의 `genuineRow`가 늘어난 핸들을 버림). ③ 접은 뒤 배치 안내 문구가 남던 것, Peek 중 목록 문구. 원인·근거는 [macOS 메뉴바와 샌드박스](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/macOS%20%EB%A9%94%EB%89%B4%EB%B0%94%EC%99%80%20%EC%83%8C%EB%93%9C%EB%B0%95%EC%8A%A4.md).
- **미실측**: macOS 27 기기가 없어 27 동작은 Hidden Bar·Thaw의 실측에 기댔다. 27 사용자는 업그레이드 후 숨길 아이콘을 한 번 다시 ⌘-드래그해야 한다(새 autosave 이름). 출시 후 27 제보로 확인.
- **절차 메모**: `upload` lane은 메타데이터를 건너뛰고 `submit` lane도 `skip_metadata: true`라 What's New가 안 올라간다. `fastlane deliver --api_key_path fastlane/asc_api_key.json --skip_binary_upload true --skip_screenshots true --build_number 8 --submit_for_review true`로 노트와 제출을 한 번에 했다. 코드: 커밋 `07aed98`, 브랜치 `fix/macos27-compact`(push, 미병합).

## 1.1.2 (7) — 2026-09-09 제출

- **결과**: **승인 → App Store 출시** (2026-09-09T16:44Z = 09-10 01:44 KST, 1회 통과). 2026-09-23 `asc state 6792326039` READY_FOR_SALE · iTunes lookup `currentVersionReleaseDate`로 확인. 위키는 09-09 WAITING에서 멈춰 있었다. Apple 통지 원문은 미수집.

숨긴 아이콘 목록에 아이콘이 두 번씩 들어가던 버그 하나만 고친 릴리스다. 원인과 재현은 [macOS 메뉴바와 샌드박스](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/macOS%20%EB%A9%94%EB%89%B4%EB%B0%94%EC%99%80%20%EC%83%8C%EB%93%9C%EB%B0%95%EC%8A%A4.md)에 있다 — **두 모니터의 윗변이 맞춰지면 창 서버가 진짜 행과 복제 행을 같은 `minY`로 보고**해 한 행으로 합쳐지고, 복제 행 전체가 ‹ 핸들 왼쪽으로 읽힌다.

실측으로 재현했다(`CGConfigureDisplayOrigin`으로 보조 모니터를 왼쪽·윗변 정렬로 옮김): 펼침 9 → **25개**, 접힘 9 → **18개**. 고친 뒤 둘 다 9개. 커밋 `c880818`.
