---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-05
updated: 2026-09-23
projects: []
---

# MyWiki 구조 lint와 pre-commit 훅

MyWiki 볼트에서 커밋이 막혔을 때 보는 문서. 2026-09-04 개편([[_wiki/LLM Wiki|LLM Wiki]] 「운영 v2」)으로 `.githooks/pre-commit`이 매 커밋 전에 `scripts/build-index.py` → `scripts/lint-structure.py --staged` → `git diff --cached --check`를 돌린다. 규칙 원본은 `AGENTS.md` 「Operation: Lint」, 수치 정본은 `scripts/wiki-config.json`.

## 핵심 정리

- **훅은 클론마다 `git config core.hooksPath .githooks`를 실행해야 켜진다.** `.git/config` 로컬 설정이라 다른 기기·클라우드 루틴 샌드박스에는 없다 — 그쪽 커밋은 검사 없이 들어온다.
- **`_wiki/index.md`는 손으로 고치지 않는다.** 훅이 `build-index.py`로 다시 만들어 `git add`까지 한다(2026-09-05). 그래서 커밋에 `_wiki/index.md`가 저절로 딸려 들어오는 것은 정상이고, 손으로 고친 index는 다음 커밋에서 덮인다. 생성은 **작업 트리** frontmatter 기준이라, 다른 세션이 허브의 `updated:`를 고쳐 두고 아직 커밋하지 않았으면 그 값이 내 커밋의 index에 먼저 실린다 — lint가 경고만 찍는다(`index 대상 문서에 stage되지 않은 변경`). 그 세션이 커밋하면 저절로 맞는다.
- **남의 미커밋 변경이 섞인 파일에서 내 변경만 커밋하려면 index에 직접 넣는다.** 대화형 `git add -p`는 쓸 수 없으니, 작업 트리 파일에서 남의 변경만 되돌린 사본(= HEAD + 내 수정)을 스크래치에 만들고 `git update-index --cacheinfo 100644,$(git hash-object -w <사본>),<경로>`로 올린다. `git diff --cached -- <경로>`로 내 hunk만 올라갔는지 확인한다. **index 대상 문서(허브·프로젝트 README)에는 못 쓴다** — 아래 표의 `staged 내용과 working tree가 달라` 검사에 막힌다. 주간·일간 계획 파일처럼 index 대상이 아닌 파일에서는 통했다(2026-09-12, `f7a2379`).
- **stage는 커밋 직전에 한다.** 같은 볼트를 쓰는 다른 세션이 그사이 `git commit`을 하면, 내가 미리 stage해 둔 파일이 그쪽 커밋에 딸려 간다.
- **예시 링크를 문서에 쓸 때는 펜스 코드 블록에 넣는다.** lint는 새로 추가된 줄의 위키링크를 인라인 코드 안까지 전부 검사하고 펜스(```)만 건너뛴다. 표 셀에 이중 대괄호 예시를 썼다가 이 문서의 첫 커밋이 막혔다(2026-09-05).
- **에러 메시지 → 원인 → 조치**

| 메시지 | 원인 | 조치 |
|---|---|---|
| `새 log 헤더 형식은 ## YYYY-MM-DD HH:mm KST — 유형 · 결과이다` | log 항목에 시각이 없거나 옛 다중행 형식 | 헤더에 `HH:mm KST`, 본문은 `입력·변경·검토·커밋` 4줄만 |
| `새 log 항목은 헤더 + 4줄이어야 한다` | `배운 것:` 같은 줄을 더 넣음 | 상세는 작업 기록·작업노트로 보내고 log엔 주소만 |
| `과거 log 항목을 수정하거나 삭제했다` | 옛 항목 편집, 또는 중간 항목 삭제 | 옛 항목은 건드리지 않는다. 아카이브는 **가장 오래된 항목부터** 잘라 `_wiki/log-YYYY-MM.md`에 그대로 붙이고 같은 커밋에 stage |
| `log 상단(첫 항목 이전)을 수정했다` | frontmatter·머리글만 고침 | 아카이브 이동 커밋에서만 허용 |
| `새 wikilink 대상을 찾을 수 없다` | 대상 파일이 없거나 basename이 둘 이상, 또는 인라인 코드 안의 예시 링크 | 경로를 풀 패스로. 예시는 펜스 블록으로. (2026-09-05 전에는 표 안 `\|` 이스케이프 별칭도 오탐했다 — 고쳐짐) |
| 같은 메시지인데 대상이 분명히 있다 | 대상 이름에 점이 있다(`1.2 Pro 계획`, `Next.js 서버와 캐싱`, `2.0 진행 2026-09-01`) — `Path.suffix`가 `.2 Pro 계획`을 확장자로 오인해 `1.md`를 찾던 결함 | 2026-09-05 수정(문자열로 `.md`만 처리). 옛 lint에서 막히면 스크립트를 최신으로 |
| `생성된 _wiki/index.md가 HEAD와 다른데 staging되지 않았다` | 훅 없이 커밋했거나 `--no-verify` | `python3 scripts/build-index.py && git add _wiki/index.md` |
| `index 드리프트: index가 오래됐다` | 작업 트리 index가 frontmatter와 다름 | 같은 명령 |
| `본문·상태를 갱신할 때 ## 현재 카드를 먼저 추가할 것` | 프로젝트 README 본문을 고쳤는데 5줄 카드가 없음 | `# 제목` 바로 아래 `## 현재 카드`(단계·현재·다음 판정·지금 할 일·하지 않을 일) |
| `staged 내용과 working tree가 달라 생성 검사를 보장할 수 없다` | index 대상 문서를 부분 stage했거나 Obsidian이 표를 재정렬해 둠. **다른 세션이 허브 변경을 stage·미커밋으로 들고 있을 때도 난다** | 그 파일을 통째로 `git add`. 남의 변경이면 아래 2026-09-20 기록의 별도 index 방식 |
| `git diff --cached --check` 실패 | 새로 추가한 줄 끝 공백 | 공백 제거 |

- **private 마커는 한 줄에 마커만 있을 때만 지워진다**(2026-09-23 `publish.sh` 4b 수정). 본문에 백틱으로 인용한 `<!-- private:start -->` 예시도 예전엔 마커로 잡혀 **그 줄부터 문서 끝까지 공개판에서 사라졌다**. 지금은 한 줄 안의 start…end 쌍이 지워지지 않고 6e 게이트가 퍼블리시를 멈춘다 — 마커는 각각 단독 줄로(앞 공백·인용 `>`는 허용), 코드 펜스 안의 마커는 예시로 본다.
- **퍼블리시는 작업 트리를 복사한다.** 다른 세션의 미커밋·미추적 노트까지 공개판에 실린다 — 내 커밋만 내보내려면 `git worktree add --detach <스크래치> HEAD` 뒤 그 안의 `scripts/publish.sh`를 돌린다(`VAULT`가 스크립트 위치 기준이라 worktree를 가리킨다). HEAD의 index가 남의 미커밋 frontmatter로 생성돼 있으면 preflight `--check`가 실패하므로 worktree 안에서 `build-index.py`를 먼저 돌린다.
- **표 안 링크는 별칭 없이 쓰는 것이 가장 안전하다.** 별칭이 꼭 필요하면 파이프를 `\|`로 이스케이프한다(볼트 관례 42곳, lint가 이제 인식).
- 우회가 필요하면 `git commit --no-verify` — 단 그 커밋은 index 드리프트를 남길 수 있으니 다음 정상 커밋의 훅이 흡수하게 둔다.

## 기록

### 2026-09-23 — 공개판에서 README 두 개가 반쯤 잘려 있었고, 전화번호가 차단 패턴을 통과했다

- 맥락: 홍 「위키를 점검해」 의미 lint. 검증 에이전트가 공개 저장소를 실측하다 발견
- 배운 것:
  - `publish.sh` 4b의 awk 패턴이 줄 앞뒤를 고정하지 않은 **부분 일치**라(줄 어디든 `<!-- private:start -->`가 있으면 매치), 규칙을 설명하려고 백틱으로 인용한 마커 예시에서 skip이 켜졌다. `학습/공부/README.md` 26행(173→25줄)과 `작업노트/README.md` 33행(180→32줄)이 그 줄부터 끝까지 공개판에서 빠져 있었다. 반대로 한 줄 안의 `start…end` 쌍(`MyCryptoDiary/학습 계획`)은 start 줄을 통째로 버리고 end를 못 봐 뒤가 다 잘렸다
  - 고친 뒤 **숨어 있던 부분이 처음 공개된다** — 그 안의 문장(공개 경계·deny-block 대상)을 먼저 정리해야 했다(금융 앱 이름이 든 허브 행 → private, 공개 페르소나에 어긋나는 문장 삭제). **새로 쓰는 기록 문장도 같은 게이트를 지난다** — 이 항목의 첫 판이 금융 앱 이름을 그대로 적어 3차 퍼블리시가 차단됐다
  - `deny-block.txt`의 휴대폰 패턴 `01[016789]…`은 `+82 10-…` 꼴을 못 잡는다 — 08-27부터 공개판에 번호가 있었다. `\+82[ -]?1[016789]…` 추가
  - 6e 게이트를 처음 돌리자 공개판 README 템플릿(`scripts/public/README.md`)의 코드 펜스 안 설명문이 걸렸다 → 4b·6e 모두 펜스 안은 예시로 보도록
- 근거: 공개 저장소 `wc -l`(수정 전 25·32줄 → 수정 후 173·180줄), 커밋 `wiki: lint 2026-09-23`·`wiki: lint 2026-09-23 2차`, 공개 스냅샷 `d73e882`·`6ec86fa`

### 2026-09-20 — 다른 세션이 **stage까지 해 둔 채** 멈춰 있을 때: 별도 index(`GIT_INDEX_FILE`) + `commit-tree` + `update-ref`

- 맥락: 커리어 포지셔닝 기록(`30f7a76`)을 커밋하려는데, WristNote sync 세션이 01:12부터 `_wiki/index.md`·`_wiki/log.md`·WristNote 2개를 **공유 index에 stage한 채** 30분 가까이 커밋하지 않고 있었다. 09-12·09-16의 `update-index --cacheinfo` 방식은 남의 변경이 **작업 트리에만** 있을 때의 해법이라, 남의 staged 항목이 index에 있으면 `git commit`이 그것까지 삼킨다. `git commit -- <경로>`(--only)도 `log.md`·`index.md`는 작업 트리본(남의 항목 포함)을 올리므로 안 된다.
- 한 것 (공유 index와 작업 트리를 건드리지 않고 커밋을 만든다):
  1. `export GIT_INDEX_FILE=<스크래치>/idx-mine; git read-tree HEAD` → 내 파일만 `git add -- <파일명들>`
  2. `_wiki/log.md`는 `git show HEAD:_wiki/log.md` + 내 항목으로 사본을 만들어 `git hash-object -w` → `git update-index --cacheinfo 100644,<blob>,_wiki/log.md`. `_wiki/index.md`도 `HEAD`본에서 **내 허브 줄만** 바꾼 사본으로 같은 방식
  3. 검증: `build-index.py`로 작업 트리 index를 재생성한 뒤 `diff <내 사본> _wiki/index.md` — **남의 줄 하나만** 달라야 한다. `GIT_INDEX_FILE=… lint-structure.py --staged`는 `staged 내용과 working tree가 달라 생성 검사를 보장할 수 없다: _wiki/index.md` **하나만** 내고 실패하는데, 이 상황에선 그 가드가 곧 「남의 미커밋 허브 변경이 있다」는 뜻이라 위 diff로 대신 확인했다. `git diff --cached --check`는 그대로 통과
  4. `tree=$(git write-tree)` → `commit=$(git commit-tree $tree -p $OLD -F msg)` (훅은 안 돈다 — 3번이 그 대신이다)
  5. **순서가 중요하다**: 먼저 `unset GIT_INDEX_FILE` 후 공유 index에 내 경로들(`log.md`·`index.md` 포함)을 `git add`하고, **그다음** `git update-ref refs/heads/main $commit $OLD`(CAS). 거꾸로 하면 그 틈에 남이 커밋할 때 남의 index에 든 옛 blob이 내 파일·log 항목을 **되돌린다.** 이 순서면 최악의 경우에도 내 변경이 남의 커밋에 섞일 뿐이고, 그때는 `update-ref`가 CAS 실패로 멈춘다
  6. 끝나고 `git diff --cached --stat`에 **남의 4개 파일·남의 delta만** 남았는지 확인 → push
- 주의: `publish.sh`는 HEAD가 아니라 **작업 트리**를 `cp -R`한다 — 남의 미커밋 공개 영역 변경도 같이 나간다.
- 근거: 커밋 `30f7a76`(5개 파일, 남의 것 0), 직후 `git diff --cached`에 WristNote delta만 잔존.

### 2026-09-18 — `publish.sh`의 `visibility: private` 게이트가 큰 파일을 통과시켰다 (`head | grep -q` + `pipefail`)

- 맥락: MyWiki 점검 중 WristNote 전사 원본(133KB, 본문이 한 줄)을 `visibility: private`로 `프로젝트/개인/WristNote/전사 샘플 2026-09-12.md`에 넣고 `scripts/publish.sh --push`를 돌렸는데, 공개 저장소(`(로컬 경로)`, 커밋 `a640b30`)에 그 파일이 그대로 올라갔다. 같은 실행에서 `학습/공부/학습 계획.md`(작은 파일)는 정상 제외됐다.
- 원인: 3/7 단계가 `if head -n 30 "$f" | grep -qE '^visibility:...'`였다. `grep -q`는 첫 매치에서 바로 끝나는데, 첫 30줄 안에 파이프 버퍼(64KB)보다 긴 줄이 있으면 `head`가 아직 쓰는 중이라 **SIGPIPE(141)로 죽는다.** 스크립트가 `set -euo pipefail`이라 파이프라인 종료 코드가 141이 되고 `if`가 거짓 → 비공개 파일이 삭제되지 않는다. **매치에 성공했기 때문에 실패하는** 구조라 작은 파일에서는 절대 재현되지 않는다.
- 수정: 파이프를 없앴다 — `awk`가 앞 30줄만 읽고 같은 정규식이 맞으면 0으로 끝나는 단일 명령(`scripts/publish.sh` 3/7 단계 참고 — POSIX 공백 클래스 표기가 위키링크로 오인돼 여기엔 옮기지 않는다). 드라이런 3/7 출력에 두 파일이 모두 제외 목록으로 찍히는 것을 확인했다.
- 일반화: `set -o pipefail` 아래에서 `… | grep -q`, `… | head -1`을 **조건문에 쓰면 입력이 클 때만 거짓이 된다.** 보안 게이트에는 파이프 없는 단일 명령을 쓴다. 퍼블리시 뒤에는 공개 저장소에서 `git ls-files`로 비공개 파일이 실제로 빠졌는지 본다 — 이번에도 그 확인으로 잡았다.
- 새로 넣은 파일은 테스트용 가상 회의 스크립트 전사라 유출 내용은 무해했다.

### 2026-09-16 — 학습/공부 대량 재편(git mv 22 + 파일 분할)에서 밟은 것 셋

- 맥락: 홍 지시로 `학습/공부/CS`·`JS`를 세부 폴더로 나누고 네트워크.md(311줄) 등 큰 파일을 개요/`학습 계획.md`/주제 노트로 쪼갰다. 볼트 전체 `학습/공부/…` 위키링크 86파일을 perl로 일괄 치환. 커밋 `wiki: 공부 폴더 재편 2026-09-16`.
- 배운 것:
  - **일괄 치환이 `_wiki/log*.md`에 닿았다.** `grep -rl … . | grep -v "^./_wiki/log"`로 뺐다고 믿었는데 `git status`에 log 3개가 M으로 떴다(log-2026-08 58줄). log는 append-only라 lint가 과거 항목 수정을 막는다 — **`git checkout -- _wiki/log*.md`로 되돌리고, 치환 뒤에는 반드시 `git status`로 닿은 파일을 확인한다.** 옛 경로 링크가 log에 남는 것은 히스토리로 받아들인다(비공개, 08-30 재편 때도 같음).
  - **`git diff --cached --name-status` 출력을 `while read`로 돌리면 한글 경로가 `"\352\263…"`로 인용돼 그 뒤 `git show ":$f"`·`git reset -- "$f"`가 전부 빈손으로 끝난다.** 증상: `diff -q <(빈 출력) <(빈 출력)`이 같다고 나와 "ok", `git diff --cached --quiet -- "$f"`가 0을 돌려줘 **staged 파일 전부가 no-op으로 unstage됐다.** 해법은 `GIT_CONFIG_PARAMETERS="'core.quotepath=false'"`(또는 `git -c core.quotepath=false`, `-z`) — 이 볼트는 경로가 전부 한글이라 스크립트마다 필요하다.
  - **`git add -A <디렉터리>`는 동시 세션의 미커밋 변경을 함께 stage한다** — 약국맵 README 현재 카드, 신규 `설계 — 09-19 색인 판정` 파일, 작업노트 README `updated:`가 내 index에 섞였다. 분리 방법: 파일마다 `git show HEAD:<f> | <내 변환 스크립트>`를 만들어 staged 내용과 `diff -q`, 다르면 `git hash-object -w --stdin` → `git update-index --cacheinfo 100644,<blob>,<f>`로 **index만 HEAD+내 변경으로 교체**한다(작업 트리는 그대로 둔다). 신규 파일은 `git reset -- <f>`.
  - awk 잡기술 하나: `awk -v pat='^### proxy\(미들웨어\)'`처럼 `-v`로 넘긴 문자열은 이스케이프가 한 번 풀려 `\(`가 `(`(그룹)이 된다 — 제목에 괄호가 있으면 `.`로 대신 매칭한다(`proxy.미들웨어.`). 이걸로 절 4개가 조용히 빠졌고, **원본의 `###` 제목 전부가 새 파일 어딘가에 있는지 grep으로 대조**해서 잡았다.
- 근거: 이 세션의 `git status`·`git diff --cached --name-status` 출력, 커밋 `wiki: 공부 폴더 재편 2026-09-16`

### 2026-09-12 — 동시 세션의 미커밋 변경을 피해 부분 커밋, log 4줄 규칙에 한 번 막힘

- 맥락: 대화 세션에서 소마 9/4 멘토링 정리·HSW 계약 템플릿·주간 회고 재료를 쓰고 의미 lint를 돌렸다. `계획/주간/2026/09/주간 2026-09-08.md`에 이 세션 것이 아닌 `status: draft → active` 미커밋 변경이 있었고, 같은 시각 다른 세션이 WristNote 작업노트를 커밋(`aacb607`)하고 있었다.
- 배운 것:
  - 위 핵심 정리의 `hash-object` + `update-index` 방식으로 status 줄을 뺀 채 회고 재료 절만 커밋했다(`f7a2379`).
  - log `검토`를 하위 불릿 여러 줄로 쓰면 `새 log 항목은 헤더 + 4줄이어야 한다`로 막힌다. 판단 대기 목록이 길어도 한 줄에 ` / `로 잇는다(`1f228a0`).
  - 백그라운드 lint 에이전트가 README를 고치는 중에 커밋하면 훅이 그 `updated:` 변경을 index에 먼저 싣는다. 경고만 나오고, 다음 커밋이 맞춘다.
- 근거: 커밋 `f7a2379`·`1f228a0`. 막힌 출력 원문 `구조 lint 실패: - _wiki/log.md: 새 log 항목은 헤더 + 4줄이어야 한다`.

### 2026-09-05 — Codex의 09-04 lint 개편을 리뷰하며 결함 3개를 재현·수정

- 맥락: [[_wiki/LLM Wiki|LLM Wiki]] 운영 v2 개편(커밋 `3a65207`, Codex)을 이튿날 리뷰. 첫 커밋에서 lint에 두 번 막힌 것이 계기.
- 배운 것:
  - `validate_added_wikilinks`가 이스케이프 별칭 링크의 `\`를 대상 경로에 남겨 오탐했다 — 함수 직접 호출로 재현. `raw.replace("\\|", "|")` 후 split으로 수정.
  - "index 대상 문서가 바뀌었지만 index가 staging되지 않았다"는 index가 안 바뀌면 통과할 수 없는 검사였다(stage할 것이 없다). `index_stage_error`로 바꿔 생성 결과가 HEAD와 다를 때만 요구.
  - log 200KB 상한 검사와 append-only 검사가 서로를 막는 교착(아카이브 = 삭제). `log_transition_errors`가 **가장 오래된 항목들이 같은 커밋에 staged된 `_wiki/log-YYYY-MM.md`에 그대로 있을 때만** 삭제를 허용한다. 중간 삭제·옛 항목 수정·머리글 단독 수정은 여전히 실패.
  - pre-commit 훅 안에서 `git add`한 파일은 그 커밋에 포함된다 — 생성물(index)을 훅이 갱신·stage하는 구조가 성립한다.
  - 두 세션이 같은 볼트에서 동시에 커밋하면 한쪽 훅이 다른 쪽의 미커밋 허브 변경을 index에 먼저 싣는다. 실패가 아니라 경고로 두고 다음 커밋이 맞추게 했다.
- 근거: `scripts/tests/test_lint_structure.py`의 `WikilinkTest`·`LogTransitionTest`·`IndexStageTest`(2026-09-05 추가, 전부 통과). 같은 날 카드 일괄 부착 커밋이 `1.2 Pro 계획` 링크로 막혀 **점이 든 이름 오탐**(네 번째 결함)을 추가로 잡았다 — `Path.suffix`·`.stem` 대신 문자열로 `.md`만 다룬다. 재현 명령:

```python
lint_structure.validate_added_wikilinks(root, "계획/README.md", ["| x | [[학습/공부/README\\|공부]] |"])
# 수정 전: ['계획/README.md: 새 wikilink 대상을 찾을 수 없다: [[학습/공부/README\\]]'] → 수정 후: []
```
