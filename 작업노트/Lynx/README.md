---
type: hub
title: "Lynx 작업노트"
summary: "ReactLynx 화면 포팅, 스레드 모델, SEED 구성요소와 빌드 도구"
status: active
created: 2026-11-19
updated: 2026-11-19
---

# Lynx 작업노트

[전체 분야](../README.md#주제-목록)

- [ReactLynx 화면 포팅 지뢰](ReactLynx%20%ED%99%94%EB%A9%B4%20%ED%8F%AC%ED%8C%85%20%EC%A7%80%EB%A2%B0.md) — 웹 CSS가 맞아도 Lynx가 받는지는 따로다. **`env()` 대비값은 선언째 버려지고, view 기본 `display`는 `linear`라 flex 속성이 무시된다**, `<image>`는 SVG를 못 그린다, 화면이 안 바뀌면 HMR 연결부터 본다. **Explorer는 `SIMCTL_CHILD_lynx_initial_url` 환경변수로 타이핑 없이 연다** (2026-10-17)
- [SEED Lynx 설치와 BGN 툴체인](SEED%20Lynx%20%EC%84%A4%EC%B9%98%EC%99%80%20BGN%20%ED%88%B4%EC%B2%B4%EC%9D%B8.md) — `@seed-design/lynx-react` 1.0.0은 rspeedy 0.18, Rsbuild 2.2.9에서 **Tailwind 없이 base.css 한 줄로** 빌드, 테스트, 화면까지 통과. 버튼 하나에 번들 +212 KB, 권장 플래그는 기존 화면 픽셀 불변, `preact-devtools` import는 메인 스레드 호출 폭주 때문에 뺀다 (2026-10-17)
- [ReactLynx 두 스레드 내부 구조](ReactLynx%20%EB%91%90%20%EC%8A%A4%EB%A0%88%EB%93%9C%20%EB%82%B4%EB%B6%80%20%EA%B5%AC%EC%A1%B0.md) — 같은 JSX를 메인(LEPUS)과 백그라운드(JS)로 두 번 컴파일, `rLynxFirstScreen`으로 hydrate 뒤 `rLynxChange` patch. **`BackgroundSnapshot not found`, `ctx not found`, `Cannot update state in main thread!`의 트리거**, MTS, 테스트 라이브러리 구조 (2026-10-17)
- [SEED Lynx 패키지 지도](SEED%20Lynx%20%ED%8C%A8%ED%82%A4%EC%A7%80%20%EC%A7%80%EB%8F%84.md) — lynx-react, lynx-css, headless 35개. 패키지 export와 CLI 스니펫 `ui:*` 구분, ActionButton 전 계층, **런타임 불변식과 에러 문자열**, 아이콘 플러그인, 문서 빈틈(PR 재료) (2026-10-17)
