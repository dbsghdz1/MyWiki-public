---
type: study
area: JS
audience: me
status: active
created: 2026-10-17
updated: 2026-10-17
projects:
  - "소프트웨어마에스트로"
---

# Lynx 구조와 React와의 차이

한 줄 요약: Lynx는 React와 CSS로 쓴 화면을 브라우저 대신 C++ 엔진이 받아 진짜 네이티브 뷰로 그리는 런타임이고, 내 코드는 메인 스레드와 백그라운드 스레드 둘에서 돈다. 2026-10-17 시점의 이해다.

읽기 쉽게 다시 쓴 판은 [Lynx가 폰에서 도는 법](https://claude.ai/artifact/Rko7n5Dku4mUvPBG5gJ5Xy)이다(2026-10-17, iOS 개념에 빗댄 그림 두 개와 네이티브, RN, Flutter, 웹 React 비교 포함). 에이전트용 세부 기록은 [ReactLynx 두 스레드 내부 구조](../../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Lynx/ReactLynx%20%EB%91%90%20%EC%8A%A4%EB%A0%88%EB%93%9C%20%EB%82%B4%EB%B6%80%20%EA%B5%AC%EC%A1%B0.md), [SEED Lynx 패키지 지도](../../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Lynx/SEED%20Lynx%20%ED%8C%A8%ED%82%A4%EC%A7%80%20%EC%A7%80%EB%8F%84.md), [SEED Lynx 설치와 BGN 툴체인](../../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Lynx/SEED%20Lynx%20%EC%84%A4%EC%B9%98%EC%99%80%20BGN%20%ED%88%B4%EC%B2%B4%EC%9D%B8.md)에 있다.

## 핵심 정리

### 어디서 왔나

- ByteDance 팀이 만들어 TikTok 안에서 먼저 썼다. 검색 패널, TikTok Studio 앱, Shop, LIVE, Disney100과 Met Gala 이벤트 화면이 공식 블로그가 밝힌 사용처다. 2025-03-05에 오픈소스로 공개했고, 공개 버전이 3.x부터 시작하는 이유는 프로덕션에서 쓰던 버전을 그대로 열었기 때문이다. 시작 연도는 공식 글에 없다.
- 코어 엔진 저장소 `lynx-family/lynx`는 C++이고 2025-03-04 생성, 첫 릴리스 3.2.0-rc.1은 2025-03-21, 4.0.0은 2026-07-20, 4.1.0은 2026-09-07이다(GitHub 릴리스, 2026-10-17 기준). 4.0 블로그(2026-09-30)는 폴더블, CSS 미디어 쿼리, `<blur-view>`, 오픈소스 `<webview>`, OpenUI 렌더러를 추가했고 「메이저지만 파괴적 변경은 아니다」라고 적었다.
- 자기 규정은 「앱 개발에 맞춘 대안 웹(alternative Web tailored for app development)」이다. 웹 지식(CSS, React)을 그대로 쓰되 문서 중심 제약을 버리고 네이티브 프리미티브 위에 그린다. React 전용이 아니며 다른 프레임워크 사용이 전체의 절반쯤이라고 밝힌다.
- 빌드 도구 Rspeedy는 같은 회사의 Rspack 계열 위에 있다. 보험찾개냥 빌드 출력이 `Rspeedy v0.18.0 (Rsbuild v2.2.9, Rspack v2.2.8)`이다.

### 폰에서 어떻게 도나

```
rspeedy build ──▶ dist/main.lynx.bundle  (이진 파일, 옛 이름 template)
                    ├ 메인 스레드 스크립트 (PrimJS 바이트코드, main-thread.js)
                    ├ 백그라운드 JS (app-service.js, background.<hash>.js)
                    └ 설정 JSON (enableNativeList, enableCSSInheritance ...)
        │ 파일로 내장하거나 HTTP로 받는다 (개발 서버 → Lynx Explorer)
        ▼
호스트 앱 안의 LynxView  (iOS: UIView 하위 클래스, Android: View 하위 클래스)
        ▼
Lynx Engine (C++)
  메인 스레드 (PrimJS): 첫 프레임 동기 렌더(IFR), 레이아웃, 페인트 지시, 메인 스레드 스크립트
  백그라운드 스레드 (iOS JavaScriptCore, Android PrimJS): React 런타임, 상태, 이펙트, 이벤트 핸들러
        │ 요소 트리 변경을 메시지로 보낸다
        ▼
플랫폼 뷰  view → UIView / ViewGroup,  text → UITextView / TextView,  image → UIImageView / ImageView
          scroll-view → UIScrollView / ScrollView,  list → UICollectionView / RecyclerView
```

- **번들 실물**: 보험찾개냥 `rspeedy build` 결과는 `main.lynx.bundle` 200.7 kB 하나와 SVG 하나다. `file`은 `data`라고 답하고, 안의 문자열에 백그라운드 JS(`app-service.js`, `background.<hash>.js`, 가장 큰 텍스트 덩어리 약 76 kB), `main-thread.js`라는 이름, `Lepus` 계열 문자열, `snapshot` 58회, 설정 JSON(`lepusStrict`, `enableNativeList`, `enableCSSInheritance: false`, `dsl: 2`)이 들어 있다. 메인 스레드 스크립트 본문은 텍스트로 안 보이는데, 공식 문서가 메인 스레드 스크립트는 PrimJS 바이트코드로 실리며 텍스트보다 약 4배 빨리 로드된다고 설명한다. Lepus는 메인 스레드 VM의 옛 이름이다(용어집 표제).
- **혼자 뜨는 앱이 아니다**: 엔진은 「Lynx 페이지를 픽셀로 바꾸는 SDK의 네이티브 부분」이고, 앱 쪽에서 `LynxEnv`를 초기화하고 번들을 읽어 주는 TemplateProvider를 구현한 뒤 `LynxView`를 뷰 트리에 넣어야 한다. 공식 문서 문장: 「Lynx has no built-in resource loading」. 개발 중에는 Lynx Explorer 앱이 이 호스트 역할을 하고 개발 서버 URL로 번들을 받는다. 웹에서는 `@lynx-js/web-core`의 `<lynx-view url>` 요소가 같은 자리다.
- **두 개의 JS 런타임**: 메인 스레드는 PrimJS(QuickJS 기반, Lynx 전용 경량 엔진)이고, 백그라운드는 iOS 기본 JavaScriptCore, Android 기본 PrimJS다(패키지 크기와 성능 이유). 지원 문법은 백그라운드 ES2015, 메인 ES2019까지이고 그 위 문법은 SWC가 빌드 때 내린다. 용어집은 Lynx 메인 스레드가 OS의 UI 스레드와 꼭 같지는 않다고 적는다.
- **첫 프레임**: 데이터가 준비돼 있으면 메인 스레드가 첫 화면을 동기로 그린다(Instant First-Frame Rendering). 번들을 동기로 못 읽거나 본문이 네트워크를 기다리면 성립하지 않는다. 백그라운드는 자기 트리를 따로 만들어 메인 트리와 비교한 뒤 차이만 동기화하고, 이후 모든 생명주기와 사용자 코드는 백그라운드에서만 돌며 변경은 메시지로 메인에 넘어간다.
- **레이아웃은 엔진 것**: `display`는 `linear`(기본, Android LinearLayout에서 왔다), `flex`(웹과 동일), `grid`(일부), `relative`, `none`. 공식 문장은 「Web layout is primarily text-based. While Lynx layout is based on elements」. 모든 요소는 블록이고 글자는 `<text>` 안에만 들어가며 `box-sizing`은 `border-box` 기본, 마진 겹침이 없다. 보험찾개냥 10-16 지뢰 「view 기본 display가 linear라 flex 속성이 무시된다」가 바로 이것이다.
- **CSS는 캐스케이드는 표준, 상속은 opt-in**: 커스텀 속성만 기본 상속되고 일반 속성 상속은 설정(`enableCSSInheritance`)으로 켠다. 번들 설정에 `false`로 박혀 있는 값이 그것이다. `-x-` 접두사 확장 속성이 있다.

### React와 뭐가 다른가

같은 것: Preact 위에 만들어 API와 동작이 React와 거의 같고, 바꾸는 건 import 출처(`react` → `@lynx-js/react`)다. 보험찾개냥 `src/index.tsx`는 `root.render(<App />)` 한 줄로 시작한다.

| 항목 | React DOM | ReactLynx |
|---|---|---|
| 실행 스레드 | 하나 | 둘. 컴포넌트 본문은 양쪽에서 실행돼 렌더 안의 `console.log`가 두 번 찍힌다 |
| DOM | `document`, `window` 있음 | 없음. `window.location.reload()` 대신 `lynx.reload` |
| 태그 | `div`, `span`, `img` | `view`, `text`, `image`, `scroll-view`, `list` |
| 이벤트 | `onClick`, `onTouchStart` | `bindtap`, `catchtap`, `bindtouchstart`. `e.stopPropagation()`은 메인 스레드 스크립트에서만 |
| 부수효과 | 어디서나 | 백그라운드 전용. 이벤트 핸들러, `useEffect`, `ref`, `'background only'` 지시어. props로 넘기는 핸들러와 커스텀 훅에 주는 콜백은 직접 표시해야 한다 |
| `useLayoutEffect` | 있음 | 없음. 측정은 `main-thread:bindlayoutchange`로 |
| 지연 민감한 처리 | 없음 | `'main thread'` 지시어 + `main-thread:bindtap`, `useMainThreadRef`, `runOnMainThread` / `runOnBackground`. 캡처한 값은 JSON으로 복사되고 수정 불가 |
| JSX | 런타임에 전부 평가 | 정적 JSX는 컴파일 타임에 snapshot으로 추출. 번들의 `snapshot` 58회가 그것이다 |
| 생명주기 시점 | 단일 스레드 기준 | 다르다. 「complete consistency with single-threaded React is not possible」 |

왜 둘로 나눴나: 백그라운드 JS가 오래 돌아도 메인 스레드의 렌더와 터치 응답을 막지 않기 위해서다. 메인 스레드 스크립트는 터치가 스레드를 여러 번 건너며 생기는 지연을 없애려는 작은 예외다.

### 네이티브, WebView, React Native, Flutter와 비교

| | 화면을 그리는 주체 | 로직이 도는 곳 | 배포 단위 |
|---|---|---|---|
| 네이티브 앱 | UIKit, Compose를 직접 호출 | 컴파일된 Swift, Kotlin. OS 메인 스레드 | 앱 바이너리 |
| WebView 앱 | 브라우저 엔진이 DOM을 그림 | JS 단일 스레드 | HTML, JS |
| React Native | 네이티브 뷰 | JS 스레드 하나와 네이티브 UI 스레드 (해석) | JS 번들 + 네이티브 셸 |
| Flutter | 자체 렌더러가 픽셀을 직접 그림 | Dart | 앱 바이너리 |
| Lynx | 엔진이 만든 네이티브 뷰. 커스텀 렌더러도 가능 | JS 두 스레드(메인 PrimJS, 백그라운드) | `lynx.bundle` + 호스트 앱 |

- 네이티브와의 핵심 차이: 같은 UIView를 쓰므로 생김새와 스크롤은 네이티브지만, 로직은 JS 두 스레드에서 돌고 화면 정의는 런타임에 받은 번들이라 앱 바이너리를 바꾸지 않고 갱신할 수 있다. 대신 호스트 앱과 엔진이 먼저 있어야 한다.
- 공식 블로그는 WebView를 덧대는 방식(PhoneGap, Cordova)과 선을 긋고, React Native(2015)는 선구자로, Flutter(2015)는 자체 렌더러 방식으로만 언급한다. 직접 비교 문장은 없다. 위 표의 React Native 행은 내 해석이고, 나머지는 공식 문서 기준이다.

## 기록

### 2026-10-17: Lynx는 React와 뭐가 다르고 폰에서 어떻게 도나

- 궁금했던 것: 보험찾개냥 Lynx 앱에 SEED를 넣기 전에 「Lynx가 어떤 건지, React와 뭐가 다른지, 역사, 휴대폰 위에서 어떻게 돌아가는지, 네이티브와 어떻게 다른지」를 먼저 알고 가자(홍).
- 예측: 아직 없음. 다음 세션은 「컴포넌트 함수 본문의 `console.log`는 Explorer에서 몇 번 찍힐까」 예측부터 시작한다.
- 파고든 길: 보험찾개냥을 `rspeedy build`로 빌드해 번들 안을 `xxd`와 `strings`로 열어 봄. lynxjs.org 공식 문서 열 쪽과 블로그 두 편, GitHub 릴리스 목록.
- 알게 된 것: 위 핵심 정리.
- 근거: 아래 참고 자료. 번들 실측은 2026-10-17 로컬 빌드.
- 새로 생긴 궁금증: SEED `lynx-react` 패키지 규칙 문서가 「intrinsic tag는 리터럴 JSX로만 쓴다」 「CSS `inherit`은 지원 안 한다」를 불변식으로 두는 이유가 이 구조(snapshot 컴파일, opt-in 상속)로 설명되는지. SEED를 넣을 때 확인한다.

### 2026-10-17 저녁: 조사 문서가 어려워서 층을 하나씩 다시 물었다

- 계기: 홍 「lynx가 폰에서 작동하는 방법들에 대해서 너가 조사했는데 나에게는 너무 어려워서」. 위 핵심 정리와 아티팩트 둘 다 한 번에 층이 너무 많았다.
- 물어본 순서와 답 한 줄씩:
  1. Lynx는 React 바탕이냐: 아니다. 엔진은 C++이고 프레임워크와 무관하다. React처럼 쓰는 층은 ReactLynx이고 Preact 위에 만들었다.
  2. 그러면 ReactLynx가 C++이냐: 아니다. C++은 엔진이고 ReactLynx는 JavaScript다. 여기서 두 층이 섞였다. 「React와 관련 없다」도 아니고 「React 호환품(Preact) 위에 만들었다」가 맞다.
  3. `main.lynx.bundle`은 누가 만드나: Rspeedy. 웹의 Vite 자리다. 나는 `src/`에 쓰고, Rspeedy가 번들을 만들고, 엔진이 폰에서 그린다.
  4. Lynx Explorer는 뭐냐: 엔진을 미리 넣어 둔 개발용 빈 껍데기 앱. 개발 서버 주소를 QR로 받아 번들을 그린다. 배포에는 안 쓴다.
  5. 앱스토어에는 엔진을 어떻게 넣나: CocoaPods로 `Lynx`와 `PrimJS`를 받고, 앱 시작 때 `LynxEnv`를 깨우고, 번들을 읽어 주는 `LynxTemplateProvider` 클래스 하나를 쓰고, `LynxView`를 `addSubview`로 올린 뒤 `loadTemplateFromURL`을 부른다. 번들 파일은 Copy Bundle Resources에 넣는다. Explorer와 완성 앱의 차이는 번들을 어디서 읽나 하나뿐이다.
  6. 화면을 바꾸면 재배포냐 서버 갱신이냐: 둘 다 된다. 5번의 Provider가 앱 안 파일을 읽으면 재배포, 서버에서 받으면 서버 갱신이다. 「바이너리 안 바꾸고 갱신」은 Provider를 서버 쪽으로 쓸 때만 생긴다. 네이티브 코드 변경과 앱 성격을 바꾸는 갱신은 어느 쪽이든 재배포다.
  7. React Native와 뭐가 다르냐: 같은 점은 React 문법, 진짜 네이티브 뷰, JS 번들 + 네이티브 껍데기. 다른 점은 스타일(RN은 `StyleSheet.create` JS 객체, Lynx는 진짜 CSS 파일과 선택자), JS 스레드 수(하나 대 둘), 첫 화면(Lynx는 메인 스레드가 JS를 기다리지 않고 그린다). RN 비교는 에이전트의 정리이고 공식 문서의 비교가 아니다.
- 막힌 지점: 「C++」이 어느 층 이야기인지. 엔진과 ReactLynx를 한 덩어리로 들으면 「ReactLynx가 C++로 만든 React 비슷한 것」으로 뭉개진다. 표로 층을 나누자 풀렸다.
- 홍이 「여기까지만 알고 넘어가도 될까」로 닫았다. 지금 보험찾개냥 PR에 필요한 건 세 줄이다: React처럼 쓰되 태그는 `view`, `text`이고 이벤트는 `bindtap`이다. 스타일은 진짜 CSS 파일이되 `display` 기본이 `linear`이고 상속이 꺼져 있다. 화면은 `rspeedy dev` 뒤 Explorer로 본다. 나머지는 「그런 게 있다」까지만.
- 숙제: 다음 Explorer 세션에서 컴포넌트 함수 맨 위에 `console.log('render')`를 넣고 몇 번 찍힐지 예측한 뒤 확인한다. 위 10-17 항목의 예측 문제와 같다.
- 근거: 5번은 [Integrate with Existing Apps](https://lynxjs.org/guide/start/integrate-with-existing-apps.html) iOS 절을 다시 열어 확인했다(2026-10-17, Pod 이름과 메서드 이름). 나머지는 위 참고 자료. 출처: Claude 대화, 2026-10-17.

## 더 알아보면 좋은 것

- Lynx for Web이 같은 번들을 브라우저 안에서 어떻게 그리는가.
- `<list>` 자식은 UI가 늦게 만들어지는데 `useEffect`와 `ref`는 먼저 불리고 `main-thread:ref`만 보임 여부를 따른다는 규칙.

## 참고 자료

- [Introducing Lynx: Unlock Native for More](https://lynxjs.org/blog/lynx-unlock-native-for-more.html) — 2025-03-05 공개 블로그. 사용처, 3.x부터 공개한 이유, IFR과 MTS, PrimJS, 「대안 웹」 (2026-10-17 확인)
- [ReactLynx Introduction](https://lynxjs.org/react/introduction.html) — Preact 기반, DOM 없음, 요소와 이벤트 이름, snapshot 컴파일 (2026-10-17 확인)
- [Thinking in ReactLynx](https://lynxjs.org/react/thinking-in-reactlynx.html) — 두 스레드 실행 규칙, background only 세 규칙, 지시어 (2026-10-17 확인)
- [Main Thread Script](https://lynxjs.org/react/main-thread-script.html) — `'main thread'`, `main-thread:` 이벤트, `useMainThreadRef`, 캡처 값 제약 (2026-10-17 확인)
- [Rendering Process and Lifecycle](https://lynxjs.org/react/lifecycle.html) — 첫 렌더와 백그라운드 동기화, `useLayoutEffect` 미지원, list 자식 규칙 (2026-10-17 확인)
- [Elements and Components](https://lynxjs.org/guide/ui/elements-components.html) — 요소와 iOS, Android 네이티브 클래스 대응표 (2026-10-17 확인)
- [Understanding Layout](https://lynxjs.org/guide/ui/layout/index.html) — display 다섯 값, 요소 기반 레이아웃, border-box, 마진 겹침 없음 (2026-10-17 확인)
- [Styling with CSS](https://lynxjs.org/guide/ui/styling.html) — 캐스케이드, 커스텀 속성, opt-in 상속 (2026-10-17 확인)
- [JavaScript Runtime](https://lynxjs.org/guide/scripting-runtime/index.html) — 백그라운드 런타임의 플랫폼별 엔진과 ES 지원 범위 (2026-10-17 확인)
- [Main Thread Runtime](https://lynxjs.org/guide/scripting-runtime/main-thread-runtime.html) — PrimJS와 바이트코드 로드 (2026-10-17 확인)
- [Instant First-Frame Rendering](https://lynxjs.org/guide/interaction/ifr.html) — 동기 첫 프레임의 조건 (2026-10-17 확인)
- [Integrate with Existing Apps](https://lynxjs.org/guide/start/integrate-with-existing-apps.html) — LynxEnv, TemplateProvider, LynxView, 웹 `lynx-view` (2026-10-17 확인)
- [Terms and Spec 용어집](https://lynxjs.org/guide/glossary.html) — Engine, Framework, 메인 스레드와 UI 스레드의 구분, template이라는 옛 이름 (2026-10-17 확인)
- [Lynx 4.0 블로그](https://lynxjs.org/blog/lynx-4-0.html) — 2026-09-30 발표, 4.0에서 더해진 것 (2026-10-17 확인)
- [lynx-family/lynx 릴리스](https://github.com/lynx-family/lynx/releases) — 버전 연표의 근거 (2026-10-17 확인)
