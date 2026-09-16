---
type: study
area: CS
audience: me
status: active
created: 2026-09-14
updated: 2026-09-16
aliases: [HTTP, HTTPS, 인증서, CA, HTTP/2, HTTP/3, QUIC]
projects:
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# HTTP와 TCP/IP의 역할

**HTTP는 무엇을 주고받을지 정하고, TCP는 어느 포트의 프로그램에 빠짐없이 순서대로, IP는 어느 컴퓨터까지 전달한다.** (2026-09-16 [[학습/공부/CS/네트워크/네트워크|네트워크]]에서 분리)

## 핵심 정리

- **HTTP = 메서드·경로·헤더·상태 코드·본문**을 정하는 규약. Fastify 같은 서버는 TCP가 아니라 「TCP 포트에서 HTTP 요청을 받는 프로그램」이다 — `app.listen({ port: 3000 })`.
- **3000번 포트에 듣는 프로그램이 없으면 HTTP 전에 TCP 연결이 거절된다.** 어느 층에서 실패했는지가 어디를 봐야 하는지를 정한다.
- **HTTPS 인증서는 도메인과 공개키를 묶고, CA가 그 관계에 서명한다.** 브라우저는 기기의 Root Store에서 출발한 체인을 확인한다. 셀프 서명도 암호화는 되지만 신원을 믿을 근거가 없다.
- **HTTP/1.1은 TCP 연결 재사용, HTTP/2는 한 연결의 여러 스트림, HTTP/3는 QUIC/UDP.** HTTP/3의 핵심은 한 스트림의 손실이 다른 스트림을 막지 않고 TLS가 전송에 통합된다는 것 → [[학습/공부/CS/네트워크/학습 계획|학습 계획]] N6에서 QUIC으로 확인.

## 기록

### 2026-09-14 — HTTP는 무엇을 정하고 TCP/IP는 무엇을 운반하나

- 궁금했던 것: [[프로젝트/개인/약국맵/README|약국맵]]의 두 HTTP 홉(`브라우저 → Fastify → apis.data.go.kr`)을 `curl -v`로 보다가, HTTP가 TCP/IP 기반이라는 말과 경로·헤더·인증서가 각각 무엇인지 궁금해졌다.
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

## 참고 자료

- [MDN — HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP) — HTTP 레퍼런스. 헤더·캐싱·상태 코드 (2026-08-15 확인)
- [Cloudflare — What is HTTP?](https://www.cloudflare.com/en-gb/learning/ddos/glossary/hypertext-transfer-protocol-http/) — 요청·응답과 HTTP 기본 구조를 가장 먼저 잡는 입문 자료 (2026-09-14 확인)
- [cs.fyi — Everything you need to know about HTTP](https://cs.fyi/guide/http-in-depth) — HTTP/0.9~2의 발전 이유. 지금은 persistent connection까지만, N5 뒤에 전체 복습 (2026-09-14 확인)
- [How HTTPS Works — Certificate Authorities](https://howhttps.works/ko/certificate-authorities/) — 인증서·CA·Root Store·신뢰 체인을 만화로. N6 TLS 뒤에 다시 보기 (2026-09-14 확인)
- [Key differences between HTTP/1.0 and HTTP/1.1](https://www.ra.ethz.ch/cdstore/www8/data/2136/pdf/pd1.pdf) — 1999년 당시 설계 이유를 9개 영역으로 설명한 역사 자료. N5 뒤에 §5 연결 관리만 복습 (2026-09-14 확인)
- [The New Stack — HTTP/3 Is Now a Standard](https://thenewstack.io/http-3-is-now-a-standard-why-use-it-and-how-to-get-started/) — QUIC/UDP·스트림별 손실 격리·내장 TLS의 입문 설명. 2022년 표준화 직후 글이라 구현·지원 현황은 현재값으로 쓰지 않는다 (2026-09-14 확인)
- [RFC 9114 — HTTP/3](https://www.rfc-editor.org/rfc/rfc9114) — HTTP/3 현행 표준 원문. N6에서는 개요와 QUIC 스트림 매핑만 확인 (2026-09-14 확인)
