---
type: study
area: JS
audience: me
status: active
created: 2026-09-26
updated: 2026-09-29
aliases: [HTML 문서 구조, HTML 뼈대, strong, em, b vs strong, 하이퍼링크, target _blank, document fragment, 시맨틱 태그, semantic HTML, landmark, viewport, 뷰포트, charset, 문자 인코딩, UTF-8, void element, 빈 요소, boolean attribute, HTML entity, quirks mode, defer]
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
- 확인 방법: 콘솔에서 `innerWidth`(레이아웃 폭)·`visualViewport.width`(보이는 폭)·`document.documentElement.scrollHeight`(문서 길이) vs `innerHeight`(창 높이). **핀치 줌**(트랙패드 두 손가락)으로 확대하면 `visualViewport.width`만 줄어든다. **Cmd +**(브라우저 줌)는 다르다 — CSS 픽셀 자체를 키워서 레이아웃 뷰포트가 CSS px로 좁아지므로 `innerWidth`·`innerHeight`가 같이 준다(09-30 실험에서 `innerHeight`도 줄어 발견). DevTools를 아래에 붙여 열어도 창이 좁아져 `innerHeight`가 준다.

### 2026-09-30 — MDN `<meta name="viewport">` 레퍼런스 정리
- 표준 한 줄: `<meta name="viewport" content="width=device-width, initial-scale=1">`. `content`는 `이름=값`을 쉼표로 잇는다.
- `width` = 레이아웃 뷰포트 폭(창문 폭). `device-width`는 기기 폭, 숫자면 px(1~10000). 숫자를 주면 그 폭이 **최소 폭**처럼 동작하고, 화면이 더 넓으면 브라우저가 창문을 넓힌다(확대하지 않고).
- `initial-scale` = 처음 열 때 배율(기본 1, 0.1~10). `minimum-scale`·`maximum-scale` = 축소·확대 한계(기본 0.1·10).
- `user-scalable` = 사용자 확대 허용(기본 yes). `no`·낮은 `maximum-scale`은 저시력 사용자를 막는다 — WCAG 최소 2배, 권장 5배.
- `height`·`interactive-widget`(가상 키보드가 뜰 때 창문을 줄일지 덮을지, 실험적)는 거의 안 쓴다.
- 고밀도 화면: `initial-scale=1`이어도 브라우저가 CSS 1px을 물리 픽셀 여러 개로 그린다(300dpi 이상 ≈ 2배) — 그래서 CSS px ≠ 화면 픽셀.

### 2026-09-30 — MDN 「Structuring documents」 (빵집 미션 2단계 앞)
- 페이지는 대개 다섯 칸: 머리말 `header` · 메뉴 `nav` · 본문 `main` · 옆 정보 `aside` · 바닥글 `footer`. 머리말·메뉴·바닥글은 사이트 모든 페이지에서 같고, **`main`만 페이지마다 다르다**.
- `main` — 이 페이지에만 있는 내용. **페이지에 한 번**, `body` 바로 아래. `nav`는 **주요** 이동 링크만(부차 링크는 넣지 않는다).
- `article` — 떼어 내도 혼자 말이 되는 덩어리(블로그 글 하나). `section` — 페이지의 한 기능 부분, **제목으로 시작**. 둘은 서로 안에 들어갈 수 있다 — 맥락이 정한다.
- `aside` — 본문과 간접적으로만 관련된 것(용어 풀이·저자 소개·관련 링크). `header`·`footer`는 `body` 바로 아래면 페이지 전체용, `article`·`section` 안이면 그 절 전용.
- `div`·`span`은 **의미 없는 포장지** — 맞는 시맨틱 태그가 없을 때만. 화면은 같아도 스크린 리더가 「본문으로 가기」「메뉴 찾기」를 태그로 한다(시각장애 인구 4~5%).
- `br` = 주소·시처럼 줄을 강제로 끊어야 할 때만, 닫는 태그 없음(`</br>`은 에러). `hr` = 주제가 바뀌는 지점.

