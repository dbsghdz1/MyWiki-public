---
type: project
status: shipped
aliases:
  - Personal Mac App
  - 개인 Mac 앱
  - BarStack
  - CollectionTopBar
created: 2026-07-18
updated: 2026-08-18
slack_channel: barstack
repos:
  - "github.com/dbsghdz1/MacTopTopBarIconCollection"
related_wiki: []
---

# BarStack — 개인 Mac 앱

macOS 메뉴바 아이콘 정리 앱이다. 동작 규칙은 하나뿐이다: **"BarStack 아이콘 왼쪽의 `‹` 경계 핸들 기준, 왼쪽에 둔 아이콘은 숨기고 오른쪽은 항상 보여준다."** 리포지터리명은 `CollectionTopBar`(`(로컬 경로)`), 사용자에게 보이는 제품명은 BarStack이다.

## 핵심 결정 (2026-07-25 기준)

| 항목 | 현재 상태 |
|---|---|
| 앱 이름 | BarStack (리포지터리명은 CollectionTopBar) |
| 한 줄 문제 정의 | 노치 맥에서 메뉴바 아이콘이 넘치고 어수선해지는 문제를, 선택한 아이콘을 숨겨서 해결 |
| 대상 사용자 | 백그라운드 유틸리티를 많이 쓰는 일반 Mac 사용자·전문가 |
| 현재 개발 단계 | **1.1.1 App Store 출시됨** (2026-08-17 승인, 1회 통과 — 숨긴 아이콘 목록) — Pro IAP는 코드만 있고 숨김, [[프로젝트/개인/BarStack/1.2 Pro 계획|1.2 계획]] |
| 기술 스택 | SwiftUI + AppKit(NSStatusItem), 완전 App Sandbox, 네트워크 없음. 권한은 **화면 기록 1개(옵트인, 숨긴 아이콘 목록용)** — 2026-08-16에 "권한 제로" 폐기 |
| 핵심 기능 | Compact Mode 숨김 · 10초 자동 접힘 · Quick Reveal(⌃⌥⌘B) · 호버로 펼치기(기본 꺼짐, 1.1) · **숨긴 아이콘 목록(팝오버, 실제 글리프 캡처 — 1.1.1로 출시)** · 단축키 변경 · 로그인 시 실행 · 4개 언어(en/ko/ja/es) |
| 배포 방식 | Mac App Store 우선. 태그 릴리즈 시 공증 DMG도 GitHub Actions로 병행 |

## 프로젝트를 규정한 결정 3가지

1. **App Store 전용 + 권한 제로.** App Sandbox가 다른 앱의 메뉴바 아이템 읽기(Accessibility API)를 차단한다는 사실을 실기기 실측으로 확인했다(샌드박스 on: 감지 0개, off: 15개+). 이에 따라 AX 기반 기능을 전부 제거하고 어떤 권한도 요청하지 않는 앱으로 재설계했다.
   → **2026-08-16 수정: "권한 제로"는 폐기.** AX 차단은 재검증(권한 허용 상태에서도 0 vs 29)으로 재확인했지만, 화면 기록 권한이 있으면 window server가 메뉴바 아이템의 소유 앱(번들 ID)을 알려준다는 걸 찾아 **숨긴 아이콘 목록**을 옵트인 권한으로 넣었다. 클릭 실행은 여전히 불가. 상세: [[프로젝트/개인/BarStack/BarStack 개발 기록 2026-08-16|개발 기록 2026-08-16]].
2. **보이는 경계 핸들.** 투명한 경계는 사용자가 위치를 알 수 없어 앱이 코드로 경계를 추적해야 했고, 그 과정에서 아이콘 실종·사각지대 버그가 연쇄 발생했다. 펼침 상태에서만 보이는 `‹` 핸들로 바꾸면서 추적 코드를 전부 삭제했다.
3. **안 되는 기능은 팔지 않는다.** AX 시절의 잔재(실행 앱 자동 그룹, 프리셋, 아이템 규칙 저장)는 샌드박스에서 약속을 지킬 수 없어 전부 제거했다. 남은 기능은 모두 실동작한다.

상세한 경위와 기술 노트는 [[프로젝트/개인/BarStack/BarStack 개발 기록 2026-07-22|BarStack 개발 기록 2026-07-22]]·[[프로젝트/개인/BarStack/BarStack 개발 기록 2026-08-16|2026-08-16]], 버전·심사 진행은 [[프로젝트/개인/BarStack/App Store 심사 이력|App Store 심사 이력]], 향후 계획은 [[프로젝트/개인/BarStack/BarStack 로드맵과 유료화 계획|로드맵과 유료화 계획]] 참고. 버전별 실행 계획은 [[프로젝트/개인/BarStack/1.1 계획과 출시 직후 작업|1.1 계획]]·[[프로젝트/개인/BarStack/1.2 Pro 계획|1.2 Pro 계획]], 미확정 아이디어는 [[프로젝트/개인/BarStack/BarStack 아이디어 인박스|아이디어 인박스]]에 있다.

## 관련 리소스

- 코드: `(로컬 경로)` — GitHub `dbsghdz1/MacTopTopBarIconCollection`, 브랜치 `chore/notarized-release-ci`
- 심사 회신 자료: `(로컬 경로)` (영문 답변 + 스크린 레코딩 체크리스트)
- App Store 스크린샷 템플릿: Figma "개인" 파일 — 한국어 6장(y=2000)·영어 6장(y=4200), 2880×1800

## 배운 것

- [[공부/App Store 성장 도구|App Store 성장 도구]] — Mac 전용 앱이라 Apple Ads·인앱 이벤트·커스텀 제품 페이지를 못 쓴다. Pro 출시 때 쓸 수 있는 것은 피처링 노미네이션(최소 3주 전)과 프로모션 텍스트뿐.
- [[공부/macOS 메뉴바와 샌드박스|macOS 메뉴바와 샌드박스]] — 샌드박스는 AX를 통째로 막고, 메뉴바 아이템 식별은 CGWindowList + 화면 기록 권한(재실행 필요)으로만 가능. 디스플레이별 행·명명 규칙 차이 주의.

## 다른 프로젝트와의 경계

- [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]와는 별개의 프로젝트다.
- Swift/macOS 프로젝트이므로 React·TypeScript 학습 위키와 연결하지 않는다.
