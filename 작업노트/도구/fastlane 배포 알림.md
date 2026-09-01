---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-01
updated: 2026-09-01
projects:
  - "탭탭"
---

# fastlane 배포 알림

한 줄 요약 — "배포 알림이 안 온다"는 대개 알림 코드가 **실패한 게 아니라 조용히 안 불린 것**이다. `notify_discord`는 webhook ENV가 없으면 `return`하고, 새로 추가한 lane에는 호출 자체가 빠져 있다. 둘 다 로그에 아무 흔적을 안 남긴다.

## 핵심 정리

- **탭탭 Fastfile(`TapTap/fastlane/fastfile`)의 `notify_discord`는 `ENV["DISCORD_WEBHOOK_URL"]`이 없으면 `return unless webhook_url`로 조용히 빠진다.** 에러도 경고도 없고, fastlane summary의 스텝 목록에서 `curl -H`가 통째로 사라질 뿐이다. **알림이 나갔는지 확인하는 방법은 summary에 `curl -H` 스텝이 있는지 보는 것.**
- **webhook은 `TapTap/fastlane/.env`에만 있고 그 파일은 `.gitignore`의 `*.env`에 걸린다.** 그래서 **로컬 실행에서만 알림이 나가고 CI에서는 원래부터 안 나갔다** — `.github/workflows/iOS.yml`의 Fastlane 스텝 `env:`에 `DISCORD_WEBHOOK_URL`이 없다. 마지막 성공 런(2026-07-28 `v1.1.2`, run 30361095983)의 로그에도 curl이 없고 summary가 7단계 `upload_to_app_store`에서 끝난다. CI에서 알림을 살리려면 리포 시크릿 등록 + 워크플로 `env:` 전달 둘 다 필요하다.
- **플랫폼 lane을 새로 만들 때 알림 호출이 같이 안 따라온다.** `platform :ios`의 4개 lane(`beta`·`appstore`·`ci_beta`·`ci_release`)에는 다 있는데 나중에 추가된 `platform :macos`의 `beta`에는 없었다.
- **`def notify_discord`가 `platform :ios do ... end` 블록 안에 정의돼 있어도 `platform :macos`의 lane에서 그대로 호출된다.** Fastfile은 하나의 객체 컨텍스트로 평가돼서 블록 밖 lane에서도 같은 메서드가 잡힌다 — 2026-09-01 macOS lane에서 실제로 호출 성공(아래 근거). 알림을 붙일 때 메서드를 복사할 필요가 없다.
- **알림은 `upload_to_testflight` 뒤에 두는 게 안전하다.** 알림이 깨져도 업로드는 이미 끝나 있다.

## 기록

### 2026-09-01 — 탭탭 macOS 1.1.0 build 4: "요즘 Discord 알림이 안 온다"의 원인 3겹

- 맥락: 탭탭 TestFlight 업로드 때 오던 Discord 알림이 최근 안 온다는 물음에서 출발. 최신 빌드를 올리는 김에 원인을 확인했다.
- 배운 것: 위 「핵심 정리」 전부. 원인이 하나가 아니라 **① macOS lane에 호출 없음 ② iOS CI가 2026-07-28 이후 안 돎(`iOS.yml`은 `main` push/`v*` 태그 전용, `main` 최신은 PR #120) ③ CI에는 webhook ENV가 없어 돌아도 무음** 3겹이었다. 최근 맥 업로드 3번(1.1.0 build 1·2·3 — 08-07, 08-20, 08-26)이 전부 ①에 해당한다.
- 근거: `gh run list --workflow iOS.yml`로 마지막 실행이 2026-07-28임을 확인, `gh run view 30361095983 --log | grep -i discord` → 0건. macOS `beta` lane 끝에 `notify_discord` 추가 후 build 4 업로드 → fastlane summary에 `7 | curl -H 'Content-Type: application/json' ...` 스텝이 찍히고 Discord에 "🚀 macOS TestFlight 업로드 완료 / 버전: 1.1.0 (4)" 전송(226바이트, 에러 없음). 수정 커밋 `b3330a7` (브랜치 `fix/mac-discord-알림`).

### 2026-09-01 — 업로드 결과 조회는 lane 밖에서 하면 2FA로 샌다

- `fastlane run latest_testflight_build_number app_identifier:… version:1.1.0 platform:osx`를 그냥 실행하면 **API 키 없이 Spaceship(Apple ID) 로그인으로 떨어져** `Error: Incorrect verification code`가 반복된다. lane 안에서는 앞선 `app_store_connect_api_key`가 세션을 깔아줘서 되던 것이다.
- 단독 조회는 `api_key_path:<key.json>`을 같이 줘야 한다([[작업노트/도구/fastlane 로컬 실행 환경|fastlane 로컬 실행 환경]]에 JSON 형식). 그게 번거로우면 **업로드 로그의 `Successfully exported and signed the pkg file` → `Successfully uploaded package to App Store Connect`와, 업로드 직전 lane이 찍은 `Latest upload for version 1.1.0 on osx platform is build: N`** 조합으로 확인하는 게 빠르다.