### 2026-09-30 — MDN 「Creating links」 (빵집 미션 3단계)
- 계기: 인스타그램 링크를 `href="www.instagram.com"`로 써서 `localhost:5500/www.instagram.com`으로 갔다 — 스킴(`https://`)이 없으면 브라우저는 **상대 경로(내 사이트 안 파일 이름)**로 읽는다([절대 URL과 상대 URL](../../CS/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/%EC%A0%88%EB%8C%80%20URL%EA%B3%BC%20%EC%83%81%EB%8C%80%20URL.md)). 외부 링크는 항상 전체 URL.
- 경로: 같은 폴더 `contacts.html`(=`./contacts.html`) · 하위 `projects/index.html` · 상위 `../` · 사이트 루트 기준 `/pdfs/a.pdf`(파일을 옮겨도 안 깨진다). 내부 링크는 도메인 없이 쓰는 게 권장(이식성).
- 조각 링크: 대상에 `id`, 링크는 `#id`(같은 문서) 또는 `other.html#id`.
- 링크 텍스트: 「여기를 클릭」 금지 — 스크린 리더는 링크만 모아 읽고, 검색엔진은 링크 텍스트로 대상을 색인한다. URL·「링크」라는 말을 텍스트에 넣지 않는다.
- 새 탭 `target="_blank"`: 뒤로 가기가 안 먹어 헷갈리고, 스크린 리더 사용자는 새 탭이 열린 걸 모를 수 있다 → 흔한 관례는 **외부 링크만 새 탭**, 그리고 아이콘·「(새 탭)」 같은 **표시를 붙인다**. `title`은 마우스 호버에만 보여 키보드·터치 사용자는 못 본다.
- 기타: 다운로드 `download` 속성, 메일 `mailto:주소?subject=…&body=…`(값은 URL 인코딩).
- **한국어판과 차이** (09-30 비교): 한국어 번역에는 「새 탭은 언제 여나」 절이 없고 `target="_blank"`는 동영상 예제에만 스치듯 나온다. 영어판에는 루트 기준 경로 `/`, URL이 서버 파일 경로로 바뀌는 과정(`index.html` 기본 페이지·끝 `/`의 의미)도 있다. **MDN은 영어판이 원본**이다.

### 2026-09-30 — MDN 「Emphasis and importance」 (빵집 미션 4단계 ① 앞)
- `em` = **말할 때 힘주는 강조**. 힘준 단어가 바뀌면 뜻이 바뀐다(「I am *glad* you weren't *late*」 → 비꼬기). 기본 모양은 기울임.
- `strong` = **중요함**(「highly toxic」「Do not be late!」). 기본 모양은 굵게. 둘 다 스크린 리더가 인식하고 억양을 바꿔 읽도록 설정할 수 있다. 서로 중첩 가능.
- `i`·`b`·`u`는 HTML5에서 **모양이 아니라 관례적 의미**로 다시 정의됐다: `i` = 외국어·학명·기술 용어·속생각, `b` = 키워드·제품명·첫 문장, `u` = 고유명사·맞춤법 오류 표시. MDN: 더 알맞은 요소가 없을 때만 — 「대개 있다」.
- 밑줄은 링크로 오해받으니 웹에선 링크에만. `big`·`font`는 모양만 바꾸는 폐기 요소.
- 그래서 H3 질문의 답: `b`와 `strong`은 화면이 같아도 **의미가 다르다** — 알레르기 경고처럼 「중요하다」면 `strong`, 그냥 눈에 띄게 할 단어면 `b`.

## 참고 자료
- roadmap.sh, [Anatomy of an HTML document](https://roadmap.sh/packs/html) — HTML pack 2강, 로그인 필요(본문은 홍이 붙여 준 원문으로 확인, 2026-09-26)
- MDN, [Basic HTML syntax](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Basic_HTML_syntax) — doctype·head/body·void 요소·boolean 속성·엔티티 (2026-09-26 확인)
- WHATWG HTML Living Standard, [Void elements](https://html.spec.whatwg.org/multipage/syntax.html#void-elements) — void 요소 13개의 정본 목록 (2026-09-26 확인)
- MDN, [`<meta>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/meta) — charset 선언은 앞 1024바이트 안, void 요소라 닫는 태그 금지 (2026-09-29 확인)
- W3C Internationalization, [Declaring character encodings in HTML](https://www.w3.org/International/questions/qa-html-encoding-declarations) — BOM > HTTP 헤더 > meta 우선순위, `head` 바로 뒤에 두라는 권고 (2026-09-29 확인)
- MDN, [Viewport(용어 사전)](https://developer.mozilla.org/ko/docs/Glossary/Viewport) — 레이아웃·비주얼 뷰포트 구분 (홍이 준 링크, 2026-09-29 확인)
- MDN, [Viewport meta tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Guides/Viewport_meta_element) — 980px 가상 뷰포트, `width=device-width` 권장, `user-scalable=no` 접근성 경고 (2026-09-29 확인)
- MDN, [`<meta name="viewport">`](https://developer.mozilla.org/ko/docs/Web/HTML/Reference/Elements/meta/name/viewport) — content 값 목록·범위·기본값, 접근성 경고 (홍이 준 링크, 2026-09-30 확인)
- MDN, [Structuring documents](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Structuring_documents) — 다섯 칸과 시맨틱 태그, div/span, br/hr (홍이 준 링크, 2026-09-30 확인)
- MDN, [Creating links](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Creating_links) — 경로·조각 링크·링크 텍스트·새 탭·mailto (홍이 준 링크, 2026-09-30 확인). [한국어판](https://developer.mozilla.org/ko/docs/Learn_web_development/Core/Structuring_content/Creating_links)은 새 탭 절 등이 빠진 번역 (2026-09-30 확인)
- MDN, [Emphasis and importance](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Emphasis_and_importance) — em·strong, i·b·u의 HTML5 의미 (홍이 준 링크, 2026-09-30 확인)
