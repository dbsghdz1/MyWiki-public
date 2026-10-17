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

# ReactLynx 두 스레드 내부 구조

ReactLynx는 같은 JSX를 메인 스레드용(LEPUS, snapshot `create` 함수, 핸들러 자리에 `1`)과 백그라운드용(JS, Preact 런타임, 핸들러 본체)으로 두 번 컴파일한다. 런타임은 메인이 첫 화면을 그린 뒤 `rLynxFirstScreen`으로 트리를 보내 백그라운드가 hydrate하고, 이후에는 `rLynxChange` patch 배열로만 갱신한다. 근거는 lynx-stack `bb2f9ba92c8fe936be39b882047ab94aae58809f`(2026-10-16, `@lynx-js/react` 0.128.0) 소스이고, 보험찾개냥은 0.126.2라 세부가 다를 수 있다. 경로는 lynx-stack 저장소 루트 기준이다.

## 핵심 정리

### 빌드 타임 분리

- 엔트리가 둘이다. `${entryName}__main-thread`(layer `react:main-thread`, 파일 `main-thread.js`)와 `${entryName}`(layer `react:background`, `background[.contenthash:8].js`). `packages/rspeedy/plugin-react/src/entry.ts:136-145,170-176`, `packages/webpack/react-webpack-plugin/src/layer.ts:4-7`. 중간 디렉터리 기본값 `.lynx`.
- 레이어별 변환 옵션: 메인은 `jsxImportSource '@lynx-js/react/lepus'`, snapshot `target 'LEPUS'`, `__MAIN_THREAD__ true`, shake 켜짐. shake가 지우는 호출은 `useEffect`, `useLayoutEffect`, `__runInJS`, `useLynxGlobalEventListener`, `useImperativeHandle`. 백그라운드는 `'@lynx-js/react'`, target `'JS'`, shake 꺼짐. `packages/webpack/react-webpack-plugin/src/loaders/options.ts:195-203,266-340,362-393`.
- snapshot은 JSX 한 덩어리를 `__snapshot_{파일해시}_{내용해시}_{순번}` id로 뽑아 `snapshotCreatorMap[id]`에 지연 등록한 정적 템플릿이다. 리터럴 속성은 `create()` 안 PAPI 호출(`__CreateView`, `__AppendElement`, `__SetAttribute`, `__SetInlineStyles`, `__AddDataset`)로 굳고, 동적 부분은 `Attr, Spread, Slot, ListSlot`과 `values=[...]`, 자식은 `$0, $1` prop으로 표현된다. JS target은 `create`와 updater를 `null`로 비우고, LEPUS target은 이벤트와 ref 값을 `1`로 치환한다. 핸들러 본체는 백그라운드에만 남는다. `packages/react/transform/crates/swc_plugin_snapshot/lib.rs:116-121,724-758,1163-1221,1395-1398,1533-1547,1623-1625`.
- 백그라운드에서만 도달 가능한 snapshot과 worklet 정의는 가상 모듈 `__lynx-react-defines.<entry>.js`로 메인 번들에 주입된다. 못 합치면 빌드 에러 `The main thread lacks the worklet definition(s) ...`. `packages/webpack/react-webpack-plugin/src/DefinesInjection.ts:139-196,171-175`.
- `'background only'`(`"use js only"`, `"background-only"`도 같음) 함수는 LEPUS 빌드에서 params와 body가 비워진다. `import 'background-only'`는 메인 레이어에서 빌드 에러 `'background-only' cannot be imported from a main-thread module.`이다. `packages/react/transform/crates/swc_plugin_directive_dce/lib.rs:20-38`, `packages/rspeedy/plugin-react/src/backgroundOnly.ts:22-65`.
- 번들 조립: 메인 chunk 배너는 `var globDynamicComponentEntry=globDynamicComponentEntry||'__Card__'`. `app-service.js`는 `manifest['/app-service.js']`로 생성되는 백그라운드 진입 스크립트이고 인라인 chunk를 `lynx.requireModule("<경로>", '__Card__')`로 부른다. chunk는 `tt.define` AMD 래퍼 안에 있다. `.lynx.bundle` 인코딩은 저장소 밖 `@lynx-js/tasm`이 `{compilerOptions, sourceContent{dsl:'react_nodiff', appType:'card', config{lepusStrict, enableNativeList, enableCSSInheritance, ...}}, css, lepusCode, manifest}`를 직렬화한다. `DEBUG=lynx` 빌드는 `tasm.json`을 남긴다. `packages/webpack/template-webpack-plugin/src/LynxEncodePlugin.ts:218-251,334-387`, `LynxTemplatePlugin.ts:1236-1299,1381-1390`. 보험찾개냥 번들에서 본 `__Card__`, `app-service.js`, `snapshot` 58회가 이것이다.

