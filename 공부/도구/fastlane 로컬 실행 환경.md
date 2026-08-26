---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-26
updated: 2026-08-26
projects:
  - "탭탭"
---

# fastlane 로컬 실행 환경

한 줄 요약 — 로컬에서 fastlane을 돌릴 때 깨지는 건 대개 fastlane이 아니라 **Ruby 환경이 자식 프로세스로 새는 것**이다. CI에서는 Ruby가 하나뿐이라 안 걸리고 로컬에서만 터진다.

## 핵심 정리

- **`bundle exec`는 `RUBYOPT=-rbundler/setup`을 환경에 심고, 그게 `xcodebuild` 자식 프로세스까지 상속된다.** Xcode가 부르는 빌드 스크립트·SwiftPM 플러그인은 **시스템 ruby(`/usr/bin/ruby`, 2.6)** 로 도는데, 그 2.6이 Homebrew ruby(4.x)의 bundler를 로드하려다 죽는다.
  ```
  uninitialized constant Gem::Resolver::APISet::GemParser (NameError)
    from /System/.../Ruby.framework/Versions/2.6/.../kernel_require.rb:54
  ```
- **이때 gym 로그는 쓸모가 없다.** `xcodebuild ... | tee <gym.log> | xcpretty` 구조라 **stderr는 tee를 안 거친다.** 진짜 에러는 stderr로 나가서 fastlane 콘솔에만 찍히고, `(로컬 경로)`는 8줄쯤에서 끊긴다. `grep error:`로는 아무것도 안 잡힌다 — "원인 불명 build/archive error"로 보인다.
- **원인 판별은 환경을 벗겨서 같은 아카이브를 직접 돌려보면 1분이면 끝난다.**
  ```bash
  env -u RUBYOPT -u BUNDLE_GEMFILE -u BUNDLE_BIN_PATH -u GEM_HOME -u GEM_PATH \
    xcodebuild -workspace X.xcworkspace -scheme S -destination 'generic/platform=macOS' \
    -archivePath /tmp/T.xcarchive archive
  ```
  이게 성공하면 코드가 아니라 환경 문제다.
- **해법 둘.** ① fastlane이 Homebrew로도 깔려 있고 Gemfile과 같은 버전이면 **`bundle exec` 없이** 실행한다(가장 간단). ② Fastfile에서 `build_app` 앞에 `ENV.delete("RUBYOPT")` — 자식에게 안 물려준다. fastlane 자신은 이미 로드된 뒤라 영향 없다.
- **`/usr/bin/bundle`이 먼저 잡히면 그것부터 막힌다.** 비로그인 셸에서는 PATH가 달라 시스템 ruby 2.6의 bundle이 잡히고 `Could not find 'bundler' (4.0.16) required by your Gemfile.lock`이 난다. `/opt/homebrew/bin/bundle`을 절대 경로로 부르거나 PATH를 맞춘다.
- **`bundle install`은 `Gemfile.lock`의 CHECKSUMS 섹션을 채워 넣는다.** 결국 bundler를 안 쓰기로 했다면 되돌려야 팀 저장소에 노이즈가 안 남는다.

## 기록

### 2026-08-26 — 탭탭 macOS 1.1.0 build 3 TestFlight 업로드

- 맥락: 탭탭 macOS QA 결함 8건(작업 기록)을 머지하고 TestFlight에 올리려는데 `fastlane macos beta version:1.1.0`이 `build_app`에서 죽었다.
- 배운 것: 위 「핵심 정리」 전부. **에러 메시지가 Ruby 스택트레이스인데 정작 실패한 건 Xcode 아카이브**라 처음에 코드 문제로 의심했다. gym 로그가 비어 있는 게 오히려 단서였다 — 빌드가 시작도 못 했다는 뜻이라 컴파일 에러일 수가 없다.
- 근거: `bundle exec fastlane` 실패(build_app 23초 만에 💥) → 같은 아카이브를 `env -u RUBYOPT`로 직접 실행 → `** ARCHIVE SUCCEEDED **` → `bundle exec` 없이 `/opt/homebrew/bin/fastlane macos beta version:1.1.0` 재실행 → build 3 업로드 성공.

이 프로젝트 CI(`iOS.yml`)는 `bundle exec fastlane ios ci_beta`를 쓰는데 잘 돈다. **러너에 Ruby가 사실상 하나뿐이라 시스템 ruby와 버전이 엇갈릴 일이 없어서**다. 그래서 "CI는 되는데 내 맥에서만 안 된다"로 보인다.

업로드 검증은 로그의 "Successfully uploaded"만 믿지 말고 조회로 확인한다:

```bash
fastlane run latest_testflight_build_number \
  app_identifier:com.example.app version:1.1.0 platform:osx api_key_path:<key.json>
# → Latest upload for version 1.1.0 on osx platform is build: 3
```

`api_key_path`에 넣을 JSON은 `{"key_id":…, "issuer_id":…, "key":"<PEM 개인키 전문 — BEGIN/END PRIVATE KEY 줄까지 그대로, 줄바꿈은 \n>", "in_house":false}` 형태다. 임시로 만들었으면 **쓰고 나서 지운다.**

> PEM 헤더를 그대로 적으면 퍼블리시 차단 패턴(`BEGIN … PRIVATE KEY`)에 걸린다. 형식 설명이라도 리터럴로 쓰지 않는다 (2026-08-26 실제로 게이트에 걸려 수정).

> 참고: `latest_testflight_build_number`는 `platform:osx`를 줘야 맥 빌드를 본다. 안 주면 iOS 쪽을 조회해 엉뚱한 숫자가 나온다.
