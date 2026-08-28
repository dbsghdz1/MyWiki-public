---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-21
updated: 2026-08-21
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# StoreKit 2 권리 확인 — `currentEntitlements`의 빈 결과는 "미구매"가 아니다

## 핵심

- `Transaction.currentEntitlements`는 **지금 이 기기에서 검증 가능한 거래**만 흘려준다. 오프라인, App Store 로그아웃, 앱 실행 직후(StoreKit 데몬 준비 전), 일시 오류에서는 **아무 것도 없이 끝난다**. 이것은 "구매한 적 없음"과 구별되지 않는다.
- 따라서 권리 상태 갱신은 **비대칭**이어야 한다: 유효 거래가 보이면 올리고, 내리는 건 `revocationDate`가 찍힌 거래(환불·가족 공유 해제)를 **실제로 관측했을 때**만. 빈 결과는 마지막 확인값을 유지한다.
- 대칭으로 짜면(`owned = false; for … { owned = true }; set(owned)`) 앱이 뜰 때마다 유료 기능이 "잠깐 꺼졌다 켜지는" 깜빡임이 생기고, 그 값을 위젯·익스텐션에 미러링하면 그쪽까지 튄다.
- 환불 반영은 `Transaction.updates`(앱 밖 거래 스트림)로 따로 받는다 — 여기서 오는 `revocationDate != nil`이 유일한 "내릴 근거".

## 기록

### 2026-08-21 — Zappy 위젯이 컬러→모노로 튀는 버그
- 맥락: 사용자 제보 "위젯이 컬러로 나오다 갑자기 모노로 바뀐다". 위젯은 App Group의 `plus` 값으로 컬러 허용을 결정하고, 앱은 `update()`마다 `ZappyStore.isPlus`를 거기 쓴다.
- 원인: `refreshEntitlements()`가 빈 결과를 `setPlus(false)`로 처리 → `syncWidget`이 `plus=false`를 써서 위젯 리로드 → 다음 확인에서 true로 복귀. 사용자가 의심한 "위젯 스타일 선택 해제(앱 따라가기)"와는 무관.
- 수정: `owned`/`revoked` 두 플래그로 분리, 빈 결과는 무시 (`Store.swift`, 커밋 `6a9ad80`). 미구매자는 애초에 `plusCached=false`라 영향 없음.
- 교훈: **부재(absence)를 부정(negation)으로 읽지 말 것.** 네트워크·데몬 의존 조회는 "없음"이 "아직 모름"일 수 있다.

## 참고 자료
- Apple 문서 — Transaction.currentEntitlements: https://developer.apple.com/documentation/storekit/transaction/currententitlements (2026-08-21 확인)
