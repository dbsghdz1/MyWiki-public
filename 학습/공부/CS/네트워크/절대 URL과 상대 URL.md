---
type: study
area: CS
audience: me
status: active
created: 2026-09-22
updated: 2026-09-22
aliases: [상대 URL, 절대 URL, URL 해석, base URL]
---

# 절대 URL과 상대 URL

**절대/상대를 가르는 건 `/`가 아니라 스킴(`https:`)이 있느냐다.** `/`는 상대 URL 안에서 *어디서부터 채울지*를 정할 뿐이다.

## 핵심 정리

### 판정 기준

- **절대 URL** = 스킴부터 다 있어서 현재 페이지가 무엇이든 혼자서 목적지가 정해진다. `https://example.com/about`
- **상대 URL** = 빠진 부분이 있어서 **현재 페이지(base URL)로 빈칸을 채워야** 목적지가 나온다. 빈칸이 어디까지냐로 세 종류.

| 모양 | 빠진 것 | 현재 페이지에서 가져오는 것 | RFC 3986 이름 |
|---|---|---|---|
| `//cdn.example.com/a.js` | 스킴 | 스킴만 | network-path reference |
| `/about` | 스킴 + 호스트 | 스킴·호스트, 경로는 **루트부터** 새로 | absolute-path reference |
| `about`, `../about` | 스킴 + 호스트 + 경로 앞부분 | 스킴·호스트 + **현재 문서가 있는 폴더** | relative-path reference |

RFC 3986 §4.2는 셋을 전부 **relative reference**로 묶는다. `/about`의 이름이 "absolute-*path*"인 것이 혼동의 씨앗이다 — *경로*가 절대일 뿐 *URL*은 상대다.

### 실제로 돌려 보면

Node의 `new URL(상대, base)`가 브라우저와 같은 규칙(WHATWG URL Standard)으로 푼다. base = `https://example.com/blog/2026/post.html`:

```
https://other.com/about    -> https://other.com/about        절대: base 무시
//cdn.example.com/a.js     -> https://cdn.example.com/a.js   스킴만 빌림
/about                     -> https://example.com/about      루트부터
about                      -> https://example.com/blog/2026/about   현재 폴더부터
../about                   -> https://example.com/blog/about        한 단계 위
?q=1                       -> https://example.com/blog/2026/post.html?q=1
#top                       -> https://example.com/blog/2026/post.html#top
```

**"현재 폴더" = 마지막 `/` 뒤를 잘라낸 것.** 그래서 끝 슬래시 하나로 결과가 바뀐다:

```
new URL("about", "https://example.com/blog/2026/")  -> https://example.com/blog/2026/about
new URL("about", "https://example.com/blog/2026")   -> https://example.com/blog/about
```

두 번째는 `2026`이 파일 이름으로 취급돼 잘려 나갔다. 앱에서 `href="about"`이 페이지에 따라 다른 곳을 가리키는 버그가 이것이다. 그래서 HTML 링크는 대개 `/about`(루트 기준)을 쓴다.

### `//`로 시작하는 건 왜 쓰나 — 옛 혼합 콘텐츠 대책, 지금은 안티패턴

`scheme://host/path`에서 `//`는 "다음에 호스트가 온다"는 표시다. 스킴만 떼고 `//host/path`를 쓰면 **스킴을 현재 페이지에서 빌린다.** 그래서 protocol-relative(scheme-relative) URL이라 부른다.

쓰인 이유는 **혼합 콘텐츠(mixed content)**다. `https://` 페이지가 `http://cdn…/a.js`를 넣으면 브라우저가 경고를 띄우거나 스크립트를 **차단**한다(MDN: 스크립트는 "blockable"). 한 사이트가 http로도 https로도 열리던 시절, HTML 하나를 양쪽에서 쓰려고 `<script src="//cdn…/a.js">`로 적었다. 페이지가 http면 http로, https면 https로 따라간다.

```
https://example.com/page      + //cdn.example.com/a.js -> https://cdn.example.com/a.js
http://example.com/page       + //cdn.example.com/a.js -> http://cdn.example.com/a.js
file:///Desktop/index.html    + //cdn.example.com/a.js -> file://cdn.example.com/a.js   ← 로컬에서 열면 깨진다
```

