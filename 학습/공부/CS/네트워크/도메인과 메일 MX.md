---
type: study
area: CS
audience: me
status: active
created: 2026-09-15
updated: 2026-09-16
aliases: [도메인, MX 레코드, DNS, 메일 호스팅, RDAP, whois]
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# 도메인과 메일 MX

**도메인은 이름표, 메일 호스팅은 우편함이다.** 둘은 따로고 DNS의 MX 레코드가 잇는다. (2026-09-16 [[학습/공부/CS/네트워크/네트워크|네트워크]]에서 분리)

## 핵심 정리

- **보내는 쪽 SMTP 서버가 받는 사람 도메인의 MX를 조회해 배달할 곳을 찾는다**(RFC 5321 §5.1). MX가 없으면 A/AAAA를 암묵적 MX로 쓴다.
- 그래서 주소는 **산 도메인 이름**으로 나온다(`hello@<도메인>`). 이미 쓰는 메일 서비스(iCloud+)에 도메인을 붙이면 기존 받은편지함으로 들어온다.
- **도메인은 사는 게 아니라 몇 년 단위로 빌리는 것.** 첫해 할인이 아니라 **연장 가격**을 비교한다.
- **미등록 확인은 한 조회만 믿지 않는다.** 범용 RDAP 중계가 404를 돌려줘도 whois로 교차 확인한다.

## 기록

### 2026-09-15 — 도메인을 사면 그 주소로 메일이 오나?

- 궁금했던 것: [[프로젝트/개인/Zappy/README|Zappy]]를 메뉴바 앱 디렉토리에 올리려는데 제출 폼이 Gmail을 거절하고 "앱 도메인 이메일"만 받았다. 도메인만 사면 되는지, iCloud도 새로 사야 하는지, 주소가 `@icloud.com`으로 나오는지 헷갈렸다.
- 예측: 도메인과 iCloud를 둘 다 사야 하고, 메일 주소는 iCloud 쪽 이름으로 나올 것 같았다 → **둘 다 틀렸다.**
- 파고든 길: 후보 도메인을 등록기관 RDAP(`rdap.verisign.com/com/v1/domain/<이름>`, 404 = 미등록)와 whois로 조회했다. `.io`는 `rdap.org`에서 404가 나왔는데 whois로 다시 보니 이미 등록된 이름이었다. 도메인 가격은 등록기관 가격표로 비교했다.
- 알게 된 것:
  - **도메인은 이름표, 메일 호스팅은 우편함이다.** 둘은 따로다. 도메인 DNS에 **MX 레코드**로 "이 도메인 메일은 이 서버로"를 적어야 이어진다. 보내는 쪽 SMTP 서버가 받는 사람 도메인의 MX를 조회해서 배달할 곳을 찾는다(RFC 5321 §5.1).
  - 그래서 주소는 **산 도메인 이름**으로 나온다(`hello@<도메인>`). 이미 쓰는 메일 서비스(iCloud+)에 도메인을 붙이면 기존 받은편지함으로 같이 들어온다.
  - 도메인은 "사는" 게 아니라 **몇 년 단위로 쓸 권리를 빌리는 것**이다(MDN). 그래서 첫해 가격보다 **연장 가격**을 비교해야 한다 — 첫해 할인이 있는 `.dev`는 둘째 해부터 `.com`보다 비쌌다.
  - 미등록 확인은 한 가지 조회만 믿으면 안 된다. 범용 RDAP 중계는 레지스트리가 RDAP을 안 주면 404를 돌려줄 수 있어서, 해당 레지스트리에 직접 묻거나 whois로 교차 확인한다.
- 근거: [RFC 5321 §5.1 — Locating the Target Host](https://www.rfc-editor.org/rfc/rfc5321#section-5.1) · [MDN — What is a domain name?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name) · [Apple — Use Custom Email Domain with iCloud Mail](https://support.apple.com/en-us/102540) (2026-09-15 확인)
- 새로 생긴 궁금증: MX만 넣으면 받기는 되는데, 내가 보낸 메일이 스팸함에 안 가려면 왜 SPF·DKIM·DMARC가 따로 필요할까? → 실제로 연결할 때 확인한다.

## 참고 자료

- [How DNS Works](https://howdns.works/) — DNS 조회 흐름을 만화로 (2026-08-15 확인)
- [RFC 5321 — SMTP, §5.1 Locating the Target Host](https://www.rfc-editor.org/rfc/rfc5321#section-5.1) — 메일 배달이 MX 레코드로 다음 서버를 찾고, MX가 없으면 A/AAAA를 암묵적 MX로 쓴다는 원전 (2026-09-15 확인)
- [MDN — What is a domain name?](https://developer.mozilla.org/en-US/docs/Learn_web_development/Howto/Web_mechanics/What_is_a_domain_name) — 도메인 구조·등록기관·"살 수 없고 사용권을 빌린다" (2026-09-15 확인)
- [Apple — Use Custom Email Domain with iCloud Mail](https://support.apple.com/en-us/102540) — iCloud+에 소유 도메인을 붙여 기존 받은편지함으로 받는 절차 (2026-09-15 확인)
