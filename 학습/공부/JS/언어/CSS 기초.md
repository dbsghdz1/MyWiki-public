---
type: study
area: JS
audience: me
status: active
created: 2026-10-02
updated: 2026-10-02
aliases: [CSS 기초, 캐스케이드, 명시도, specificity, 박스 모델, box model, Flexbox, Grid, position absolute, CSS 변수, 미디어 쿼리, 컨테이너 쿼리]
projects: []
---

# CSS 기초

roadmap.sh CSS 로드맵을 「밀과 오븐」 빵집 페이지(HTML 미션 결과물)에 `style.css`를 입히며 C1~C14로 밟았다 (2026-10-01~02). 세션 목록은 [JS/언어 학습 계획](%ED%95%99%EC%8A%B5%20%EA%B3%84%ED%9A%8D.md). HTML 쪽은 [HTML 문서 구조](HTML%20%EB%AC%B8%EC%84%9C%20%EA%B5%AC%EC%A1%B0.md).

## 핵심 정리

**규칙이 충돌하면** — `!important` > 인라인 `style` > **명시도**(id 100 · class/속성 10 · 태그 1, 왼쪽 칸부터 비교) > **소스 순서**(같은 힘이면 나중 것). 외부 `.css`냐 `<style>`이냐는 힘이 같다 — 갈리는 건 **파일이 도착한 시간이 아니라 HTML에 쓰인 위치**다. 그래서 `@media` 같은 예외는 **파일 맨 아래**에 둔다.

**상속** — 글자 속성(`color`·`font-*`·`line-height`)은 내려가고 박스 속성(`margin`·`padding`·`border`·`background`)은 안 내려간다. 폼 요소는 브라우저가 글꼴을 직접 지정해 상속이 끊긴다 → `font: inherit`.

**틀린 값은 조용히 버려진다** — `font-size: 40`(단위 없음), `border-radius: 4`, `linear-gradient()`(빈 괄호), `outline: nones`. 에러 없이 그 줄만 무시된다. "왜 안 먹지" → Styles 탭의 **노란 삼각형**부터 본다.

**박스 모델** — content → padding → border → margin. 기본(`content-box`)은 `width`가 내용만이라 `200 + 20·2 + 2·2 = 244px`. `box-sizing: border-box`면 `width`가 테두리까지 전체(내용 156). 디자인 수치와 맞으려면 `*, *::before, *::after { box-sizing: border-box }`를 맨 위에.

**block / inline / inline-block** — 규칙 하나: **inline은 세로 방향이 안 먹는다**(`width`·`height`·위아래 `margin` 무시, 위아래 `padding`은 칠해지기만 하고 주변을 못 밀어 겹침). inline-block = 세로도 먹는 inline. Flex·Grid 안에서는 자식이 뭐든 다 먹어서 이 고민이 사라진다.

**단위** — `px` 고정 · `em` 부모 글자 기준 · `rem` `<html>` 기준(16px) · `%` 대부분 부모 폭 · `vw`/`vh` 창문 1%. 글자는 `rem` — 브라우저 글꼴 크기를 키운 사용자에게 `px`는 반응하지 않는다. `line-height: 1.6`·`opacity`는 단위 없는 게 정상.

**Flexbox**(한 줄) — 부모에 `display: flex`, 효과는 자식에. `justify-content` = **늘어선 방향(주축)**, `align-items` = **수직 방향(교차축)**. `flex-direction: column`이면 둘이 뒤바뀐다. `gap`은 사이에만. `flex: 1`로 남는 공간 나눔.

**Grid**(두 방향) — `grid-template-areas`로 배치를 글자로 그리고, 각 요소에 `grid-area: 이름`. `fr`은 남는 공간의 몫. `repeat(auto-fit, minmax(200px, 1fr))` 한 줄이 미디어 쿼리 없는 반응형. HTML을 안 건드리고 `body` 하나에서 header·nav·main·aside·footer를 배치했다 — HTML 단계에서 시맨틱 다섯 칸을 세워 둔 덕.

**position** — `absolute`는 포스트잇: **`static`이 아닌 가장 가까운 조상**에 붙고, 없으면 페이지 전체. 부모에 `position: relative`(offset 없이)를 주는 이유 = 기준 상자 만들기. `fixed`는 창문 기준, `sticky`는 흐름대로 있다가 `top: 0`에 닿으면 붙음. `z-index`는 같은 쌓임 문맥 안에서만 비교 — 자식은 부모 문맥을 못 벗어난다(`transform`·`opacity<1`도 문맥을 만든다).

**가상 클래스/요소** — `:hover`·`:focus-visible`·`:nth-child(even)`은 상태·위치로 고르기, `::before`/`::after`는 `content` 필수인 가짜 요소. `::after` 글자는 스크린 리더가 못 읽을 수 있어 장식만. `outline: none`은 키보드 사용자의 현재 위치를 지운다 — 지우려면 대체 표시.

