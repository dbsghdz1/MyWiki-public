---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-08
updated: 2026-09-09
projects:
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# 공공데이터포털 오픈API

`apis.data.go.kr` 계열 오픈API를 붙일 때 **착수 전에 5분이면 확인되는 것들**. 키 없이도 엔드포인트 생존·경로 오타·CORS는 전부 확인된다 — 이걸 안 하고 설계하면 "프록시가 필수"처럼 **틀린 전제 위에 아키텍처를 세운다.**

## 핵심 정리

- **키 없이 먼저 찔러본다.** 더미 키로 호출하면 응답이 셋 중 하나로 갈린다:
  - `SERVICE_KEY_IS_NOT_REGISTERED_ERROR` (`returnReasonCode 30`) → **엔드포인트는 살아 있다.** 경로가 맞다는 뜻
  - `해당 오픈API 서비스가 없거나 폐기됨` → **경로가 틀렸다**
  - 이 구분만으로 문서에 도는 오타 경로를 걸러낸다
- **CORS는 기본적으로 안 막힌다.** `Origin`을 붙여 보내면 `Access-Control-Allow-Origin`에 그 값이 **반사**돼 돌아온다. 단순 GET은 프리플라이트도 안 탄다 → **브라우저에서 직접 호출된다.** 프록시가 필요한 진짜 이유는 **서비스키 노출**과 **캐싱**이지 CORS가 아니다.
- **에러도 HTTP 403으로 온다.** 본문은 `200 + 에러 XML`이 아니라 **`403` + XML**이다. `res.ok`/`response.status` 체크에서 먼저 걸리므로, 클라이언트 코드가 XML 에러 본문을 읽기도 전에 throw한다.
- **응답은 XML이다** (`Content-Type: application/xml`). JSON이 필요하면 오퍼레이션별 `_type=json` 지원 여부를 따로 확인한다.
- **FullData 내려받기 오퍼레이션이 있으면 설계가 바뀐다.** 전량을 받아 캐싱하면 일 호출 한도(개발계정 1,000회/일)가 무의미해진다. 자주 안 바뀌는 마스터 데이터는 매 요청 호출하지 않는다.

## 기록

### 2026-09-09 — Fastify 프록시로 옮겼다. **키가 `undefined`여도 응답 메시지는 「등록되지 않은 서비스키」로 똑같다**

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[프로젝트/개인/약국맵/사다리 3-A — Fastify 프록시 2026-09-09|사다리 3-A]]. 브라우저 직접 호출 → Fastify 프록시 경유로 전환
- **`serviceKey=undefined`가 전송되면 응답은 `SERVICE_KEY_IS_NOT_REGISTERED_ERROR`다** — 09-08의 이중 인코딩 403과 **글자 그대로 같은 메시지**다. 즉 이 메시지는 「키가 틀렸다」가 아니라 **「내가 보낸 문자열을 못 알아보겠다」**는 뜻이고, 원인은 셋 중 하나다: ① 키가 안 읽힘(`.env` 누락·`--env-file` 누락) ② 이중 인코딩 ③ 진짜 미등록. **①을 1초에 배제하는 법**: `console.log("KEY 길이:", KEY?.length)` → 이 API 키는 96자
- **`numOfRows` 오타를 코드에서 뒤늦게 잡았다.** 09-08에 *"모르는 파라미터는 조용히 무시된다"*고 여기 적어놓고도 `numbersOfRows`가 코드에 남아 있었다. **파라미터 오타는 응답 개수를 세야 잡힌다**:

```bash
curl -s localhost:3000/api/pharmacies | grep -o '"dutyName"' | wc -l   # numOfRows=3 → 3
```

- **프록시 구성** (재현 가능한 최소형):

