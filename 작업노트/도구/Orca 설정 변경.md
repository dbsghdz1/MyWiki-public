---
title: "Orca 설정 변경"
summary: "앱이 로컬 설정을 덮어쓰는 시점과 재실행 후 설정 적용을 확인하는 방법"
type: study
area: 도구
audience: ai
status: active
created: 2026-09-22
updated: 2026-11-24
projects:
  - "Ghostty 설정"
---

# Orca 설정 변경

**Orca(stably.ai, Electron) 설정은 `orca-data.json`의 `settings`에 있지만, 앱이 켜진 채로 파일을 고치면 메인 프로세스가 메모리 값으로 덮어쓴다.** Orca 안에서 연 Claude 세션은 앱을 재시작할 수 없으므로 `orca computer` CLI로 Settings 화면을 직접 조작한다.

## 핵심 정리

- **설정 파일**: `(로컬 경로)`. `settings.*`(앱 설정 192개: `theme`·`appIcon`·`terminalFontSize`·`terminalFontFamily`·`terminalThemeDark`·`workspaceDir`…)과 `ui.*`(`uiZoomLevel`·`editorFontZoomLevel`). 백업 `.bak.0~4`가 옆에 있다. 메인 프로세스는 시작 때 한 번 읽고(`out/main/index.js`) 이후는 메모리 → 저장이라 **실행 중 파일 편집은 무의미**.
- **`orca` CLI(`/opt/homebrew/bin/orca`)에는 settings 명령이 없다.** `orca agent-context`에도 없음. 대신 `orca computer *`(macOS 접근성 기반 computer-use, 권한은 `orca computer permissions`)로 UI를 누른다.
- **Claude가 Orca 안에서 도는지 판별**: `TERM_PROGRAM=Orca`, `ORCA_WORKTREE_ID`·`ORCA_TAB_ID` 환경변수. 이 경우 Orca를 끄면 세션이 죽는다.
- `orca computer` 지뢰:
  - **React 버튼은 `--element-index`(AXPress)로 눌러도 `ok: true`만 돌려주고 아무 일도 안 일어난다**(사이드바 탭·UI Zoom ±·아이콘 ‹›). **`--x --y` 좌표 클릭**은 먹는다. 좌표는 window 기준 pt이고 스크린샷은 2배 px — 1280폭으로 줄여 본 좌표 ×2.
  - `set-value`는 stepper에 먹는다(터미널 Font Size 14→16). 응답의 `verification.state: unverified, value_mismatch`는 무시해도 되고 파일 값으로 확인한다.
  - `scroll --x --y --pages N`은 `ok`지만 설정 페이지가 안 움직였다. 대신 **섹션 헤더(Interface·Terminal)를 클릭해 접으면** 아래 항목(앱 아이콘 캐러셀)이 보인다.
  - 상호작용마다 element index가 바뀐다(`element 198 is stale`). 매번 `get-app-state` 다시.
  - 키 입력(`hotkey`)은 `--restore-window` 없이는 `window_not_focused`.
- **UI Zoom**: `ui.uiZoomLevel`은 Electron zoom level(0.5 단위 = 10%). `Cmd+=`/`Cmd+-`는 터미널 pane 밖에서만. 홍 판정: 120%는 "너무 커졌어", **110%가 적당**. 터미널 폰트는 16px.
- **앱 아이콘 후보는 3개뿐**: `classic`·`watercolor`·`blue`(`out/shared/app-icon.js`). 캐러셀 › 순서도 classic→watercolor→blue.
- 셸 지뢰: zsh는 `set -- $xy`로 단어 분리를 안 한다 — 좌표 두 개를 한 변수에 넣고 `$1 $2`로 꺼내면 `Invalid numeric value for --x`.

