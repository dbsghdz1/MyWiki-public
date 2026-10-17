---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-30
updated: 2026-10-17
projects:
  - "계획"
---

# Claude 클라우드 루틴 운영

> 2026-10-17부터 홍은 Codex를 주로 쓰되 Claude Code 설정은 복제본으로 유지한다. 이 노트는 그대로 유효하다. Codex 쪽 대응은 [Codex CLI 설정과 Claude Code 이식](Codex%20CLI%20%EC%84%A4%EC%A0%95%EA%B3%BC%20Claude%20Code%20%EC%9D%B4%EC%8B%9D.md).

한 줄 요약 — 루틴 실행 상태 `ROUTINE_RUN_STATUS_SUCCEEDED`는 "세션이 정상 종료됨"일 뿐 **작업이 수행됐다는 뜻이 아니다**. 그리고 CCR 샌드박스의 git 상태는 실제 저장소와 다르다 — detached HEAD가 진짜 최신이고, `refs/remotes/origin/main`은 낡은 캐시일 수 있으며, merge-base 실패는 이력 단절이 아니라 샌드박스 아티팩트다.

## 핵심 정리

- **진단 경로**: `/schedule` 스킬 → `RemoteTrigger` `list` (루틴별 `last_run.status`·`next_run_at`) → `list_runs` → `get_run_log <session_id>`. 상태가 SUCCEEDED여도 커밋이 원격에 없으면 로그를 읽어야 한다 — 루틴이 스스로 판단해 push 없이 종료한 경우가 실제로 있었다.
- **CCR 샌드박스 git 상태의 함정** (2026-08-29 실측, run `cse_01QzpHvADwJer7UYvpseDxkw`):
  - 체크아웃은 `HEAD detached from refs/heads/main` — 이 detached HEAD(`43d3b88`)가 **실제 GitHub main의 최신 커밋**이었다.
  - 그런데 샌드박스 안의 `refs/remotes/origin/main`은 6일 전 커밋(`9e0def6`, 08-23)을 가리켰고, `git merge-base 43d3b88 9e0def6`는 **공통 조상 없음**을 반환했다(얕은/그래프트된 이력으로 추정).
  - 실제 저장소에서 같은 명령은 `9e0def6`을 반환한다 — 즉 정상적인 조상 관계다. **샌드박스의 merge-base·remote-tracking ref로 "이력이 단절됐다"는 판단을 내리면 안 된다.**
- **루틴 프롬프트가 detached HEAD를 다루지 않으면 이 함정을 밟는다**: `git checkout main`을 하는 순간 낡은 ref 기준의 main으로 이동해 "원격이 정체됐다"는 오판으로 이어진다. 커밋·push는 detached HEAD 위에서 `git push origin HEAD:main`으로 하거나, 프롬프트의 `git pull --rebase origin main`을 detached 상태 그대로 수행해야 한다.
- **루틴이 중단을 선택하면 흔적은 푸시 알림뿐이다** — 저장소에는 아무것도 남지 않아, 다음 날 "파일이 안 생겼다"로만 발견된다. Hermes 07:20 "파일 없음" 감시가 이 장애를 잡는 설계인데(계획 루틴 표), 이번 건은 사용자가 먼저 알아챘다.

## 기록

### 2026-10-15 루틴 프롬프트를 최신 결정에 맞춰 갱신: update는 job_config 전체, 검증은 재조회 대조

