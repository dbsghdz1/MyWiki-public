---
type: project
title: "PourCalc"
summary: "미국 DIY 사용자를 위한 콘크리트 자재량 계산기 웹사이트로, 구글 검색 유입과 웹 광고 수익 가능성을 시험한다"
status: active
aliases:
  - pourcalc
  - 콘크리트 계산기
  - Concrete Calculator
created: 2026-11-25
updated: 2026-11-25
repos:
  - "~/Developer/pourcalc — GitHub 원격 미생성"
related_wiki: []
launch_gate: exempt
launch_exception: "검색 유입 실험이라 20, 5, 2 게이트 대신 Search Console 3개월 노출 추이로 판정한다. 홍이 2026-11-25 대화에서 실험 계획을 듣고 바로 만들라고 지시했다"
launch_approved: 2026-11-25
---

# PourCalc

## 현재 카드

- **단계**: 7일 MVP
- **현재**: 2026-11-25 https://pourcalc.vercel.app 배포. 계산기 5개와 설명 글, sitemap, robots.txt 응답 200 확인, 계산 단위 테스트 9개 통과. 브라우저 화면 확인은 못 함(Aside 미실행). Search Console 미등록, 광고 없음
- **다음 판정**: 2026-12-25 도메인 구매 여부(계속할 마음이 들면 사서 옮김), 2027-02-25 Search Console 노출 판정(꾸준히 늘면 분야 확장과 애드센스 신청, 거의 없으면 접거나 방향 전환)
- **지금 할 일**: 홍이 Google 계정으로 Search Console에 URL 접두어 방식으로 등록하고 HTML 인증 태그를 넘기면, 태그를 넣어 재배포하고 `sitemap-index.xml`을 제출한다
- **하지 않을 일**: 검색 노출 확인 전 다른 분야(페인트, 바닥재 등) 계산기 추가

## 무엇

미국 DIY 사용자가 콘크리트를 몇 포대 사야 하는지 바로 알려 주는 계산기 사이트다. 영어로만 만든다.

| 페이지 | 내용 |
|---|---|
| `/concrete-slab-calculator` | 슬래브, 파티오, 진입로 |
| `/post-hole-concrete-calculator` | 울타리, 데크 기둥 구멍. 기둥이 차지하는 부피와 바닥 자갈을 뺀다 |
| `/sonotube-calculator` | 원통 거푸집(Sonotube) 기초 |
| `/concrete-steps-calculator` | 계단과 상단 참 |
| `/concrete-bag-calculator` | 부피를 80, 60, 40lb 포대 수로 변환 |

- 결과: 세제곱야드, 세제곱피트, m³, 포대 수, 레미콘 주문량(0.25야드 단위), 쇼핑 목록 복사
- 포대 수율은 표준 프리믹스 포대에 흔히 적힌 값(80lb 0.60, 60lb 0.45, 40lb 0.30 cu ft)을 쓰고 About 페이지에 밝혔다
- About, Privacy 페이지는 나중 애드센스 신청에 대비해 미리 넣었다

## 왜 이것

2026-11-25 대화에서 웹 광고 수익을 따져 본 결과다. 해석이며 이 세션에서 실측하지 않은 추정이다.

- MAU 5만 한국 트래픽은 애드센스로 월 10만~100만 원대이고, 한 번 쓰고 나가는 도구는 그 아래쪽이다
- 미국 트래픽 RPM은 한국의 3~5배이고 홈인테리어 주제는 광고 단가가 높다
- 대신 Omni Calculator, Calculator.net, Inch Calculator, Home Depot와 Lowe's가 이미 같은 계산기를 갖고 있다. 큰 사이트가 하나로 끝내는 계산을 상황별로 잘게 나누고(기둥 구멍, 계단 참 포함), UI와 쇼핑 목록으로 차별화하는 전략이다
- 홍은 이 분야를 잘 모른다는 점을 걱정했다. 계산은 제조사 공개값과 기하로 충분하지만, 사용자가 실제로 막히는 지점은 r/DIY 같은 곳을 읽어 채워야 한다. 대안으로 한국 관련 외국인 도구(아는 분야, 낮은 단가)도 비교했고 홍이 콘크리트를 골랐다

## 판정 방식

1. 도메인 없이 `vercel.app`으로 먼저 올렸다. Search Console은 URL 접두어 방식만 된다. 애드센스는 소유 도메인이 필요해 수익화 단계에서는 도메인을 사야 한다
2. 늦게 옮길수록 검색 신호 이전 손해가 커서, 한 달 안에 계속할지 정한다
3. 3개월 뒤 Search Console 노출 수로 계속 여부를 정한다

## 코드 저장소

- 로컬 `(로컬 경로)`, GitHub 원격 미생성. Astro 7.4 정적 사이트, `@astrojs/sitemap`, Vercel 프로젝트 `pourcalc`
- 계산 로직은 `src/lib/concrete.ts`(순수 함수), 테스트는 `npm test`(Node 내장 test runner로 TS 직접 실행)
- 커밋: `c0689ea` 계산기 5개 첫 배포
