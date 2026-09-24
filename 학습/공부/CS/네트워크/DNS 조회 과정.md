---
type: study
area: CS
audience: me
status: active
created: 2026-09-22
updated: 2026-09-24
aliases: [DNS, 재귀 확인자, 리졸버, 루트 서버, TLD, 네임서버, dig, Anycast, CNAME]
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

### NS와 A — "누가 아는지"와 "주소"

- **네임서버** = DNS 질문에 답하는 컴퓨터. 단, **자기 구역만** 안다. Cloudflare 네임서버에 `example.com`을 물으면 `flags: aa`(authoritative answer)로 답하고, `google.com`을 물으면 `status: REFUSED`.
- `NS` 레코드 = "이 이름 담당 네임서버는 얘" (루트·TLD가 주는 것). `A` 레코드 = Address, IPv4 주소 (마지막 담당만 줌).
- 그래서 **IP를 바꾸면 고칠 곳은 권한 네임서버(Cloudflare) 하나**다. 루트·TLD는 "Cloudflare에 물어봐"만 들고 있다.
- **한 이름에 여러 대**가 반복된다 — 확인자 2개, 루트 13, `.com` 13, 네임서버 2, `A` 2개. 한 대 죽어도 멈추지 않게.

### TTL은 저울이다

- 서버 이전 전엔 TTL을 낮춘다. 단 **낮춘 설정도 옛 TTL이 끝나야 퍼지므로** 옛 TTL만큼 먼저 낮춰 둔다: 낮춤 → 옛 TTL 대기 → IP 변경 → 다시 올림.
- 평소엔 높게: 짧으면 질의 비용↑, 캐시 미스마다 수백 ms(루트·TLD·네임서버 왕복), 네임서버 장애 시 캐시로 버티지 못한다.

### 루트 서버 — 13개 IP, 기계는 수천 대 (Anycast)

- 루트는 `.`을 맡은 네임서버. 아는 건 "각 TLD 담당이 누구인가"뿐 — `dig @a.root-servers.net example.com`은 `ANSWER: 0`, `com. NS gtld-servers` 13개만 준다.
- **Anycast** = 여러 기계가 **같은 IP**를 광고하고 라우터가 경로상 가장 가까운 기계로 보낸다(대표번호처럼). 유니캐스트는 IP 하나 = 기계 하나.
- 한국(KT)에서 `id.server CH TXT`로 "너 누구?"를 물은 결과: `e`·`f`·`j`·`l`·`m`은 인천·서울 기계가 6~8ms, `h`·`k`는 도쿄 38~49ms, `a`는 시애틀·`c`는 LA 140ms대. **가깝다 = 지도 거리가 아니라 네트워크 경로** — `a`는 한국에서 물어도 시애틀로 갔다.
- 확인자는 13개의 응답 시간을 재 두고 빠른 곳을 쓴다. 게다가 `com.` NS 답은 TTL 172800(2일)이라 루트에 묻는 일 자체가 드물다.
- 운영: 루트는 ICANN 감독, TLD 배정은 IANA. TLD는 gTLD(`.com`·`.dev`, `gtld-servers`의 g)와 ccTLD(`.kr`·`.jp`).

> Cloudflare 글은 루트 인스턴스를 "600개 이상"이라 쓰지만 root-servers.org는 2026-09-24 기준 2045개 운영 중이라 표시한다 — 글이 오래됐다.

### CNAME — "이 이름은 저 이름의 별명"

| 레코드 | 답 |
|---|---|
| `A` | IP는 이거 |
| `NS` | 이 **구역**은 저 네임서버한테 물어봐 |
| `CNAME` | 이 **이름**은 저 이름과 같다, 저 이름으로 처음부터 다시 찾아 |

```
www.naver.com.   13412  CNAME  www.naver.com.nheos.com.   ← 거의 안 바뀜, TTL 김
www.naver.com.nheos.com.  8  A  223.130.192.248 외 3       ← 수시로 바뀜, TTL 8초
```

- **IP 관리를 남의 장부로 넘기는 장치.** `blog.hong.dev CNAME cname.vercel-dns.com`이면 IP(`76.76.21.98`)는 Vercel 네임서버에 적혀 있어서, Vercel이 IP를 바꿔도 내 Cloudflare 장부는 고칠 게 없다. 내가 `A`로 IP를 직접 적었으면 내가 고쳐야 한다. Vercel·Netlify가 CNAME 연결을 안내하는 이유.

## 기록

### 2026-09-22 — Cloudflare «What is DNS?»의 8단계를 붙여넣고 "자세히"