**CSS 변수** — `:root { --color-ink: … }` + `var(--color-ink)`. **상속되는 속성**이라 `aside { --color-ink: red }`면 그 안만 바뀐다. Sass 변수는 빌드 때 한 번 치환, CSS 변수는 **그릴 때 요소마다** 정해진다 → 영역별 덮어쓰기·다크 모드·JS 변경이 된다. `clamp(최소, 보통, 최대)`.

**반응형** — `@media (max-width: 600px)`는 **창문** 폭, `@container`는 **지정한 조상(`container-type: inline-size`)** 폭. 미디어 쿼리가 안 먹으면 ① 소스 순서(기본 규칙보다 아래인가) ② `grid-template-areas`도 같이 바꿨나 ③ viewport 메타가 있나.

**움직임** — `transform`은 레이아웃 안 건드리고 그림만 옮김. `transition`은 **기본 규칙에** 둬야 올릴 때·뗄 때 둘 다 부드럽다(`:hover`에 두면 뗄 때 뚝). `@keyframes` + `animation`은 스스로 반복. `@media (prefers-reduced-motion: reduce)`에서 끈다.

## 기록

### 2026-10-01~02 — 빵집 `style.css` C1~C14 (예측이 틀렸던 것 위주)
- C1 예측 「로드되는 순서: `<style>` → css → 인라인」 → 반만 맞음. 인라인이 이기는 건 맞지만, `<link>`와 `<style>`은 **쓰인 순서**로 갈린다. 실험 중 `href="style.csss"` 오타로 파랑 규칙이 아예 안 실려 있었다 — Network 탭 404가 첫 확인 지점.
- C3 「나중 규칙이 지는 경우」 = id·더 자세한 선택자·`!important`·인라인.
- C4 `font-size: 40` 단위 누락 → 적용 안 됨. 같은 실수 C6 `border-radius: 4`, C11 `outline: nones`.
- C6 「`width: 200px`면 200」 예측 → 244. `border-box` 추가 → 200, 내용 156.
- C7 inline/inline-block/margin이 언제 먹는지 혼란 → 「inline은 세로가 안 먹는다」 한 줄로 정리. `from label` 오타로 라벨만 block이 안 됐다.
- C8 `align-items: center`를 가로 가운데로 착각 → 세로. `justify-content`가 가로(주축).
- C9 「flex로 2:1이 불편하다」 → 정확히는 header·footer가 두 열을 가로질러야 해서 감싸는 `div`가 더 필요해진다는 것. grid는 `body` 하나로 끝.
- C10 「static이 아닌 가장 가까운 조상」을 트리로 위로 거슬러 올라가며 설명 → 이해.
- C13 `@media`를 파일 위에 둬서 `li` `width: 200px`이 이김 → 맨 아래로 이동. `grid-template-columns: 1fr`만 바꾸고 areas를 안 바꾸면 두 번째 열이 암시적으로 생겨 여전히 나란히.
- C12·C14는 홍 요청으로 내가 대신 작성(변수화·`clamp`·hover 떠오름·배지 pulse·reduced-motion).
- 학습 방식: 「무엇을 하고 싶은가 → 그걸 하는 속성 하나 → 안 되면 그때 이유」. 외우지 않는다. 폼(HTML 5단계)과 C12 이후에 「굳이 알아야 하나」 신호가 와서 핵심만 코드로 받고 넘어갔다 — 지루함 쪽 신호에 맞는 처치.

## 더 알아보면 좋은 것
- 전역 CSS가 커질 때 깨지는 것(이름 충돌·`ul { display: flex }` 같은 광역 규칙)과 CSS Modules(파일 단위로 class 이름을 고유하게 바꿈)·CSS-in-JS(컴포넌트 안에 스타일을 두고 런타임/빌드에 주입)가 그걸 막는 방식 — React로 넘어갈 때.
- Sass·PostCSS·BEM — 로드맵 마지막 노드, 이름만 들어 둠.
- `@property`로 등록한 변수의 애니메이션.

## 참고 자료
- roadmap.sh, [CSS Roadmap](https://roadmap.sh/css) — 주제 33개·하위 71개 (2026-10-01 확인)
- MDN, [CSS styling basics](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Getting_started) — Getting started · Basic selectors · Combinators · Attribute selectors · Handling conflicts · Values and units · Sizing · Box model · Backgrounds and borders · Images, media, form elements · Tables · Pseudo-classes and pseudo-elements (2026-10-01 URL 확인, 10-01~02 본문 확인)
- MDN, [Text styling](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Text_styling/Fundamentals) — Fundamentals · Web fonts · Styling lists · Styling links (2026-10-01 확인)
- MDN, [CSS layout](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Introduction) — Introduction · Flexbox · Grids · Floats · Positioning · Responsive design · Media queries (2026-10-01 확인)
- MDN, [Stacking context](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Positioned_layout/Stacking_context) · [Using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascading_variables/Using_custom_properties) · [Container queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Containment/Container_queries) · [Using CSS transitions](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Transitions/Using) · [Using CSS transforms](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Transforms/Using) · [Using CSS animations](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Animations/Using) (2026-10-01 확인)
