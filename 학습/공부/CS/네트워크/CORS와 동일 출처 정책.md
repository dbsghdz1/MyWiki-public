---
type: study
area: CS
audience: me
status: active
created: 2026-09-08
updated: 2026-09-08
projects:
  - "약국맵"
---

# CORS와 동일 출처 정책

**브라우저는 요청을 막는 게 아니라, 응답을 「읽게 해줄지」를 판단한다.** 이 순서를 알면 CORS 에러가 왜 그런 모양으로 나타나는지 설명된다.

## 핵심 정리

### 출처(origin) = 스킴 + 호스트 + 포트

셋 중 하나만 달라도 다른 출처다.

| 비교 대상 | `http://store.company.com/dir/page.html` 기준 |
|---|---|
| `https://store.company.com/page.html` | 다름 (스킴) |
| `http://store.company.com:81/dir/page.html` | 다름 (포트) |
| `http://news.company.com/dir/page.html` | 다름 (호스트) |

**동일 출처 정책**은 다른 출처끼리의 상호작용을 제한한다. 중요한 건 **무엇을 제한하느냐**다 (MDN 분류):

- **쓰기**(링크·리다이렉트·폼 제출) — 대체로 **허용**
- **끼워넣기**(`<img>` `<script>` `<link>` `<iframe>`) — 대체로 **허용**
- **읽기** — **금지**가 기본

그래서 `<img src="남의사이트/사진.png">`는 되는데 `fetch("남의사이트/데이터")`의 **결과를 읽는 건** 안 된다. 목적은 악성 사이트가 내 웹메일·사내망 내용을 몰래 읽어가는 걸 막는 것.

### CORS는 서버가 내주는 허가증이다

서버가 응답에 **`Access-Control-Allow-Origin`**을 실어 *"이 출처에는 보여줘도 된다"*고 선언하면 브라우저가 읽기를 허용한다. 값이 요청 Origin과 맞아야 한다.

**요청은 이미 서버에 갔다.** 브라우저는 응답을 받아본 다음에 *"읽게 해줄까?"*를 판단하고, 아니면 JS에게 에러를 준다. 그래서 **서버 로그에는 요청이 찍혀 있는데 브라우저에서는 CORS 에러**인 상황이 정상이다.

### 프리플라이트 — "단순 요청"이면 건너뛴다

위험해 보이는 요청은 본 요청 전에 `OPTIONS`로 먼저 물어본다(**프리플라이트**). 다만 **단순 요청**이면 이 절차가 없다. 대략:

- 메서드가 `GET` / `HEAD` / `POST`이고
- 직접 붙인 헤더가 없거나 아주 기본적인 것뿐이고
- `Content-Type`이 몇 가지 평범한 값일 때

**헤더 하나 안 붙인 맨 `GET`은 프리플라이트를 안 탄다.** 반대로 `POST` + `Content-Type: application/json`이 되는 순간 `OPTIONS`가 하나 더 생긴다.

### 실무 함정: "CORS 때문에 프록시가 필요하다"가 항상 참은 아니다

많은 공공/외부 API가 **Origin을 그대로 반사해서** `Access-Control-Allow-Origin`으로 돌려준다. 이러면 브라우저에서 직접 부를 수 있다. **확인은 `curl -D -`로 응답 헤더를 보는 것으로 30초면 끝난다** — 안 해보고 "CORS 때문에"라고 적으면 **틀린 전제 위에 아키텍처를 세우게 된다.**

프록시가 필요한 진짜 이유는 보통 **키 노출**과 **캐싱**이다.

## 기록

### 2026-09-08 — localhost에서 공공 API를 직접 불렀는데 되더라 (실습 실습 세션 1)

- 맥락: [약국맵](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/%EC%95%BD%EA%B5%AD%EB%A7%B5/README.md) [실습 세션 1](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/%EC%95%BD%EA%B5%AD%EB%A7%B5/%EC%8B%A4%EC%8A%B5%201%20%E2%80%94%20fetch%EC%99%80%20useState%202026-09-08.md). `localhost:5173` → `apis.data.go.kr` 직접 호출
- 예측: **"남의 서버라 막힌다"** → **틀렸다.** 200으로 왔고 화면에 XML이 떴다
- 실제로 본 것:
  - 응답 헤더 `access-control-allow-origin: http://localhost:5173` — **우리 Origin이 그대로 반사돼 있다**
  - Network에 **`OPTIONS` 줄이 없다** — 헤더 없는 맨 `GET`이라 단순 요청
- 배운 것: 직관("기본은 못 읽는다")은 맞았고, 이 서버가 허락한 것뿐이다. **막느냐 마느냐를 정하는 건 브라우저가 아니라 서버의 헤더다**
- 프로젝트 영향: 약국맵 Fastify 도입 근거에서 CORS가 빠지고 **키 노출 + 캐싱**만 남았다. 실습 1·2는 프록시 없이 진행
- 근거: [공공데이터포털 오픈API](../../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/%EB%8F%84%EA%B5%AC/%EA%B3%B5%EA%B3%B5%EB%8D%B0%EC%9D%B4%ED%84%B0%ED%8F%AC%ED%84%B8%20%EC%98%A4%ED%94%88API.md), `pharmacy-map` 커밋 `3c81de8`

## 더 알아보면 좋은 것

- 실습 8번(제보 버튼)에서 `POST` + JSON을 보내면 **`OPTIONS`가 처음 등장한다.** 오늘 본 것과 대비해서 볼 것

## 참고 자료

- [MDN — Same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy) — 출처의 정의(스킴/호스트/포트)와 쓰기·끼워넣기는 허용, **읽기는 금지**라는 분류 (2026-09-08 확인)
- [MDN — Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) — 단순 요청 조건과 프리플라이트 (2026-09-08 확인)
- [Fetch Standard §3.3 — CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol) — 원전. 요청/응답 헤더와 자격증명 포함 시의 규칙 (2026-09-08 확인)
