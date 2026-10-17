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

# SEED Lynx 패키지 지도

SEED의 Lynx 쪽은 `@seed-design/lynx-react`(스타일드 컴포넌트)와 `@seed-design/lynx-css`(토큰과 recipe CSS), 그리고 lynx-react가 끌고 오는 headless 패키지 35개로 되어 있다. 컴포넌트 일부는 패키지 export이고 일부는 CLI 스니펫(`ui:*`)이다. ActionButton은 패키지 export다. 기준은 `daangn/seed-design` dev `0684acaa7`(2026-10-16), 경로는 그 레포 루트 기준이다. 설치와 툴체인 호환은 [SEED Lynx 설치와 BGN 툴체인](SEED%20Lynx%20%EC%84%A4%EC%B9%98%EC%99%80%20BGN%20%ED%88%B4%EC%B2%B4%EC%9D%B8.md)에 있다.

## 핵심 정리

### 소비 모델

- 설치는 두 패키지뿐이다. headless는 lynx-react의 dependencies로 따라온다(`docs/content/lynx/updates/upgrade/v1.mdx:78`).
- Registry 위치: `docs/registry/lynx/registry-ui.ts`(id `ui`, 항목 39개)와 `registry-lib.ts`(`manner-temp-level` 1개). 원본은 `docs/registry/lynx/ui/*.tsx`, 생성물은 `docs/public/__registry__/lynx/{index.json,ui/<id>.json}`, CLI는 `${BASE_URL}/__registry__/lynx/...`를 fetch한다(`packages/cli/src/utils/fetch.ts:62,88,115`).
- snippet 39개: accordion, alert-dialog, app-bar, attachment-display-field(+reorderable), attachment-field(+reorderable), avatar, badge, bottom-sheet, callout, checkbox, chip-tabs, content-placeholder, dialog, field-button, floating-action-button, help-bubble, identity-placeholder, list(+list-header), manner-temp, manner-temp-badge, menu, menu-sheet, page-banner, progress-circle, pull-to-refresh, quantity-picker, radio-group, result-section, segmented-control, select, select-box, slider, switch, tabs, tag-group, text-field, wheel-picker. 전부 `@seed-design/lynx-react`의 compound part를 조립하는 얇은 래퍼이고(`docs/registry/lynx/ui/switch.tsx:1-28`가 예), result-section만 대응 컴포넌트가 없는 합성 전용, reorderable 둘만 `@seed-design/lynx-react-sortable`이 더 필요하다.
- 패키지에서 직접 import하는 것(snippet 없음): ActionButton(`docs/content/lynx/components/action-button.mdx:17-19`), Chip, ContextualFloatingButton, Divider, ImageFrame, ReactionButton, ScrollFog, Skeleton, KeyboardAvoidingScrollView, Box/HStack/VStack, Text, Icon/PrefixIcon/SuffixIcon, ScaleFeedback(`packages/lynx-react/src/components/index.ts:1-54`). 레포 규칙은 registry가 있으면 snippet, 없는 것만 패키지 export(`docs/examples/lynx/AGENTS.md:39`).
- snippet은 복사본이라 패키지 업그레이드로 자동 갱신되지 않는다(`v1.mdx:90`). 아이콘 import가 있는 14개는 `@karrotmarket/lynx-monochrome-icon`을 이름만으로 설치한다.
- CLI: `seed-design.json`은 zod strict로 `$schema?`, `rsc`(기본 false), `tsx`(기본 true), `framework`("react"|"lynx", 기본 "react"), `path`(필수), `telemetry`(기본 true)다(`packages/cli/src/utils/get-config.ts:13-22`). `init`은 `<cwd>/seed-design.json` 한 파일만 쓰고 존재 확인 없이 덮어쓴다(`packages/cli/src/utils/init-config.ts:80-95`). Lynx 자동 감지는 deps, devDeps, peerDeps의 `@lynx-js/react` 또는 `@seed-design/lynx-react`(`:18-31`). **init 없이 `add`하면 자동 생성 설정이 `framework: "react"`라 React registry를 받는다**(`get-config.ts:41,44`, `init-config.ts:13`). `init`을 먼저 하거나 `-f lynx`를 쓴다. `add ui:switch`는 `<path>/ui/switch.tsx`를 쓰고 기존 파일과 다르면 덮어쓰기, 백업(`legacy-<이름>-<타임스탬프>`), 건너뛰기 프롬프트를 띄운다(`packages/cli/src/utils/write.ts:71-126`). tsconfig, lynx.config, CSS는 건드리지 않아 alias는 수동이다(`installation.mdx:106-152`). 텔레메트리는 `"telemetry": false` 또는 `DISABLE_TELEMETRY=true`.