```js
// server/main.js — 라우트는 전부 listen 위에
const KEY = process.env.DATA_GO_KR_KEY;
app.get("/api/pharmacies", async () => {
  const url = `https://apis.data.go.kr/B552657/ErmctInsttInfoInqireService/getParmacyListInfoInqire`
    + `?serviceKey=${KEY}&numOfRows=3&Q0=${encodeURIComponent("서울특별시")}&Q1=${encodeURIComponent("관악구")}&_type=json`;
  const res = await fetch(url);
  return await res.json();
});
await app.listen({ port: 3000 });
```

```bash
node --env-file=.env.local server/main.js    # Node 20.6+ 내장, dotenv 불필요
```

- **`_type=json`이 이 오퍼레이션에서 동작한다** — `getParmacyListInfoInqire`는 `_type=json`을 주면 JSON으로 답한다(기본은 XML). 응답 모양은 `response.body.items.item[]`, 약국명은 `dutyName`, 고유 ID는 `hpid`
  - **주의**: `numOfRows=1`이면 `item`이 배열이 아니라 객체로 오는 계열의 API다. 개수를 1로 줄여 테스트할 때 `.map`이 터질 수 있다
- **키는 이미 인코딩된 형태**(끝이 `%3D`)로 발급된다. 서버 코드에서도 `encodeURIComponent`를 씌우면 안 된다 → [[학습/공부/CS/URL과 퍼센트 인코딩|URL과 퍼센트 인코딩]]
- 영향: 번들에서 `serviceKey=` 검색 결과 **1 → 0**. 브라우저는 이제 `/api/pharmacies`만 안다. 개발 중 전달은 Vite `server.proxy`가 하고, **배포에서는 Fastify가 직접 해야 한다**(사다리 3-B)

### 2026-09-08 (2) — 이중 인코딩 403과 「모르는 파라미터는 조용히 무시」 — 브라우저에서 처음 붙이며

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] [[프로젝트/개인/약국맵/사다리 1 — fetch와 useState 2026-09-08|사다리 세션 1]]. 발급받은 서비스키로 첫 실호출
- **같은 키인데 `curl`은 200, 브라우저는 403 `SERVICE_KEY_IS_NOT_REGISTERED_ERROR`가 났다.** 원인은 키가 아니라 **URL 안의 인코딩 안 된 한글**(`Q0=서울특별시`)이다 — macOS `open`/브라우저가 URL 전체를 인코딩하면서 키 안의 `%`까지 `%25`로 바꿔버린다(이중 인코딩). 한글을 미리 퍼센트 인코딩해두면 그대로 열린다
- 키 모양 4가지를 대조했다. **`%`를 한 번 더 인코딩한 경우에만 403이 재현된다** — 나머지 셋은 전부 200이라, **Encoding 키/Decoding 키 중 뭘 넣느냐는 사실 문제가 아니었다**

```bash
set -a; source .env.local; set +a
B="https://apis.data.go.kr/B552657/ErmctInsttInfoInqireService/getParmacyListInfoInqire?numOfRows=1&Q0=%EC%84%9C%EC%9A%B8%ED%8A%B9%EB%B3%84%EC%8B%9C&Q1=%EA%B4%80%EC%95%85%EA%B5%AC"

curl -s "$B&serviceKey=$VITE_DATA_GO_KR_KEY"   # Encoding 키 그대로 → 200 NORMAL SERVICE
# 한 번 푼 키를 raw로 / 다시 인코딩해서 → 둘 다 200
DBL=$(python3 -c "import urllib.parse,os;print(urllib.parse.quote(os.environ['VITE_DATA_GO_KR_KEY'],safe=''))")
curl -s "$B&serviceKey=$DBL"                    # % → %25 이중 인코딩 → 403 SERVICE_KEY_IS_NOT_REGISTERED_ERROR
```

- **모르는 파라미터는 조용히 무시된다.** `numOfRows`를 `numbersOfRows`로 오타냈는데 **에러 없이 기본 개수로 정상 응답**이 왔다. 400도 경고도 없다 — 파라미터 오타는 **응답 개수를 세어야** 잡힌다
- **브라우저에서 직접 호출이 실제로 된다** — `localhost:5173`(Vite dev) → `apis.data.go.kr`, HTTP 200, 응답 헤더 `access-control-allow-origin: http://localhost:5173`, **`OPTIONS` 프리플라이트 없음**(헤더 없는 맨 `GET`이라 단순 요청). 09-08 오전의 `curl` 실측이 실제 브라우저에서도 그대로 확인됐다
- **`.env` 함정**: Vite 템플릿 `.gitignore`는 `*.local`만 덮는다. `.env`라는 이름으로 만들면 **키가 커밋에 딸려 올라간다.** `.env.local`을 쓰거나 `.env`·`.env.*`를 무시 목록에 추가할 것
- 영향: 사다리 1·2는 프록시 없이 확정. Fastify는 3번에서 **키 노출 + 캐싱** 때문에 붙인다