- 맥락: 계획과 취업 운영의 2026-10-14 기준(옛 계획은 의무가 아니다, 주간 분량 조절, Web FE 지원)이 문서에만 반영되고 루틴 프롬프트는 옛 지시(약국맵 실습, 이력서 뼈대 9회, 이월 3회 규칙, 분야별 필수 슬롯)를 들고 있었다. 세 루틴을 `RemoteTrigger update`로 갱신했다: 일간 계획 생성 `trig_014hB2B75srhp5a8HGpmz5qt`, 주간 회고 스캐폴드 `trig_017pmFKgZFA1WWhrKqvo6qAX`, 채용 공고 JD 분석 `trig_01P2oUMvMLGFtn6yJ2LwEjpn`.
- 알아낸 것:
  - **실제 목록을 먼저 본다.** `scripts/wiki-config.json`과 계획 문서의 루틴 표는 기대 상태다. `list` 응답의 `derived_state.prompt`가 지금 도는 프롬프트이고, `last_run`과 `next_run_at`이 실제 실행 상태다. 주간 일정 채우기 루틴은 `enabled: false`로 그대로 두었다.
  - **프롬프트만 바꿀 때도 `job_config.ccr`를 통째로 보낸다**: `environment_id`, `session_context`(`model`, `sources`, `allowed_tools`), `events`(새 uuid, `message.role` 필수). 기존 값을 `list` 응답에서 복사해 넣으면 cron, enabled, `mcp_connections`, config의 나머지 필드는 바뀌지 않는다. 갱신 전후 `list` 전체를 저장해 두고 그 필드들을 diff해서 확인했다.
  - **검증은 재조회 대조다.** 갱신 뒤 `list`를 다시 받아 `derived_state.prompt`가 로컬 파일과 바이트 단위로 같은지 비교한다. 루틴 API에는 발송 없는 미리보기가 없다. `run`은 실제 커밋과 push를 만들므로 테스트용으로 돌리지 않았고, 다음 정기 실행 결과(일간 파일, 일요일 회고)로 본다.
  - **프롬프트는 문서 구조 변화에 느슨해야 한다.** 주간 템플릿의 접힌 콜아웃 이름은 `참고`인데 이번 주 파일은 `배분 초안과 지난주 근거`다. 이름으로만 지시하면 못 찾으므로 `> [!note]-` 블록이면 같은 것으로 본다고 적었다.
  - **서버 쪽 짝은 남아 있다.** Hermes `weekly_retro_briefing.py`의 고정 회고 질문 2번이 「3회 이월되거나 계속 미뤄진 항목」을 묻는다. ssh 읽기로 확인만 했고 수정은 auto 모드 차단이라 홍이 서버에서 고친다. `job_watch.py`는 이미 iOS를 제외하고 FE와 AI 트랙만 모으므로 그대로 둔다.
- 근거: 갱신 응답의 `updated_at` 2026-10-14T15:51~15:5x Z, 재조회 대조 스크립트(세션 스크래치), 서버 `hermes cron list`와 스크립트 grep 읽기.

### 2026-09-17 — 루틴 샌드박스는 임의 외부 호스트에 못 나간다 · 생성 시 커넥터가 전부 붙는다

- 맥락: 외부 공개 JSON API를 읽어 위키에 노트를 쓰는 루틴을 새로 만들고 즉시 실행(`RemoteTrigger run`)으로 검증했다.
- 알아낸 것:
  - **`curl`이 exit 56으로 죽는다.** 로그 원문: `[agent-proxy] … <호스트>:443 — connect_rejected (the egress proxy denied the CONNECT (organization policy) or could not reach the destination)`. GitHub(저장소 push)는 되지만 임의 호스트는 안 된다. **루틴이 외부 데이터를 읽어야 하면, 밖에 있는 머신(Hermes 서버)이 받아서 저장소에 커밋해 두고 루틴은 저장소 파일만 읽게 한다.**
  - **`create` 본문에 `mcp_connections`를 안 넣으면 계정의 커넥터가 전부 붙는다**(Slack·Gmail·Notion·Drive·Atlassian·Docs 등 7개). 저장소만 만지는 루틴은 생성 직후 `update {"clear_mcp_connections": true}`로 뗀다 — 외부 쓰기 능력은 규칙이 아니라 도구에서 뺀다([Hermes cron 운영](Hermes%20cron%20%EC%9A%B4%EC%98%81.md) 09-15 교훈과 같다).
  - 검증 순서: `run` → 응답의 `session_id` → `get_run_log`. 실패 원인이 도구 결과에 그대로 찍힌다.
- 서버 쪽 짝: 서버 클론에서 커밋할 때는 브리핑 스크립트와 같은 `(로컬 경로)`을 `fcntl.flock`으로 잡고 `pull --ff-only` → 파일명으로 `add`·`commit -- <paths>` → `push origin HEAD:main`. 서버에는 pre-commit 훅이 없다(훅은 클론에 안 따라온다).


