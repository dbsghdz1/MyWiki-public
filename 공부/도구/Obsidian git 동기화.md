---
type: study
area: 도구·인프라
audience: ai
status: active
created: 2026-08-27
updated: 2026-08-27
projects:
  - "계획 운영"
---

# Obsidian git 동기화

obsidian-git의 자동 백업은 `autoCommitOnlyStaged: true`면 **사실상 무동작**이다 — Obsidian 편집(체크박스 포함)은 파일을 수정만 하고 stage하지 않으므로, 자동 커밋이 돌아도 커밋할 것이 없다. "체크하면 자동으로 push된다"는 믿음은 이 설정 하나로 조용히 깨진다.

## 핵심 정리

- **obsidian-git 자동 백업 경로**: `autoSaveInterval`/`autoPushInterval`이 잡혀 있어도, `autoCommitOnlyStaged`(설정 UI: "Commit only staged files")가 켜져 있으면 staged 파일만 커밋 대상이라 Obsidian 편집이 하나도 올라가지 않는다. 실패 알림도 없다 — "커밋할 변경 없음"으로 정상 종료하기 때문. 판별법: `git log --grep "vault backup"`으로 마지막 자동 커밋 날짜를 보면 된다(MyWiki는 2026-07-14 이후 0건이었다).
- **클라우드 루틴은 원격만 본다**: 일간 계획 생성 루틴(07:00 KST, 클라우드 실행)은 저장소를 clone해서 어제 일간 파일을 읽는다. 로컬에만 있는 체크는 보이지 않으므로, push 안 된 `[x]`는 `[ ]`로 취급되어 이월된다. 로컬 수정이 원격에 반영됐는지가 루틴 정확성의 전제다.
- **로컬 dirty 파일은 `pull --rebase`를 막는다**: 추적 파일에 unstaged 변경이 있으면 rebase가 거부된다. 위키 오퍼레이션이 전부 `pull --rebase`로 시작하므로, Obsidian 손편집이 커밋되지 않고 쌓이면 모든 세션의 sync가 막힌다. 이번에 6개 파일이 최장 6일(08-21~) 방치돼 있었다.
- **Obsidian 표 에디터는 셀 폭 정렬을 다시 쓴다**: 표가 있는 파일을 Obsidian에서 열어 편집하면 셀 padding이 재정렬되어 내용 변화 없이 수백 줄 diff가 난다(`지출 기록 2026-07.md` 304줄). diff가 커 보여도 `git diff | grep "^+"` 몇 줄로 포맷 변경인지 먼저 판별할 것.

## 기록

### 2026-08-27 — 체크한 항목이 다음 날 다시 이월된 원인

- 맥락: 계획 운영 — 08-26 일간에서 `[x]` 체크한 SSH-295가 08-27 일간에 `(이월 2회)`로 재등장
- 배운 것: 체크가 로컬 작업 트리에만 있고 push되지 않아, 클라우드 루틴이 원격 HEAD 기준(`[ ]`)으로 이월했다. 근본 원인은 obsidian-git `autoCommitOnlyStaged: true`로 자동 백업이 7월 중순부터 무동작이었던 것
- 근거: 마지막 `vault backup` 커밋 `ce9d07e`(2026-07-14) · 정리 커밋 `a0141af` · 설정 수정 커밋 `91b10d3`(`.obsidian/plugins/obsidian-git/data.json`, false로 변경 — 적용은 Obsidian 재시작 필요)