### 런타임 흐름

- 메인 첫 렌더: `renderPage(data)` → `__CreatePage('0',0)` → `__root.ensureElements()` → `renderMainThread()`. 렌더러는 preact-render-to-string 포크로 동기 실행하며 `options[SKIP_EFFECTS]=true`를 건다. 메인이 만든 요소 id는 음수다. `packages/react/runtime/src/snapshot/lynx/calledByNative.ts:98-118`, `packages/react/runtime/src/snapshot/renderToOpcodes/index.ts:49-90`, `packages/react/runtime/src/snapshot/snapshot/snapshot.ts:171`.
- 동기화: `syncFirstScreen()`이 `JSON.stringify(__root)`를 `__OnLifecycleEvent(['rLynxFirstScreen', {root, firstScreenEventIdSwap}])`로 보낸다(기본 timing `'immediately'`). 백그라운드 `root.render(jsx)`는 Preact `render`이고 `document.createElement`가 만드는 것은 `BackgroundSnapshotInstance`다. hydrate는 type(snapshot id, 두 레이어에서 같다)끼리 짝지어 백그라운드 id를 메인 id로 바꾸고 차이는 patch로 보정한다. 불일치는 에러가 아니다. `packages/react/runtime/src/snapshot/lifecycle/event/firstScreenSync.ts:17-41`, `packages/react/runtime/src/snapshot/lynx/tt.ts:84-124`, `packages/react/runtime/src/snapshot/snapshot/backgroundSnapshot.ts:517-620`.
- 이후 업데이트: setState → Preact diff → `__globalSnapshotPatch`에 op 기록 → `options[COMMIT]` 훅이 `lynx.getNativeApp().callLepusMethod('rLynxChange', {data: JSON.stringify({patchList:[{id, snapshotPatch}], ...}), patchOptions:{reloadVersion, ...}}, cb)`를 보낸다(보험찾개냥 번들 속 `LepusMethod` 문자열). op 코드는 `CreateElement 0, InsertBefore 1, RemoveChild 2, SetAttribute 3, SetAttributes 4, nodesRefInsertBefore 5, nodesRefRemoveChild 6`. 메인 `updateMainThread`가 적용한 뒤 `__FlushElementTree(__page, flushOptions)`. 오래된 `reloadVersion` patch는 버린다. `packages/react/runtime/src/snapshot/lifecycle/patch/commit.ts:98-190`, `snapshotPatch.ts:11-23`, `updateMainThread.ts:20-106`.
- 이벤트 역방향: 메인이 `__AddEvent(el,'bindEvent','tap',"<id>:<idx>:")`를 등록하고 네이티브가 `tt.publishEvent(sign, data)`를 부르면 서명으로 백그라운드 핸들러를 찾는다. hydrate 전 이벤트는 큐잉했다가 `firstScreenEventIdSwap`으로 id를 바꿔 재생한다. `packages/react/runtime/src/snapshot/snapshot/event.ts:22-29`, `tt.ts:43,138-151,210-260`.
- 메인에서 훅 동작: `useState` setter는 아무 일도 안 하고 dev에서 `console.error('Cannot update state in main thread!')`만 낸다. `useMemo`와 `useCallback`은 매번 재계산, `useRef`는 렌더마다 새 객체, 일반 vnode의 `ref` prop은 무시되고 컴파일된 snapshot에서는 `react-ref-<id>-<idx>=1` 속성만 붙는다. `packages/react/runtime/src/core/hooks/mainThreadImpl.ts:124-156`, `packages/react/runtime/src/snapshot/snapshot/ref.ts:35-58`.
- `useLayoutEffect`는 export되지만 `useEffect`와 같은 `usePreactEffect`의 별칭이다(프로파일링이 켜지면 trace 이름만 다르다). 0.126.0부터 언마운트된 컴포넌트의 effect cleanup은 after-paint flush에서 돈다. `packages/react/runtime/src/core/hooks/react.ts:141-162`, `packages/react/CHANGELOG.md:88-96`.
- 네이티브 엔진이 `renderPage`와 `rLynxChange`를 어느 스레드에서 어떤 순서로 부르는지는 이 저장소 밖이라 미확인. 동기 에뮬레이션은 테스트 환경에만 있다.

