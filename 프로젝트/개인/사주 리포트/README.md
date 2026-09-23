---
type: project
title: "사주 리포트"
summary: "생년월일시를 넣으면 만세력 원국을 세우고 AI가 즉시 문서 풀이를 써 주는 웹 서비스"
status: active
aliases:
  - saju-report
  - 사주 풀이
created: 2026-09-23
updated: 2026-09-23
repos:
  - "~/Desktop/saju-report — (GitHub 미등록)"
related_wiki: []
launch_gate: exempt
launch_exception: "남는 토큰으로 만드는 실험 — 홍 지시 2026-09-23 '그냥 토큰 남아서 만드려구 서비스 하나를'"
launch_approved: 2026-09-23
launch_evidence: "161/0/0"
launch_first_ten: "미정 — 실험이라 유통 조건 없음"
launch_channels: []
launch_continue_if: "해당 없음"
launch_stop_if: "해당 없음"
---

# 사주 리포트

## 현재 카드
- **단계**: 7일 MVP
- **현재**: 2026-09-23 MVP 완성 — 만세력 원국(년·월·일·시주, 십신, 지장간, 오행, 대운) + Claude 스트리밍 풀이. 로컬 mock 모드로 화면·스트리밍 검증 완료, 실제 모델 호출은 API 키 대기
- **다음 판정**: Anthropic API 키를 넣고 실제 풀이 1건 → 품질 확인 → Vercel 배포
- **지금 할 일**: `ANTHROPIC_API_KEY` 설정 후 실제 풀이 품질 확인
- **하지 않을 일**: 결제·계정·저장 기능 (실험 범위 밖)

## 배경

크몽 실측(크몽 수요 실측 — 비공개)에서 사주 문서 풀이는 상위 gig 리뷰 1.5만 건·3만 원인데 **결과물이 2주~8개월 밀린다**는 불만이 최대 덩어리였다(1~3★ 161건 중 납기 21건). 사람 처리량이 병목인 자리에 "즉시 납기"를 놓아 보는 실험. 20·5·2 게이트는 면제(홍 승인 2026-09-23, 비수익 실험).

## 만든 것 (2026-09-23)

- **스택**: Vite + React + TS 클라이언트, Hono 서버(로컬은 `@hono/node-server`, 배포는 `hono/vercel`), `lunar-javascript`로 만세력, `@anthropic-ai/sdk` 스트리밍(`claude-opus-5`, adaptive thinking, effort medium, 서버측 fallback `claude-opus-4-8`).
- **입력**: 양력/음력(윤달)·시각 모름 허용·성별·진태양시 보정(-30분, 기본 켬)·질문 300자.
- **계산**: 절기 기준 월주, 야자시는 당일 일주 유지(`setSect(2)`), 대운 8개, 올해 세운, 오행 분포(지장간 제외).
- **안전장치**: IP당 시간당 5회(`RATE_LIMIT_PER_HOUR`), zod 검증, `MOCK_LLM=1`이면 모델 호출 없이 가짜 스트림. 입력은 저장하지 않는다.
- **검증**: 2000-01-01 12:00 → 己卯 丙子 戊午 戊午 (알려진 값과 일치). 00:10 + 진태양시 → 전날 23:40 야자시 처리 확인. 음력 1999-11-25 → 양력 2000-01-01 확인.

## 실행

```
cp .env.example .env   # ANTHROPIC_API_KEY 채우거나 MOCK_LLM=1
npm run dev            # api :8787 + web :5173
npm run typecheck
```

## 배운 것

(아직 없음)