- **개발 빌드를 격리해 띄우기**: 쓰고 있는 Orca를 건드리지 않고 Orca 저장소의 e2e 시험 틀로 임시 HOME과 userData를 쓰는 앱을 창 없이 띄울 수 있다. 컴포넌트 조각(`pnpm fixtures:shot`)과 실제 앱이 다르게 나온 적이 있어서, 버그로 올리기 전에는 이렇게 확인한다. 방법은 아래 2026-11-24 기록.
## 기록

### 2026-09-22 — 앱 아이콘·폰트 키우기
- 맥락: 홍이 Orca 안에서 연 MyWiki 세션에서 "앱 아이콘 바꾸고 폰트 전체적으로 2pt 키워줘". 관련: [Ghostty 설정](Ghostty%20%EC%84%A4%EC%A0%95.md)(터미널 테마 `Ghostty Default Style Dark`를 Orca가 그대로 씀).
- 배운 것: 위 핵심 정리 전부. 결과 `appIcon: blue`, `terminalFontSize: 16`, `uiZoomLevel: 0.5`(110%).
- 근거: `orca-data.json` grep으로 매 단계 확인. Orca 1.4.205, macOS 26.5.

### 2026-11-24: 격리한 개발 빌드로 화면 재현
- 맥락: Orca 기여 중 「화면 언어를 바꾸면 일부 문구가 예전 언어로 남는다」는 가설을 컴포넌트 조각에서 세웠고, 실제 앱에서 확인해야 했다.
- 배운 것:
  - 빌드: Orca 저장소에서 `pnpm exec electron-vite build --mode e2e`(10초쯤). 일반 빌드에는 시험용 `window.__store`가 없다.
  - 실행: 일회용 spec을 `tests/e2e/`에 두고 `ORCA_BACKGROUND_LAUNCH=1 SKIP_BUILD=1 pnpm exec playwright test --config tests/playwright.config.ts --project=electron-headless <spec>`(20초쯤). `tests/e2e/desktop/electron-fixtures/orca-app`의 `test`를 쓰면 `orcaPage`가 격리된 앱 창이다.
  - 조작: `window.__store!.getState()`의 `updateSettings({ uiLanguage: 'ko' })`, `openSettingsPage()`, `closeSettingsPage()`, `openAutomationsPage()`로 상태를 만들고, 판단은 화면 글자(DOM)로 한다. `orcaAppExtraEnv`로 PATH 같은 환경 변수를 넘길 수 있고 HOME은 못 바꾼다.
  - 지뢰: pnpm을 corepack으로만 쓰면 global-setup의 `execSync('pnpm run build:cli')`가 `/bin/sh: pnpm: command not found`로 멈춘다. `exec corepack pnpm "$@"` 한 줄짜리 `pnpm` 스크립트를 PATH 앞에 두면 된다.
  - 뒷정리: spec은 지우고 `git status`가 깨끗한지 본다. 사진이 쌓이는 `test-results/`는 git이 무시한다.
  - 설정 칸 열기: `openSettingsPage()` 뒤 `openSettingsTarget({ pane: 'integrations', repoId: null })`처럼 칸을 고르면 그 칸이 열린다. 저장소 설정은 `pane: 'repo'`와 그 저장소 id다.
  - 사진에서 일부 가리기: Playwright `screenshot({ mask })`는 줄바꿈된 인라인 요소면 여러 줄을 감싼 사각형 전체를 덮어 같은 줄의 다른 글자까지 가린다. `getClientRects()`로 줄마다 덮개를 얹고 찍은 뒤 치우면 된다.
- 결과: 설정 화면에서 언어를 영어에서 한국어로 바꾸자 전역 `translate()`로 만든 필터 이름까지 바로 바뀌었다. 조각에서 영어로 남은 것은 조각 환경 탓이었다. 대신 그 화면 사진에서 번역 조각을 이어 붙인 문장이 한국어로 깨진 것을 찾았다.
- 근거: Orca `main` `231551f9f6`, macOS. 시험 spec과 출력은 비공개 기여 기록에 둠.
