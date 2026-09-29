---
type: study
area: JS
audience: me
status: active
created: 2026-09-26
updated: 2026-09-29
aliases: [HTML 문서 구조, HTML 뼈대, viewport, 뷰포트, charset, 문자 인코딩, UTF-8, void element, 빈 요소, boolean attribute, HTML entity, quirks mode, defer]
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
- **템플릿**(`{{ pageTitle }}`)도 결국 서버가 빈칸을 채운 **HTML 문자열**을 보낸다 — 그래서 프레임워크가 써 줘도 charset·viewport·defer 같은 뼈대가 틀리면 그대로 사고가 난다. DOM은 그 문자열을 받은 브라우저가 만든다([브라우저 렌더링과 DOM](%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%20%EB%A0%8C%EB%8D%94%EB%A7%81%EA%B3%BC%20DOM.md)).

## 기록

### 2026-09-26 — roadmap.sh HTML pack 2강 「Anatomy of an HTML document」
- 원문 요약이 위 핵심 정리다. 직접 해 볼 것(레슨의 Try it): 위 뼈대를 `index.html`로 저장 → Elements 탭에서 `head`·`body` 중첩 확인 → `<title>` 바꾸면 탭 제목, `<h1>` 바꾸면 본문이 바뀌는지 → `lang` 지워도 페이지는 돌지만 접근성 신호가 사라짐 → `data-author="you"`를 붙이면 화면엔 변화 없고 DevTools에만 보임 → HTML 검증기에 넣어 오류 고치기.
- 원전과 대조: void 요소 목록은 HTML 표준에 13개로 정의돼 있다 — `area` `base` `br` `col` `embed` `hr` `img` `input` `link` `meta` `source` `track` `wbr`. 레슨은 그중 6개만 예로 든다.

### 2026-09-29 — 「밀과 오븐」 빵집 미션 중 `<meta charset>`
- 계기: 미션 `index.html`에 `<meat charset>` 오타 → 인코딩이 뭔지, 왜 `head`에 두는지 물었다. 붙여 준 블로그 글의 설명 「브라우저마다 인코딩 방식이 달라서 깨진다」는 반만 맞다.
- **인코딩** = 글자 ↔ 바이트 변환 규칙. 파일에는 바이트만 저장된다. 「밀과 오븐」을 UTF-8로 저장하면 `eb b0 80 ea b3 bc 20 ec 98 a4 eb b8 90`이고, 같은 바이트를 다른 규칙으로 읽으면 깨진다(Python으로 확인):
  - UTF-8 → `밀과 오븐` / windows-1252 → `ë°€ê³¼ ì˜¤ë¸�` / EUC-KR → `諛�怨� ��ㅻ��`
- 그래서 정확한 원인은 **저장한 규칙과 읽는 규칙이 다를 때**다. 선언이 없으면 브라우저는 추측하고, 그 추측이 브라우저·언어 설정마다 다를 뿐이다.
- 브라우저가 규칙을 정하는 우선순위: **BOM > HTTP `Content-Type` 헤더의 charset > `<meta charset>`**. 파일에 `utf-8`이라 써도 서버가 다른 charset을 보내면 서버가 이긴다.
- `<meta charset>`은 **파일 앞 1024바이트 안에** 끝나야 한다 — 브라우저는 앞부분만 훑어 규칙을 정한 뒤 본문을 읽기 때문이다. 그래서 `<head>`를 열자마자 첫 줄에 둔다. 블로그 예제처럼 `<title>` 뒤에 둬도 짧으면 동작하지만, `<title>`의 한글이 이미 규칙 없이 읽힐 수 있어 첫 줄이 안전하다.
- `meta`는 void 요소라 `</meta>`를 쓰면 **틀린 HTML**이다(MDN: 「must not have an end tag」). 블로그의 「안 써도 된다」보다 강하다.

### 2026-09-29 — 빵집 미션 1단계 검증에서 `user-scalable=no` 경고 → viewport가 뭔가
- 처음 생각: viewport = 휴대폰 화면. 스크롤도 되니까.
- 고쳐 잡은 것: viewport는 화면이 아니라 **문서를 들여다보는 창**이다. 문서는 긴 종이, viewport는 창틀이고 스크롤은 창 밑에서 종이를 미는 것이다. MDN: 「뷰포트 바깥의 콘텐츠는 스크롤하기 전엔 보이지 않는다」. 폰에선 주소창 등을 뺀 브라우저 영역이 창이다.
- 창은 둘이다. **레이아웃 뷰포트** = 페이지를 배치할 때 기준으로 쓰는 폭. `<meta name="viewport">`가 정하는 게 이것이다. **비주얼 뷰포트** = 지금 실제로 보이는 부분. 두 손가락으로 확대하면 비주얼만 작아지고 레이아웃은 그대로다.
- meta가 없으면 폰은 레이아웃 뷰포트를 980px 같은 가짜 폭으로 잡고 그린 뒤 화면에 맞게 축소한다 — 모바일을 고려 안 한 옛 페이지가 안 깨지게 하려는 호환 동작. `width=device-width`가 이걸 끈다.
- `user-scalable=no`는 비주얼 뷰포트 확대를 막는다 → 저시력 사용자가 못 읽는다. WCAG는 최소 2배 확대를 요구한다(MDN 경고). 그래서 검증기가 경고를 냈다.
- 확인 방법: 콘솔에서 `innerWidth`(레이아웃 폭)·`visualViewport.width`(보이는 폭)·`document.documentElement.scrollHeight`(문서 길이) vs `innerHeight`(창 높이). 확대하면 `visualViewport.width`만 줄어든다.

## 참고 자료
- roadmap.sh, [Anatomy of an HTML document](https://roadmap.sh/packs/html) — HTML pack 2강, 로그인 필요(본문은 홍이 붙여 준 원문으로 확인, 2026-09-26)
- MDN, [Basic HTML syntax](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Basic_HTML_syntax) — doctype·head/body·void 요소·boolean 속성·엔티티 (2026-09-26 확인)
- WHATWG HTML Living Standard, [Void elements](https://html.spec.whatwg.org/multipage/syntax.html#void-elements) — void 요소 13개의 정본 목록 (2026-09-26 확인)
- MDN, [`<meta>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta) — charset 선언은 앞 1024바이트 안, void 요소라 닫는 태그 금지 (2026-09-29 확인)
- W3C Internationalization, [Declaring character encodings in HTML](https://www.w3.org/International/questions/qa-html-encoding-declarations) — BOM > HTTP 헤더 > meta 우선순위, `head` 바로 뒤에 두라는 권고 (2026-09-29 확인)
- MDN, [Viewport(용어 사전)](https://developer.mozilla.org/ko/docs/Glossary/Viewport) — 레이아웃·비주얼 뷰포트 구분 (홍이 준 링크, 2026-09-29 확인)
- MDN, [Viewport meta tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Viewport_meta_element) — 980px 가상 뷰포트, `width=device-width` 권장, `user-scalable=no` 접근성 경고 (2026-09-29 확인)