### 에러 문자열과 트리거 (grep 키)

- `BackgroundSnapshot not found: <type>`: 백그라운드 `new BackgroundSnapshotInstance(type)`에서 `__snapshot` id가 `snapshotManager.values`와 `snapshotCreatorMap` 어디에도 없을 때. 그 JSX가 든 모듈의 등록문이 백그라운드에서 실행되지 않은 경우(중복 런타임 복사본, intrinsic tag를 문자열로 넘김 등). `packages/react/runtime/src/snapshot/snapshot/backgroundSnapshot.ts:157-168`.
- `Snapshot not found: <type>`(dev에서는 `REACT_ALOG=true` 안내가 붙음): 메인 `new SnapshotInstance(type)` 실패. 백그라운드가 보낸 `CreateElement` patch의 type이 메인 번들에 없을 때. `snapshot.ts:153-164`, `packages/react/runtime/src/snapshot/lifecycle/patch/snapshotPatchApply.ts:31-35`.
- `snapshotPatchApply failed: ctx not found, snapshot type: ..., operation: ..., commitId: ..., isHydration: ...`(`lynx.reportError`로 보고): patch가 가리키는 id가 메인 `snapshotInstanceManager`에 없을 때. id가 음수면 메인이 첫 화면에서 만든 노드, 양수면 백그라운드가 만든 노드. `operation`과 `commitId`는 0.127.0에서 추가됐으니 0.126.2 문구에는 없을 수 있다. `packages/react/runtime/src/snapshot/lifecycle/patch/error.ts:62-67`, `snapshotPatchApply.ts:45-53`.
- 트리 불일치 계열: `Invalid ctx format: ...`, `The node to be removed is not a child of this node.`, `Unexpected slot type: ...`(컴파일러와 런타임 버전 불일치 의심).
- 개발자 오용: `Cannot update state in main thread!`(dev), `cloneElement from compiled snapshot with children is not supported`, `Elements' "ref" property should be a function, or an object created by createRef(), but got [..] instead`.
- worklet 계열: `"bindtap" on <view> ... expected a main-thread function ... Did you forget to add a "main thread" directive to the handler?`(dev), `MainThreadRef: value of a MainThreadRef cannot be accessed in the background thread.`(dev), `runOnMainThread can only be used on the background thread.`, `runOnBackground can only be used on the main thread.`, `runOnBackground: JS function not found: ...`, `Depth of value exceeds limit of 1000.`
- 빌드 타임: `'background-only' cannot be imported from a main-thread module.`, `<component /> is not supported`, `The main thread lacks the worklet definition(s) ...`, `react-transform failed`.
- 리스트: `Unsupported: <list-item/> with defer={true} must be used with <list custom-list-name="list-container"/>`, `childCtx not found`, 네이티브 `220201 ... illegal list item-key`.
- 던지지 않고 `lynx.reportError`로만 나가는 경로가 있다. 메인 렌더 예외는 루트 자식을 비우고 보고만 하므로 첫 화면이 비었다가 hydration에서 채워질 수 있다(`packages/react/runtime/src/snapshot/lifecycle/render.ts:21-25`). `onLifecycleEvent`와 이벤트 핸들러도 같다. 디버깅 스위치는 `REACT_ALOG=true`(스레드 간 전송과 patch를 ALog로), `REACT_ALOG_ELEMENT_API`, `REACT_PROFILE`, `REACT_DEVTOOL`.

### Main Thread Script

- `"main thread"` 또는 `"main-thread"` 디렉티브 함수는 `{ _c: {캡처 변수}, _wkltId: "<sha1(파일명) 앞4자>:<내용해시>:<순번>", _jsFn: {...} }` 컨텍스트 객체로 바뀌고, 본문은 LEPUS 빌드에서만 `registerWorklet("main-thread", id, fn)`로 등록된다. `_lepusWorkletHash`는 pre-0.99 호환 필드이고 보험찾개냥 번들의 `lepusWorkletHash` 문자열은 이 호환 분기다. `packages/react/transform/crates/swc_plugin_worklet/gen_stmt.rs:143-177,433-448`, `packages/react/runtime/src/worklet-runtime/bindings/types.ts:39-50`.
- `main-thread:bindtap`은 `__AddEvent(..., {type:'worklet', value})`로 연결된다. `runOnMainThread`는 백그라운드 전용이며 hydrate 후이고 렌더 중이 아니면 즉시, 아니면 다음 commit patch에 실린다. `runOnBackground`는 메인 worklet 안에서만 쓸 수 있고 첫 화면 중 호출은 hydrate 후로 지연된다. `useMainThreadRef`의 init 값은 `rLynxChangeRefInitValue`로 메인에 전달된다. SDK 요구는 MTS > 2.13, runOnBackground > 2.15. `packages/react/runtime/src/snapshot/snapshot/workletEvent.ts:63-70`, `packages/react/runtime/src/core/thread-function-call/main-thread.ts:35-81`, `packages/react/runtime/src/core/background-function/run-on-background.ts:121-167`, `packages/react/runtime/src/core/mts-capability.ts:13`.