### 2026-08-30 — 일간 2026-08-29 미생성: 루틴의 "이력 단절" 오판으로 self-abort
- 맥락: 계획 — 루틴 "MyWiki 일간 계획 생성"(`trig_014hB2B75srhp5a8HGpmz5qt`)이 08-28 22:02 UTC(08-29 07:02 KST)에 돌았고 상태는 SUCCEEDED인데 `계획/일간/2026/08/일간 2026-08-29.md`가 로컬·원격 어디에도 없었다.
- 배운 것:
  - 로그를 보니 루틴은 파일 초안·완료판정 편집까지 **다 만들었다**. 그 후 `git checkout main`이 실패하며 위의 샌드박스 git 함정을 밟았고, "GitHub main 6일 정체 + 실제 볼트와 이력 단절(공통 조상 없음)"로 결론 내려 **push 없이 초안을 지우고 종료**했다. 모바일 푸시 2건("MyWiki 일간 계획 루틴 중단 …")만 남았다.
  - 실제 원격은 정상이었다 — 로컬에서 `git merge-base 43d3b88 9e0def6` = `9e0def6`(조상 관계 확인), origin/main도 최신. 루틴의 안전 정지 자체는 올바른 행동이었지만(강제 push 안 함), **판단 근거가 전부 샌드박스 아티팩트였다.**
  - 루틴이 샌드박스 안에 만든 `preserve-local-2026-08-29` 브랜치·stash는 샌드박스 소멸과 함께 사라진다 — 정리할 것 없음.
- 근거: `RemoteTrigger get_run_log cse_01QzpHvADwJer7UYvpseDxkw` 전문, 로컬 `git merge-base`·`git fetch` 실측 (2026-08-30 01:20 KST).
- 조치 (2026-08-30): 두 루틴("MyWiki 일간 계획 생성"·"MyWiki 주간 회고 스캐폴드") 프롬프트에 **「git 규칙」 섹션** 추가 — detached HEAD 위에서 그대로 커밋, `git checkout main` 금지, push는 `git pull --rebase origin main` → `git push origin HEAD:main`(거부 시 1회 재시도, 강제 push 금지). 일간 루틴에는 **어제 파일 부재 시 가장 최근 일간 파일로 이월 수집**, 회고 루틴에는 **일간 파일 없는 날 표기** 규칙도 추가. 원본 규격은 계획에 반영.

### 2026-10-17 — 루틴 셋을 맥 launchd + Codex exec로 이식, Claude 루틴은 등록 뒤 끈다

- 맥락: 이식 기록. 홍이 Claude 구독을 끊기로 해서 일간 계획 생성, 주간 회고 스캐폴드, 채용 공고 JD 분석 프롬프트를 `RemoteTrigger list`의 `derived_state.prompt`에서 그대로 받아 `scripts/routines/*.md`로 옮겼다.
- 배운 것:
  - 프롬프트에서 CCR 전용인 것은 세 절뿐이다. UTC 더하기(맥은 KST라 `date`로 끝), 「git 규칙 (샌드박스 함정)」의 detached HEAD와 merge-base 이야기(로컬 클론은 main 브랜치), egress 차단 문구. 나머지 규격은 글자 하나 안 바꿨다.
  - push는 프롬프트가 아니라 실행기가 한다. 루틴 세션은 커밋까지만 하고 `codex_routine.py`가 `git push origin HEAD:main`을 한다. 2026-08-29 self-abort 같은 「push 안 하고 종료」가 있어도 실행기 로그에 남는다.
  - JD 분석 루틴은 `scripts/wiki-config.json`의 `routines`에 없었다. 이식하면서 `com.hong.mywiki-codex-jd`로 넣었다.
  - 둘이 같이 돌면 같은 파일을 두 번 만들 수 있어 전환은 같은 날에 했다. 16:01 launchd 등록, 16:02 `RemoteTrigger update {"enabled": false}` 셋. 부분 update에 `enabled`만 보내도 된다(프롬프트를 바꿀 때만 `job_config.ccr` 전체가 필요하다).
- 근거: `scripts/routines/`, 루틴 목록 JSON은 세션 스크래치에만 보관.
