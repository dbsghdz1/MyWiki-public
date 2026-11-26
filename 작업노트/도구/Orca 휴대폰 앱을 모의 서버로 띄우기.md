---
title: "Orca 휴대폰 앱을 모의 서버로 띄우기"
summary: "Orca 휴대폰 앱을 iOS 시뮬레이터와 모의 서버로 띄울 때 '이 호스트의 Orca 업데이트' 화면에 막히거나 봇 카드가 대체 문구로 나올 때 읽는다"
type: study
area: 도구
audience: ai
status: active
created: 2026-11-26
updated: 2026-11-26
projects:
  - "Orca 설정 변경"
---

# Orca 휴대폰 앱을 모의 서버로 띄우기

**Orca 휴대폰 앱의 호스트 화면(채팅 포함)은 앱에 든 코드가 아니라 데스크톱이 보내 주는 웹 번들(`mobileWeb.bundle.v1`)로 그려진다.** 그래서 저장소의 모의 서버(`mobile/scripts/mock-server.ts`)를 그대로 쓰면 페어링 뒤 "이 호스트의 Orca 업데이트" 화면에 막힌다. 모의 서버가 `pnpm build:mobile-web`으로 만든 번들을 보내게 하면 데스크톱 없이 휴대폰 채팅까지 띄울 수 있다 (2026-11-26, Orca `main` 3e22725924 기준).

## 핵심 정리

### 앱 빌드와 Metro

- `cd mobile && pnpm install` 뒤 `pnpm native ensure ios --device "iPhone 17 Pro"`. 빌드 캐시가 없으면 전체 빌드를 하고 시뮬레이터에 설치까지 한다.
- Metro는 `pnpm start --dev-client`. `scripts/start-expo.mjs`가 맨 이름 `pnpm`을 부르므로 PATH에 pnpm이 있어야 한다(corepack만 있으면 shim을 PATH에 넣는다).
- 개발 메뉴 단추 없이 깨끗하게 띄우는 주소: `exp+orca-mobile://?__expo_url=http%3A%2F%2Flocalhost%3A8081&__expo_disable_fab=1&__expo_disable_onboarding=1`을 `xcrun simctl openurl`로 연다.
- 시뮬레이터 언어는 `xcrun simctl spawn <udid> defaults write "Apple Global Domain" AppleLanguages -array ko-KR en-US`와 `AppleLocale ko_KR` 뒤 재부팅. 앱은 기기 언어를 따른다.

### 모의 서버와 페어링

- 데스크톱 Orca가 켜져 있으면 6768 포트를 쓰므로 모의 서버는 `PORT=6790`처럼 다른 포트로 띄운다. `MOCK_SERVER_KEY_FILE`로 서버 키를 고정하면 서버를 다시 띄워도 다시 페어링하지 않는다.
- 페어링 링크: `orca://pair?code=<base64url(JSON)>`. JSON은 `{"v":2,"endpoint":"ws://localhost:6790","deviceToken":"mock-device-token","publicKeyB64":"<모의 서버가 출력한 키>"}`(`src/shared/remote-runtime/pairing/mobile-relay-pairing-offer.ts`). 링크를 열고 "페어링"을 누른다.
- 페어링된 호스트의 id는 앱 데이터 폴더의 `RCTAsyncLocalStorage_V1`에서 `orca:hosts` 값으로 읽는다(`xcrun simctl get_app_container <udid> com.stably.orca.mobile data`). 이 id로 `orca://h/<hostId>/session/<worktreeId>` 링크를 열면 바로 그 화면으로 간다.

### 막히는 곳 다섯 가지