### 버전

- 보험찾개냥 0.126.2 + 플러그인 0.20.3은 한 세트다. 0.127.0은 MainThreadObject API와 ctx-not-found 진단 확장, 0.128.0은 호환이 깨지는 메인 런타임 module delivery(`@lynx-js/react/worklet-runtime/init` 추가, prebuilt worklet-runtime chunk와 `loadWorkletRuntime` 제거, `Older compiler output is unsupported.`)다. 플러그인 0.21.0의 peer가 `@lynx-js/react ^0.128.0`이라 react만 올리면 안 되고 플러그인과 함께 올린다. `packages/react/CHANGELOG.md:3-37`, `packages/rspeedy/plugin-react/package.json:73`.
- `createElement`와 `cloneElement`의 문자열 태그 지원은 0.121.0, compiled snapshot clone 0.122.1로 보험찾개냥 버전에 이미 있다.

### 테스트 라이브러리

- `LynxTestingEnv`가 `mainThread`와 `backgroundThread` 전역 객체 둘을 만들고 `switchToMainThread()`, `switchToBackgroundThread()`가 그 속성을 Node `global`에 덮어쓴다. `callLepusMethod`는 메인으로 전환해 `globalThis['rLynxChange'](data)`를 부르고, `__OnLifecycleEvent`는 백그라운드 `tt.OnLifecycleEvent`를 부른다. `packages/testing-library/testing-environment/src/index.ts:152-181,683-708`.
- 테스트 로더는 `mode:'test'`, snapshot `target:'MIXED'`, `directiveDCE:false`, shake 기본 false다. 그래서 테스트에서는 `'background only'` 제거나 `useEffect` 제거가 일어나지 않고 활성 스레드에 따라 실행된다. rstest에서는 `callerName === 'rstest'`일 때 이 로더만 적용된다. `packages/webpack/react-webpack-plugin/src/loaders/testing.ts:63-103`, `packages/rspeedy/plugin-react/src/pluginReactLynx.ts:466-476`.
- `render()`는 기본 `enableMainThread=false`라 메인 첫 화면까지 보려면 `enableMainThread: true`를 준다. `container`는 `lynxTestingEnv.mainThread.elementTree.root`, `fireEvent`는 백그라운드로 전환해 DOM 이벤트를 보낸다. `withLynxConfig(options)`는 `lynx.config.*`를 rspeedy `loadConfig`로 읽어 `@rstest/adapter-rsbuild`로 변환하고 `lynx:rsbuild:qrcode`, `lynx:rsbuild:target` 플러그인을 뺀다. `packages/react/testing-library/src/pure.jsx:39-109`, `rstest-config.ts:98-189`.

## 기록

### 2026-10-17: lynx-stack 소스로 두 스레드 구조를 읽음

- 맥락: 소프트웨어마에스트로 Lynx 포팅과 SEED 도입 전에 「컴포넌트 코드가 두 번 돈다」의 실제 구조와 에러 문구를 잡아 두려고. Sonnet 에이전트가 읽고, Haiku가 인용 261건의 파일 존재와 줄 범위를 전수 확인(실패 0)하고 표본 25건을 대조(불일치 0, 형식만 다른 근사 일치 5)했으며, Fable이 `mainThreadImpl.ts`의 setter, `options.ts`의 removeCall 목록, `renderToOpcodes`의 `SKIP_EFFECTS`를 직접 다시 봤다.
- 배운 것: 위 핵심 정리. 보험찾개냥 번들에서 본 `__Card__`, `app-service.js`, `LepusMethod`, `lepusWorkletHash`, `snapshot` 58회의 정체가 여기 있다.
- 근거: lynx-stack `bb2f9ba92c8fe936be39b882047ab94aae58809f`(2026-10-16). 줄 번호는 그 커밋 기준이고 0.126.2 태그 코드는 보지 않았다.
