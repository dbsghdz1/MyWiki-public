---
type: study
area: JS
audience: me
status: active
created: 2026-09-08
updated: 2026-09-08
projects:
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# 비동기와 Promise

**`fetch`는 결과를 주지 않는다. "나중에 결과가 담길 상자"를 즉시 준다.** 그 상자가 `Promise`고, 비동기의 전부가 여기서 나온다.

## 핵심 정리

### 왜 즉시 안 주나

네트워크 왕복은 수백 ms다. 그동안 브라우저가 멈춰 서면 화면이 통째로 얼어붙는다. 그래서 요청만 띄우고 **즉시 반환**한 뒤, 응답이 오면 상자를 채운다.

콘솔에서 `fetch(url)`만 치면 상태 전이가 그대로 보인다:

```
Promise {<pending>}          ← 엔터 친 순간
  PromiseState  : "fulfilled"   ← 펼쳐볼 때쯤엔 채워져 있다
  PromiseResult : Response
```

(크롬 콘솔은 이 둘을 **내부 슬롯**이라 대괄호를 두 겹 씌워 보여준다 — JS 코드로는 못 읽는 값이라는 표시다.)

### 두 번 기다린다

```js
fetch(url)                 // ① 응답 헤더가 오면 fulfilled → Response
  .then((r) => r.text())   // ② 본문을 다 읽으면 fulfilled → 문자열
  .then((t) => /* 여기서 t가 진짜 글자 */);
```

첫 상자에 든 건 **XML이 아니라 `Response` 객체**다. `status: 200`, `ok: true`는 왔는데 `bodyUsed: false`, `body: (...)` — **본문은 아직 안 읽었다.** 헤더는 먼저 오고 본문은 스트리밍으로 뒤따르니, 본문을 꺼내는 `.text()`/`.json()`도 Promise를 돌려준다.

`.then(fn)`은 *"상자가 채워지면 fn을 실행해라"*.

### 한 줄이 두 시점에 나뉘어 실행된다

```js
fetch(location.href).then(r => r.text()).then(t => console.log(t.slice(0, 200)))
```
콘솔에 **두 개가 순서대로** 찍힌다.
1. `Promise {<pending>}` — 표현식의 반환값, 즉시
2. `<?xml version...` — 수백 ms 뒤, 응답이 도착해 `.then`이 실행된 것

**비동기가 뭔지 콘솔이 시간순으로 보여주는 장면.**

### `fetch`는 403·404에 실패하지 않는다

MDN이 명시한다 — *"if the server responds with an error like 404, then `fetch()` fulfills with a `Response`"*. **거부(reject)되는 건 네트워크 자체가 끊겼을 때뿐**이다. 그래서 `r.ok` 또는 `r.status`를 **직접 봐야 한다.**

공공 API에서 특히 아프다: 키가 틀리면 `403` + 에러 XML이 오는데, 체크가 없으면 그 에러 XML이 **정상 데이터인 척** 파싱 단계로 넘어간다. → [[작업노트/도구/공공데이터포털 오픈API|공공데이터포털 오픈API]]

## 기록

### 2026-09-08 — 콘솔에서 fetch를 처음 쳐봤다 (야생학습 사다리 세션 1)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[학습/야생학습/약국맵 사다리 1 — fetch와 useState 2026-09-08|사다리 세션 1]]. React 붙이기 전에 콘솔에서 `fetch`만 격리해서 확인
- 배운 것:
  - 예측은 "XML 글자가 찍힌다"였는데 **`Promise {<pending>}`이 찍혔다.** 예측 누락 — 그리고 이게 이 개념의 전부였다
  - `PromiseResult`에 `Response`가 들어 있고 `bodyUsed: false`인 것을 눈으로 봄 → **응답 ≠ 본문**
  - `.then` 두 번이 왜 필요한지가 여기서 풀렸다
- 근거: 브라우저 콘솔 세션, `pharmacy-map` 커밋 `3c81de8`

## 참고 자료

- [MDN — Using the Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) — Promise 반환, **404에도 reject되지 않으므로 `response.ok`를 직접 확인해야 한다** (2026-09-08 확인)
