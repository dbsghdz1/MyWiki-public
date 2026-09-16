---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-16
updated: 2026-09-16
---

# gstack

Garry Tan(YC CEO)이 공개한 Claude Code 스킬 팩. `github.com/garrytan/gstack`, MIT. 2026-09-16 홍의 맥에 설치했다. 설치 자체가 **전역 `(로컬 경로)` 네임스페이스와 `settings.json`을 건드리는 작업**이라, 뭘 심었는지 모르면 나중에 원인 불명의 훅·스킬을 만나게 된다.

## 핵심 정리

- 설치 = `git clone` + `./setup`. **`./setup`이 진짜 설치기**다 — 스킬 심볼릭 링크, Chromium 다운로드, `settings.json` 훅 등록을 전부 여기서 한다.
- **기본값은 prefix 없는 짧은 이름**(`/review`, `/ship`, `/qa`)이라 전역 스킬 네임스페이스를 잠식한다. `--prefix`를 주면 `/gstack-review` 형태로 격리된다.
- `--host`는 기본 `claude` **하나만**이다. `--host auto`를 줘야 PATH에서 `codex`/`opencode`/`kiro-cli`/`droid`를 찾아 그쪽 스킬 디렉터리까지 건드린다.
- **Aside가 깔려 있으면 gstack이 알아서 Aside를 primary 브라우저로 잡는다.** 번들 Chromium은 fallback이다 — 둘 중 하나를 고르라고 CLAUDE.md에 규칙을 쓸 필요가 없다.

## 기록

### 2026-09-16 — 설치: `--prefix`로 격리, Aside가 자동으로 primary가 된다

홍이 "gary ten의 gstack 스킬 다운로드 해줘"로 시작한 작업. 실행한 것:

```bash
git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git (로컬 경로)
cd (로컬 경로) && PATH="$HOME/.local/share/mise/shims:$PATH" ./setup --prefix
```

결과 — **스킬 54개**가 `(로컬 경로)` 심볼릭 링크로 등록됐고, 라우터 별칭 `gstack` 하나가 추가됐다. 로그 마지막 줄:

```
gstack ready (claude).
  browse: (로컬 경로)
  browser: Aside (primary) — gstack browser is the fallback
```

**`--prefix`를 쓴 이유**: 기본값(`--no-prefix`)은 `/review`·`/ship`·`/qa`·`/browse` 같은 짧은 이름을 전역에 깐다. 홍의 기존 스킬(`wiki`·`teacher`·`github-pr`·`hong-pr-review`·`coderabbit`·`appstore-release`·`aside-browser`)과 직접 충돌하진 않지만, `/review`가 내장 `/code-review`·`mattpocock-skills:code-review`와 역할이 겹쳐 라우팅이 흐려진다. 참고로 setup은 **기존 동명 스킬을 지우지 않고** `(로컬 경로)`로 백업한 뒤 교체한다(setup:209·215).

**Aside 자동 감지**는 setup 스크립트에 박혀 있다 — `command -v aside`가 잡히면 로그가 "Aside (primary) — gstack browser is the fallback"으로 바뀌고, 렌더된 `gstack-browse/SKILL.md` 설명도 "Drive a real browser through Aside"가 된다(setup:424, 3230). `GSTACK_SKIP_ASIDE=1`이 opt-out. 홍은 `(로컬 경로)` + `/Applications/Aside.app`이 있어서 자동으로 이 경로를 탔다.

### 2026-09-16 — `./setup`이 전역 `settings.json`에 Stop 훅을 심는다

묻지 않고 등록한다. 설치 후 `(로컬 경로)`:

```json
"Stop": [{ "_gstack_source": "gstack-timeline-stop",
  "hooks": [{ "type": "command",
    "command": "~/.claude/skills/gstack/hosts/claude/hooks/timeline-stop-hook",
    "timeout": 5 }] }]
```

세션 타임라인 항목이 스킬 중단 시에도 닫히게 하는 용도. 원본은 `settings.json.bak.<타임스탬프>`로 백업된다(`(로컬 경로)`). 제거:

```bash
(로컬 경로) remove-source --source gstack-timeline-stop
```

`_gstack_source` 필드로 자기 항목을 식별하므로, 손으로 지우지 말고 이 명령을 쓴다. **안 깔린 것 둘** — plan-tune 훅(AskUserQuestion Pre/PostToolUse 로깅; 비대화형 setup이라 건너뜀, `./setup --plan-tune-hooks` 또는 `GSTACK_PLAN_TUNE_HOOKS=yes`), gbrain(`/gstack-setup-gbrain`).

### 2026-09-16 — Chromium 274MB는 `GSTACK_SKIP_PLAYWRIGHT=1`로 건너뛸 수 있다 (Aside가 있으면 사실상 안 쓴다)

setup은 Playwright Chromium(178.7MiB) + headless shell(94.7MiB) + ffmpeg를 `(로컬 경로)`에 받는다. Aside가 primary면 이건 fallback이라 평소 안 쓰이고, **번들 브라우저 자체가 필요한 스킬은 `/gstack-pair-agent` 하나**다(setup:3234). 안 받으려면 `GSTACK_SKIP_PLAYWRIGHT=1 ./setup`. 홍의 설치에서는 플래그를 안 붙여서 그냥 받았다.

setup이 읽는 환경변수 중 쓸 만한 것: `GSTACK_SKIP_PLAYWRIGHT`, `GSTACK_SKIP_ASIDE`, `GSTACK_PLAYWRIGHT_INSTALL_TIMEOUT`(느린 회선), `GSTACK_CHROMIUM_NO_SANDBOX`(Ubuntu 24.04+ AppArmor), `GSTACK_HOME`(기본 `(로컬 경로)`).

### 2026-09-16 — Codex 설정을 읽지만 `--host claude`면 Codex 디렉터리는 안 건드린다

setup 첫 줄이 `Codex skill profile: gpt-5.6-sol / Source: (로컬 경로)`라서 Codex에도 까는 것처럼 보이는데, **실제 설치 대상은 `--host`가 정한다**(setup:590~625). 기본 `claude`면 `INSTALL_CLAUDE=1`만 서고 `(로컬 경로)`·`(로컬 경로)`·`(로컬 경로)`는 그대로다. 홍의 맥은 이 셋이 전부 존재하므로 `--host auto`는 의도하지 않는 한 쓰지 않는다.