### 스타일링과 테마

- `base.css`는 필수다. 토큰 변수, 라이트와 다크 셀렉터, icon slot 클래스가 여기 있고(`packages/lynx-css/base.css:1-54`) recipe CSS는 컴포넌트 import 때 따라온다. `all.css`는 문서에 없다. 순서 규칙은 「스타일이 컴포넌트보다 먼저 등록돼야 한다」(`docs/examples/lynx/AGENTS.md:31`). 문서 예제 31개의 `styles.ts`는 `base.css`와 plain `preview.css`만 import한다(`:32`).
- 테마: Lynx는 `data-*` 셀렉터를 지원하지 않아 클래스를 쓴다(`getting-started/styling/theming.mdx:13`). 루트 `<page className={getSeedClassName({ colorMode: "system" })}>`가 `seed-user-color-scheme-light|dark`를 붙이고 system은 `lynx.__globalProps.theme`을 읽는다(`packages/lynx-react/src/utils/get-seed-class-name.ts:36-47`). 하위 영역은 `seed-color-mode-light-only|dark-only`. 런타임 전환에는 `useGlobalProps`를 구독하는 `useSeedClassName`이 필요하고(`packages/lynx-react/src/hooks/useSeedClassName.ts:29-35`) 레포 앱도 그것을 쓴다(`examples/lynx-spa/src/index.tsx:26-33`). host가 `theme` global prop을 줘야 한다. Tailwind v4는 루트 테마만 지원.
- Box/HStack/VStack은 막힌 것이 아니라 prop 해석 비용 때문에 정적 CSS를 권장한다(`getting-started/styling/index.mdx:15-28`).

### ActionButton 전 계층

