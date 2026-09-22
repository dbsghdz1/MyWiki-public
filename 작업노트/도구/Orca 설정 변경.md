---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-22
updated: 2026-09-22
projects:
  - "[[작업노트/도구/Ghostty 설정|Ghostty 설정]]"
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

## 기록

### 2026-09-22 — 앱 아이콘·폰트 키우기
- 맥락: 홍이 Orca 안에서 연 MyWiki 세션에서 "앱 아이콘 바꾸고 폰트 전체적으로 2pt 키워줘". 관련: [[작업노트/도구/Ghostty 설정|Ghostty 설정]](터미널 테마 `Ghostty Default Style Dark`를 Orca가 그대로 씀).
- 배운 것: 위 핵심 정리 전부. 결과 `appIcon: blue`, `terminalFontSize: 16`, `uiZoomLevel: 0.5`(110%).
- 근거: `orca-data.json` grep으로 매 단계 확인. Orca 1.4.205, macOS 26.5.
