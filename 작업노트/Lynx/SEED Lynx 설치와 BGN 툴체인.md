---
type: study
area: Lynx
audience: ai
status: active
created: 2026-10-17
updated: 2026-10-17
projects:
  - "소프트웨어마에스트로"
---

# SEED Lynx 설치와 BGN 툴체인

`@seed-design/lynx-react` 1.0.0은 보험찾개냥 Lynx 앱(BGN) 툴체인에서 Tailwind 없이 `base.css` 한 줄로 빌드, 타입 검사, rstest까지 통과한다. 비용은 버튼 하나에 번들 약 212 KB이고, 권장 플래그는 앱 전체의 CSS 상속을 켠다. 화면 렌더 확인은 따로 한다. 2026-10-17 기준.

## 핵심 정리

- **툴체인 조합**: BGN은 `@lynx-js/rspeedy` 0.18.0(Rsbuild 2.2.9, Rspack 2.2.8), `@lynx-js/react` 0.126.2, `@lynx-js/react-rsbuild-plugin` 0.20.3, `@lynx-js/types` 4.3.0, `@rstest/core` 0.12.3. SEED 레포가 검증한 조합(rspeedy 0.16.4, plugin 0.18.3, React 0.123.3)보다 새 조합인데 문제가 없었다. SEED peer는 `@lynx-js/react >=0.117.0`, `@lynx-js/types >=3.9.0`.
- **최소 배선 셋**: ① `npm install @seed-design/lynx-react @seed-design/lynx-css` ② `src/styles/global.css`에 `@import "@seed-design/lynx-css/base.css";`, `src/index.tsx`에서 App import보다 앞에 `import './styles/global.css';` ③ `import { ActionButton } from '@seed-design/lynx-react'`, 탭은 `bindtap`. `onClick`은 `Property 'onClick' does not exist on type ...ActionButtonProps` 타입 에러다.
- **스타일링 라이브러리는 SEED 컴포넌트 렌더에 필수가 아니다.** 설치 문서(`docs/content/lynx/getting-started/installation.mdx`)는 Vanilla Extract 또는 Tailwind v4 선택을 단계로 두지만, 그것은 앱 자신의 레이아웃용이다. recipe CSS는 `@seed-design/lynx-css/recipes/action-button.mjs` 첫 줄 `import './action-button.css';`로 컴포넌트 import 때 따라오고, 토큰 변수 정의(`--seed-color-bg-brand-solid:` 등)는 `base.css`에만 있다. **base.css 없이도 빌드는 통과하므로 빌드 성공을 화면 정상의 증거로 쓰면 안 된다.** Vanilla Extract 플러그인 0.2.0은 peer `@rsbuild/core ^1`이라 Rsbuild 2에서는 문서상 불가.
- **번들 비용**: 복사본 기준 196,579 B → 408,656 B. 분해는 `base.css`만 +30,852 B, ActionButton만(base.css 없이) +181,141 B. ActionButton이 `useScaleFeedback`(눌림 축소, main thread 애니메이션), `ProgressCircle`(loading), 접근성 유틸, Icon slot, headless `@seed-design/lynx-react-action-button`과 `use-press-tap`, `@lynx-js/lynx-ui-common`을 함께 끌고 오고, 같은 코드가 백그라운드 JS와 메인 스레드 바이트코드로 두 번 실린다(해석). `strings dist/main.lynx.bundle | grep -c seed-`가 0 → 1104. 쓰지 않는 컴포넌트의 root 클래스는 번들에 없으므로 CSS 트리셰이킹은 동작한다.
- **권장 플래그는 앱 전체에 걸린다**: `npm install -D @lynx-js/config-rsbuild-plugin`(0.2.5, rspeedy 0.18과 호환) 뒤 `pluginReactLynx({ engineVersion: '3.6' })`과 `pluginLynxConfig({ enableCSSInheritance: true, enableCSSInlineVariables: true, fontScaleEffectiveOnlyOnSp: true, enableFixedNew: true })`. 번들 설정 JSON에서 `"enableCSSInheritance":false`가 `true`로 바뀌고 `enableCSSInlineVariables`, `enableFixedNew`, `fontScaleEffectiveOnlyOnSp`가 새로 생긴다. `engineVersion: '3.6'`만으로는 번들 JSON과 크기에 보이는 변화가 없었다. **기존 화면은 변하지 않았다.** 고객센터, 로그인, 온보딩을 플래그 ON과 OFF로 전해상도 픽셀 비교해 차이 0(2026-10-17, iPhone 17 시뮬레이터). 상속되는 속성은 엔진 기준 13개(font-size, font-family, text-align, line-spacing, letter-spacing, line-height, font-style, font-weight, color, text-decoration, text-shadow, direction, cursor. 엔진 `dynamic_css_styles_manager.cc:30-38`)인데 BGN의 모든 `<text>`가 font-size와 color를 직접 선언하기 때문이다. 앞으로 font-size를 생략한 `<text>`는 14px이 아니라 base.css의 16px이 된다. jsdom 테스트는 이것을 못 잡으므로 화면으로 본다. 설정 확인 명령: `strings dist/main.lynx.bundle | grep -m1 lepusStrict`.
- **base.css의 전역 규칙 둘**: `:root { font-size: 16px; }`(rem 기준이 Lynx 기본 14px에서 16px로)와 `text { background-color: var(--seed-color-bg-transparent); }`. BGN CSS에 rem이 없어 지금은 영향 없음. plain CSS에서는 `t5-regular` 같은 타이포 클래스가 없고 `--seed-font-size-t1`~`t10`, `--seed-line-height-t5`, `--seed-font-weight-regular|medium|bold` 변수를 쓴다.
- **테스트**: 기존 `rstest.config.ts`(`withLynxConfig()`) 그대로 SEED 컴포넌트 렌더, `fireEvent.tap`, loading과 disabled 차단이 통과한다. `withLynxConfig()`는 `lynx.config.ts`의 플러그인 옵션을 읽으므로 `pluginLynxConfig` 추가가 테스트를 깨지 않는다. 함정 하나: `useSeedClassName({ colorMode: 'system' })`은 `TypeError: Cannot read properties of undefined (reading 'theme')`로 죽는다. 테스트 환경에 `lynx.__globalProps`가 없어서이고, 렌더 전에 `lynx.__globalProps = { theme: 'dark' }`를 심으면 통과한다. `getSeedClassName`은 옵셔널 체이닝이라 안전하다.
- **개발 모드 경고**: `warn: No need to use this API when 'globalPropsMode' is not 'event', updates will be triggered automatically by full re-render.`가 SEED 컴포넌트의 `useGlobalProps()` 호출마다 찍힌다(`@lynx-js/react/runtime/lib/core/globalProps.js`의 `warnGlobalPropsMode`, `__DEV__`에서만). `pluginReactLynx({ globalPropsMode: 'event' })`로 사라지고 빌드와 테스트는 그대로 통과한다. 기기 확인은 안 했다.
- **다크 모드**: `<page>`가 없는 BGN 구조(`root.render(<App />)`)에서는 `:root` 단독 선택자가 라이트 토큰만 적용한다. 다크는 루트 `<page className={...}>`에 `useSeedClassName`을 달아야 하고 host가 `theme` global prop을 줘야 한다. 미실험.
- **devtools 위험**: SEED 예제 앱 `examples/lynx-spa/src/index.tsx`는 `@lynx-js/preact-devtools` import를 `Temporarily disabled due to a getUniqueIdListBySnapshotId runtime error.`라는 주석과 함께 꺼 두었다. BGN `src/index.tsx`는 맨 위에서 import한다. **BGN 조합(snapshot 런타임, 개발 모드)에서는 그 TypeError가 나지 않는다**(시뮬레이터 로그 20초, 예외 0, 2026-10-17). 메인 스레드 함수는 `@lynx-js/react/runtime/src/snapshot/lynx/injectLepusMethods.ts:6-11`이 `lynx.ts:37-53`의 `__DEV__ || __REACT_DEVTOOL__` 조건에서 주입하고, TypeError가 나는 조합은 자체 메인 스레드 부트스트랩을 쓰는 Element Template 런타임(`experimental_useElementTemplate`, lynx-stack PR #3809와 issue #3394, 미해결)이다. 대신 `@lynx-js/preact-devtools/lib/adapter/shared/idMapper.js:19-36`의 `updateIdToUniqueIdRelation`이 커밋마다 컴포넌트 vnode 수만큼 `callLepusMethod("getUniqueIdListBySnapshotId")`를 메인으로 보낸다(ActionButton 하나 화면의 첫 렌더에 27회). DevTool이 켜져 있으면 5초에 800회를 넘길 때 엔진 경고 `CallLepusMethod called too frequently`가 난다. 당근 lynx-console PR #68(2026-09-08)이 같은 이유로 import를 뺐다. 완화는 `src/index.tsx:1`의 `import '@lynx-js/preact-devtools';` 제거(`@lynx-js/react/debug`는 별개라 유지), 또는 `lynx.config.ts`에 `resolve: { alias: { '@lynx-js/preact-devtools$': false } }`(개발 번들 약 169 KB 감소 확인). 프로덕션은 `react-alias-rsbuild-plugin`이 이미 `false`로 alias하므로 해당 없음. `REACT_DEVTOOL=1`을 프로덕션에 남기지 않는다.
- **필요 없던 것**: Tailwind, Vanilla Extract, `seed-design.json`과 CLI(`ActionButton`은 snippet이 아니라 패키지 export), tsconfig alias, 아이콘 플러그인(아이콘을 쓰기 전까지). Tailwind v4 경로도 되긴 한다: `tailwindcss@4`, `@tailwindcss/postcss@4`, `postcss@8`, `@seed-design/tailwind4-theme`와 루트 `postcss.config.js`(문서 그대로, `lynxTailwindCompatibility()` 포함)만으로 빌드와 테스트 통과, 번들 +1,928 B.
- **화면 확인**: 별도 iPhone 17 시뮬레이터에 Explorer를 설치해 SEED ActionButton 화면(주황 brandSolid, 흰 굵은 라벨, 오류 토스트 없음)과 기존 화면 셋을 촬영했다. Explorer를 타이핑 없이 여는 법과 픽셀 비교 방법은 [ReactLynx 화면 포팅 지뢰](ReactLynx%20%ED%99%94%EB%A9%B4%20%ED%8F%AC%ED%8C%85%20%EC%A7%80%EB%A2%B0.md) 2026-10-17 기록.
- **첫 적용 대상은 온보딩 다음 버튼이다.** 카카오 로그인 버튼은 카카오 디자인 가이드가 컨테이너 `#FEE500`, 심볼 `#000000`, 레이블 `#000000 85%`를 고정하고 「색상 규정에 벗어난 색상을 적용해서는 안 됩니다」라고 적어 SEED 버튼으로 바꾸면 안 된다.

## 기록

### 2026-10-17: 복사본에서 SEED ActionButton을 올려 본 실험

- 맥락: 소프트웨어마에스트로 Lynx 포팅에 SEED를 넣기 전에 「홍 툴체인에서 되는가」를 확인하려고. 원본 저장소는 손대지 않고 `git clone`한 복사본(원격 제거)에서 Sonnet 에이전트가 실험하고, Fable이 핵심 주장을 원본 파일로 재확인했다.
- 배운 것: 위 핵심 정리 전부. 가장 비싼 세 가지는 (1) 빌드 통과가 화면 정상을 뜻하지 않는다(base.css 없이도 빌드됨), (2) 권장 플래그는 앱 전체에 걸리지만 BGN 기존 화면은 픽셀 단위로 불변이었다, (3) devtools import는 에러가 아니라 메인 스레드 호출 폭주 때문에 빼는 것이 맞다. 두 번째와 세 번째는 Fable 에이전트가 별도 시뮬레이터와 엔진 소스로 닫았다.
- 근거: 복사본 로그는 세션 스크래치에 두고 볼트에 넣지 않았다. 재현 명령은 핵심 정리의 명령 그대로. SEED 문서 근거는 `docs/content/lynx/getting-started/installation.mdx:13-15`(설치 대상 두 패키지), `feature-flags.mdx:17-35`(권장 플래그), `styling/vanilla-extract.mdx:22`(Rsbuild 1 peer), `packages/lynx-css/base.css:1-3,15-17`(전역 규칙), `packages/lynx-react/package.json:39-43`(peer). 레포는 `daangn/seed-design` dev `0684acaa7`.
