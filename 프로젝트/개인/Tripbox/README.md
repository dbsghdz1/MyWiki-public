---
type: project
title: "Tripbox"
summary: "가고 싶은 곳 링크를 둘이 공유해 두면 AI가 일정과 근처 현지 맛집을 정리해 주는 iOS 여행 앱"
status: active
aliases:
  - 여행앱
created: 2026-09-23
updated: 2026-09-24
repos:
  - "github.com/dbsghdz1/Tripbox"
related_wiki: []
launch_gate: exempt
launch_exception: "비수익 개인 도구 — 두 사람 여행용, TestFlight·직접 빌드로만 배포하고 스토어에 올리지 않는다"
launch_approved: 2026-09-23
---

# Tripbox

## 현재 카드
- **단계**: 7일 MVP
- **현재**: 2026-09-23 grilling 13문항으로 범위 확정 → PRD v1 작성, 레포 `docs/PRD.md`가 정본(09-24). 코드 없음
- **다음 판정**: 2026-10-30 출국 전 두 사람 폰 설치 · 2026-11-08 PRD 성공 기준 4번("다음 여행에도 쓴다")으로 닫을지·스토어 게이트로 갈지
- **지금 할 일**: 첫 기능 spec — 공유 확장으로 링크 담기 → 공유 목록
- **하지 않을 일**: App Store 출시 준비(스크린샷·심사·수익화)

## 문제

같이 여행 가는 두 사람이 인스타·블로그·유튜브·구글맵 링크를 각자 흩어 두고, 출발 전에 손으로 날짜별 일정에 옮긴다. 현지에서는 근처 맛집을 매번 새로 찾고, 그 나라 사람들이 믿는 신호를 모른다. 상세는 [[프로젝트/개인/Tripbox/PRD Tripbox v1 2026-09-23|PRD v1]].

## 주요 결정 (2026-09-23 grilling)

| # | 결정 | 버린 것 |
|---|---|---|
| Q1 | 두 사람용 개인 앱, TestFlight·직접 빌드 — 게이트 `exempt` | 스토어 제품 |
| Q2 | 링크는 태그 없이 전부 던져 두고 AI가 정리 | 공유 때 도시·일차 태그 |
| Q3 | v1에 AI 일정 짜기 + 근처 맛집 추천 둘 다 | 하나만 먼저 |
| Q4 | 둘이 같은 목록 — 작은 서버(Supabase)로 동기화, AI 호출도 서버 | CloudKit 공유, 각자 폰 |
| Q5 | 네이티브 셸(공유 확장·지도) + Next.js 웹뷰 + 브릿지 | 전부 SwiftUI, React Native |
| Q6 | 여행 뼈대는 앱에서 입력, 이번 여행은 초기 데이터로 | Notion 연동, 하드코딩 |
| Q7 | 맛집 후보는 Google Places, AI는 고르고 이유만 | MapKit 검색, AI 웹검색 |
| Q8 | AI 초안 → 둘이 수정·고정 → 고정 빼고 재생성 | 통째 재생성, 대화형 |
| Q9 | Sign in with Apple + 초대 링크, 담은 사람 표시 | 여행 코드, 기기 고정 |
| Q10 | 🏅 빕구르망·🍜 호커 배지(정적 목록 매칭) + 호치민 Foody 검색 링크 | AI 설명만, 틱톡·샤오홍슈 |
| Q11 | 장소 못 뽑은 링크는 링크로만 보관 | 메모 칸, 스크린샷 OCR |
| Q12 | 일정 = 동선 + 영업시간 + 식사 자리 비움 + 이동일 반나절 | 동선만, 날씨·페이스 |
| Q13 | 이름 Tripbox, 모노레포 `(로컬 경로)` (`ios/`·`web/`·`supabase/`), GitHub 비공개 | — |

## 나라별 현지 맛집 신호 (2026-09-23 조사)

- 세 나라 모두 Google Maps가 기본 도구이고 합법 데이터 경로는 Places API뿐이다.
- 싱가포르 — 호커센터가 단위, 미쉐린 빕구르망 2026판 97곳(노점 포함). Burpple은 API 없음.
- 쿠알라룸푸르 — Google Maps + 인스타·틱톡, 중국계는 샤오홍슈. 빕구르망 2025판 KL 24곳.
- 호치민 — Foody.vn(리뷰 148만+)이 카카오맵 자리지만 약관이 스크래핑 금지·API 없음. 빕구르망 2025판 베트남 63곳.

## 코드 저장소

- `(로컬 경로)` — [github.com/dbsghdz1/Tripbox](https://github.com/dbsghdz1/Tripbox) (비공개, 2026-09-24 생성)
- **PRD 정본은 레포 `docs/PRD.md`**, 결정 표는 `docs/decisions.md`. 위키의 PRD 파일은 09-23 작성본 사본이다.

## 기록

- [[프로젝트/개인/Tripbox/PRD Tripbox v1 2026-09-23|PRD Tripbox v1 2026-09-23]]
