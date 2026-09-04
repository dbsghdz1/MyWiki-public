---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-05
updated: 2026-09-05
projects: []
---

# MyWiki 구조 lint와 pre-commit 훅

MyWiki 볼트에서 커밋이 막혔을 때 보는 문서. 2026-09-04 개편([[_wiki/LLM Wiki|LLM Wiki]] 「운영 v2」)으로 `.githooks/pre-commit`이 매 커밋 전에 `scripts/build-index.py` → `scripts/lint-structure.py --staged` → `git diff --cached --check`를 돌린다. 규칙 원본은 `AGENTS.md` 「Operation: Lint」, 수치 정본은 `scripts/wiki-config.json`.

## 핵심 정리

- **훅은 클론마다 `git config core.hooksPath .githooks`를 실행해야 켜진다.** `.git/config` 로컬 설정이라 다른 기기·클라우드 루틴 샌드박스에는 없다 — 그쪽 커밋은 검사 없이 들어온다.
- **`_wiki/index.md`는 손으로 고치지 않는다.** 훅이 `build-index.py`로 다시 만들어 `git add`까지 한다(2026-09-05). 그래서 커밋에 `_wiki/index.md`가 저절로 딸려 들어오는 것은 정상이고, 손으로 고친 index는 다음 커밋에서 덮인다. 생성은 **작업 트리** frontmatter 기준이라, 다른 세션이 허브의 `updated:`를 고쳐 두고 아직 커밋하지 않았으면 그 값이 내 커밋의 index에 먼저 실린다 — lint가 경고만 찍는다(`index 대상 문서에 stage되지 않은 변경`). 그 세션이 커밋하면 저절로 맞는다.
- **예시 링크를 문서에 쓸 때는 펜스 코드 블록에 넣는다.** lint는 새로 추가된 줄의 위키링크를 인라인 코드 안까지 전부 검사하고 펜스(```)만 건너뛴다. 표 셀에 이중 대괄호 예시를 썼다가 이 문서의 첫 커밋이 막혔다(2026-09-05).
- **에러 메시지 → 원인 → 조치**

| 메시지 | 원인 | 조치 |
|---|---|---|
| `새 log 헤더 형식은 ## YYYY-MM-DD HH:mm KST — 유형 · 결과이다` | log 항목에 시각이 없거나 옛 다중행 형식 | 헤더에 `HH:mm KST`, 본문은 `입력·변경·검토·커밋` 4줄만 |
| `새 log 항목은 헤더 + 4줄이어야 한다` | `배운 것:` 같은 줄을 더 넣음 | 상세는 작업 기록·작업노트로 보내고 log엔 주소만 |
| `과거 log 항목을 수정하거나 삭제했다` | 옛 항목 편집, 또는 중간 항목 삭제 | 옛 항목은 건드리지 않는다. 아카이브는 **가장 오래된 항목부터** 잘라 `_wiki/log-YYYY-MM.md`에 그대로 붙이고 같은 커밋에 stage |
| `log 상단(첫 항목 이전)을 수정했다` | frontmatter·머리글만 고침 | 아카이브 이동 커밋에서만 허용 |
| `새 wikilink 대상을 찾을 수 없다` | 대상 파일이 없거나 basename이 둘 이상, 또는 인라인 코드 안의 예시 링크 | 경로를 풀 패스로. 예시는 펜스 블록으로. (2026-09-05 전에는 표 안 `\|` 이스케이프 별칭도 오탐했다 — 고쳐짐) |
| `생성된 _wiki/index.md가 HEAD와 다른데 staging되지 않았다` | 훅 없이 커밋했거나 `--no-verify` | `python3 scripts/build-index.py && git add _wiki/index.md` |
| `index 드리프트: index가 오래됐다` | 작업 트리 index가 frontmatter와 다름 | 같은 명령 |
| `본문·상태를 갱신할 때 ## 현재 카드를 먼저 추가할 것` | 프로젝트 README 본문을 고쳤는데 5줄 카드가 없음 | `# 제목` 바로 아래 `## 현재 카드`(단계·현재·다음 판정·지금 할 일·하지 않을 일) |
| `staged 내용과 working tree가 달라 생성 검사를 보장할 수 없다` | index 대상 문서를 부분 stage했거나 Obsidian이 표를 재정렬해 둠 | 그 파일을 통째로 `git add` |
| `git diff --cached --check` 실패 | 새로 추가한 줄 끝 공백 | 공백 제거 |

- **표 안 링크는 별칭 없이 쓰는 것이 가장 안전하다.** 별칭이 꼭 필요하면 파이프를 `\|`로 이스케이프한다(볼트 관례 42곳, lint가 이제 인식).
- 우회가 필요하면 `git commit --no-verify` — 단 그 커밋은 index 드리프트를 남길 수 있으니 다음 정상 커밋의 훅이 흡수하게 둔다.

## 기록

### 2026-09-05 — Codex의 09-04 lint 개편을 리뷰하며 결함 3개를 재현·수정

- 맥락: [[_wiki/LLM Wiki|LLM Wiki]] 운영 v2 개편(커밋 `3a65207`, Codex)을 이튿날 리뷰. 첫 커밋에서 lint에 두 번 막힌 것이 계기.
- 배운 것:
  - `validate_added_wikilinks`가 이스케이프 별칭 링크의 `\`를 대상 경로에 남겨 오탐했다 — 함수 직접 호출로 재현. `raw.replace("\\|", "|")` 후 split으로 수정.
  - "index 대상 문서가 바뀌었지만 index가 staging되지 않았다"는 index가 안 바뀌면 통과할 수 없는 검사였다(stage할 것이 없다). `index_stage_error`로 바꿔 생성 결과가 HEAD와 다를 때만 요구.
  - log 200KB 상한 검사와 append-only 검사가 서로를 막는 교착(아카이브 = 삭제). `log_transition_errors`가 **가장 오래된 항목들이 같은 커밋에 staged된 `_wiki/log-YYYY-MM.md`에 그대로 있을 때만** 삭제를 허용한다. 중간 삭제·옛 항목 수정·머리글 단독 수정은 여전히 실패.
  - pre-commit 훅 안에서 `git add`한 파일은 그 커밋에 포함된다 — 생성물(index)을 훅이 갱신·stage하는 구조가 성립한다.
  - 두 세션이 같은 볼트에서 동시에 커밋하면 한쪽 훅이 다른 쪽의 미커밋 허브 변경을 index에 먼저 싣는다. 실패가 아니라 경고로 두고 다음 커밋이 맞추게 했다.
- 근거: `scripts/tests/test_lint_structure.py`의 `WikilinkTest`·`LogTransitionTest`·`IndexStageTest`(2026-09-05 추가, 전부 통과). 재현 명령:

```python
lint_structure.validate_added_wikilinks(root, "계획/README.md", ["| x | [[학습/공부/README\\|공부]] |"])
# 수정 전: ['계획/README.md: 새 wikilink 대상을 찾을 수 없다: [[학습/공부/README\\]]'] → 수정 후: []
```
