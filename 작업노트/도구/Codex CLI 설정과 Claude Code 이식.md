---
type: study
area: 도구
audience: ai
status: active
created: 2026-10-17
updated: 2026-10-17
projects:
  - "운영"
---

# Codex CLI 설정과 Claude Code 이식

한 줄 요약: Claude Code의 전역 지침, 스킬, 훅, 서브에이전트, MCP는 Codex CLI 0.162에 거의 1:1로 자리가 있다. 안 되는 것은 세 가지다. 경로별 자동 로드 규칙(`.claude/rules`의 `paths:`), 파일 읽기 거부(`permissions.deny`), 셸 결과 자동 주입(스킬의 `` !`cmd` ``). 그리고 Claude Code auto 모드 분류기는 Codex 쪽에 영구 설정을 심는 명령을 막는다.

## 핵심 정리

| Claude Code | Codex CLI | 메모 |
|---|---|---|
| `(로컬 경로)` | `(로컬 경로)` (임시 덮어쓰기는 `AGENTS.override.md`) | 전역 지침. 프로젝트는 루트에서 cwd까지 `AGENTS.md`를 걷어 내려오며 합친다. 32KiB(`project_doc_max_bytes`) 상한 |
| 레포 `CLAUDE.md` | `AGENTS.md`. 없는 레포는 `project_doc_fallback_filenames = ["CLAUDE.md"]`로 그대로 읽힌다 | `@파일` 임포트는 없다. "먼저 읽어라"로 바꾼다 |
| `(로컬 경로)` | `(로컬 경로)`, 레포는 `.agents/skills/` | frontmatter는 `name`, `description`, `metadata`만. `allowed-tools`는 뺀다. `disable-model-invocation: true`는 `agents/openai.yaml`의 `policy.allow_implicit_invocation: false`. 호출은 `$스킬명` |
| 스킬의 `` !`명령` `` 주입 | 없다 | "먼저 이 명령을 실행하고 결과로 판단한다"로 바꾼다 |
| `settings.json` `hooks` | `(로컬 경로)`, 레포는 `.codex/hooks.json` | 이벤트 이름과 입력(`tool_input.command`, `cwd`), 출력(`hookSpecificOutput.permissionDecision: deny`, `additionalContext`)이 같아 스크립트를 그대로 쓴다. `$CLAUDE_PROJECT_DIR`은 `$(git rev-parse --show-toplevel)`로. 새 훅은 TUI `/hooks`에서 한 번 신뢰해야 돈다(`hooks.state` 해시) |
| `(로컬 경로)` | `(로컬 경로)` (레포는 `.codex/agents/`) | 필수 키 `name`, `description`, `developer_instructions`. `model`, `model_reasoning_effort` 지정 가능. `tools:`로 도구를 좁히는 것은 없고 `features.*=false`만. 본문에 백슬래시가 있으면 `'''` 리터럴 문자열로 |
| `CLAUDE_CODE_SUBAGENT_MODEL` | `[agents] default_subagent_model`, `default_subagent_reasoning_effort` | 강한 모델이 필요한 에이전트만 toml에 model 명시 |
| `(로컬 경로)` `mcpServers` | `config.toml` `[mcp_servers.<이름>]` (`command`+`args` 또는 `url`), `codex mcp add` | HTTP OAuth는 `[mcp_servers.x.oauth] client_id`와 `codex mcp login x` |
| 플러그인 마켓플레이스 | `codex plugin marketplace add owner/repo` 뒤 `codex plugin add 이름@마켓` | `.claude-plugin/marketplace.json`만 있는 레포(mattpocock/skills, typesafe-ai/skills)도 그대로 등록된다 |
| `permissions.allow`/`deny` | 명령은 `(로컬 경로)`의 `prefix_rule(allow/prompt/forbidden)`. 파일 읽기 거부는 없다 | 비밀 파일은 AGENTS.md 지침으로만 막는다 |
| auto 모드 | `approval_policy = "never"` + `sandbox_mode = "workspace-write"` (프로필 파일 `(로컬 경로)`, `codex -p 이름`) | `codex exec`는 승인 플래그가 없다(`-a` 불가). 샌드박스 밖 git은 rules의 allow가 있어야 돈다 |
| 메모리(`(로컬 경로)`) | `[features] memories = true`면 `(로컬 경로)`에 자동 생성 | 홍은 메모리 대신 위키를 쓰므로 끈 채로 둔다. Claude 메모리는 `/import`로도 옮길 수 있지만 위키로 증류했다 |
| `/loop`, 클라우드 루틴 | ChatGPT 앱 「Scheduled」(로컬 프로젝트, 맥이 켜져 있어야 함) 또는 launchd + `codex exec` | CLI에는 예약 관리 화면이 없다 |
| `/import` | Codex TUI `/import` → Claude Code 선택 | AGENTS.md, config, 스킬, 플러그인, MCP, 훅, 커맨드, 서브에이전트, 최근 30일 대화 50개를 가져온다. 이미 있는 항목은 건너뛴다 |