### 2026-09-08 — 약국맵 착수 전 「전국 약국 정보 조회 서비스」를 키 없이 검증했다

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] 착수. 09-05 문서가 *"공공 API 다수가 브라우저에서 직접 호출되지 않는다(CORS·키 노출). 프록시가 없으면 동작하지 않는다"*고 Fastify 도입 근거를 적어 뒀는데, **CORS 쪽을 실측하니 틀렸다.**
- 대상: 국립중앙의료원 전국 약국 정보 조회 서비스 (공공데이터포털 신청번호 `15000576`)
  - 정상 경로: `https://apis.data.go.kr/B552657/ErmctInsttInfoInqireService/getParmacyListInfoInqire`
  - **틀린 경로**: `ErmctInsttInsttInfoInqireService` (`Instt` 중복) — 일부 문서에 이렇게 돌아다니는데 `해당 오픈API 서비스가 없거나 폐기됨`이 온다
- 배운 것:
  - **오퍼레이션 생존 확인은 더미 키로 된다.** `getParmacyListInfoInqire`·`getParmacyFullDown`·`getParmacyLcinfoInqire`·`getParmacyBassInfoInqire` 넷 모두 `SERVICE_KEY_IS_NOT_REGISTERED_ERROR`를 반환 = 넷 다 존재한다. 「FullData 내려받기」도 xls 파일이 아니라 **API 오퍼레이션**이다.
  - **CORS가 안 막힌다.** `Origin: http://localhost:5173` → 응답 헤더 `Access-Control-Allow-Origin: http://localhost:5173`. Origin을 그대로 반사한다.
  - **상태코드가 403이다.** Origin 유무와 무관하게 잘못된 키는 `HTTP/1.1 403 Forbidden` + 에러 XML. 200을 기대하면 안 된다.
  - 조회 조건이 **날짜가 아니라 요일**이고 영업시간 필드가 `dutyTime1s/1c` ~ `dutyTime8s/8c` 8슬롯(월~일 + 공휴일)의 정적 주간 스케줄이다 → **명절 당번약국은 못 담을 가능성이 높다**(키 발급 후 대조 필요). `dutyTime8`은 "공휴일에 몇 시에 연다"는 상시 선언이지 "이번 추석 당번"이 아니다.
  - 안내문에 *"실제 영업시간은 약국에 전화 후 방문"*이 적혀 있다 — **원천 데이터 운영 주체가 신뢰도 한계를 스스로 명시한 것**이고, 약국맵 「확인됨」 배지의 근거가 된다.
- 재현:

```bash
# ① 오퍼레이션 생존 — 더미 키로 확인
curl -s "https://apis.data.go.kr/B552657/ErmctInsttInfoInqireService/getParmacyListInfoInqire?serviceKey=DUMMY&numOfRows=1"
# → <errMsg>SERVICE_KEY_IS_NOT_REGISTERED_ERROR</errMsg> <returnReasonCode>30</returnReasonCode>

# ② CORS — Origin 반사 확인
curl -s -D - -o /dev/null -H "Origin: http://localhost:5173" \
  "https://apis.data.go.kr/B552657/ErmctInsttInfoInqireService/getParmacyListInfoInqire?serviceKey=DUMMY&numOfRows=1" \
  | grep -i "^HTTP\|access-control"
# → HTTP/1.1 403 Forbidden
# → Access-Control-Allow-Origin: http://localhost:5173
```

- 영향: 약국맵 사다리 **세션 1·2를 프록시 없이 진행**할 수 있게 됐다. Fastify는 세션 3에서 **키를 감추고 FullData를 캐싱하려고** 붙인다 — 스택 선택은 유지, 근거만 좁아졌다.

## 참고 자료

- [공공데이터포털 — 국립중앙의료원 전국 약국 정보 조회 서비스](https://www.data.go.kr/data/15000576/openapi.do) — 신청번호 `15000576`, 이용허락 제한 없음·개발/운영 자동승인 (2026-09-08 확인)
- [MDN — Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CORS) — 단순 요청(simple request)이 프리플라이트를 타지 않는 조건 (2026-09-08 확인)
