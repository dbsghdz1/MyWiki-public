---
type: study
area: Flutter
audience: ai
status: active
created: 2026-08-27
updated: 2026-08-27
projects:
  - "보험찾개냥"
---

# iOS 접근성 트리 공백

## 핵심 정리

- 보험찾개냥 iOS에서 **일부 화면은 접근성 트리가 통째로 빈다**(상태바 요소만 남음). XCTest 기반 도구(Maestro)의 텍스트 셀렉터·assert가 전부 실패하고, VoiceOver 사용자도 그 화면을 못 읽는다는 뜻이다.
- **자동화가 특정 화면에서만 "Element not found"를 내면 셀렉터를 고치기 전에 트리부터 덤프한다** — `maestro hierarchy`.

## 기록

### 2026-08-27 — 04c(직접 입력)에서 재현 (보험찾개냥 SSH-292 스크린샷 자동화)

- 온보딩·로그인·홈·04(업로드)까지는 트리가 정상인데 **04c 진입 직후부터 텍스트가 하나도 안 잡혔다.** `maestro hierarchy` 라이브 덤프로 확인 — 상태바 3개 요소뿐.
- 코드에는 `ExcludeSemantics`/`BlockSemantics` 없음. 04c는 병원 검색 오버레이(`hospital_search_field`, CompositedTransform/Overlay 계열)를 쓰는 화면이라 **오버레이 위젯이 의심 지점**이나 원인 미확정. 05(추출 확인)도 같은 위젯을 써서 같을 가능성.
- 영향: Maestro 텍스트 탭·assert 불가 → 좌표 탭만 남는데, 좌표 blind 체이닝은 화면 전환 검증이 없어 드리프트로 실패했다. 안드로이드는 adb(트리 무관) + 매 탭 스크린샷 검증으로 우회 성공.
- **접근성 버그로서 별도 티켓감** — 자동화 문제가 아니라 VoiceOver 사용자 문제다. (2026-08-27 기준 티켓 미생성)
