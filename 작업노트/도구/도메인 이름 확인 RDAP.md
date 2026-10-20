---
type: study
area: 도구
audience: ai
status: active
created: 2026-10-20
updated: 2026-10-20
projects:
  - "일상 블로그"
---

# 도메인 이름 확인 RDAP

서비스나 블로그 이름 후보가 생겼을 때 **도메인이 비어 있는지, 프리미엄 값이 붙는지, Cloudflare Registrar에서 살 수 있는지**를 로그인 없이 확인하는 방법. 2026-10-20 일상 블로그 이름을 고르며 정리했다. 결제는 사람이 등록기관 화면에서 한다.

## 비어 있는지: 등록기관 RDAP를 직접 찌른다

- TLD별 RDAP 서버는 IANA 부트스트랩 `https://data.iana.org/rdap/dns.json`에 있다. `.com`과 `.net`은 `https://rdap.verisign.com/com/v1/domain/<이름>`, `.day`와 `.page`와 `.app`은 `https://pubapi.registry.google/rdap/domain/<이름>`, `.life`와 `.today`와 `.world`와 `.kitchen`과 `.recipes`는 `https://rdap.identitydigital.services/rdap/domain/<이름>`, `.blog`는 `https://rdap.blog.fury.ca/rdap/domain/<이름>`.
- HTTP 200이면 등록됨, 404면 등록 기록 없음. 200일 때 JSON의 `events`에 등록일과 만료일, `entities`에 등록기관, `nameservers`에 네임서버가 있다(예: haru.day는 2022-02 등록, 2028-02 만료).
- **`rdap.org`의 404를 「비어 있음」으로 읽지 말 것.** rdap.org가 서버를 모르는 TLD(`.blog`)는 모든 이름이 404로 나온다. wordpress.blog까지 404였다. 등록기관 서버로 다시 확인해야 한다.
- Identity Digital은 연달아 부르면 429를 준다. 요청 사이에 1초 쉬면 된다.
- zsh에서 `for n in $names`는 문자열을 단어로 나누지 않는다. `printf '%s\n' a b c | while read -r n`로 돌린다.

## 프리미엄인지: 404라도 값은 따로 본다

- 짧거나 흔한 이름(hong.day, hong.blog, haru.blog, seoul.blog)은 등록 기록이 없어도 등록기관이 프리미엄 값을 매긴다. RDAP에는 값이 없다.
- **Cloudflare Registrar는 프리미엄 도메인을 등록하지 못한다**([API 문서](https://developers.cloudflare.com/api/typescript/resources/registrar): premium tier는 registration 미지원). 대시보드의 도메인 검색은 로그인이 필요하다.
- 로그인 없이 값을 보려면 Porkbun 공개 검색 `https://porkbun.com/checkout/search?q=<이름>`을 Aside로 열어 snapshot을 읽는다. 「Premium」 배지와 「renews at」 값이 같이 보인다. 2026-10-20 실측: hong.day 연 $141.35, hong.blog 첫해 $135.89 뒤 연 $546.35, haru.blog 첫해 $54.02 뒤 연 $218.86, girok.day 연 $10.81(표준). GoDaddy의 `find.godaddy.com/domainsapi`는 curl에 Access Denied를 준다.

## Cloudflare에서 살 수 있는 TLD와 값

- 지원 목록은 [Cloudflare TLD policies](https://www.cloudflare.com/tld-policies/). `.com`, `.net`, `.org`, `.blog`, `.day`, `.page`, `.dev`, `.app`, `.life`, `.me`, `.co`, `.kitchen`, `.recipes`, `.cooking`, `.today`, `.travel`, `.world`는 되고 **`.kr`, `.jp`, `.cn`, `.tw`는 없다.** 한국 도메인이 필요하면 다른 등록기관에서 사고 DNS만 Cloudflare에 둔다.
- 원가 판매(등록기관과 ICANN 가격, 마진 없음, [Registrar FAQ](https://developers.cloudflare.com/registrar/faq/)). 2026-10 가격 추적 기준 `.com` 연 $10.46, `.blog` 연 $20.20, `.day` 연 $10.20이고 갱신도 같은 값([domainoffer.net](https://domainoffer.net/tld/com/cloudflare)).

## 이름을 고를 때 본 것

- 2음절 한국어 단어(기록, 담다, 소풍, 마실, 단골, 조각, 동네, 일기)는 `.com`이 전부 팔려 있다. `.blog`와 `.day`는 대부분 비어 있고 표준 가격이다.
- 해외 충돌은 웹 검색(미국 결과 위주)으로, 국내 충돌은 네이버 통합검색을 Aside로 열어 본다. 소풍(Sopung)은 튀르키예와 말레이시아의 한식 체인과 겹쳐 뺐고, 기록(girok)은 같은 로마자를 쓰는 카페와 개인 계정만 있어 통과했다.

## 근거

- IANA RDAP 부트스트랩: https://data.iana.org/rdap/dns.json
- Cloudflare 이전 안내(프리미엄과 IDN 미지원): https://blog.cloudflare.com/a-step-by-step-guide-to-transferring-domains-to-cloudflare
- 결정 기록(비공개): 이름과 도메인 2026-10-20