- 문서 `docs/content/lynx/components/action-button.mdx`. 예제 14개 `docs/examples/lynx/action-button/*.tsx`, `loading.tsx:7-17`은 `bindtap` 핸들러에 `"background only"`를 둔다. 미지원 prop은 `color`, `fontWeight`(`:160-167`).
- 컴포넌트 `packages/lynx-react/src/components/ActionButton/ActionButton.tsx`(Root :70-133, 공개 :264-359). recipe className, headless `useActionButton`, ScaleFeedback, 접근성 라벨, 아이콘 slot 배분을 결정한다.
- headless `packages/lynx-react-headless/action-button/src/useActionButton.ts:61-136`. `interactive = !disabled && !loading`(:73), 기본값 `disabled = false`, `loading = false`(:63-64). `usePressTap`(`packages/lynx-react-headless/use-press-tap/src/usePressTap.ts:52-129`)이 `pressed` state와 background-only tap을 맡는다.
- 생성 recipe `packages/lynx-css/recipes/action-button.{css,mjs,d.ts}`. slot 7개 root, content, text, prefixIcon, suffixIcon, icon, loadingIndicator. 기본값 `variant brandSolid, size medium, layout withText, pressed/disabled/loading false`(`.mjs:35-42`). 클래스 형식 `.seed-action-button__root--variant_brandSolid`. 생성물이라 직접 수정하지 않는다.
- recipe 원천 `packages/lynx-qvism-preset/src/recipes/action-button.ts`(`defineSlotRecipe` :12-14, variants :81-221, compound :222-621), `bun qvism:generate`. rootage 스펙 `packages/rootage/components/action-button.yaml`(`scaleScope: self`), `bun rootage:generate`가 `packages/lynx-css/vars/component/action-button.mjs`를 만들고 컴포넌트는 런타임에 `actionButtonVars`로 ProgressCircle 크기를 읽는다(`ActionButton.tsx:53-66`).
- 공개 props: `variant` brandSolid|neutralSolid|neutralWeak|criticalSolid|brandOutline|neutralOutline|ghost, `size` xsmall|small|medium|large, `layout` withText|iconOnly, `disabled`, `loading`. 탭은 `bindtap` 또는 `main-thread:bindtap`이고 `onClick`은 없다(`packages/lynx-react/src/types.ts:41-44`). disabled나 loading이면 둘 다 막힌다. `className`은 root와 clsx 병합, `style`은 인라인이며 사용자 값이 마지막에 이긴다, `ref`는 root `<view>`. 추가로 `flexGrow`, `bleed/bleedX/bleedY`(px, 0, "safeArea"만)와 `<view>` native props 전부.
- 요소 트리: `<view root flatten=false>` 안에 `[prefix <view><image/></view>] <text>label</text> [suffix]`. iconOnly는 `<Icon>` wrapper 하나. loading은 `<view loadingIndicator>`(ProgressCircle)와 `<view content>`(opacity 0으로 폭 유지). `<button>`은 없다.
- 눌림 피드백은 3겹: CSS `:active`가 enabled이고 loading이 아닐 때 root 배경을 pressed 색으로(`lynx-css/recipes/action-button.css:344-346`), Main Thread 터치 핸들러가 `runOnBackground`로 `pressed` state를 갱신해 `...pressed_true` 클래스, Main Thread `animate`로 scale((basis-2)/basis), basis=max(높이, 폭/4, 24), 150ms, `cubic-bezier(0, 0, 0.15, 1)`. root에 `flatten:false`, `main-thread:ref`, `main-thread:bindlayoutchange`, `main-thread:bindtouchstart/end/cancel`이 주입되고 `GlobalProps.motion === "reduced"`면 건너뛴다(`packages/utils/lynx-react-scale-feedback/src/useScaleFeedback.ts:62-89,129-130,153`).
- 접근성: root `accessibility-element`(조상이 접근성 요소면 false), `accessibility-traits` "button" 또는 비활성과 로딩이면 "disabled", `accessibility-label`은 명시값이 없으면 문자열 children에서 추출하고 iconOnly는 필수. children에 intrinsic `<text>`가 있으면 추출을 포기하므로 명시가 필요하다. text slot과 아이콘 wrapper는 접근성 트리에서 숨긴다. dev 경고 `` <Name>: `accessibility-label` is required when children have no text. ``.

### 런타임 불변식과 에러 문자열

