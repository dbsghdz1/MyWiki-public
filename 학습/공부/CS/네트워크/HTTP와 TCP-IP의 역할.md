---
type: study
area: CS
audience: me
status: active
created: 2026-09-14
updated: 2026-10-15
aliases: [HTTP, HTTPS, 인증서, CA, HTTP/2, HTTP/3, QUIC, HPACK, QPACK, HOL, Head-of-line blocking, 멀티플렉싱]
projects:
  - "약국맵"
---

# HTTP와 TCP/IP의 역할

**HTTP는 무엇을 주고받을지 정하고, TCP는 어느 포트의 프로그램에 빠짐없이 순서대로, IP는 어느 컴퓨터까지 전달한다.** (2026-09-16 [네트워크](%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC.md)에서 분리)

## 핵심 정리

- **HTTP = 메서드·경로·헤더·상태 코드·본문**을 정하는 규약. Fastify 같은 서버는 TCP가 아니라 「TCP 포트에서 HTTP 요청을 받는 프로그램」이다 — `app.listen({ port: 3000 })`.
- **3000번 포트에 듣는 프로그램이 없으면 HTTP 전에 TCP 연결이 거절된다.** 어느 층에서 실패했는지가 어디를 봐야 하는지를 정한다.
- **HTTPS 인증서는 도메인과 공개키를 묶고, CA가 그 관계에 서명한다.** 브라우저는 기기의 Root Store에서 출발한 체인을 확인한다. 셀프 서명도 암호화는 되지만 신원을 믿을 근거가 없다.
- **각 버전은 앞 버전의 막힘(HOL) 하나를 푼다.** HTTP/2는 HTTP/1.1의 「응답은 순서대로」를, HTTP/3는 TCP의 「바이트는 순서대로」를 푼다. 위에 얹힌 HTTP의 뜻(메서드, 헤더, 상태 코드)은 그대로이고 아래 운반 방식만 바뀐다. 2026-10-15 실측으로 확인.

| 버전 | 운반 | 패킷 하나가 사라지면 |
|---|---|---|
| HTTP/1.1 | TCP | 앞 응답이 끝나야 다음 응답. 그래서 브라우저가 연결을 6개씩 연다 |
| HTTP/2 | TCP 연결 하나에 여러 스트림 | 파일을 섞어 보내지만 TCP가 번호 순서를 지켜서 전부 같이 멈춘다 |
| HTTP/3 | UDP 위의 QUIC | 사라진 패킷이 속한 스트림만 재전송을 기다리고 나머지는 계속 받는다 |

## 기록

### 2026-09-14 — HTTP는 무엇을 정하고 TCP/IP는 무엇을 운반하나

- 궁금했던 것: [약국맵](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/%EC%95%BD%EA%B5%AD%EB%A7%B5/README.md)의 두 HTTP 홉(`브라우저 → Fastify → apis.data.go.kr`)을 `curl -v`로 보다가, HTTP가 TCP/IP 기반이라는 말과 경로·헤더·인증서가 각각 무엇인지 궁금해졌다.
- 예측: Fastify를 쓰는 이유는 브라우저에 공공 API 키를 노출하지 않기 위해서라고 맞혔다. 반면 처음에는 TCP를 Fastify가 데이터를 가져오는 방식, `localhost`의 IP를 와이파이 네트워크 주소로 생각했다.
- 파고든 길: `curl -v http://localhost:3000/api/pharmacies -o /dev/null`에서 `GET /api/pharmacies`, `Host: localhost:3000`, `200 OK`를 보고, 가짜 키로 공공 API를 직접 호출해 `403 Forbidden`을 확인했다. 이어 HTTP 입문 글·HTTPS 인증기관 만화·HTTP 버전 비교 자료를 읽었다.
- 알게 된 것:
  - **HTTP는 무엇을 주고받을지** 정한다(메서드·경로·헤더·상태 코드·본문). **TCP는 어느 포트의 프로그램에 빠짐없이 순서대로**, **IP는 어느 컴퓨터까지** 전달한다. Fastify는 TCP가 아니라 `app.listen({ port: 3000 })`으로 TCP 3000번 포트에서 HTTP 요청을 받는 프로그램이다.
  - `localhost`는 와이파이 주소가 아니라 자기 컴퓨터를 가리키는 루프백(`::1`·`127.0.0.1`)이다. 3000번 포트에 듣는 프로그램이 없으면 HTTP 전에 TCP 연결이 거절된다.
  - HTTPS 인증서에는 도메인과 공개키가 묶이고, CA는 둘의 관계에 서명한다. 브라우저는 기기의 Root Store에서 출발한 인증서 체인을 확인한다. 셀프 서명도 암호화는 되지만 공개 사이트에서 신원을 믿을 근거가 없다.
  - HTTP/1.1은 TCP 연결 재사용, HTTP/2는 한 TCP 연결의 여러 스트림, HTTP/3는 QUIC/UDP를 쓴다. HTTP/3의 핵심은 한 스트림의 패킷 손실이 다른 스트림 전체를 막지 않고 TLS가 전송에 통합된다는 것.
