---
type: study
area: CS
audience: me
status: active
created: 2026-09-01
updated: 2026-09-16
aliases: [cron, crontab, 스케줄러, 데몬, Vixie cron]
projects:
  - "계획"
---

# cron

**OS 데몬이 1분마다 깨어나 crontab을 보고 명령을 대신 실행한다.** 5필드 `분 시 일 월 요일`, 함정은 시간대(UTC)·실행 환경·조용한 실패. (2026-09-16 [[학습/공부/CS/운영체제/운영체제|운영체제]]에서 분리)

## 핵심 정리


- **cron은 유닉스 계열 OS에 상주하는 데몬(백그라운드 프로세스)**이다. 사용자가 로그인해 있지 않아도 **정해진 시각에 정해진 명령을 대신 실행**한다. 1975년 벨연구소, Version 7 Unix부터. 지금 리눅스·BSD의 표준 구현은 Paul Vixie의 **Vixie cron(1987)**. 이름은 시간의 신 **Chronos**.
- **동작 원리는 단순하다**: 데몬이 **1분마다 깨어나** 각 사용자의 `crontab`(cron table) 파일을 읽고, "지금 이 분"에 맞는 줄이 있으면 그 명령을 자식 프로세스로 실행하고 다시 잔다. 그래서 **최소 단위가 1분**이고 초 필드가 없다.
- **표현식은 5필드 + 명령**: `분 시 일 월 요일 <명령>`. 범위는 분 0–59 · 시 0–23 · 일 1–31 · 월 1–12 · 요일 0–6(0=일요일; Vixie는 7도 일요일). `*`=모든 값, `*/N`=N마다, `a-b`=범위, `a,b`=목록. `@daily`·`@hourly`·`@reboot` 같은 별칭도 있다(Vixie 확장, POSIX엔 없음).
  - 내 루틴 셋: `0 22 * * *` = 매일 22:00 UTC(= 07:00 KST) · `0 2 * * 2` = 화요일 02:00 UTC(= 화 11:00 KST) · `0 11 * * 0` = 일요일 11:00 UTC(= 일 20:00 KST).
- **함정 셋** — 전부 이 볼트 운영에서 실제로 겪은 것:
  1. **시간대는 데몬이 도는 곳의 시간대다.** Claude 클라우드 루틴은 UTC라 KST에서 9시간을 빼서 써야 하고, 22:00 UTC는 **다음 날** 07:00 KST다 — 루틴 프롬프트마다 "UTC 날짜 그대로 쓰면 하루 틀린다"가 박혀 있는 이유.
  2. **실행 환경이 로그인 셸과 다르다.** PATH가 최소(`/usr/bin:/bin`)이고 `.zshrc` 같은 건 안 읽는다 — 터미널에서 되던 명령이 cron에서만 "command not found"가 나는 고전적 원인.
  3. **실패가 조용하다.** 결과를 보는 사람이 없으니 안 돌아도 아무도 모른다. 감시가 따로 필요하다 — Hermes 07:20 "일간 파일 없음" 보고가 07:00 Claude 루틴의 감시 장치인 것이 그 설계.
- **"cron"은 표현식 문법의 이름으로도 통용된다.** Claude 클라우드 루틴은 리눅스 cron 데몬을 쓰는 게 아니라 Anthropic 스케줄러가 cron 표현식으로 시각만 받는 것이고(최소 간격 1시간은 그 서비스 제한, cron 자체 제한이 아님), Oracle 서버의 Hermes cron은 hermes 프로그램 안의 자체 스케줄러다. GitHub Actions `schedule:`, Kubernetes `CronJob`, Spring `@Scheduled(cron=…)`(이건 초 필드가 붙어 6자리)도 같은 문법을 빌려 쓴다.

## 기록

### 2026-09-01 — cron: 루틴 표에 나오는 "cron"이 뭔지
- 맥락: 계획 루틴 운영 — 주간 소마·일정 채우기 루틴(`0 2 * * 2`)을 만들고 "지금 루틴이 어떻게 돌아가는지 어디를 보나"를 설명하던 중 "cron이라는게 뭐야?" 질문
- 배운 것: 위 핵심 정리 「cron」 절. 요점 — **OS 데몬이 1분마다 깨어나 표를 보고 명령을 대신 실행**하는 것, 5필드 문법, 시간대·실행 환경·조용한 실패 세 함정
- 근거: crontab(5) man page · POSIX crontab 스펙 · 위키 `계획/README.md` 루틴 표의 실제 cron 식 3개 (아래 참고 자료)


## 참고 자료

- [crontab(5) — Linux man page (man7.org)](https://man7.org/linux/man-pages/man5/crontab.5.html) — cron 표 문법의 원전. 5필드 범위, `*`·`/`·`-`·`,`, `@daily`류 별칭, 환경변수 규칙 (2026-09-01 확인)
- [POSIX crontab — IEEE Std 1003.1-2017](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/crontab.html) — 표준이 보장하는 최소 문법(5필드, 요일 0=일요일). 별칭·`*/N`은 여기 없다 → Vixie 확장임을 확인하는 용도 (2026-09-01 확인)
- [Cron — Wikipedia](https://en.wikipedia.org/wiki/Cron) — 역사(1975 벨연구소·V7 Unix·Vixie cron 1987), 데몬 동작 방식, Chronos 어원 (2026-09-01 확인)
- [crontab.guru](https://crontab.guru/) — cron 식을 사람 말로 풀어주는 편집기. `0 2 * * 2`가 맞는지 눈으로 확인할 때 (2026-09-01 확인)
