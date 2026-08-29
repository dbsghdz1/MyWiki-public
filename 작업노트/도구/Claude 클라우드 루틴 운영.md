---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-30
updated: 2026-08-30
projects:
  - "계획"
---

# Claude 클라우드 루틴 운영

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

### 2026-08-30 — 일간 2026-08-29 미생성: 루틴의 "이력 단절" 오판으로 self-abort
- 맥락: 계획 — 루틴 "MyWiki 일간 계획 생성"(`trig_014hB2B75srhp5a8HGpmz5qt`)이 08-28 22:02 UTC(08-29 07:02 KST)에 돌았고 상태는 SUCCEEDED인데 `계획/일간/일간 2026-08-29.md`가 로컬·원격 어디에도 없었다.
- 배운 것:
  - 로그를 보니 루틴은 파일 초안·완료판정 편집까지 **다 만들었다**. 그 후 `git checkout main`이 실패하며 위의 샌드박스 git 함정을 밟았고, "GitHub main 6일 정체 + 실제 볼트와 이력 단절(공통 조상 없음)"로 결론 내려 **push 없이 초안을 지우고 종료**했다. 모바일 푸시 2건("MyWiki 일간 계획 루틴 중단 …")만 남았다.
  - 실제 원격은 정상이었다 — 로컬에서 `git merge-base 43d3b88 9e0def6` = `9e0def6`(조상 관계 확인), origin/main도 최신. 루틴의 안전 정지 자체는 올바른 행동이었지만(강제 push 안 함), **판단 근거가 전부 샌드박스 아티팩트였다.**
  - 루틴이 샌드박스 안에 만든 `preserve-local-2026-08-29` 브랜치·stash는 샌드박스 소멸과 함께 사라진다 — 정리할 것 없음.
- 근거: `RemoteTrigger get_run_log cse_01QzpHvADwJer7UYvpseDxkw` 전문, 로컬 `git merge-base`·`git fetch` 실측 (2026-08-30 01:20 KST).
- 조치 (2026-08-30): 두 루틴("MyWiki 일간 계획 생성"·"MyWiki 주간 회고 스캐폴드") 프롬프트에 **「git 규칙」 섹션** 추가 — detached HEAD 위에서 그대로 커밋, `git checkout main` 금지, push는 `git pull --rebase origin main` → `git push origin HEAD:main`(거부 시 1회 재시도, 강제 push 금지). 일간 루틴에는 **어제 파일 부재 시 가장 최근 일간 파일로 이월 수집**, 회고 루틴에는 **일간 파일 없는 날 표기** 규칙도 추가. 원본 규격은 계획에 반영.