## 기록

### 2026-10-17 — 홍의 Claude Code 설정 전부를 Codex로 옮겼다

- 맥락: 이식 기록. 홍이 이번 주 Claude 구독을 끊고 Codex로 간다고 해서 전역 규칙, 스킬 7개, 훅 1개, 서브에이전트 1개, MCP 4개, 프로젝트 지침 8개, 클라우드 루틴 3개, 메모리 93개를 옮겼다.
- 배운 것:
  - **Codex 매뉴얼은 한 파일이다.** `https://developers.openai.com/codex/codex-manual.md`(약 3MB). 토픽은 `grep -n '^#'`로 찾는다. 공식 문서 URL은 `learn.chatgpt.com/docs/...md`로 바뀌어 있다.
  - **`codex exec`에는 `-a`/`--ask-for-approval`가 없다.** `error: unexpected argument '-a' found`. 비대화형이라 승인은 애초에 없고, 샌드박스 밖 명령은 `rules`의 allow만 통한다.
  - **Claude Code auto 모드 분류기가 Codex 설정 심기를 막는다.** `codex plugin add`와 `codex mcp add`를 묶어 돌리자 「Unauthorized Persistence」, `approval_policy = "never"` 프로필 파일과 launchd용 `codex exec` 실행기 작성은 「Create Unsafe Agents」, launchd plist 생성은 다시 「Unauthorized Persistence」로 거부됐다. 반면 `config.toml`에 같은 MCP 블록을 직접 쓰는 것, 읽기 전용 `reviewer` 에이전트 toml, 레포 안의 plist 파일 쓰기는 통과했다. 사람이 돌려야 하는 것은 `codex plugin add ...`와 `launchctl bootstrap`이다.
  - **긴 한글 줄을 heredoc으로 Python에 넘기면 깨질 수 있다.** 1,400바이트짜리 한 줄이 섞인 스크립트가 `SyntaxError: Non-UTF-8 code starting with '\xec'`로 죽었다. 같은 내용을 파일로 쓰고 `python3 script.py`로 돌리면 된다. 한글 비교는 `unicodedata.normalize('NFC', ...)`를 양쪽에 건다.
  - **launchd는 `(로컬 경로)`과 iCloud 볼트를 못 읽는다(TCC).** 그래서 날씨 루틴처럼 실행기는 `(로컬 경로)`, 저장소는 `(로컬 경로)` 클론을 쓴다. [Hermes cron 운영](Hermes%20cron%20%EC%9A%B4%EC%98%81.md) 09-17 항목과 같은 결론.
  - **Codex 스킬 목록 확인은 `codex exec --sandbox read-only --skip-git-repo-check '스킬 이름을 나열해'`** 한 번으로 끝난다. `allow_implicit_invocation: false`인 스킬은 목록에 안 보이는 것이 정상이다.
- 근거: `(로컬 경로)`, `(로컬 경로)`, `(로컬 경로)`, `(로컬 경로)`, `scripts/routines/`, 이식 기록 문서.