- 맥락: MDN Learn → 도메인·호스팅에서 이어진 질문. Cloudflare Learning 글의 8단계 영한 대역을 붙여넣음
- 파고든 길: `scutil --dns`로 내 확인자 확인 → `dig +trace example.com`으로 루트·TLD·네임서버 세 번의 위임을 실제로 봄 → 같은 질의 두 번으로 TTL 카운트다운 확인 → RFC 1034 §2.4·§5.3.3
- 결론: 8단계는 「나↔확인자」 2단계와 「확인자↔서버 셋」 6단계로 나뉘고, 뒤 6단계는 캐시가 살아 있으면 건너뛴다. 루트·TLD는 IP가 아니라 "누가 아는지"를 준다
- 새로 생긴 궁금증: 확인자가 `h.root-servers.net`의 IP는 어떻게 아나 (닭과 달걀) → 루트 힌트 파일이 확인자에 내장돼 있다는데, 그 13개 IP가 바뀌면 어떻게 갱신되나?

### 2026-09-24 — 이틀 뒤 복습: 내가 어디까지 기억하나

- 맥락: 같은 Cloudflare 글로 이해 점검을 요청
- 예측(기억): "브라우저가 서버에 묻고, com 안에서 example을 찾아 IP를 돌려준다" → **계층은 맞았지만 세 군데가 비었다**: ① 브라우저는 확인자 한 곳(KT `168.126.63.1`)에만 묻는다 ② `.com` 위에 루트가 있다 ③ `.com` 서버는 IP가 아니라 NS(누가 아는지)를 준다
- 파고든 길: `dig +trace`에서 앞 세 덩어리가 전부 `NS`, 마지막만 `A`임을 확인 → Cloudflare 네임서버에 직접 `example.com`(aa) vs `google.com`(REFUSED) → 5초 간격 질의로 TTL 299→294→289 카운트다운(첫 값 122는 KT 내부의 다른 캐시 서버로 추정)
- 결론: 네임서버는 자기 구역만 아는 답변 컴퓨터. IP 변경은 권한 네임서버만 고친다. TTL은 바꾸기 쉬움 ↔ 빠름·튼튼함의 저울
- 새로 생긴 궁금증: 네임서버를 Cloudflare에서 다른 곳으로 옮기면 그땐 `.dev` 쪽 NS를 고쳐야 할 텐데, TLD 서버에 내가 직접 쓸 수는 없을 테니 누가 대신 고쳐 주나? (등록기관?)

### 2026-09-24 (이어서) — Cloudflare «DNS server types»: 루트 서버가 뭐야, CNAME은?

- 맥락: 같은 Cloudflare Learning의 서버 유형 글을 붙여넣고 "루트 서버가 뭐야"
- 예측: 한국에서 `k` 루트를 물었는데 도쿄가 답한 이유 → "가까워서, 한국엔 없고 일본엔 있어서" → **가까워서는 맞고, 기준은 지도 거리가 아니라 경로**. 한국 기계 유무는 이 결과로 알 수 없다
- 파고든 길: 13개 루트 전부에 `id.server CH TXT` + 응답 시간 → `traceroute`로 `k`(NTT망 경유 44ms)·`l`(17ms) 경로 비교 → `www.github.com`·`www.naver.com`의 CNAME 체인 → `cname.vercel-dns.com`의 A와 NS
- 결론: 루트 "13개"는 IP 13개, 뒤에 Anycast 기계 수천 대. CNAME은 IP 관리를 남의 장부로 넘기는 것 — Vercel IP가 바뀌어도 내 장부는 그대로
- 새로 생긴 궁금증: 루트 도메인(`hong.dev` 자체, `www` 없이)에는 CNAME을 못 쓴다고 들었는데 왜일까? Cloudflare의 "CNAME flattening"은 그걸 어떻게 우회하나?

관련: [[학습/공부/CS/네트워크/도메인과 메일 MX|도메인과 메일 MX]] — 같은 DNS, 이쪽은 MX 레코드 · [[학습/공부/CS/네트워크/포트와 localhost|포트와 localhost]] — IP를 얻은 다음은 포트

## 참고 자료

- Cloudflare Learning «What is DNS?» — 붙여넣은 8단계의 출처. 확인자·루트·TLD·권한 네임서버 넷과 브라우저·OS·확인자 3단 캐시 설명 (2026-09-22 봇 차단(403)으로 URL 자동 확인 실패, 링크 생략)
- [RFC 1034 — Domain Names: Concepts and Facilities](https://www.rfc-editor.org/rfc/rfc1034) — §2.4 네임서버(authority)·확인자 정의, §5.3.3 확인자 알고리즘(캐시 우선 · 위임을 받으면 다음 서버로) (2026-09-22 확인)
- [How DNS Works](https://howdns.works/) — 같은 8단계를 만화로 (2026-09-22 확인)
- Cloudflare Learning «What are the different types of DNS server?» — 확인자·루트·TLD·권한 네임서버 넷, Anycast, CNAME (2026-09-24 역시 403, 링크 생략)
- [IANA — Root Servers](https://www.iana.org/domains/root/servers) — a~m 13개 루트와 운영 기관 (2026-09-24 확인)
- [root-servers.org](https://root-servers.org/) — 루트 인스턴스 위치 지도, 2045개 운영 중 (2026-09-24 확인)
