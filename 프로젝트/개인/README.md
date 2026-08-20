---
type: hub
status: active
aliases:
  - 개인프로젝트
  - Personal Projects
created: 2026-07-22
updated: 2026-08-19
related_wiki: []
---

# 개인 프로젝트

개인 프로젝트를 모두 묶는 영역 허브다. 2026-08-07 영역별 구조 개편으로 최상위에 있던 인스타카드뉴스, MyCryptoDiary가 이곳으로 편입되었다.

> [!note] `paused`의 의미 (2026-08-19 명시)
> 이 허브에서 `paused`는 **잠시 중지**다 — 실패·폐기가 아니라 **2026-11-14 지원 마감을 역산한 슬롯 배분 결정**이고, 각 프로젝트는 마지막 정상 상태 그대로 멈춰 있어 그 지점에서 재개한다. 재개 조건은 각 README의 보류 노트에 적혀 있다.

## 프로젝트 목록

| 프로젝트 | 설명 | 상태 |
|---|---|---|
| [[프로젝트/개인/BarStack/README|BarStack]] | macOS 메뉴바 아이콘 정리 앱 — 1.1 출시(2026-07-30), **1.1.1 출시(2026-08-17, 숨긴 아이콘 목록)**. 1.2 Pro는 08-15 보류(11-14 이후) | shipped |
| [[프로젝트/개인/DayTune/README|DayTune]] | 수면 기반 하루 계획 iOS 앱 — MVP 화면 merge 완료. **2026-08-14 잠시 중지**(중단 아님, 슬롯 배분 결정) — 지원 마감 2026-11-14 이후 재개 검토 | paused |
| [[프로젝트/개인/Zappy/README|Zappy]] | 배터리 캐릭터 메뉴바 앱 — **1.6.0 출시(2026-08-17, 위젯 첫 도입)**, 1.5.0은 자진 취소·흡수. 테마 14종(8종 무료) + Zappy+ IAP. 개발 정지·런치 플랜 실행 단계 | active |
| [[프로젝트/개인/math-sprint/README|math-sprint]] | Apps in Toss용 60초 암산 스프린트 게임 — React·Granite 기반, build·`.ait` 통과 상태로 멈춤. **2026-08-18 잠시 중지**(중단 아님, 슬롯 배분 결정) — 지원 마감 11-14 이후 재개 검토 | paused |
| [[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]] | 인스타그램 카드뉴스 생성·발행 자동화 도구 (Python) — Oracle systemd timer 08:00·23:00 자율 발행(08-14~) | active |
| [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] | 가상 1,000만원 모의투자 거래소 + 매매일기 + 유저 랭킹(08-15 전환) — iOS→React 전환용 핵심 포트폴리오, 2주 계획 D2(Neon+Drizzle) 진행 중, 실시간·WebSocket은 Phase 2(08-28~) | active |

## 새 프로젝트 추가 규칙

- `프로젝트/개인/<이름>/README.md`를 AGENTS.md의 프로젝트 frontmatter로 생성한다.
- 이 목록과 [[_wiki/index|Wiki Index]]에 등록한다.
- 코드 저장소가 있으면 README의 `repos`와 `코드 저장소` 섹션에 연결한다.