- `packages/lynx-react/AGENTS.md:16` 「어기면 타입 검사는 통과하고 기기에서만 깨진다.」 :18 intrinsic tag(`view`, `text`, `image`)는 컴포넌트 파일 안에서 리터럴 JSX로만 쓴다. 변수 tag, `React.createElement("view", ...)`, `withProvider`와 `withContext`에 문자열 전달은 `BackgroundSnapshot not found`를 낸다. :19-21 `children`은 `nativeProps`에 섞지 않고 분리, ref는 `{...(ref ? { ref } : {})}`로 있을 때만, CSS `inherit`은 지원 안 한다. :32-35 React 레이어에 `style` 직접 사용 금지(Recipe로), variant는 `splitVariantProps`로 분리, 런타임 상태는 `React.createContext`로 전달하고 context가 없으면 throw. :37-52 raw `<overlay>` 금지, `@lynx-js/lynx-ui-overlay`의 `OverlayView` 직접 사용, overlay 모드는 `event-through` 기본 true. :54-58 프레임 애니메이션은 `requestAnimationFrame`, 반복 애니메이션은 state 대신 `setStyleProperty`.
- `assertNotIntrinsicComponent`(`packages/lynx-react/src/utils/create-slot-recipe-context.tsx:20-26`, 호출 :105, :139, :176) 메시지 원문: `${caller} cannot wrap intrinsic tag "${Component}" in Lynx. Native <view>/<text> slots must be rendered as literal JSX in the component file; passing intrinsic strings through createSlotRecipeContext can cause BackgroundSnapshot not found errors.`
- production에서도 throw: `useClassNames must be used within a ClassNamesProvider. Did you forget to wrap your component in a ClassNamesProvider?`, 파트가 자기 Root 밖에 있을 때 `<X/> must be rendered inside <XRoot/>` 또는 `inside a styled <XRoot/>`(Switch, RadioGroup, Checkbox, Menu, Select, AppBar, Slider, Accordion), `Tabs must be rendered inside a styled TabsRoot`, `FloatingActionButtonLabel must be used within FloatingActionButtonRoot.`, `ContentPlaceholder.Asset must be rendered inside ContentPlaceholder.Root.`, `MenuSheet item slots must be used within MenuSheetItem.`, `WheelPicker.Column must be rendered inside WheelPicker.Root.`, headless `useActionButtonContext must be used within an ActionButtonRoot`. 트리거는 styled Root 대신 headless Root를 쓰거나 part를 트리 밖에 둘 때.
- 개발 모드 전용: `Icon-only Component must render only one <Icon /> under children. ...`, `... must not be nested within another Icon-Only. ...`, `... must render <Icon /> as a child. Check if you are using raw icon element instead of <Icon icon={} />.`(`packages/lynx-react/src/components/Icon/Icon.tsx:49,64,69`), `` FieldButton.ClearButton requires `accessibility-label` for accessibility. ``, PageBanner `solid`+`magic` 조합 `console.error`.
- 기타 실측 제약: `accessibility-element` view의 라벨은 하위 `<text>`에서 만들어지지 않는다(`packages/lynx-react/src/utils/accessibility.tsx:7-9`). 인라인 `calc()` 안의 `var()`는 Lynx가 버린다(`ActionButton.test.tsx:83`). SEED 컴포넌트에 주는 class에 `display`를 두지 않는다(`docs/examples/lynx/AGENTS.md:23`). Box/Stack `style`의 `flex: 1` 축약은 native에서 적용되지 않는다(`:24`). `overflow: scroll` 미지원, `position` 기본 `relative`(`getting-started/styling/tailwind-css-4.mdx:214-220`). qvism preset 금지 목록은 `boxSizing`, `verticalAlign`, SVG stroke/fill, `content`, `initial/inherit/unset`, `inset` shorthand(`packages/lynx-qvism-preset/README.md:28-37`).

### 아이콘

- `<image>`는 SVG를 못 그린다(`packages/rsbuild-plugin-lynx-icon/README.md:5` "Lynx 3.5 does not support SVG natively"). 아이콘 패키지 `@karrotmarket/lynx-monochrome-icon`, `lynx-multicolor-icon`은 `.svg`를 import해 `<image src tint-color>`로 렌더하므로 빌드가 `.svg`를 `<image src>`가 읽는 값으로 바꿔야 한다. `@seed-design/rsbuild-plugin-lynx-icon`을 `pluginLynxIcon({ include: /node_modules\/@karrotmarket\/assets-(monochrome|multicolor)\/svg\// })`로 등록한다(`docs/content/lynx/foundation/iconography.mdx:31-33,58-61`). 로더는 `sharp`로 폭 `maxSize`(기본 72)로 줄여 WebP data URI를 낸다(`src/loader.ts:4-5,12-34`). peer는 `@rsbuild/core ^1.1.0`이지만 레포는 Rsbuild 2.1.10에서 쓴다(2.2.9 미확인). lynx-spa는 추가로 모든 rspack rule에 `.svg` 제외와 별도 svg 로더를 덧붙이는데 이유는 문서에 없다(`examples/lynx-spa/lynx.config.ts:95-129`). **아이콘이 안 보이면 이 블록부터 비교한다.**
- slot 안에서는 `Icon`, `PrefixIcon`, `SuffixIcon` wrapper를 쓴다. wrapper가 recipe의 CSS `color`를 main thread에서 읽어 `<image tint-color>`로 동기화하고(`packages/lynx-react/src/hooks/useIconColor.ts:15-36,82-117`) 크기는 slot이 정한다. 안쪽 아이콘에 `color`나 `tint-color`를 주지 않는다.

### 레포 쪽 테스트 설정

