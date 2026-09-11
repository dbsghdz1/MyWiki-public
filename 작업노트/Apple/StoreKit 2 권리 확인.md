---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-21
updated: 2026-09-12
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
  - "[[프로젝트/개인/한능검/README|한능검]]"
---

# StoreKit 2 권리 확인 — `currentEntitlements`의 빈 결과는 "미구매"가 아니다

## 핵심

- `Transaction.currentEntitlements`는 **지금 이 기기에서 검증 가능한 거래**만 흘려준다. 오프라인, App Store 로그아웃, 앱 실행 직후(StoreKit 데몬 준비 전), 일시 오류에서는 **아무 것도 없이 끝난다**. 이것은 "구매한 적 없음"과 구별되지 않는다.
- 따라서 권리 상태 갱신은 **비대칭**이어야 한다: 유효 거래가 보이면 올리고, 내리는 건 `revocationDate`가 찍힌 거래(환불·가족 공유 해제)를 **실제로 관측했을 때**만. 빈 결과는 마지막 확인값을 유지한다.
- 대칭으로 짜면(`owned = false; for … { owned = true }; set(owned)`) 앱이 뜰 때마다 유료 기능이 "잠깐 꺼졌다 켜지는" 깜빡임이 생기고, 그 값을 위젯·익스텐션에 미러링하면 그쪽까지 튄다.
- 환불 반영은 `Transaction.updates`(앱 밖 거래 스트림)로 따로 받는다 — 여기서 오는 `revocationDate != nil`이 유일한 "내릴 근거".

- **유료 앱을 무료+IAP로 바꿀 때 기존 구매자는 `AppTransaction.originalAppVersion`으로 가린다.** iOS에서 이 값은 마케팅 버전이 아니라 **최초 구매 시점의 빌드 번호(CFBundleVersion)** 문자열이다. 유료로 팔던 마지막 빌드 이하면 이미 값을 낸 사람이므로 전량 해제한다. 그 상수는 한 번 정하면 올리지 않는다 — 올리는 순간 무료 시절 사용자까지 공짜로 열린다.
- **`AppTransaction`은 iOS 16+**다. 배포 타깃이 15면 그 구간에서는 판단이 불가능한데, 여기서 false를 돌리면 **이미 산 사람이 잠긴다.** 잘못 열어 주는 비용 < 잘못 잠그는 비용이므로 열어 주는 쪽으로 기운다.
- **샌드박스·StoreKit 테스트에서 `originalAppVersion`은 항상 "1.0"이다.** 그래서 시뮬레이터에서는 전원이 기존 구매자로 잡히고 IAP 구매 흐름이 아예 안 보인다 — **구매 경로 검증은 TestFlight(실제 빌드 번호)로만 된다.**
- 가격 전환 순서에 함정이 있다. **잠금이 들어간 새 버전이 승인됐는데 앱은 아직 유료면**, 그 사이에 산 사람은 앱 값을 내고도 빌드 번호가 새 것이라 기존 구매자로 안 잡혀 **IAP를 또 사야 한다.** 반대로 먼저 무료로 바꾸면 잠금 없는 구버전이 공짜로 풀린다. 어느 쪽도 공짜가 아니므로 **승인 뒤 수동 출시로 잡고, 가격 변경과 출시를 같은 자리에서 한다.**

## 기록

### 2026-08-21 — Zappy 위젯이 컬러→모노로 튀는 버그
- 맥락: 사용자 제보 "위젯이 컬러로 나오다 갑자기 모노로 바뀐다". 위젯은 App Group의 `plus` 값으로 컬러 허용을 결정하고, 앱은 `update()`마다 `ZappyStore.isPlus`를 거기 쓴다.
- 원인: `refreshEntitlements()`가 빈 결과를 `setPlus(false)`로 처리 → `syncWidget`이 `plus=false`를 써서 위젯 리로드 → 다음 확인에서 true로 복귀. 사용자가 의심한 "위젯 스타일 선택 해제(앱 따라가기)"와는 무관.
- 수정: `owned`/`revoked` 두 플래그로 분리, 빈 결과는 무시 (`Store.swift`, 커밋 `6a9ad80`). 미구매자는 애초에 `plusCached=false`라 영향 없음.
- 교훈: **부재(absence)를 부정(negation)으로 읽지 말 것.** 네트워크·데몬 의존 조회는 "없음"이 "아직 모름"일 수 있다.

## 참고 자료
- Apple 문서 — Transaction.currentEntitlements: https://developer.apple.com/documentation/storekit/transaction/currententitlements (2026-08-21 확인)
