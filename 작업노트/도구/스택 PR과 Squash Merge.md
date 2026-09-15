---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-15
updated: 2026-09-15
projects:
  - "보험찾개냥"
---

# 스택 PR과 Squash Merge

앞 PR을 Squash Merge한 뒤 뒤 PR은 **`mergeable`이 `CONFLICTING`일 때만** 잘라 옮긴다(`rebase --onto` + force push). `MERGEABLE`/`CLEAN`이면 그대로 머지해도 squash 결과는 뒤 PR 자기 변경만 들어간다.

## 핵심 정리

- **충돌 여부는 "뒤 PR이 앞 PR이 바꾼 hunk를 또 고쳤나"로 갈린다.** 앞 PR 커밋과 main의 squash 커밋은 내용이 같아서 git 3-way merge가 흡수한다. 뒤 PR이 같은 자리를 더 고쳤으면 그 hunk만 충돌한다.
- **`CLEAN`인데 그대로 머지할 때의 대가는 PR 화면뿐이다** — Files changed에 앞 PR 변경이 섞여 보인다(#141: 자기 14개인데 93개로 표시). squash 커밋 자체는 자기 파일만 담는다.
- **먼저 `gh pr view <n> --json mergeable,mergeStateStatus`를 본다.** 무조건 rebase + force push부터 하지 않는다. repo `CLAUDE.md`의 "머지 직후 잘라 옮긴다"는 규칙은 충돌 시에만 필요하다.
- **잘라낼 기준점은 끝 커밋 하나로 판정하지 않는다.** 뒤 브랜치가 앞 PR **중간 커밋**에서 따였을 수 있다 — `git merge-base --is-ancestor <앞 PR 끝> <뒤>`가 `no`여도 앞 PR 커밋 일부를 품고 있을 수 있다. `git log <더 앞 기준점>..origin/<뒤 브랜치>`로 커밋을 나열해 경계를 눈으로 확인한다.
- **auto mode는 `git push --force-with-lease`를 `[Git Destructive]`로 거부한다.** rebase·검증까지 하고 push는 홍에게 `! git -C <worktree> push --force-with-lease=<branch>:<옛 head> origin <branch>` 한 줄로 넘긴다.

## 기록

### 2026-09-15 — 보험찾개냥 #138~#142 스택 머지

- 맥락: 보험찾개냥 #138(SSH-557) 위에 #139(SSH-558), 그 위에 #141(SSH-561)·#142(SSH-562)가 쌓인 채 순서대로 머지
- 배운 것:
  - #139는 #138이 바꾼 `claimant_info_view_model.dart`·`profile_edit_view_model.dart` 등을 더 고쳐서 #138 squash(`4c22816`) 뒤 `CONFLICTING`/`DIRTY`. `git rebase --onto origin/main 767aa63 feat/SSH-558`은 충돌 없이 9커밋 이동 → force push 뒤 `MERGEABLE`, 머지 `f8e611e`
  - #141은 #139 파일을 안 건드려 **rebase 없이 `MERGEABLE`/`CLEAN`**(PR 화면 93파일). 그대로 머지 `c865928`, main 클라이언트 검사 success
  - #142는 #141 끝(`43e76a0`)이 아니라 중간 `b09a220`에서 따였다 — 끝 커밋 ancestor 검사만 해서 처음엔 "#141 위가 아니다"로 오판. 머지 `c2c3710`의 squash 파일 16개 = #142 자기 커밋 3개(`b09a220..e70ee1c`) 파일과 정확히 일치
- 근거: `gh pr view --json mergeable,mergeStateStatus,changedFiles`, `gh api repos/ASMSSH/Boheomgaenyang/commits/c2c3710 --jq '.files[]'`, 로컬 `(로컬 경로)`·`wt-561` 워크트리
