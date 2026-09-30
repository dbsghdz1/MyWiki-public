---
type: project
title: "BarStack"
summary: "macOS 메뉴바 아이콘을 정리하는 앱의 작업 진입점"
status: shipped
aliases:
  - Personal Mac App
  - 개인 Mac 앱
  - BarStack
  - CollectionTopBar
created: 2026-07-18
updated: 2026-09-30
slack_channel: 23-개인-barstack
repos:
  - "github.com/dbsghdz1/MacTopTopBarIconCollection"
related_wiki: []
---

# BarStack — 개인 Mac 앱

## 현재 카드
- **단계**: 운영
- **현재**: **1.1.3(build 8) 심사 대기** — 2026-09-30 제출, macOS 27 Compact 대응(미실측) + 1.1.2의 접힘 상태 목록 0개 버그 수정 — [심사 이력](App%20Store%20%EC%8B%AC%EC%82%AC%20%EC%9D%B4%EB%A0%A5.md). 출시 중인 것은 1.1.2. 1.2 Pro IAP는 코드만 있고 숨김 — [1.2 Pro 계획](1.2%20Pro%20%EA%B3%84%ED%9A%8D.md)
- **다음 판정**: 1.2 Pro 착수 여부 — 소마 종료(2026-12) 이후 재판정(계획 우선순위 표의 보류 결정). 착수 시 피처링 노미네이션은 출시 3주 전
- **지금 할 일**: 1.1.3 심사 결과 확인, 출시 후 macOS 27 사용자 제보로 Compact 동작 확인
- **하지 않을 일**: 1.2 Pro 착수(다음 판정 전)

macOS 메뉴바 아이콘 정리 앱이다. 동작 규칙은 하나뿐이다: **"BarStack 아이콘 왼쪽의 `‹` 경계 핸들 기준, 왼쪽에 둔 아이콘은 숨기고 오른쪽은 항상 보여준다."** 리포지터리명은 `CollectionTopBar`(`(로컬 경로)`), 사용자에게 보이는 제품명은 BarStack이다.

## 핵심 결정 (2026-07-25 기준)

| 항목 | 현재 상태 |
|---|---|
| 앱 이름 | BarStack (리포지터리명은 CollectionTopBar) |
| 한 줄 문제 정의 | 노치 맥에서 메뉴바 아이콘이 넘치고 어수선해지는 문제를, 선택한 아이콘을 숨겨서 해결 |
| 대상 사용자 | 백그라운드 유틸리티를 많이 쓰는 일반 Mac 사용자·전문가 |
| 현재 개발 단계 | **1.1.2 App Store 출시됨** (2026-09-10 KST, 1회 통과 — 숨긴 아이콘 목록 중복 수정. 1.1.1은 08-17) — Pro IAP는 코드만 있고 숨김, [1.2 계획](1.2%20Pro%20%EA%B3%84%ED%9A%8D.md) |
| 기술 스택 | SwiftUI + AppKit(NSStatusItem), 완전 App Sandbox, 네트워크 없음. 권한은 **화면 기록 1개(옵트인, 숨긴 아이콘 목록용)** — 2026-08-16에 "권한 제로" 폐기 |
| 핵심 기능 | Compact Mode 숨김 · 10초 자동 접힘 · Quick Reveal(⌃⌥⌘B) · 호버로 펼치기(기본 꺼짐, 1.1) · **숨긴 아이콘 목록(팝오버, 실제 글리프 캡처 — 1.1.1로 출시)** · 단축키 변경 · 로그인 시 실행 · 4개 언어(en/ko/ja/es) |
| 배포 방식 | Mac App Store 우선. 태그 릴리즈 시 공증 DMG도 GitHub Actions로 병행 |

## 프로젝트를 규정한 결정 3가지

1. **App Store 전용 + 권한 제로.** App Sandbox가 다른 앱의 메뉴바 아이템 읽기(Accessibility API)를 차단한다는 사실을 실기기 실측으로 확인했다(샌드박스 on: 감지 0개, off: 15개+). 이에 따라 AX 기반 기능을 전부 제거하고 어떤 권한도 요청하지 않는 앱으로 재설계했다.
   → **2026-08-16 수정: "권한 제로"는 폐기.** AX 차단은 재검증(권한 허용 상태에서도 0 vs 29)으로 재확인했지만, 화면 기록 권한이 있으면 window server가 메뉴바 아이템의 소유 앱(번들 ID)을 알려준다는 걸 찾아 **숨긴 아이콘 목록**을 옵트인 권한으로 넣었다. 클릭 실행은 여전히 불가. 상세: [개발 기록 2026-08-16](BarStack%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-16.md).
