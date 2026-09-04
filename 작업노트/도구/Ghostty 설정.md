---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-30
updated: 2026-09-05
projects: []
---

# Ghostty 설정

Ghostty 커스텀은 텍스트 config 하나로 하는데, **macOS에선 config 파일이 두 군데고, cmux가 깔린 머신에선 `ghostty` CLI와 `GHOSTTY_RESOURCES_DIR`가 cmux 것이라 검증이 엉뚱한 곳을 본다.**

## 핵심 정리

- **macOS Ghostty의 config 경로는 두 개다**: XDG `(로컬 경로)`와 `(로컬 경로)`. 둘 다 로드된다. 앱에서 `Cmd+,`(Open Config)로 열면 **Application Support 쪽**이다 — 홍이 붙여넣은 곳도 여기였다. 한 군데만 쓰는 게 안전한데, **통일은 XDG 쪽으로 해야 한다: cmux는 `(로컬 경로)`만 읽는다**(`cmux config paths` 출력에 명시, 08-31 확인 — 처음엔 App Support로 통일했다가 cmux에 테마가 안 먹어서 뒤집음).
- **config는 `key = value`만 허용한다.** 안내문의 셸 명령어(`ghostty +list-themes`)를 그대로 붙여넣으면 `config.ghostty:4:ghostty +list-themes: unknown field`. 에러 포맷은 `<파일>:<줄>:<내용>: unknown field`라 위치는 바로 찾힌다.
- **테마 이름은 대문자+공백이다** (Ghostty 1.3 기준): `Catppuccin Mocha`, `Catppuccin Latte`. 흔히 떠도는 `catppuccin-mocha`(소문자-하이픈)는 `theme "catppuccin-mocha" not found`. 정확한 이름은 `ghostty +list-themes`로 확인.
- **cmux가 깔려 있으면 PATH의 `ghostty`는 cmux 번들 바이너리다** (`/Applications/cmux.app/Contents/Resources/bin/ghostty`, `app runtime: .none`). 게다가 cmux 터미널 세션엔 `GHOSTTY_RESOURCES_DIR=/Applications/cmux.app/Contents/Resources/ghostty`가 박혀 있어서, **진짜 `/Applications/Ghostty.app/Contents/MacOS/ghostty`를 직접 불러도 테마를 cmux 리소스 폴더에서 찾는다.** 검증은 `env -u GHOSTTY_RESOURCES_DIR /Applications/Ghostty.app/Contents/MacOS/ghostty +validate-config`.
- 셰이더(`custom-shader`)는 파일이 실제로 있어야 한다 — 받기 전에 줄부터 넣으면 에러.
- 설정 리로드는 `Cmd+Shift+,`.
- **cmux 터미널도 사용자의 Ghostty config를 그대로 읽는다.** 바이너리에 `CmuxGhosttyConfigPathResolver`·`GhosttyConfigFileReading`이 있고, cmux의 `reload_config` 액션 설명이 "Reload Ghostty config, cmux settings, and refresh terminals"다. 테마·폰트·패딩·커서는 cmux 패널에도 적용되지만 `macos-titlebar-style`·`macos-icon`·`background-blur` 같은 앱 수준 옵션은 Ghostty.app 전용. cmux 자체 설정은 `(로컬 경로)`(JSONC, 전부 주석 처리된 템플릿 — 주석 해제하면 파일 관리로 전환), 사이드바를 터미널 배경색에 맞추는 키는 `sidebarAppearance.matchTerminalBackground: true`.
- macOS 앱 아이콘은 `macos-icon`(독·앱 전환기만, Finder는 번들 고정): 공식 변형 8종(`blueprint`·`chalkboard`·`microchip`·`glass`·`holographic`·`paper`·`retro`·`xray`), `custom`(+`macos-custom-icon` 이미지 경로), `custom-style`(+`macos-icon-frame`/`-ghost-color`/`-screen-color`, 실험적). 반영은 앱 재시작. 탭 세로 배치는 macOS에선 불가 — 네이티브 탭이라 위치 옵션이 없다(`gtk-tabs-location`은 Linux 전용, top/bottom만).

## 기록

### 2026-08-30 — 첫 커스텀 config 작성: unknown field · 테마명 · cmux 리소스 하이재킹 3연타

- 맥락: 홍의 로컬 머신 Ghostty.app(1.3.2 tip) 첫 커스텀. 안내 텍스트를 통째로 `Cmd+,` config에 붙여넣어 에러.
- 배운 것: 위 핵심 정리 전부 이 세션에서 재현. 최종 config는 Application Support 쪽 하나로 통일(XDG 쪽은 삭제), `theme = light:Catppuccin Latte,dark:Catppuccin Mocha` + JetBrainsMono Nerd Font + opacity/blur + titlebar tabs로 `+validate-config` 통과.
- 관련: [[작업노트/도구/셸 초기화와 터미널 통합|셸 초기화와 터미널 통합]] — cmux가 터미널 환경에 끼어드는 또 다른 사례(셸 함수 덮어쓰기).

### 2026-08-31 — 아이콘·탭 위치·cmux 연동 확인

- 맥락: 홍의 후속 질문 3건(로고 변경 / 탭 세로 배치 / cmux를 Ghostty 디자인처럼).
- 배운 것: 핵심 정리의 cmux config 공유·`macos-icon`·탭 항목 전부 이 세션에서 실기기 확인(strings 분석 + `+validate-config`). 홍의 최종 선택: `macos-icon = xray`, cmux는 `matchTerminalBackground: true`.
- **정정**: cmux는 XDG 경로만 읽으므로 config를 App Support → `(로컬 경로)`로 이동. `cmux reload-config` CLI가 Ghostty config + cmux.json을 앱 재시작 없이 둘 다 리로드한다(`OK Reloaded config`). cmux 편에서 편집 전 백업 권고: cmux.json은 `.bak` 사본을 만들라고 자체 문서(`cmux docs settings`)가 안내.
- 갱신: `updated: 2026-08-31`.

### 2026-09-05 — 홍은 Ghostty를 직접 쓴다 (cmux 아님) — 기본 키바인드는 바이너리에서 뽑는다

- 맥락: 홍이 "ghostty 단축키 알려줘"라고 물어 기본 키바인드를 정리하며, 08-27~31 기록의 cmux 언급을 근거로 "cmux가 키를 먼저 잡을 수 있다"고 덧붙였다가 홍이 "아니야 고스티 쓰고 있어"라고 정정.
- 배운 것: **2026-09-05 기준 홍의 터미널은 Ghostty.app 1.3.1 단독**이다. cmux 관련 기록(테마 리로드 `cmux reload-config`, XDG config만 읽음)은 그 시점 사실이고, 현재 환경을 cmux로 가정하지 않는다. `(로컬 경로)`에 `keybind` 항목은 없어 macOS 기본값이 그대로다.
- 근거: 기본 키바인드 전체는 `/Applications/Ghostty.app/Contents/MacOS/ghostty +list-keybinds --default`로 뽑는다(설치본 기준, 문서보다 정확). 자주 쓰는 것: ⌘D/⌘⇧D 분할, ⌘⌥화살표 스플릿 이동, ⌘⇧Enter 줌, ⌘↑/↓ 프롬프트 점프, ⌘⇧P 커맨드 팔레트, ⌘⇧, 리로드.