1. **"이 호스트의 Orca 업데이트" 화면**: 상태 응답의 capabilities에 `mobileWeb.bundle.v1`이 없어서 웹 번들 검사(`evaluateMobileWebBundleCompat`)가 막는다. 휴대폰 폭이면 모든 호스트 화면이, iPad에서도 세션 화면이 여기에 걸린다. 해결: 루트에서 `pnpm build:mobile-web`(결과 `out/mobile-web/`), 모의 서버에서 데스크톱의 `MOBILE_WEB_BUNDLE_METHODS`(`src/main/runtime/rpc/methods/mobile-delivery/mobile-web-bundle.ts`)를 불러 `mobileWeb.bundle.*` 요청에 `handler(params, { connectionId })`로 답한다. 번들 위치는 `setAppEnvironment`(`src/shared/app-lifecycle/app-environment.ts`)의 `getAppPath`를 저장소 루트로 주면 찾는다. 이 모듈들은 Electron 없이 tsx로 불러진다.
2. **봇 목록이 "Update Orca on Host 1 to see its bots."**: capabilities에 `src/shared/bot-runtime-capabilities.ts`의 값(`bots.mobile.v1` 등)을 넣고 `bots.list`에 봇 하나를 돌려준다. 봇은 `id, name, folder, folderWorkspaceId, avatar, paused, agent, notifications, members`와 `slack, linear, github: null`이면 스키마를 통과한다.
3. **카드 대신 대체 문구가 나온다**: 카드를 싣는 줄은 role `system`이고, 텍스트 블록에 `presentation`(`google-setup`, `composio-setup`, `bot-import-plan`, `bot-import-done`)이 있어야 알림 줄로 읽힌다(`nativeChatNoticeBlock`).
4. **Google, Composio 카드만 대체 문구다**: 이 카드들은 휴대폰이 아는 봇에게만 그려진다(`useHostBotById`). 세션을 봇의 홈으로 열어야 봇 저장소가 붙는다: worktreeId를 `folder:<folderWorkspaceId>`로 해서 `orca://h/<hostId>/session/folder%3A<id>`.
5. **카드가 데스크톱에 묻는 RPC**: `bots.googleSetup.read`(`{ run, blocker }`, run은 `src/shared/google/google-setup-run.ts`의 스키마), `google.status`, `bots.composio.status`, `bots.composio.connections`, `bots.composio.apps`, `bots.composio.eventsStatus`, `webhooks.card`. 빠지면 카드에 "Unknown method" 줄이 뜬다.

### 조작과 캡처

- Maestro로 누르고 넘긴다: `maestro --device <udid> test flow.yaml`, 화면 요소는 `maestro hierarchy`. 버튼은 보이는 글자가 아니라 접근성 이름으로 찾는다(예: "채팅 UI 사용" 버튼의 이름은 "채팅 UI에서 세션 열기").
- 캡처는 `xcrun simctl io <udid> screenshot`. 이전과 이후를 비교하려면 `ko.json`을 바꾸고 `pnpm build:mobile-web`을 다시 한 뒤 모의 서버를 다시 띄운다. 번들의 언어 카탈로그는 manifest 해시로 검증되므로 파일만 바꾸면 안 된다.
- 시뮬레이터에 iOS의 "Apple 계정 확인" 창이 뜨면 계정 이메일이 보인다. 그 캡처는 쓰지 말고 지운다.
- 11-26 `main`에서 `pnpm build:mobile-web`의 크기 예산 검사(eager budget)가 1,364바이트 초과로 실패했다. 번들 자체는 만들어지고, 언어 카탈로그는 그 예산 밖이다.

## 기록

### 2026-11-26: 휴대폰 봇 카드 번역을 시뮬레이터에서 확인

- 맥락: Orca 휴대폰 봇 카드의 한국어 205개를 PR로 내기 전에 실제 앱에서 보려고 했다. 저장소 모의 서버로는 채팅 화면이 막혀, 위 다섯 가지를 차례로 풀었다.
- 배운 것: 휴대폰의 호스트 화면은 데스크톱이 보내는 웹 번들이라, 휴대폰 문구를 바꾸는 PR은 그 번들을 다시 빌드해야 화면에서 확인된다. 모의 서버에 번들 전송만 붙이면 나머지는 모의 RPC로 채울 수 있다.
- 결과: 같은 채팅을 `main` 카탈로그와 브랜치 카탈로그로 띄워 카드가 영어에서 한국어로 바뀌는 비교 캡처 세 장을 얻었다. PR은 같은 키를 다른 기여자가 먼저 올려서 내지 않았다.
- 근거: `mobile/README.md`의 Mock Server 절, `mobile/src/transport/web-bundle/mobile-web-bundle-compat.ts`, `src/main/runtime/rpc/methods/mobile-delivery/`.