**지금은 쓰지 않는다.** Paul Irish가 2010년에 이 기법을 퍼뜨렸다가 2014년에 스스로 "안티패턴"이라고 고쳤다 — http 페이지에서는 자산도 http로 내려와 중간자 공격에 열리고, 반대로 http 페이지에서 `https://` 자산을 부르는 건 아무 문제가 없다. 그러니 그냥 `https://`를 박는다. 세 번째 줄처럼 HTML 파일을 더블클릭해 `file://`로 열면 `file://cdn.example.com/…`을 찾다 실패하는 것도 `//`의 부작용이다.

### 한국어 MDN이 헷갈리게 쓴 부분

내가 읽은 [한국어 MDN](https://developer.mozilla.org/ko/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL)은 `//…`와 `/ko/docs/Learn`을 **"절대 URL"** 항목 아래에 "암시적 프로토콜·암시적 도메인 이름"으로 넣고, `/` 없는 것만 "상대 URL"이라 부른다. 그래서 "`/` 유무 = 절대/상대"로 읽힌다. 현행 영어 원문은 같은 예를 **scheme-relative URL · domain-relative URL**이라 부르며 상대 URL로 분류하고, RFC 3986·WHATWG도 그쪽이다. 한국어 번역이 옛 판이다.

## 기록

### 2026-09-22 — "앞에 `/` 있냐 없냐 차이야?"

- 맥락: MDN «What is a URL» 한국어판을 읽다가. 예측은 *"`/`가 있으면 절대, 없으면 상대"*
- 파고든 길: 영어 원문 → RFC 3986 §4.2 · §5.4.1 예제표 → WHATWG URL 파서의 `base` 인자 → Node `new URL(r, base)`로 7가지 모양을 실제로 풀어 봄
- 결론: 예측이 틀렸다. 기준은 **스킴 유무**이고, `/`는 상대 URL의 시작점(루트 vs 현재 폴더)을 정한다. 혼동은 한국어 MDN의 옛 분류에서 왔다
- 새로 생긴 궁금증: 브라우저는 `<base href>`가 있으면 현재 문서 대신 그걸 base로 쓴다는데, SPA 라우터가 `/user/1`에서 새로고침할 때 자산 경로가 깨지는 문제가 이것과 같은 원인인가?

### 2026-09-22 — "그럼 `//`로 시작하는 건 왜 쓰는거야"

- 맥락: 위 항목 바로 다음 질문
- 파고든 길: Node로 https·http·`file:` base 세 가지에 `//cdn…`을 풀어 봄 → MDN Mixed content(스크립트는 차단) → Paul Irish 2010 글의 2014년 정정
- 결론: 한 HTML을 http·https 양쪽에서 쓰기 위한 옛 혼합 콘텐츠 대책. HTTPS가 기본인 지금은 `https://`를 명시하는 게 맞고, `file://`로 열면 깨지는 부작용까지 있다

관련: [[학습/공부/CS/네트워크/URL과 퍼센트 인코딩|URL과 퍼센트 인코딩]] — 같은 URL이지만 이쪽은 값을 어떻게 싣느냐

## 참고 자료

- [MDN — What is a URL? (영어 원문)](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL) — scheme-relative · domain-relative · sub-resource 세 예제, 상대 URL로 분류 (2026-09-22 확인)
- [MDN — URL이란? (한국어)](https://developer.mozilla.org/ko/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_URL) — 내가 읽은 판. `/…`를 절대 URL로 분류한 옛 번역 (2026-09-22 확인)
- [RFC 3986 §4.2 Relative Reference](https://datatracker.ietf.org/doc/html/rfc3986#section-4.2) — network-path / absolute-path / relative-path 정의. §5.4.1의 base `http://a/b/c/d;p?q` 예제표가 한눈에 보는 정답지 (2026-09-22 확인)
- [WHATWG URL Standard](https://url.spec.whatwg.org/#urls) — 브라우저·Node `new URL()`이 실제로 따르는 규격. 파서가 `base`를 받아 상대 문자열을 푼다 (2026-09-22 확인)
- [Paul Irish — The Protocol-relative URL (2010, 2014 정정)](https://www.paulirish.com/2010/the-protocol-relative-url/) — 기법을 퍼뜨린 글. 2014년 업데이트에서 "this technique is now an anti-pattern"이라고 스스로 고침 (2026-09-22 확인)
- [MDN — Mixed content](https://developer.mozilla.org/en-US/docs/Web/Security/Mixed_content) — https 페이지의 http 자산 처리. 이미지 등은 upgradable, 스크립트는 blockable로 차단 (2026-09-22 확인)
