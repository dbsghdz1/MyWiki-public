---
type: project
title: "BookMini"
summary: "MacBook을 맥미니처럼 상시 가동 호스트로 쓰게 하는 내부 도구 — 잠자기 방지·살아있음 감시·로그인 세션 점검"
status: active
aliases:
  - book mini agent
  - 맥북 상시 가동
created: 2026-09-12
updated: 2026-09-12
repos:
  - "~/Desktop/개인 앱/BookMini — 로컬 git"
related_wiki: []
launch_gate: exempt
launch_exception: "내부 자동화 — 홍보 오토파일럿의 실행 호스트(Mac)를 밤새 살려두는 도구. 판매를 검토할 때 20·5·2를 따로 한다"
launch_approved: 2026-09-12
---

# BookMini

## 현재 카드
- **단계**: 7일 MVP
- **현재**: 2026-09-12 v0 설치·가동. Mac launchd 3개(`com.hong.bookmini.awake`·`.heartbeat`·`.sessions`)가 돌고 `PreventSystemSleep 1`이 켜졌다. 하트비트가 Oracle `(로컬 경로)`에 도착하고, 서버 `bookmini-watch.timer`(5분)가 등록됐다. 로그인 점검 드라이런 결과: X `@DevHongX`·Instagram은 확정, Threads는 약한 판정. 로컬 git `fb5c73b`·`a5b8069`
- **다음 판정**: 2026-09-19 — 7일 밤 동안 하트비트 공백 없이 돌았는가. 스크립트로 안 풀린 불편이 남았으면 그것만 앱으로 만든다
- **지금 할 일**: 7일간 그대로 두고 매일 아침 `bookmini-status`와 Slack 경보 유무만 본다 — 끊김이 나오면 원인을 여기 기록한다
- **하지 않을 일**: 메뉴바 앱·판매·공개(7일 검증 전), 게시 자동화 자체(→ 홍보 자동화 몫)

## 왜 만드나

**홍이 직접 부딪힌 문제다** (2026-09-12).

- 홍보 오토파일럿의 게시를 API 대신 **Mac 화면(Aside 브라우저) 조작**으로 하기로 했다(설계안). 그러려면 MacBook이 밤새 깨어 있어야 한다.
- 실측해 보니 네 가지가 막혀 있었다([[작업노트/도구/MacBook 상시 가동|MacBook 상시 가동]]).
  - 잠자기 방지 장치가 없었다. 보이던 `caffeinate`는 Claude Code 세션이 작업 중에만 거는 것이었다.
  - AC 전원에서도 입력 없이 1분이면 잠든다.
  - FileVault 때문에 재부팅 뒤 멈춘다.
  - 원격으로 살릴 경로가 없다.
- 홍: *"맥북을 맥미니 처럼 만들수 있는 무언가"* → *"실제로 내가 불편해서 만드는 거니 괜찮지 않을까"* → *"하던거 하자"*.

## 20·5·2 게이트와의 관계

**내부 도구로 면제 받았다**(`launch_gate: exempt`, 홍 승인 2026-09-12).

- 이유: 판매가 목적이 아니라 홍의 실행 호스트를 살리는 도구다.
- 판매를 검토하는 순간 20·5·2를 따로 한다. 불만 신호 후보는 아이디어 인박스에 적어뒀다(hermes-agent #101991·#95819 등).
- **잠자기 방지 단독 제품은 얇다**고 판단했다: Amphetamine이 무료이고, Hermes에 이미 내장돼 있다.

## v0 설계 (스크립트 먼저)

| 조각 | 위치 | 주기 | 하는 일 |
|---|---|---|---|
| `bookmini-awake` | Mac launchd (KeepAlive) | 상시 | `caffeinate -i -s` — 전원 연결 시 시스템 잠자기를 막는다. 디스플레이는 설정대로 꺼진다 |
| `bookmini-heartbeat` | Mac launchd | 300초 | 시각·유휴·전원·배터리·덮개 상태를 Oracle `(로컬 경로)`에 쓴다 |
| `bookmini-sessions` | Mac launchd | 1800초 | **입력 유휴가 10분 이상일 때만** Aside로 X·Threads·Instagram 로그인 상태를 본다. 로그아웃이면 Slack으로 알린다 |
| `bookmini_watch.py` | Oracle systemd user timer | 300초 | 하트비트가 15분 넘게 없으면 Slack에 한 번 알리고, 돌아오면 한 번 더 알린다 |

**설치 위치**: launchd가 실행하는 파일은 `(로컬 경로)`에 둔다.
- 이유: 옛 `com.instacardnews.publish`가 `(로컬 경로)`의 스크립트를 부르다 exit 127로 죽어 있었다(TCC로 추정).
- 소스는 `(로컬 경로)`에 두고 `install.sh`로 복사한다.

## 코드 저장소

- 로컬: `(로컬 경로)` (git, 원격 없음)

## 기록

- 2026-09-12 — 착수. 면제 승인, v0 범위 확정.
- 2026-09-12 — **v0 설치·가동.**
  - 서버 감시 판단 로직 단위 테스트 5개 통과.
  - **첫 설치에서 하트비트가 launchd로 돌 때 로그 없이 `last exit code = 141`로 죽었다.** 원인은 `set -o pipefail` + `pmset -g batt | head -1`의 SIGPIPE — 출력을 변수에 담고 파싱하도록 고쳤다(`a5b8069`). 손으로 실행하면 대개 멀쩡해 보여서 놓치기 쉽다 → [[작업노트/도구/MacBook 상시 가동|MacBook 상시 가동]].
  - 서버 감시 드라이런: 끊긴 하트비트·파일 없음 두 경우 모두 경보 문구 확인.
  - 로그인 점검은 홍 사용 중(유휴 93초)이라 `skip`으로 정상 동작.
  - 덮개를 닫은 채(외부 모니터) AC 연결 상태에서 설치했다.