2. **보이는 경계 핸들.** 투명한 경계는 사용자가 위치를 알 수 없어 앱이 코드로 경계를 추적해야 했고, 그 과정에서 아이콘 실종·사각지대 버그가 연쇄 발생했다. 펼침 상태에서만 보이는 `‹` 핸들로 바꾸면서 추적 코드를 전부 삭제했다.
3. **안 되는 기능은 팔지 않는다.** AX 시절의 잔재(실행 앱 자동 그룹, 프리셋, 아이템 규칙 저장)는 샌드박스에서 약속을 지킬 수 없어 전부 제거했다. 남은 기능은 모두 실동작한다.

상세한 경위와 기술 노트는 [BarStack 개발 기록 2026-07-22](BarStack%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-07-22.md)·[2026-08-16](BarStack%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-16.md), 버전·심사 진행은 [App Store 심사 이력](App%20Store%20%EC%8B%AC%EC%82%AC%20%EC%9D%B4%EB%A0%A5.md), 향후 계획은 [로드맵과 유료화 계획](BarStack%20%EB%A1%9C%EB%93%9C%EB%A7%B5%EA%B3%BC%20%EC%9C%A0%EB%A3%8C%ED%99%94%20%EA%B3%84%ED%9A%8D.md) 참고. 버전별 실행 계획은 [1.1 계획](1.1%20%EA%B3%84%ED%9A%8D%EA%B3%BC%20%EC%B6%9C%EC%8B%9C%20%EC%A7%81%ED%9B%84%20%EC%9E%91%EC%97%85.md)·[1.2 Pro 계획](1.2%20Pro%20%EA%B3%84%ED%9A%8D.md), 미확정 아이디어는 [아이디어 인박스](BarStack%20%EC%95%84%EC%9D%B4%EB%94%94%EC%96%B4%20%EC%9D%B8%EB%B0%95%EC%8A%A4.md)에 있다.

## 관련 리소스

- 코드: `(로컬 경로)` — GitHub `dbsghdz1/MacTopTopBarIconCollection`, 브랜치 `chore/notarized-release-ci`
- 심사 회신 자료: `(로컬 경로)` (영문 답변 + 스크린 레코딩 체크리스트)
- App Store 스크린샷 템플릿: Figma "개인" 파일 — 한국어 6장(y=2000)·영어 6장(y=4200), 2880×1800

## 배운 것

- [App Store 성장 도구](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/AppStore/App%20Store%20%EC%84%B1%EC%9E%A5%20%EB%8F%84%EA%B5%AC.md) — Mac 전용 앱이라 Apple Ads·인앱 이벤트·커스텀 제품 페이지를 못 쓴다. Pro 출시 때 쓸 수 있는 것은 피처링 노미네이션(최소 3주 전)과 프로모션 텍스트뿐.
- [macOS 메뉴바와 샌드박스](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/macOS%20%EB%A9%94%EB%89%B4%EB%B0%94%EC%99%80%20%EC%83%8C%EB%93%9C%EB%B0%95%EC%8A%A4.md) — **macOS 27은 메뉴바가 창 하나라 아이템별 창이 없고, 폭 절반 이상인 상태 아이템을 제거한다**(2026-09-30, 1.1.3). 샌드박스는 AX를 통째로 막고, 메뉴바 아이템 식별은 CGWindowList + 화면 기록 권한(재실행 필요)으로만 가능. 디스플레이별 행·명명 규칙 차이 주의. **행을 `minY`로만 묶으면 두 모니터의 윗변이 맞춰졌을 때 진짜 행과 복제 행이 합쳐져 목록이 두 배가 된다**(2026-09-09 재현·수정).

- [가상머신과 컨테이너](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/CS/%EC%9A%B4%EC%98%81%EC%B2%B4%EC%A0%9C/%EA%B0%80%EC%83%81%EB%A8%B8%EC%8B%A0%EA%B3%BC%20%EC%BB%A8%ED%85%8C%EC%9D%B4%EB%84%88.md) — macOS 27 시험 환경을 고르다 궁금해짐: VM은 커널까지 새로 부팅, 컨테이너는 호스트 커널 공유라 다른 OS 버전은 VM만 가능.

## 다른 프로젝트와의 경계

- [MyCryptoDiary](../MyCryptoDiary/README.md)와는 별개의 프로젝트다.
- Swift/macOS 프로젝트이므로 React·TypeScript 학습 위키와 연결하지 않는다.
