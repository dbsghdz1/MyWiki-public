---
type: study
area: 백엔드
audience: me
status: active
created: 2026-09-12
updated: 2026-09-12
projects: []
---

# UTM 파라미터와 유입 분석

URL 뒤의 `utm_*`는 **링크를 만든 쪽이 "이 방문은 어디서 왔다"를 직접 적어 붙인 꼬리표**다. 서버 동작은 바뀌지 않고 분석 도구만 읽는다. 그중 `utm_medium`은 **"어떤 종류의 통로로 왔나"**(social·email·cpc)를 적는 칸이다.

## 핵심 정리

- **왜 필요한가**: 분석 도구가 유입 경로를 아는 기본 수단은 `Referer` 헤더다. 그런데 메일 앱이나 SNS 앱 같은 **네이티브 앱에서 브라우저로 넘어오면 보낼 "이전 웹페이지"가 없어** 비는 경우가 많고, 그러면 `(direct)`로 뭉친다. 그래서 출처를 링크 자체에 적어 넣는다.
- **칸 다섯 개** (Google 공식 정의):

  | 파라미터 | 뜻 | 예 |
  |---|---|---|
  | `utm_source` | **누가** 보냈나 — 사이트·매체 이름 | `google`, `newsletter4`, `ig` |
  | `utm_medium` | **어떤 종류의 통로**로 — 마케팅 매체 | `cpc`, `email`, `social` |
  | `utm_campaign` | **무슨 캠페인**으로 | `spring_sale` |
  | `utm_content` | 같은 곳 안에서 **어느 링크·소재** | `toplink`, `link_in_bio` |
  | `utm_term` | 유료 검색 키워드 | |

  Google은 source·medium·campaign 셋은 항상 쓰라고 권한다.
- **source는 고유명사, medium은 분류다.** "인스타그램"은 source, "소셜"은 medium. 뉴스레터 이름은 source, "email"은 medium.
- **채널은 medium이 정한다.** GA4 기본 채널 그룹에서 **Organic Social** = source가 소셜 사이트 목록과 맞거나, **또는** medium이 `social`·`social-network`·`social-media`·`sm`·`social network`·`social media` 중 하나일 때다. 그래서 `utm_source=ig`가 목록에 없는 약칭이더라도 `medium=social`이 있으면 소셜로 잡힌다. 소셜 사이트 source에 medium이 `cp` 포함·`ppc`·`retargeting`·`paid…`면 **Paid Social**이다. 채널 분류 규칙 자체는 대소문자를 구분하지 않는다.
- **값은 글자 그대로 따로 집계된다.** `ig`·`instagram`·`Instagram`은 리포트에서 세 줄로 갈린다. 해법은 이름 규칙을 한 곳에 정해 두는 것뿐이다.
- **서버 입장에선 그냥 쿼리 스트링이다.** `?utm_...`를 떼도 페이지는 똑같이 뜨고, 페이지에 심은 분석 스크립트가 URL에서 읽어 방문 기록에 붙인다.

## 기록

### 2026-09-12 — 인스타에서 넘어간 링크의 `utm_medium=social`

- 맥락: 인스타그램에서 어떤 사이트로 넘어갔더니 URL에 `utm_source=ig&utm_medium=social&utm_content=link_in_bio`가 붙어 있었다. 이걸 보고 "utm_medium=social이 뭐야"라고 물었다. 프로젝트 작업 중은 아니었다.
- 배운 것:
  - 해석: 인스타(`ig`)에서 소셜 통로(`social`)로, 프로필 소개란 링크(`link_in_bio`)를 눌러 들어온 방문이다. 사이트 운영자의 GA4에는 **Organic Social**로 잡힌다.
  - **`utm_campaign`이 없다.** Google이 늘 쓰라고 권하는 칸이 빠졌다는 건, 캠페인마다 사람이 만든 링크라기보다 **고정된 bio 링크에 일괄로 붙은 값**일 가능성이 크다. 누가 붙였는지는 확인하지 못했다(아래 막힌 것).
- 근거: 아래 참고 자료의 GA4 도움말 두 문서. 이 문자열을 인스타가 자동으로 붙이는지 검색했지만 공식 근거는 찾지 못했다.

## 막힌 것

- 이 `ig / social / link_in_bio` 조합은 Instagram이 bio 링크에 자동으로 붙이는가, 아니면 링크인바이오 도구(Later 등)가 붙이는가? Later 도움말 「Link in Bio UTM Parameters」가 답일 수 있지만 403으로 열리지 않았다.
- Instagram 인앱 브라우저는 `Referer`를 보내는가? 서드파티 글들은 "보내지 않는다"고 하지만 원전은 확인하지 못했다.

## 더 알아보면 좋은 것

- 앱 설치 유입은 UTM으로 이어지지 않는다. 앱스토어를 거치면서 연결이 끊기므로 AppsFlyer·Airbridge 같은 어트리뷰션 툴이 따로 필요하다 → [[작업노트/도구/GA4와 Amplitude 앱 계측|GA4와 Amplitude 앱 계측]]
- 쿼리 스트링 값의 인코딩 규칙 → [[학습/공부/CS/URL과 퍼센트 인코딩|URL과 퍼센트 인코딩]]

## 참고 자료

- [GA4 — Collect campaign data with custom URLs](https://support.google.com/analytics/answer/10917952) — utm 파라미터 공식 정의와 필수 여부 (2026-09-12 확인)
- [GA4 — Default channel group](https://support.google.com/analytics/answer/9756891) — medium·source 값이 Organic/Paid Social 등 채널로 나뉘는 규칙 원문 (2026-09-12 확인)
- [How to Track Traffic From Instagram — Later](https://later.com/blog/track-traffic-from-instagram/) — 인스타 링크에 UTM을 붙이는 실무 예시, 대소문자를 일관되게 쓰라는 경고 (2026-09-12 확인)
