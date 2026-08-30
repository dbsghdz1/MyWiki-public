---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-30
updated: 2026-08-30
projects: []
---

# Ghostty 설정

Ghostty 커스텀은 텍스트 config 하나로 하는데, **macOS에선 config 파일이 두 군데고, cmux가 깔린 머신에선 `ghostty` CLI와 `GHOSTTY_RESOURCES_DIR`가 cmux 것이라 검증이 엉뚱한 곳을 본다.**

## 핵심 정리

- **macOS Ghostty의 config 경로는 두 개다**: XDG `(로컬 경로)`와 `(로컬 경로)`. 둘 다 로드된다. 앱에서 `Cmd+,`(Open Config)로 열면 **Application Support 쪽**이다 — 홍이 붙여넣은 곳도 여기였다. 한 군데만 쓰는 게 안전하다.
- **config는 `key = value`만 허용한다.** 안내문의 셸 명령어(`ghostty +list-themes`)를 그대로 붙여넣으면 `config.ghostty:4:ghostty +list-themes: unknown field`. 에러 포맷은 `<파일>:<줄>:<내용>: unknown field`라 위치는 바로 찾힌다.
- **테마 이름은 대문자+공백이다** (Ghostty 1.3 기준): `Catppuccin Mocha`, `Catppuccin Latte`. 흔히 떠도는 `catppuccin-mocha`(소문자-하이픈)는 `theme "catppuccin-mocha" not found`. 정확한 이름은 `ghostty +list-themes`로 확인.
- **cmux가 깔려 있으면 PATH의 `ghostty`는 cmux 번들 바이너리다** (`/Applications/cmux.app/Contents/Resources/bin/ghostty`, `app runtime: .none`). 게다가 cmux 터미널 세션엔 `GHOSTTY_RESOURCES_DIR=/Applications/cmux.app/Contents/Resources/ghostty`가 박혀 있어서, **진짜 `/Applications/Ghostty.app/Contents/MacOS/ghostty`를 직접 불러도 테마를 cmux 리소스 폴더에서 찾는다.** 검증은 `env -u GHOSTTY_RESOURCES_DIR /Applications/Ghostty.app/Contents/MacOS/ghostty +validate-config`.
- 셰이더(`custom-shader`)는 파일이 실제로 있어야 한다 — 받기 전에 줄부터 넣으면 에러.
- 설정 리로드는 `Cmd+Shift+,`.

## 기록

### 2026-08-30 — 첫 커스텀 config 작성: unknown field · 테마명 · cmux 리소스 하이재킹 3연타

- 맥락: 홍의 로컬 머신 Ghostty.app(1.3.2 tip) 첫 커스텀. 안내 텍스트를 통째로 `Cmd+,` config에 붙여넣어 에러.
- 배운 것: 위 핵심 정리 전부 이 세션에서 재현. 최종 config는 Application Support 쪽 하나로 통일(XDG 쪽은 삭제), `theme = light:Catppuccin Latte,dark:Catppuccin Mocha` + JetBrainsMono Nerd Font + opacity/blur + titlebar tabs로 `+validate-config` 통과.
- 관련: [[작업노트/도구/셸 초기화와 터미널 통합|셸 초기화와 터미널 통합]] — cmux가 터미널 환경에 끼어드는 또 다른 사례(셸 함수 덮어쓰기).