- 다시 보기: **지금**은 Cloudflare HTTP 입문과 How HTTPS Works만 이해한다. **N5(HTTP/1.0·1.1)** 뒤에 cs.fyi와 1999년 비교 논문 5장, **N6(HTTP/2·3·TLS)** 뒤에 The New Stack HTTP/3 글과 RFC 9114를 다시 본다. 세부 구현 목록은 외우지 않는다.
- 근거: [Cloudflare — What is HTTP?](https://www.cloudflare.com/en-gb/learning/ddos/glossary/hypertext-transfer-protocol-http/) · [cs.fyi — Everything you need to know about HTTP](https://cs.fyi/guide/http-in-depth) · [How HTTPS Works — Certificate Authorities](https://howhttps.works/ko/certificate-authorities/) · [Key differences between HTTP/1.0 and HTTP/1.1 (1999)](https://www.ra.ethz.ch/cdstore/www8/data/2136/pdf/pd1.pdf) · [The New Stack — HTTP/3 Is Now a Standard](https://thenewstack.io/http-3-is-now-a-standard-why-use-it-and-how-to-get-started/) · [RFC 9114 — HTTP/3](https://www.rfc-editor.org/rfc/rfc9114) (2026-09-14 확인)
- 새로 생긴 궁금증: HTTP/3가 TCP를 버리고도 어떻게 순서·재전송·혼잡 제어를 제공하는가? → N6에서 QUIC으로 확인한다.

### 2026-10-15: HTTP/2와 HTTP/3는 각각 무엇을 고쳤나

- 궁금했던 것: [네트워크 학습 계획](%ED%95%99%EC%8A%B5%20%EA%B3%84%ED%9A%8D.md) N6의 61강(HTTP/2 vs 3)을 듣고 발전 과정을 다시 잡고 싶었다. 09-14 기록에 남긴 「HTTP/3는 TCP 없이 어떻게 재전송하나」를 이어서 확인했다.
- 예측: HTTP/2는 HEADERS와 DATA 프레임으로 나눠 병렬로 보내고, 같은 헤더면 압축한다고 이해했다. 프레임 분리, 스트림 ID, 같은 연결로 CSS를 부르는 것은 맞았다. 「병렬」과 「압축」은 실제 동작과 달랐다. HTTP/3에서 HTML 패킷 하나가 사라지면 파일 전송이 안 된다고 예측했는데 틀렸다.
- 파고든 길:
  - `curl -sI --http2 https://cloudflare-quic.com` 응답에서 `alt-svc: h3=":443"; ma=86400`을 봤다. 서버가 HTTP/2로 답하면서 HTTP/3 포트를 광고하고, 브라우저는 다음 접속부터 갈아탄다.
  - `curl -sv --http2`로 두 URL을 받자 `Re-using existing connection` 뒤에 스트림 `[1]`과 `[3]`이 같은 연결에 열렸다.
  - 맥 기본 curl 8.7.1은 HTTP/3를 못 해서 Python `aioquic` 1.2.0으로 요청 두 개를 보냈다. 핸드셰이크 1회 14ms, 스트림 0과 4, 연결 ID 8바이트가 찍혔다.
  - Python 소켓으로 닫힌 포트에 보내 보니 TCP는 `Connection refused`가 즉시 오고, UDP는 보내기만 성공하고 답이 끝내 없었다.
- 알게 된 것:
  - **HTTP/2는 병렬이 아니라 끼워 넣기다.** 모든 프레임은 길이, 타입, 플래그, 스트림 ID를 담은 9바이트 헤더로 시작한다. 클라이언트가 여는 스트림은 홀수다. 선로는 TCP 하나라 프레임은 한 줄로 흐르고, 빠른 이유는 느린 응답 하나가 뒤를 막지 않아서다.
  - **HPACK은 압축보다 사전 참조다.** 양쪽이 같은 표를 들고 헤더를 번호로 보낸다. 정적 표는 61개(`:method: GET`이 2번)이고, 동적 표는 연결에서 한 번 보낸 헤더를 쌓는다. 그래서 두 번째 요청의 `user-agent`는 번호 하나로 간다.
  - **서버 푸시는 별개 기능이다.** 요청 전에 서버가 CSS를 밀어 넣는 것인데 Chrome 106에서 제거됐고, 지금은 `103 Early Hints`와 preload가 대신한다.
  - **TCP HOL**: TCP는 스트림을 모르고 바이트 번호만 지킨다. 3번 패킷이 사라지면 다른 파일인 4번, 5번도 재전송을 기다린다. 순서를 고집하는 쪽은 HTTP/2가 아니라 TCP다.
  - **HTTP/3는 같은 HTTP에 운반만 바꿨다.** QUIC이 UDP 위에서 재전송과 순서 보장을 스트림별로 다시 만들었다. 사라진 패킷은 재전송으로 결국 오고, 그 스트림만 기다린다. TCP와 TLS 핸드셰이크를 합쳐 왕복 1회, 재접속은 0-RTT다.
  - **연결 ID 덕분에 네트워크가 바뀌어도 이어진다.** TCP 연결은 양쪽 IP와 포트 4개 값이 이름이라 와이파이에서 LTE로 바뀌면 끊긴다. 그래서 손실과 전환이 잦은 지하철 같은 환경에서 HTTP/3가 유리하다.
  - 스트림이 순서 없이 도착하니 순서에 기대는 HPACK 대신 QPACK을 쓴다. QUIC 스트림 ID는 하위 2비트가 유형이라 클라이언트 양방향 스트림이 0, 4, 8로 오른다. TCP를 고치지 않고 UDP로 간 이유는 TCP가 커널과 중간 장비에 굳어 새 기능을 넣을 수 없어서다(ossification).
- 내 비유: 인터넷은 상자를 던지는 택배다. UDP는 던지고 끝이고, TCP는 상자에 번호를 붙여 답장을 받고 안 온 것은 다시 보낸다. HTTP/2는 HTML과 CSS를 한 트럭에 섞어 TCP 기사에게 맡기는데, 기사는 번호 순서만 지킨다. HTTP/3는 파일마다 줄을 따로 세는 QUIC 기사로 바꿨다.
- 근거: 위 실측(2026-10-15 실행), [RFC 9113 §4.1](https://www.rfc-editor.org/rfc/rfc9113) 프레임 형식, [RFC 7541 부록 A](https://www.rfc-editor.org/rfc/rfc7541) 정적 표, [RFC 9000 §2.1](https://www.rfc-editor.org/rfc/rfc9000) 스트림 ID, [Chrome 블로그](https://developer.chrome.com/blog/removing-push) 푸시 제거 (2026-10-15 확인)
- 새로 생긴 궁금증: TLS 핸드셰이크는 왕복마다 무엇을 주고받는가? → N6의 62~63강.

## 참고 자료

- [MDN — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) — HTTP 레퍼런스. 헤더·캐싱·상태 코드 (2026-08-15 확인)
- [Cloudflare — What is HTTP?](https://www.cloudflare.com/en-gb/learning/ddos/glossary/hypertext-transfer-protocol-http/) — 요청·응답과 HTTP 기본 구조를 가장 먼저 잡는 입문 자료 (2026-09-14 확인)
- [cs.fyi — Everything you need to know about HTTP](https://cs.fyi/guide/http-in-depth) — HTTP/0.9~2의 발전 이유. 지금은 persistent connection까지만, N5 뒤에 전체 복습 (2026-09-14 확인)
- [How HTTPS Works — Certificate Authorities](https://howhttps.works/ko/certificate-authorities/) — 인증서·CA·Root Store·신뢰 체인을 만화로. N6 TLS 뒤에 다시 보기 (2026-09-14 확인)
- [Key differences between HTTP/1.0 and HTTP/1.1](https://www.ra.ethz.ch/cdstore/www8/data/2136/pdf/pd1.pdf) — 1999년 당시 설계 이유를 9개 영역으로 설명한 역사 자료. N5 뒤에 §5 연결 관리만 복습 (2026-09-14 확인)
- [The New Stack — HTTP/3 Is Now a Standard](https://thenewstack.io/http-3-is-now-a-standard-why-use-it-and-how-to-get-started/) — QUIC/UDP·스트림별 손실 격리·내장 TLS의 입문 설명. 2022년 표준화 직후 글이라 구현·지원 현황은 현재값으로 쓰지 않는다 (2026-09-14 확인)
- [RFC 9114 — HTTP/3](https://www.rfc-editor.org/rfc/rfc9114) — HTTP/3 현행 표준 원문. N6에서는 개요와 QUIC 스트림 매핑만 확인 (2026-09-14 확인)
- [RFC 9113 HTTP/2](https://www.rfc-editor.org/rfc/rfc9113): HTTP/2 현행 표준. §4.1 프레임 형식과 §5 스트림만 보면 된다 (2026-10-15 확인)
- [RFC 9000 QUIC](https://www.rfc-editor.org/rfc/rfc9000): QUIC 전송 표준. §2.1 스트림 ID 유형과 §5.1 연결 ID (2026-10-15 확인)
- [Chrome for Developers: Remove HTTP/2 Server Push from Chrome](https://developer.chrome.com/blog/removing-push): 서버 푸시를 Chrome 106에서 뺀 이유와 대안인 103 Early Hints (2026-10-15 확인)
