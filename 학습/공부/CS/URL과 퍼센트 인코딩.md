---
type: study
area: CS
audience: me
status: active
created: 2026-09-08
updated: 2026-09-08
projects:
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# URL과 퍼센트 인코딩

**인코딩은 정확히 한 번이어야 한다.** 두 번 하면 값이 조용히 망가지고, 서버는 "그런 값 없다"고만 답한다.

## 핵심 정리

### 퍼센트 인코딩이 뭔가

URL은 ASCII만 실을 수 있게 설계됐다. 한글·공백처럼 못 싣는 글자, 그리고 `?` `&` `=` `/`처럼 **URL 안에서 이미 뜻을 가진 글자**를 값으로 담으려면 `%XX`(퍼센트 + 16진수 두 자리)로 바꿔 싣는다.

RFC 3986 §2.1의 정의 그대로:
> *"A percent-encoded octet is encoded as a character triplet, consisting of the percent character "%" followed by the two hexadecimal digits representing that octet's numeric value."*

`서울` → `%EC%84%9C%EC%9A%B8` · 공백 → `%20`

### `%`는 탈출 문자다 — 그래서 이중 인코딩이 생긴다

`%`는 *"다음 두 글자는 16진수 코드다"*라는 신호다. 따라서 **`%` 자체를 데이터로 담으려면 `%25`로 써야 한다**(RFC 3986이 명시).

여기서 사고가 난다. **이미 인코딩된 문자열을 한 번 더 인코딩하면** `%2B` → `%252B`가 되고, 서버는 규칙대로 **한 번만** 풀어서 `%2B`라는 **글자 그대로**를 값으로 읽는다.

```
원래 값   +          (이 값을 담고 싶다)
1회 인코딩 %2B        ← 정상
2회 인코딩 %252B      ← 서버가 풀면 "%2B"라는 글자가 나온다. 값이 달라졌다
```

**증상이 고약하다**: 문법 에러도 400도 아니고 *"등록되지 않은 키"* 같은 **의미 수준의 실패**로 나타난다. 그래서 원인이 인코딩이라는 걸 떠올리기 어렵다.

### 질의문자열 구조

```
?이름=값&이름=값&이름=값
```
`?`로 시작, `&`로 구분, `이름=값` 쌍. 값만 이어 붙이면(`?a=1=2=3`) 서버가 읽을 수 없다.

**서버는 모르는 파라미터를 조용히 무시한다.** `numOfRows`를 `numbersOfRows`로 잘못 쓰면 에러가 아니라 **기본값으로 응답이 정상 반환**된다 — 틀렸는데 에러가 안 나는 종류의 버그.

### JS에서: `encodeURIComponent`

값 하나를 인코딩할 땐 `encodeURIComponent`. `A–Z a–z 0–9 - _ . ! ~ * ' ( )`를 뺀 전부를 이스케이프하고, **`? = / & :` 같은 구분자까지 인코딩한다.** `encodeURI`는 URL 전체용이라 구분자를 남겨두므로 **값에는 `encodeURIComponent`가 맞다.**

**규칙**: 파라미터 **값**에만 걸고, **이미 인코딩된 값에는 절대 걸지 않는다.**

## 기록

### 2026-09-08 — 브라우저에서만 403이 났다 (야생학습 사다리 세션 1)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[학습/야생학습/약국맵 사다리 1 — fetch와 useState 2026-09-08|사다리 세션 1]]. 공공데이터포털 서비스키로 첫 호출
- 증상: 같은 키인데 **`curl`은 200, 브라우저는 `SERVICE_KEY_IS_NOT_REGISTERED_ERROR`(403)**
- 원인: URL에 **인코딩 안 된 한글**(`Q0=서울특별시`)이 들어 있었다 → macOS `open`/브라우저가 *"인코딩이 필요한 URL"*로 보고 **문자열 전체를 인코딩** → 키 안에 이미 있던 `%`까지 `%25`로 바뀜 → 이중 인코딩
- 확인: 키를 4가지 모양으로 보내 대조. **`%`를 한 번 더 인코딩한 경우에만** 그 403이 재현됐다

  | 보낸 모양 | 결과 |
  |---|---|
  | Encoding 키 그대로 | 200 NORMAL SERVICE |
  | 한 번 푼 키를 raw로 | 200 |
  | 한 번 푼 키를 다시 인코딩 | 200 |
  | **`%2B` → `%252B`** | **403 SERVICE_KEY_IS_NOT_REGISTERED_ERROR** |

- 배운 것: **범인은 키가 아니라 옆에 있던 한글이었다.** 한 층(`open`)이 "내가 인코딩해줘야 하나?"를 스스로 판단하는 순간, 이미 인코딩된 다른 부분이 같이 망가진다. 층이 여러 개(내 코드 → `open` → 브라우저 → 서버)일 때 각 층이 독립적으로 판단하는 게 이 버그의 구조
- 근거: [[작업노트/도구/공공데이터포털 오픈API|공공데이터포털 오픈API]]에 재현 명령, `pharmacy-map` 커밋 `3c81de8`

## 참고 자료

- [RFC 3986 §2.1 — Percent-Encoding](https://datatracker.ietf.org/doc/html/rfc3986#section-2.1) — 정의 원전. `%` 자체는 `%25`로 인코딩해야 한다고 명시 (2026-09-08 확인)
- [MDN — Percent-encoding (Glossary)](https://developer.mozilla.org/en-US/docs/Glossary/Percent-encoding) — 문자별 대응표(`/`→`%2F`, `&`→`%26`, `=`→`%3D`) (2026-09-08 확인)
- [MDN — `encodeURIComponent()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent) — 이스케이프 대상과 `encodeURI`와의 차이 (2026-09-08 확인)
