---
type: study
area: CS
audience: me
status: active
created: 2026-09-22
updated: 2026-09-22
aliases: [DNS, 재귀 확인자, 리졸버, 루트 서버, TLD, 네임서버, dig]
---

# DNS 조회 과정

**DNS는 「누가 아는지」를 위에서 아래로 되묻는 것이다.** 루트는 `.com`을 누가 아는지, `.com`은 `example.com`을 누가 아는지, `example.com`의 네임서버만 IP를 안다. 이 되묻기를 내 대신 해 주는 게 재귀 확인자(recursive resolver)이고, 답은 TTL 동안 캐시된다.

## 핵심 정리

### 등장인물 넷

| 역할 | 누구 | 무엇을 아나 | 내 맥에서 본 것 |
|---|---|---|---|
| 재귀 확인자 (resolver) | 통신사·회사·`1.1.1.1`·`8.8.8.8` | 아무것도 모르지만 대신 찾아다닌다 | `210.220.163.82` (`scutil --dns`) |
| 루트 서버 (`.`) | 13개 이름(a~m.root-servers.net), 실제로는 수백 대 | 각 TLD 네임서버가 어디인지 | `h.root-servers.net` |
| TLD 서버 (`.com`) | 등록소(Verisign) | 각 도메인 네임서버가 어디인지 | `m.gtld-servers.net` |
| 권한 네임서버 (authoritative) | 도메인 주인이 지정 (Cloudflare 등) | **그 도메인의 IP** | `hera.ns.cloudflare.com` |

RFC 1034 §2.4: 네임서버는 자기 존(zone)에 **AUTHORITY**를 가진 프로그램, 확인자는 클라이언트 대신 네임서버들을 찾아다니는 프로그램.

### 8단계를 두 덩어리로 보면

1. **나 ↔ 확인자** (1단계·8단계): 브라우저는 OS에 묻고, OS는 설정된 확인자 한 곳에만 묻는다. 나는 루트도 TLD도 모른다.
2. **확인자 ↔ 세 서버** (2~7단계): 확인자가 루트 → TLD → 권한 네임서버 순서로 세 번 묻는다. 앞의 둘은 답을 모르고 **"거기 가서 물어봐"(referral)**만 준다.

```
dig +trace example.com

.            NS  a~m.root-servers.net      ← 확인자에게 받음 (루트 목록)
com.         NS  a~m.gtld-servers.net      ← h.root-servers.net 답: ".com은 얘들이 안다"
example.com. NS  hera/elliott.ns.cloudflare.com  ← m.gtld-servers.net 답: "example.com은 얘들이 안다"
example.com. A   104.20.23.154             ← hera.ns.cloudflare.com 답: 드디어 IP
```

`+trace`는 확인자가 하는 2~7단계를 **내 맥이 직접 흉내 내는** 옵션이다. 각 단계에 `;; Received … from h.root-servers.net`처럼 누가 답했는지 찍힌다.

### 재귀(recursive)와 반복(iterative)

- 나 → 확인자: **재귀 질의.** "끝까지 찾아서 IP를 줘."
- 확인자 → 루트·TLD·네임서버: **반복 질의.** "너 아니? 모르면 누가 아는지만 알려줘." 서버들은 답 대신 위임(referral)을 돌려주고 확인자가 다음 서버로 옮겨 간다(RFC 1034 §5.3.3: "a better delegation to other servers → cache the delegation, go to step 2").

루트·TLD 서버가 재귀를 안 해 주는 이유: 전 세계 질의를 대신 찾아다니면 견딜 수 없다. 자기 몫만 답하고 넘긴다.

### 캐시와 TTL — 8단계가 매번 도는 게 아니다

권한 네임서버의 답에는 TTL(초)이 붙어 있고, 확인자는 그동안 답을 저장한다. 같은 질문을 다시 하면 2~7단계 없이 캐시에서 준다. 캐시는 확인자 앞에도 있다: 브라우저 캐시 → OS 캐시 → 확인자 캐시.

```
dig example.com A +noall +answer     example.com. 220 IN A 104.20.23.154   Query time: 4 msec
(잠시 후)                              example.com. 189 IN A 104.20.23.154
```

TTL이 220 → 189로 **줄어든다** = 확인자가 캐시에서 주면서 남은 시간을 알려주는 것. 4ms는 확인자까지 왕복만 한 시간이다(`+trace`의 루트·TLD 왕복은 각 38ms). 도메인 DNS를 바꾸고 "전파에 시간이 걸린다"는 말은 각 확인자의 캐시가 TTL만큼 옛 답을 들고 있다는 뜻이다.

## 기록

### 2026-09-22 — Cloudflare «What is DNS?»의 8단계를 붙여넣고 "자세히"

- 맥락: MDN Learn → 도메인·호스팅에서 이어진 질문. Cloudflare Learning 글의 8단계 영한 대역을 붙여넣음
- 파고든 길: `scutil --dns`로 내 확인자 확인 → `dig +trace example.com`으로 루트·TLD·네임서버 세 번의 위임을 실제로 봄 → 같은 질의 두 번으로 TTL 카운트다운 확인 → RFC 1034 §2.4·§5.3.3
- 결론: 8단계는 「나↔확인자」 2단계와 「확인자↔서버 셋」 6단계로 나뉘고, 뒤 6단계는 캐시가 살아 있으면 건너뛴다. 루트·TLD는 IP가 아니라 "누가 아는지"를 준다
- 새로 생긴 궁금증: 확인자가 `h.root-servers.net`의 IP는 어떻게 아나 (닭과 달걀) → 루트 힌트 파일이 확인자에 내장돼 있다는데, 그 13개 IP가 바뀌면 어떻게 갱신되나?

관련: [[학습/공부/CS/네트워크/도메인과 메일 MX|도메인과 메일 MX]] — 같은 DNS, 이쪽은 MX 레코드 · [[학습/공부/CS/네트워크/포트와 localhost|포트와 localhost]] — IP를 얻은 다음은 포트

## 참고 자료

- Cloudflare Learning «What is DNS?» — 붙여넣은 8단계의 출처. 확인자·루트·TLD·권한 네임서버 넷과 브라우저·OS·확인자 3단 캐시 설명 (2026-09-22 봇 차단(403)으로 URL 자동 확인 실패, 링크 생략)
- [RFC 1034 — Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034) — §2.4 네임서버(authority)·확인자 정의, §5.3.3 확인자 알고리즘(캐시 우선 · 위임을 받으면 다음 서버로) (2026-09-22 확인)
- [How DNS Works](https://howdns.works/) — 같은 8단계를 만화로 (2026-09-22 확인)
