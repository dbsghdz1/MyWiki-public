---
type: study
area: 언어·프레임워크
status: active
created: 2026-08-22
updated: 2026-08-22
projects:
  - "[[프로젝트/개인/DayTune/README|DayTune]]"
---

# HealthKit 수면 데이터 조회

"지난밤 수면"은 **달력 날짜로 자를 수 없다.** 수면은 자정을 가로지르고, 사용자는 자정 이후 취침 전에도 앱을 연다. 자정 기준 윈도우(`startOfDay - 6h ~ now`)는 **자정~취침 사이에 열면 직전 밤이 통째로 빠져** "데이터 없음"이 된다 — 워치 설정이 다 켜져 있어도.

## 핵심 정리

- **윈도우는 "지금부터 N시간 전"으로 잡고, 세션으로 묶은 뒤 마지막 세션만 쓴다.** 36시간 거슬러 올라가면 이틀 밤이 같이 들어오므로, 샘플을 시작 시각순으로 훑어 **3시간 넘는 공백에서 세션을 끊고** 마지막 세션만 합산한다. 새벽 2시에 열면 어젯밤(20일 23시~21일 07시)이, 아침 8시에 열면 방금 잔 밤이 잡힌다. 윈도우만 넓히고 세션 분리를 안 하면 아침에 두 밤이 합산된다.
- **`HKCategoryValueSleepAnalysis.allAsleepValues`로 거른다.** iOS 16+ 워치는 `asleepCore`/`asleepDeep`/`asleepREM`으로 단계를 쪼개 기록하고, 구형·서드파티는 `asleepUnspecified`. `inBed`는 "침대에 있음"이라 수면 시간에서 제외. `allAsleepValues`가 이 넷을 모두 묶어준다(iOS 16+).
- **같은 구간을 여러 소스가 기록할 수 있다** — 워치 + 아이폰 "수면 집중 모드", 서드파티 앱. 단순 합산하면 수면이 두 배로 나온다. 구간 **합집합**으로 세거나(`coveredUntil` 포인터 유지), 소스를 하나로 제한한다.
- `predicateForSamples(withStart:end:options: .strictStartDate)`는 시작 시각이 윈도우 안인 샘플만. 윈도우 경계를 걸치는 긴 샘플을 놓치기 쉬우니 "최근 N시간" 방식에서는 옵션을 비우는 게 낫다.
- HealthKit은 **읽기 권한 거부를 알려주지 않는다** — `authorizationStatus`는 쓰기 권한만 의미 있고, 읽기 거부는 그냥 빈 결과로 온다. "데이터 없음"이 권한 문제인지 정말 없는지 코드로 구분할 수 없으므로 UI는 둘 다 안내해야 한다.
- 세션 분리 같은 날짜 로직은 **HealthKit에서 떼어 순수 함수로** 두면(`[DateInterval] → Summary`) 실기기 없이 단위 테스트된다. 시뮬레이터에는 수면 데이터가 없으니 이게 유일한 검증 수단이다.

## 기록

### 2026-08-22 — 워치 수면 추적이 켜져 있는데 "수면 데이터 없음"
- 맥락: [[프로젝트/개인/DayTune/README|DayTune]] 새벽 2시 테스트. `fetchLastNightSleep()`이 `startOfDay(now) - 6h`(어제 18:00) 이후 샘플만 조회 → 직전 밤(시작 20일 23시경)이 윈도우 밖 → nil.
- 수정: `HealthKitHealthDataRepository` — 36h lookback + `latestSleepSession(from:)` 순수 함수(3h 공백 세션 분리, 합집합 합산). 테스트 6개 추가(`HealthKitHealthDataRepositorySleepSessionTests`): 단일 밤 단계 합산, 두 밤 중 마지막만, 짧은 각성은 같은 세션, 중복 소스 미중복, 정렬 안 된 입력.
- 근거: 수정 전 코드 `HealthKitHealthDataRepository.swift:55-72`, 수정 후 테스트 18/18 통과.

## 참고 자료

- Apple — HKCategoryValueSleepAnalysis: https://developer.apple.com/documentation/healthkit/hkcategoryvaluesleepanalysis (2026-08-22 확인)
- Apple — Reading data from HealthKit (권한·빈 결과 규칙): https://developer.apple.com/documentation/healthkit/reading-data-from-healthkit (2026-08-22 확인)
