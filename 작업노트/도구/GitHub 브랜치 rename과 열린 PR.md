---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-12
updated: 2026-09-12
projects:
  - "보험찾개냥"
---

# GitHub 브랜치 rename과 열린 PR

## 핵심 정리

- `POST /repos/{owner}/{repo}/branches/{branch}/rename`(= `gh api .../rename -f new_name=...`)으로 **열린 PR의 head 브랜치**를 바꾸면, 그 PR은 따라오지 않고 **`closed` + `head_ref_deleted` 이벤트로 닫힌다.** 다른 PR의 **base**로 쓰인 브랜치는 정상적으로 새 이름으로 갱신된다(#128의 base가 `feat/SSH-462`로 바뀜).
- 닫힌 PR은 `gh pr reopen`이 `Could not open the pull request`로 거부된다 — head가 없기 때문. 복구는 **새 PR**뿐이다. 리뷰·코멘트 이력은 옛 PR에 남으니 새 본문에서 링크한다.
- 그래서 이름을 바꿔야 하면: 새 이름으로 push → 새 PR 생성 → 옛 PR 닫기 순이 맞다. rename API는 PR이 안 걸린 브랜치에만.
- `gh pr close N --delete-branch`는 팀원 브랜치도 그대로 지운다 — 되돌리기는 PR 페이지의 「Restore branch」.

## 기록

### 2026-09-12 — 보험찾개냥 브랜치명 CI 규칙 교정

- 맥락: `rules-ci.yml` ④가 브랜치명을 `^<type>/SSH-[0-9]+$`로 검사해 `feat/SSH-462-home`·`feat/SSH-464-report`(#127·#130)가 빨갰다. 성식님 브랜치(#116·#117)를 정리한 뒤 rename API로 이름을 바꿨더니 두 PR이 닫혔다.
- 근거: #127 이벤트 `closed 01:43:18` → `head_ref_deleted 01:43:19`. #131·#132로 다시 열었고 협업 규칙 CI 통과.
