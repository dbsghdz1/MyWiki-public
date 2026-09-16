---
type: study
area: Flutter
audience: ai
status: active
created: 2026-09-15
updated: 2026-09-16
projects:
  - "보험찾개냥"
---

# Flutter iOS TestFlight 업로드

fastlane 없이 **ASC API 키 하나로** Flutter 앱을 남의 팀 계정에 서명·업로드한다. 로컬에 그 팀 배포 인증서가 없어도 된다.

## 핵심 정리

- **순서: `flutter build ios --release --no-codesign` → `xcodebuild archive` → `xcodebuild -exportArchive`(`destination=upload`).** `flutter build ipa`는 xcodebuild에 인증 플래그를 못 넘기므로 쪼갠다. archive·export 둘 다에 `-allowProvisioningUpdates -authenticationKeyPath … -authenticationKeyID … -authenticationKeyIssuerID …`를 준다.
- **로컬 배포 인증서가 0개여도 업로드된다.** 아카이브는 그 팀의 `Apple Development` 인증서로 서명되고, export 단계에서 **Apple 클라우드 서명**이 배포 서명을 대신한다. 업로드 후에도 키체인의 그 팀 identity 수는 그대로 0 — 로컬에 인증서를 만들지 않는다.
- ExportOptions: `method=app-store-connect` · `destination=upload` · `teamID` · `signingStyle=automatic` · `manageAppVersionAndBuildNumber=false`(빌드 번호는 `--build-number`로 직접).
- **`--dart-define` 값은 `ios/Flutter/Generated.xcconfig`의 `DART_DEFINES`에 base64 쉼표 목록으로 남는다.** `xcodebuild archive`가 이걸 다시 읽으므로 flutter build 때 넘긴 값이 아카이브에 그대로 들어간다. 업로드 전 검증은 여기를 디코드해서 `APP_ENV=dev`가 있는지 본다.
- **`grep -q`로 파이프 검증을 하면 `pipefail`에서 거짓 실패가 난다.** `grep -q`는 찾자마자 끝나고 앞 명령이 SIGPIPE로 죽는데 `pipefail`이 그걸 파이프 실패로 본다 — "찾았는데 없다"로 중단된다. 변수에 담아 zsh 조건식의 `==` 글롭 비교로 판정한다(파이프 없음).
- **설정 파일에서 키를 뽑을 땐 `^KEY=` 줄로 좁힌다.** `Kakao.xcconfig`는 주석에도 `KAKAO_NATIVE_APP_KEY`가 3번 나와 `awk '/KAKAO_NATIVE_APP_KEY/'`는 여러 줄을 잡고 길이 검사(32자)에서 떨어진다.
- **iOS 앱 아이콘은 1024·알파 없음·사각형.** 디자이너가 둥근 모서리를 투명으로 잘라 준 PNG는 그대로 못 쓴다 — 모서리를 배경색으로 채운다. 배경이 세로 그라데이션이면 **줄마다 왼쪽에서 처음 만나는 불투명 픽셀 색**을 그 줄 배경으로 깔고 원본을 합성하면 이음새가 안 보인다. Android 레거시 `mipmap-*/ic_launcher.png`는 둥근 투명본 그대로 둔다.
- **내부 테스터는 API로 한 번에 넣는다.** `POST /v1/betaGroups`(`isInternalGroup:true`, `hasAccessToAllBuilds:true`, app 관계) → `POST /v1/betaTesters`(email + betaGroups 관계). 대상은 **그 앱을 볼 수 있는 ASC 사용자**여야 한다 — `GET /v1/users/{id}/visibleApps`로 먼저 확인(`allAppsVisible:false`인 사용자도 앱이 배정돼 있으면 된다).

## 기록

### 2026-09-16 — build 4 업로드, 그리고 "빌드 번호는 기록이 아니라 서버에 묻는다" (보험찾개냥 SSH-539)

- 맥락: 자주 묻는 질문·고객센터 화면(PR #148)을 홍이 테플에서 보려 해서 `feat/SSH-539`(`9fef0ff`, 머지 전) 그대로 올렸다. 전날 만들어 둔 `soma-asc/`(키·ExportOptions·`archive_upload.sh`)를 그대로 재사용했다.
- **실패 한 번**: 아래 09-15 기록이 「build 2까지」라 build 3으로 올렸더니 export에서 거부됐다 — `The bundle version must be higher than the previously uploaded version: '3'`. 기록에 없는 build 3이 ASC에 이미 있었다. **내 기록은 ASC의 상태가 아니다** — 번호는 올리기 전에 서버에 묻거나, 실패 메시지가 알려주는 값(+1)으로 바로 다시 올린다. 아카이브는 31초라 재시도가 싸다.
- 재빌드(build 4) → `** ARCHIVE SUCCEEDED **` → `Progress 100%: Upload succeeded` / `** EXPORT SUCCEEDED **`(17:12:54).
- **빌드 조회 수단이 없다**: 스킬의 `asc` CLI는 `apps | state | cancel-review | app-infos | add-locale | set-privacy | set-primary`뿐이고 `state <appId>`는 **App Store 버전**(PREPARE_FOR_SUBMISSION)을 보여줄 뿐 TestFlight 빌드가 아니다. 이 맥에는 `jwt`·`cryptography` 파이썬 모듈이 없어 ASC API를 바로 부르기도 어렵다 — 처리 상태는 사람이 TestFlight에서 보거나, 필요해지면 `asc`에 builds 명령을 붙여야 한다.


### 2026-09-15 — 보험찾개냥 iOS 첫 TestFlight 업로드 (팀원 개인 팀 계정)

- 맥락: 보험찾개냥 홍 "테플에 올리자". 저장소에 fastlane·iOS 배포 CI가 없고 `pubspec` 버전은 `1.0.0+1`, 서명 팀 `34X3BT977M`은 로컬 `Apple Development: <팀원 이름>` 인증서의 `OU`로 팀원 개인 팀임을 확인. 홍 개인 팀(`WN2B884S76`) ASC에는 앱·번들 ID가 없었다. 홍이 그 팀 API 키를 받아 줬다(ASC 앱 `6804209753`은 이미 있음, 홍은 `APP_MANAGER`).
- 배운 것: 위 「핵심 정리」 전부. dev 서버(`APP_ENV=dev`)·`#142` 머지된 main `c2c3710`으로 **build 1**(16:48 업로드, 3분 뒤 `VALID`), 새 아이콘으로 **build 2**(23:10, `IN_BETA_TESTING`). 내부 그룹 `내부 테스트` 생성 후 홍 초대 → `INSTALLED`.
- 근거: `archive.log` `** ARCHIVE SUCCEEDED **` · `export.log` `Upload succeeded` / `** EXPORT SUCCEEDED **` · `security find-identity -v -p codesigning | grep -c 34X3BT977M` = 0 · 아카이브에서 꺼낸 `AppIcon60x60@2x.png` 육안 확인 · 아이콘 커밋 `18d61ac`(로컬 `design/app-icon`, Jira 번호 받기 전이라 미푸시). 확인 스크립트의 `grep -q` 거짓 실패로 첫 실행이 업로드 전에 중단됐다.
