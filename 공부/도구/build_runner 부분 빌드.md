---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-27
updated: 2026-08-27
projects:
  - "소프트웨어마에스트로"
---

# build_runner 부분 빌드

`--build-filter`로 부분만 재생성하면 **필터 밖의 기존 생성 파일이 지워질 수 있다.**

## 핵심 정리

- DTO 하나만 빨리 재생성하려고 `dart run build_runner build --delete-conflicting-outputs --build-filter="lib/data/services/api/dto/treatment/**"`를 돌렸더니, **필터 밖**의 `.g.dart`들(document·sample DTO)이 사라졌다. 직후 `flutter analyze`가 무관한 파일에서 `uri_has_not_been_generated`·`undefined_method`(`_$...FromJson`)를 쏟아낸다.
- 증상이 "내가 안 건드린 파일이 깨짐"이라 원인을 못 찾기 쉽다 — **부분 빌드 직후 무관 DTO 에러가 나면 필터 없이 전체 `build_runner build`를 다시 돌리는 게 복구다** (3초면 끝난다, 어차피 캐시가 있다).

## 기록

### 2026-08-27 — 진료내역 DTO 개명 후 재생성하다가

- 맥락: 소프트웨어마에스트로 SSH-289 — `TreatmentRecordResponse.id`를 `treatmentRecordId`로 바꾸고 `.g.dart`만 부분 재생성
- 배운 것: 위 핵심 정리. 필터로 아낀 시간보다 원인 찾는 시간이 길다 — 그냥 전체 빌드가 낫다
- 근거: analyze 에러 13건 실측 → 전체 `build_runner build`로 0건 복구. PR #83
