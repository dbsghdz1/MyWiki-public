---
type: study
area: CS
audience: me
status: active
created: 2026-08-18
updated: 2026-09-16
aliases: [포트, localhost, 루프백, 사설 IP, LISTEN]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# 포트와 localhost

**서버는 특별한 하드웨어가 아니라 포트를 잡고 LISTEN 하는 프로그램이다.** `localhost`는 루프백이고, `*:3000`은 같은 와이파이의 다른 기기에도 열려 있다. (2026-09-16 [[학습/공부/CS/네트워크/네트워크|네트워크]]에서 분리)

## 핵심 정리

- **포트 = 한 컴퓨터 안에서 어느 프로그램인지 구분하는 번호**(0~65535). IP는 「어느 컴퓨터」까지만 알려준다. 3000번은 Node 생태계 관습일 뿐이다.
- **`localhost` = `127.0.0.1`(IPv6는 `::1`) = 루프백.** 요청이 랜카드로 나가지 않고 OS 안에서 되돌아온다. 랜선을 뽑아도 동작한다.
- **「나만 볼 수 있다」는 절반만 맞다.** LISTEN 주소가 `127.0.0.1:3000`이면 정말 나만 보지만 `*:3000`이면 모든 인터페이스에서 듣는다 — 같은 와이파이의 기기는 `192.168.x.x:3000`으로 들어올 수 있고, 사설 IP라 인터넷에서는 못 찾아온다.
- 확인 명령: `lsof -nP -iTCP:3000 -sTCP:LISTEN` · `ipconfig getifaddr en0`.

## 기록

### 2026-08-18 — 포트·localhost·사설 IP (강의 전에 프로젝트에서 먼저 만남)

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1에서 `npm run dev`로 띄운 서버가 왜 `localhost:3000`으로 보이는지 물으며 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]]). W1 강의(38강~) 시작 전에 실물로 먼저 부딪힌 내용이라 여기 남긴다.
- 배운 것:
  - **서버 = 특별한 하드웨어가 아니라 포트를 잡고 LISTEN 하고 있는 프로그램.** `lsof -nP -iTCP:3000 -sTCP:LISTEN`으로 `node ... TCP *:3000 (LISTEN)` 확인. 끄면 서버가 사라진다.
  - **포트 = 한 컴퓨터 안에서 어느 프로그램인지 구분하는 번호**(0~65535). IP는 "어느 컴퓨터"까지만 알려준다. 80=HTTP, 443=HTTPS, 5432=Postgres. **3000번은 아무 의미 없고 Node 생태계 관습**일 뿐 — `-p 4000`이면 그대로 동작한다.
  - **`localhost` = `127.0.0.1` = 루프백.** 요청이 랜카드로 나갔다 오지 않고 OS 안에서 되돌아온다. 랜선을 뽑아도 동작한다.
  - **"나만 볼 수 있다"는 절반만 맞다.** LISTEN 주소가 `127.0.0.1:3000`이면 정말 나만 보지만, `*:3000`이면 모든 인터페이스에서 듣는 것이라 **같은 와이파이의 다른 기기는 `192.168.x.x:3000`으로 접속 가능**하다(핸드폰 테스트에 씀). 다만 `192.168.x.x`는 사설 IP라 공유기 밖 인터넷에서는 찾아올 수 없다. 즉 정확히는 "인터넷에 공개되지 않았다".
  - 왜 브라우저가 외부 API를 직접 부르면 안 되는가 → **CORS**. 우리 서버에 same-origin 창구(route handler)를 두는 이유. 자세한 건 [[학습/공부/JS/Next.js/Next.js 서버와 캐싱|Next.js 서버와 캐싱]], CORS 자체는 W3-4 94강에서.
- 근거: 로컬 `lsof` 출력, `ipconfig getifaddr en0` → `192.168.35.199`


## 참고 자료

- [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/) — 소켓 프로그래밍 고전 무료 가이드. 코드로 확인하고 싶을 때 (2026-08-15 확인)
- [RFC 9293 — Transmission Control Protocol](https://www.rfc-editor.org/rfc/rfc9293) — 현행 TCP 명세. 상태 다이어그램(§3.3.2)만 봐도 가치 있음 (2026-08-15 확인)
