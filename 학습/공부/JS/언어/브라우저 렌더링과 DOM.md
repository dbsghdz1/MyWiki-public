---
type: study
area: JS
audience: me
status: active
created: 2026-09-26
updated: 2026-09-26
aliases: [DOM, DOM 트리, CSSOM, 렌더 트리, 중요 렌더링 경로, critical rendering path]
projects: []
---

# 브라우저 렌더링과 DOM

DOM 트리는 **서버가 아니라 브라우저가** 만든다. 서버는 HTML **텍스트**를 보낼 뿐이다.

## 핵심 정리

```
서버 ──HTML 텍스트──▶ 브라우저
                      HTML 파싱 → DOM 트리   (구조: 어떤 요소가 어떤 부모 아래 있나)
                      CSS  파싱 → CSSOM 트리 (스타일: 어떤 요소에 어떤 규칙이 붙나)
                      DOM + CSSOM → 렌더 트리 (화면에 보일 노드만 + 계산된 스타일)
                      → 레이아웃(위치·크기) → 페인트(픽셀)
```

- **DOM에 CSS는 안 들어간다.** DOM과 CSSOM은 따로 만든 독립된 트리이고, 합쳐지는 건 렌더 트리 단계다.
- **JS는 DOM을 만드는 재료가 아니라 DOM을 고치는 쪽이다.** `document.querySelector`·`appendChild`가 이미 만들어진 DOM을 바꾸고, 바뀌면 렌더 트리부터 다시 계산된다.
- `display: none`인 요소는 DOM에는 있지만 렌더 트리에는 없다. 구조(DOM)와 화면(렌더 트리)이 다르다는 증거.
- 헷갈린 지점: SSR에서 "서버가 HTML을 만든다"는 건 **완성된 HTML 문자열**을 만든다는 뜻이지 DOM을 만든다는 뜻이 아니다 — DOM은 그 문자열을 받은 브라우저가 파싱해서 만든다 ([[학습/공부/JS/React/렌더링 방식과 SEO|렌더링 방식과 SEO]]).

## 기록

### 2026-09-26 — "서버가 DOM을 만든다"고 생각했다
- 처음 이해: HTML을 만들면 서버가 HTML·CSS·JS를 고려해 DOM 트리를 만들고, DOM은 웹사이트 구조를 나타낸다.
- 고친 것: 구조를 나타낸다는 건 맞다. 만드는 주체는 브라우저이고, CSS는 CSSOM이라는 별도 트리로, JS는 만들어진 DOM을 조작하는 쪽으로 들어온다.
- 확인하는 법: 개발자 도구 Elements 탭은 **지금의 DOM**, 「페이지 소스 보기」는 **서버가 보낸 HTML 원문**이다. JS가 요소를 추가한 페이지에서 둘을 비교하면 다르다.

### 2026-09-26 — roadmap.sh 「What is HTML?」 요약
- **HTML은 마크업이지 프로그래밍 언어가 아니다** — 변수·반복·함수가 없다. 태그로 「이건 제목, 이건 문단, 이건 링크」라고 **라벨을 붙여 묘사**할 뿐이다. `hello.html`을 서버 없이 브라우저로 열어도 페이지가 나온다(컴파일도 서버도 없음).
- **HTML이 DOM이 된다** — 브라우저가 파일을 파싱해 트리로 만든다. 요소 하나 = 노드 하나, 중첩된 요소 = 중첩된 노드(`html` → `head`·`body` → `h1`·`p`).
- **모두가 다루는 건 HTML 원문이 아니라 DOM이다** — CSS는 DOM에서 요소를 찾아 스타일을 붙이고, JS는 DOM을 읽고 바꾸고, 스크린 리더·검색엔진·확장 프로그램도 DOM을 본다. *"HTML is the input, and the DOM is what the browser actually works with."*
- **DOM은 로드 뒤에도 바뀐다** — 초보가 가장 많이 걸리는 지점. 「페이지 소스 보기」 = 서버가 보낸 원문(안 변함), Elements 탭 = 지금의 DOM(JS가 돌 때마다 갱신). SPA에서는 소스가 거의 빈 껍데기이고 Elements에만 페이지가 차 있다.
- **서버의 몫은 HTML을 「만드는」 것까지** — 백엔드는 템플릿에 데이터를 섞어 HTML을 만들어 보낸다. DOM으로 바꾸는 건 그걸 받은 브라우저다. 처음에 「서버가 DOM을 만든다」로 읽은 건 이 두 단계를 하나로 합친 것이었다.
- 원문과 MDN은 어긋나지 않는다. 원문은 CSS가 「DOM을 써서」 스타일을 붙인다고만 하고, MDN은 그 사이에 CSSOM이라는 별도 트리가 있고 렌더 트리에서 합쳐진다고 한 단계 더 들어간다.

## 참고 자료
- roadmap.sh, [What is HTML?](https://roadmap.sh/packs/html/what-is-html) — HTML pack 첫 레슨, 로그인 필요(본문은 홍이 붙여 준 원문으로 확인, 2026-09-26)
- MDN, [Populating the page: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work) — 파싱 → DOM·CSSOM → 렌더 트리 → 레이아웃 → 페인트 (2026-09-26 확인)
- web.dev, [Constructing the Object Model](https://web.dev/articles/critical-rendering-path/constructing-the-object-model) — bytes → characters → tokens → nodes → DOM, CSS도 같은 과정으로 CSSOM (2026-09-26 확인)
