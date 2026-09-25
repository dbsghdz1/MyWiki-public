---
type: study
area: JS
audience: me
status: active
created: 2026-09-26
updated: 2026-09-26
aliases: [HTML 문서 구조, HTML 뼈대, void element, 빈 요소, boolean attribute, HTML entity, quirks mode, defer]
projects: []
---

# HTML 문서 구조

거의 모든 페이지가 같은 뼈대를 갖는다. 줄마다 맡은 일이 있다.

## 핵심 정리

```html
<!DOCTYPE html>                 <!-- 선언(태그 아님). 없으면 quirks mode(옛 호환 동작) -->
<html lang="en">                <!-- 루트. lang = 스크린 리더·검색엔진·번역기용 언어 신호 -->
  <head>                        <!-- 페이지에 대한 정보. 사용자에게 안 보인다 -->
    <meta charset="UTF-8">      <!-- 없으면 André·Müller·한글이 깨진다 -->
    <meta name="viewport" content="width=device-width, initial-scale=1">  <!-- 없으면 폰이 데스크톱 폭으로 그리고 축소 -->
    <title>My first page</title> <!-- 탭 제목·검색 결과 제목 -->
    <link rel="stylesheet" href="style.css">
    <script src="app.js" defer></script>  <!-- defer: 파싱 계속, 문서 완성 뒤 실행 -->
  </head>
  <body>…</body>                <!-- 사용자가 보고 누르는 모든 것. head와 형제 -->
</html>
```

- **태그 ≠ 요소** — `<a>`(여는 태그) + `Click here`(내용) + `</a>`(닫는 태그) = 요소 하나. 태그는 표시일 뿐이다.
- **속성** = 여는 태그 안의 `name="value"`. **boolean 속성**은 이름만 쓰면 켜진다(`<button disabled>`).
- **자주 쓰는 속성 셋**: `id`(페이지에 하나 — CSS·JS·`#pricing` 앵커가 찾는 고유 이름) · `class`(여럿 공유, 한 요소에 공백으로 여러 개) · `data-*`(내 데이터 — 브라우저는 무시, JS가 읽고 CSS가 선택 가능).
- **void 요소** — 감쌀 내용이 없어 닫는 태그가 없다: `img` `br` `hr` `input` `meta` `link` 등. `<br />`은 옛 표기, HTML에선 `<br>`이 정석.
- **중첩은 괄호처럼** 연 순서의 역순으로 닫는다. `<p><strong>…</p></strong>`은 깨진 것 — 브라우저가 알아서 고치려 하지만 결과를 믿으면 안 된다.
- **엔티티** — 마크업으로 읽힐 글자를 글자로 보여 줄 때: `&lt;` `&gt;` `&amp;` `&copy;`. `<meta>`라고 쓰면 브라우저가 태그로 파싱해 버린다 → `&lt;meta&gt;`.
- **주석** `<!-- … -->` — 브라우저가 무시, 여러 줄 가능.
- **템플릿**(`{{ pageTitle }}`)도 결국 서버가 빈칸을 채운 **HTML 문자열**을 보낸다 — 그래서 프레임워크가 써 줘도 charset·viewport·defer 같은 뼈대가 틀리면 그대로 사고가 난다. DOM은 그 문자열을 받은 브라우저가 만든다([[학습/공부/JS/언어/브라우저 렌더링과 DOM|브라우저 렌더링과 DOM]]).

## 기록

### 2026-09-26 — roadmap.sh HTML pack 2강 「Anatomy of an HTML document」
- 원문 요약이 위 핵심 정리다. 직접 해 볼 것(레슨의 Try it): 위 뼈대를 `index.html`로 저장 → Elements 탭에서 `head`·`body` 중첩 확인 → `<title>` 바꾸면 탭 제목, `<h1>` 바꾸면 본문이 바뀌는지 → `lang` 지워도 페이지는 돌지만 접근성 신호가 사라짐 → `data-author="you"`를 붙이면 화면엔 변화 없고 DevTools에만 보임 → HTML 검증기에 넣어 오류 고치기.
- 원전과 대조: void 요소 목록은 HTML 표준에 13개로 정의돼 있다 — `area` `base` `br` `col` `embed` `hr` `img` `input` `link` `meta` `source` `track` `wbr`. 레슨은 그중 6개만 예로 든다.

## 참고 자료
- roadmap.sh, [Anatomy of an HTML document](https://roadmap.sh/packs/html) — HTML pack 2강, 로그인 필요(본문은 홍이 붙여 준 원문으로 확인, 2026-09-26)
- MDN, [Basic HTML syntax](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Basic_HTML_syntax) — doctype·head/body·void 요소·boolean 속성·엔티티 (2026-09-26 확인)
- WHATWG HTML Living Standard, [Void elements](https://html.spec.whatwg.org/multipage/syntax.html#void-elements) — void 요소 13개의 정본 목록 (2026-09-26 확인)
