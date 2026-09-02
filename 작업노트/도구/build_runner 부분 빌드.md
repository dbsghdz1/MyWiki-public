---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-27
updated: 2026-09-02
projects:
  - "소프트웨어마에스트로"
---

# build_runner 부분 빌드

`--build-filter`로 부분만 재생성하면 **필터 밖의 기존 생성 파일이 지워질 수 있다.**

## 핵심 정리

- DTO 하나만 빨리 재생성하려고 `dart run build_runner build --delete-conflicting-outputs --build-filter="lib/data/services/api/dto/treatment/**"`를 돌렸더니, **필터 밖**의 `.g.dart`들(document·sample DTO)이 사라졌다. 직후 `flutter analyze`가 무관한 파일에서 `uri_has_not_been_generated`·`undefined_method`(`_$...FromJson`)를 쏟아낸다.
- **build_runner가 0% CPU로 멈춰 있으면 `.dart_tool/build`를 지운다.** 죽은 실행이 남긴 asset graph가 깨지면 로그가 한 줄에서 안 넘어가고 프로세스가 CPU 0%로 무한 대기한다 — 느린 게 아니라 막힌 것이다.
- 증상이 "내가 안 건드린 파일이 깨짐"이라 원인을 못 찾기 쉽다 — **부분 빌드 직후 무관 DTO 에러가 나면 필터 없이 전체 `build_runner build`를 다시 돌리는 게 복구다** (3초면 끝난다, 어차피 캐시가 있다).

## 기록

### 2026-08-27 — 진료내역 DTO 개명 후 재생성하다가

- 맥락: 소프트웨어마에스트로 SSH-289 — `TreatmentRecordResponse.id`를 `treatmentRecordId`로 바꾸고 `.g.dart`만 부분 재생성
- 배운 것: 위 핵심 정리. 필터로 아낀 시간보다 원인 찾는 시간이 길다 — 그냥 전체 빌드가 낫다
- 근거: analyze 에러 13건 실측 → 전체 `build_runner build`로 0건 복구. PR #83

### 2026-09-02 — 0% CPU로 20분 멈춘 build_runner, 캐시를 지우니 16초

- 맥락: 소프트웨어마에스트로 SSH-441 — 로그인 응답 DTO(`SocialLoginResponse`)를 새로 만들고 `.g.dart` 생성
- 증상: `dart run build_runner build`가 `0s freezed on 335 inputs; lib/app.dart` 한 줄에서 20분 넘게 안 넘어갔다. `ps`로 보면 **CPU 0.0%**, `sample`은 이벤트 루프 대기. 타임아웃으로 죽이고 다시 돌려도 같은 자리
- 원인: 앞서 타임아웃으로 죽인 실행이 `.dart_tool/build`의 asset graph를 깨뜨린 상태였다. `.dart_tool/build/lock/build_runner.lock`을 지우는 것만으로는 안 풀린다
- 해법: **`rm -rf .dart_tool/build` 후 재실행 → 16초에 28개 출력 완료.** 전체 재빌드라 다른 `.g.dart`가 손상되지도 않았다(`git status`로 확인)
- **오래 걸리는 것과 막힌 것을 CPU %로 가른다** — 0%면 기다려도 안 끝난다
- 곁가지: 이 버전은 `--delete-conflicting-outputs`가 **제거돼 무시된다**(`W These options have been removed and were ignored`). 그리고 출력을 `| tail`로 파이프하면 버퍼링돼 진행이 안 보인다 — 파일로 리다이렉트해서 봐야 한다