- `packages/lynx-react/vitest.config.ts`가 `createLynxVitestConfig()`를 호출하고, `packages/lynx-vitest-config/src/index.mjs:7-65`가 `@lynx-js/react/testing-library/vitest-config`를 감싼다. internal preact alias, `include: src/**/*.test.{ts,tsx}`, `optimizeDeps.exclude`와 `ssr.noExternal`에 `@lynx-js/react`, `@lynx-js/gesture-runtime`, `preact`, `@lynx-js/internal-preact`(`:43-62`).
- 패턴: Main Thread 경로는 `render(<X/>, { enableMainThread: true, enableBackgroundThread: true })` + `waitSchedule()` + `fireEvent.touchstart`. 단순 단위 테스트는 `vi.mock("@lynx-js/react")`로 `runOnMainThread`를 no-op 처리(`ActionButton.test.tsx:12-19`). 전역 `lynx.__globalProps`는 `globalThis`, `lynxTestingEnv.backgroundThread.globalThis`, `mainThread.globalThis` 세 곳에 모두 넣는다(`packages/lynx-react/src/components/Dialog/Dialog.test.tsx:67-81`). rstest 언급은 레포 전체에 없다.

### 문서와 패키지의 빈틈 (PR 재료 후보, 코드로 확인한 것만)

- 사용자가 설치하는 `packages/lynx-react`, `packages/lynx-css`에 README가 없다(headless 35개, qvism-preset, 아이콘 플러그인은 있다).
- `theming.mdx:18-33,184`는 비반응형 `getSeedClassName`만 안내하고 `useSeedClassName`을 언급하지 않는다(`hooks/use-seed-class-name.mdx:55-59`와 대조).
- 「라이브러리 없이 CSS Modules」 경로가 문서에 없다(`installation.mdx:23`). 그러나 문서 예제는 plain CSS를 쓴다.
- `ActionButton.tsx:40`이 가리키는 AGENTS.md 「Native tag literal JSX constraint」 절이 없다(실제 이름은 「런타임 불변식」, `AGENTS.md:14-18`).
- init 없이 `add`하면 React registry를 받는데 문서에 경고가 없다. 문서의 prompt 흐름과 JSON 예에는 `rsc`가 없지만(`cli.mdx:29-36`) 코드는 항상 `rsc`를 묻고 기록한다(`init-config.ts:11-16,43-47`).
- 아이콘 플러그인 README는 "Lynx 3.5"인데 SEED 최소 엔진은 3.6(`feature-flags.mdx:9`). peer가 Rsbuild 1 범위인데 iconography.mdx에는 경고가 없다.
- 참조 설정은 `targetSdkVersion: "3.9"`(`docs/lynx.config.ts:30`, `examples/lynx-spa/lynx.config.ts:66`)인데 문서는 `engineVersion: "3.6"`과 `targetSdkVersion` deprecated를 안내한다(`feature-flags.mdx:59`).
- `packages/lynx-react/src/utils/resolve-recipe-token.ts:10`의 JSDoc import 경로가 exports에 없다. `ARCHITECTURE.md:22`는 snippet 원천에 Lynx registry를 빠뜨렸다. CLI 예제 `ui:action-button`(`packages/cli/src/commands/add.ts:58`)은 Lynx registry에 없다. `installation.mdx:47-53`은 코드 블록이 `<Step>` 밖에 있다.

## 기록

### 2026-10-17: SEED 레포를 읽어 Lynx 쪽 지도를 만듦

- 맥락: 소프트웨어마에스트로 앱에 SEED를 넣는 일과 [오픈소스 기여](../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/%EA%B0%9C%EB%B0%9C%EB%B0%A9%EB%B2%95/%EC%98%A4%ED%94%88%EC%86%8C%EC%8A%A4%20%EA%B8%B0%EC%97%AC.md) 둘 다를 위해. Sonnet 에이전트가 읽고, Haiku가 인용 319건의 파일 존재와 줄 범위를 확인했다(줄 범위 초과 1건은 `index.mjs:43-62`로 정정, 표본 25건 중 위치 불일치 1건은 `get-config.ts` 인용을 `init-config.ts:13`과 함께 적는 것으로 반영). Fable이 `assertNotIntrinsicComponent` 메시지, 기본 variant, `interactive` 정의를 직접 확인했다.
- 배운 것: 위 핵심 정리. 보험찾개냥에서 SEED 컴포넌트가 「기기에서만 깨지면」 런타임 불변식 절부터 본다.
- 근거: `daangn/seed-design` dev `0684acaa7`(2026-10-16 #2487), 로컬 클론에서 빌드 완료 상태로 읽음.
