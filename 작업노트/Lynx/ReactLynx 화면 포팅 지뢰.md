---
type: study
area: Lynx
audience: ai
status: active
created: 2026-10-07
updated: 2026-10-17
projects:
  - "소프트웨어마에스트로"
---

# ReactLynx 화면 포팅 지뢰

웹 CSS와 React 문법이 맞아도 Lynx가 받는지는 따로다. 가장 비싼 것은 에러 없이 무시되는 것들이다. 2026-10-17 기준 결론.

## 핵심 정리

- **조용히 무시되는 CSS**: `env()`에 대비값 인자를 주면 선언 전체가 버려진다. view의 기본 `display`는 `linear`라 `flex-direction`과 자식의 `flex: 1`이 무시된다. `font` 축약형은 지원 목록에 없다. 색은 상속되지 않아 `<text>`마다 준다.
- **요소 규칙**: 글자는 `<text>` 안에만 둔다. `<scroll-view>`는 높이가 있어야 스크롤된다. SVG 파일은 `<svg src>`로, PNG와 JPG만 `<image>`로 그린다.
- **실행 규칙**: `bindtap`에는 함수를 넘긴다. `bindtap={navigate(...)}`처럼 호출 결과를 넘기면 렌더 중에 실행된다. 값을 `import type`으로 가져오면 실행 때 사라진다. ReactLynx에는 `Link`가 없어 `bindtap`에서 `useNavigate()`로 옮긴다.
- **타입**: `TS2322: Property 'children' does not exist on type 'TextProps'`가 전 파일에 뜨면 `src/index.tsx`가 비었는지 본다.
- **개발 연결**: 화면이 안 바뀌면 코드보다 HMR 연결을 먼저 본다(`lsof`의 LynxExplorer 연결, 맥 IP, 렌더 중 예외). 번들 주소는 소문자 `http://`로 넣는다. 터미널 앱이 Desktop 권한을 잃어 `Operation not permitted`가 나면 앱을 완전히 종료한다.
- **테스트**: `@lynx-js/react/testing-library`에서 `toBeInTheDocument`가 `Invalid Chai property`로 실패하면 `import '@testing-library/jest-dom';`이 빠진 것이다.

2026-10-17에 [미분류](../%EB%AF%B8%EB%B6%84%EB%A5%98.md)에서 두 항목을 원문 그대로 옮겼다. Lynx 기록이 셋째가 되어 분야를 만들었다.

## 기록

### 2026-10-07 — ReactLynx 화면 포팅에서 밟은 지뢰

- 맥락: 소프트웨어마에스트로에서 Flutter 화면을 ReactLynx(`@lynx-js/react` 0.126, Rspeedy, react-router 6.30)로 옮기다가. 기록: Lynx 포팅 기록 2026-10-07
- 배운 것:
  - **`env(safe-area-inset-top, 48px)`처럼 env()에 둘째(대비값) 인자를 주면 Lynx는 선언 전체를 버린다** — 에러 없이 `margin-top`이 0이 되어 상단 바가 다이내믹 아일랜드 뒤로 숨는다. 대비값은 같은 속성을 두 줄로(`margin-top: 48px;` 다음 `margin-top: env(safe-area-inset-top);`) 쓰고, Biome `lint/suspicious/noDuplicateProperties`는 `biome-ignore` 주석으로 이유를 남긴다. 웹 CSS 문법이 맞아도 Lynx가 받는지는 실기기로 본다
  - **`<scroll-view>`는 높이가 있어야 스크롤된다** — 세로 flex 부모 안에서 `flex: 1`. 글자는 반드시 `<text>` 안에 둔다. 색은 기본으로 상속되지 않아 `<text>`마다 준다
  - **`TS2322: Property 'children' does not exist on type 'TextProps'`가 갑자기 전 파일에 뜨면 `src/index.tsx`가 비었는지 본다** — 엔트리가 `@lynx-js/react`를 import하며 JSX 타입 보강을 싣는다. 0바이트가 되자 이 에러가 났다
  - **Lynx Explorer에 번들 주소를 `Http://`(대문자)로 넣으면 빈 화면이다** — 에러 표시도 없다. 소문자로
  - **값을 `import type`으로 가져오면 실행 때 사라진다** — 경로 상수 객체를 `import type { routes }`로 가져와 테스트가 `routes.support`에서 깨졌다. Biome는 원인 대신 `noUnusedImports`(타입 자리에서 안 쓰임)만 알린다
  - ReactLynx엔 `Link`가 없다 — `bindtap`에서 `useNavigate()`로 이동, 시작 경로는 `MemoryRouter initialEntries`. `useNavigate`는 라우터 밖에서 던지므로 화면 단위 테스트도 `MemoryRouter`로 감싼다
  - 10-07 기준 Lynx에서 외부 URL·메일 앱을 여는 공개 API를 찾지 못했다(SSH-579 조사 대기, 미확인)
