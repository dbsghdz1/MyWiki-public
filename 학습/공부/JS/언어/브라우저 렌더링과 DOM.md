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

## 참고 자료
- MDN, [Populating the page: how browsers work](https://developer.mozilla.org/en-US/docs/Web/Performance/Guides/How_browsers_work) — 파싱 → DOM·CSSOM → 렌더 트리 → 레이아웃 → 페인트 (2026-09-26 확인)
- web.dev, [Constructing the Object Model](https://web.dev/articles/critical-rendering-path/constructing-the-object-model) — bytes → characters → tokens → nodes → DOM, CSS도 같은 과정으로 CSSOM (2026-09-26 확인)