- 근거: `ASMSSH/BGN` 커밋 `bb39db8`(env() 한 줄로 합침) → `0a89316`(두 줄로 되돌림) · `298a2f3`(routes 상수) · PR #1~#4 · Lynx Explorer iOS 시뮬레이터 재현

### 2026-10-16: ReactLynx 로그인 화면에서 밟은 지뢰

- 맥락: 소프트웨어마에스트로의 Lynx 포팅에서 로그인 화면(SSH-592, `ASMSSH/BGN` PR #21)을 옮기다가. 에이전트가 요구사항을 틀리게 줘서 홍이 레이아웃이 아닌 곳에서 시간을 쓴 것들이다
- 배운 것:
  - **Lynx view의 기본 `display`는 flex가 아니라 `linear`다.** linear는 `flex-direction`과 자식의 `flex: 1`을 에러 없이 무시한다. 증상: 버튼이 바닥으로 안 가고, `flex-direction: row`를 줘도 아이콘과 글자가 세로로 쌓인다. 자식을 배치하는 상자에는 항상 `display: flex`를 먼저 쓴다. 저장소의 `.Screen`, TopBar, 온보딩 CSS가 전부 그렇다. `align-items`와 `justify-content`는 linear에서도 먹어서 "flex 없이도 된다"고 오판하기 쉽다. 근거: [Lynx display](https://lynxjs.org/api/css/properties/display.html)의 초기값 `linear`
  - **`display: flex`는 자식까지만 적용된다.** 손주는 그 자식의 display를 따른다. 상속되지 않는다
  - **`<image>`는 SVG를 그리지 못한다.** SVG 파일은 `<svg src={logo} />`, PNG와 JPG만 `<image>`. 증상: 로고 자리가 빈다
  - **CSS 축약형 `font`는 Lynx 지원 속성 목록에 없다.** `font: 12px`는 무시된다. `font-size`를 쓴다
  - **`bindtap={navigate(routes.terms)}`는 렌더 중에 이동을 실행한다.** 이후 화면이 CSS 없이 깨진 채로 멈추고 HMR도 더 오지 않았다. `bindtap={() => navigate(...)}`로 감싸고 Explorer에서 URL을 다시 넣어야 복구됐다
  - **"저장했는데 화면이 안 바뀐다"는 코드보다 연결을 먼저 본다.** `lsof -nP -iTCP:3000 | grep ESTABLISHED`에 LynxExplorer가 없으면 HMR이 죽은 것이다. 원인은 셋이었다: dev 서버가 죽음(호스트 앱 재시작), 맥 IP 변경(`ipconfig getifaddr en0`, 번들 주소의 IP도 바꿔야 한다), 위의 렌더 중 예외. Explorer 로그에서 마지막 `lynx.hmr.css`나 `hot-update.json` 시각을 보면 언제 끊겼는지 알 수 있다. 에이전트가 이걸 안 보고 "원래 HMR이 안 된다"고 추측했다가 틀렸다
  - **호스트 터미널 앱이 Desktop 권한(TCC)을 잃으면 `Operation not permitted`가 난다.** 설정에서 전체 디스크 접근이 켜져 있어도, 앱을 Cmd+Q로 완전히 종료해야 권한이 다시 읽힌다. 창만 닫으면 Helper 프로세스가 남아 소용없다
  - 테스트(`@lynx-js/react/testing-library`)에서 `toBeInTheDocument`가 "Invalid Chai property"로 실패하면 `import '@testing-library/jest-dom';` 줄이 빠진 것이다. 같은 글자가 두 번 그려지면 "Found multiple elements"가 먼저 나서 이 원인이 가려진다
- 근거: `ASMSSH/BGN` PR #21(커밋 `43d8044`, `92bd3cd`, `07ea887`, `380e99e`, 머지 `8c78bf6`), Lynx Explorer iOS 시뮬레이터 재현, Explorer 로그(`xcrun simctl spawn booted log show --predicate 'process == "LynxExplorer"'`)
- 애플 로그인 버튼은 Lynx에 공식 버튼이 없어 이번에 넣지 않았다. RN 패키지도 네이티브 브릿지 방식이 달라 못 쓴다. 애플 로그인 티켓에서 iOS 네이티브 엘리먼트로 감싼다(SSH-592 Jira 댓글)
