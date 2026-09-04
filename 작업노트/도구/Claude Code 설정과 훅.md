---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-21
updated: 2026-09-05
projects:
  - "소프트웨어마에스트로"
---

# Claude Code 설정과 훅

**어떤 설정이 "언제 로드되는가"를 틀리면 조용히 아무 일도 안 일어난다**는 노트. 지금까지 겪은 것은 전부 문법이 아니라 **로딩 시점**의 문제였고, 실패해도 에러가 안 나서 몇 달을 모르고 지나갈 수 있다.

## 핵심 정리

- 설정이 안 먹으면 **문법보다 "언제 읽히는가"를 먼저 의심한다.** 이 계열의 실패는 전부 조용하다.
- **문서로 적은 규칙은 강제가 아니다.** 반드시 실행돼야 하는 것은 훅으로 내린다. 공식 문서도 같은 경계를 말한다 — *"CLAUDE.md instructions shape Claude's behavior but are not a hard enforcement layer."*
- 새로 만든 설정 파일은 **그 세션에서 안 먹을 수 있다.** 만들고 나서 반드시 실제로 발동하는지 확인한다.

## 기록

### 2026-08-19 — `paths: "**/*"`는 "항상 로드"가 아니라 "조건부 로드"다

소마 저장소 `.claude/rules/api-conventions.md`가 API 계약 정본인데 `paths: "**/*"` frontmatter를 달고 있었다. 의도는 "모든 파일에 해당하니 항상 로드"였는데 **정반대로 동작했다.**

- `paths` 필드가 **있으면** path-scoped 규칙이 된다 → Claude가 매칭되는 파일을 **Read한 뒤에야** 주입된다
- `paths` 필드가 **없어야** 세션 시작 시 무조건 로드된다 (*"Rules without a `paths` field are loaded unconditionally"*)

즉 `**/*`는 "모든 파일에 해당"이라 항상 로드되는 게 아니라, **아무 파일이나 하나 읽기 전까지는 컨텍스트에 없다.** "API 하나 만들어줘"로 시작하면 경로·상태코드·에러 형태에 대한 첫 판단을 **계약을 모르는 채로** 내리게 된다.

같은 세션에서 실측으로 확인했다 — 초기 컨텍스트에 없었고, `.kt` 파일을 Read한 **뒤에** 주입됐다. 고치는 방법은 `paths` 블록을 통째로 지우는 것.

**세 문서가 같이 거짓말을 하고 있었다.** 규칙 본문("무조건 따르는 계약")과 두 스킬("항상 로드된다")이 전부 사실이 아니었고, 아무도 몰랐다. 로딩 실패에 에러가 없기 때문이다.

근거: [Claude Code 문서 — memory](https://code.claude.com/docs/en/memory) 「Path-specific rules」

### 2026-08-19 — path-scoped 규칙은 Read에만 걸린다. 새 파일을 만들 때는 안 실린다

*"Path-scoped rules trigger when Claude reads files matching the pattern, not on every tool use."*

Write·Edit은 트리거하지 않는다. **컨벤션이 가장 필요한 순간이 새 파일을 만들 때인데 거기서 안 실린다.** 기존 파일을 먼저 읽으면 걸리지만 그건 우연에 기대는 것이다.

부수 효과 — 세션이 Bash `cat`으로 파일을 읽으면(도구 대신 셸을 쓰는 모드) **Read 도구를 안 거치므로 규칙이 영영 안 실린다.**

### 2026-08-19 — 스킬의 `allowed-tools`는 제한이 아니라 사전승인이다

이름만 보고 "이 스킬은 이 도구만 쓸 수 있다"로 읽었는데 반대였다.

> The `allowed-tools` field **grants permission** for the listed tools during the turn that invokes the skill... **It does not restrict which tools are available**: every tool remains callable.

그래서 스킬이 필요로 하는 도구가 목록에 빠져 있어도 **막히지 않고 권한 프롬프트만 뜬다.** "스킬이 도구를 못 써서 실패한다"고 진단하면 틀린다.

근거: [Claude Code 문서 — skills](https://code.claude.com/docs/en/skills)

### 2026-08-21 — 권한 규칙에서 복합 명령은 조각별로 매칭된다

소마에 `.claude/settings.json`을 처음 만들면서 `"Bash(cd Server && ./gradlew:*)"`를 넣었는데 **영원히 매칭되지 않는 죽은 규칙**이었다.

> Claude Code is aware of shell operators... **A rule must match each subcommand independently.**
> 구분자: `&&`, `||`, `;`, `|`, `|&`, `&`, 개행

`cd Server && ./gradlew build`는 `cd Server`와 `./gradlew build` **두 조각으로 나뉘어 각각** 검사된다. 그러니 `Bash(./gradlew:*)` 하나면 덮이고, `cd`를 포함한 규칙은 쓸 이유가 없다.

문법 하나 더 — **`Bash(ls:*)`와 `Bash(ls *)`는 같다.** 단 `:*`는 **패턴 맨 끝에서만** 인식되고, `Bash(git:* push)` 같은 중간 위치의 콜론은 그냥 문자로 취급돼 아무것도 매칭되지 않는다.

근거: [Claude Code 문서 — permissions](https://code.claude.com/docs/en/permissions)

### 2026-08-21 — 훅 필터를 `if`로 걸면 복합 명령을 놓친다

커밋 직전 검사를 훅으로 걸면서 `"if": "Bash(git commit *)"`로 거르려 했는데, 위와 같은 이유로 **앞부분만 보기 때문에** `git add a && git commit -m b`나 `cd Server && git commit ...`을 놓친다. 에이전트가 만드는 커밋이 실제로 그 모양인 일이 잦다.

해법은 **스크립트가 stdin으로 명령 전체를 받아 스스로 판정**하게 하는 것. 훅 입력은 JSON으로 들어오므로 셸에서 이렇게 걸러도 충분하다.

```bash
case "$(cat)" in
    *'git commit'*) ;;
    *) exit 0 ;;
esac
```

**판정을 git 호출보다 앞에 둔다.** 그래야 커밋이 아닌 Bash 호출에 비용이 안 붙는다(실측 0.00초).

### 2026-08-21 — 새로 만든 `settings.json`은 그 세션에서 안 먹는다

훅을 다 만들고 일부러 컴파일 에러를 낸 코드를 커밋해봤는데 **막히지 않고 그냥 통과했다.** 스크립트는 손으로 돌리면 정상 동작했고 JSON도 유효했다.

원인은 감시 범위였다.

> the settings watcher... only watches directories that **had a settings file when this session started.**

`.claude/settings.json`이 그 세션에서 **처음 생긴 파일**이라 감시 대상이 아니었다. `/hooks`를 한 번 열거나 재시작해야 반영된다.

**교훈은 "훅을 만들었으면 반드시 실제로 발동시켜 봐야 한다"는 것.** 스크립트 단위 테스트가 전부 통과해도 배선이 안 됐을 수 있고, 이 실패 역시 조용하다.

### 2026-08-21 — 훅으로 차단하려면 `deny` JSON을 stdout으로 낸다

`PreToolUse` 훅이 도구 호출을 막는 방법은 종료 코드가 아니라 **stdout JSON**이다.

```json
{"hookSpecificOutput": {
  "hookEventName": "PreToolUse",
  "permissionDecision": "deny",
  "permissionDecisionReason": "왜 막았는지 — 이 문구가 모델에게 그대로 전달된다"
}}
```

`permissionDecisionReason`에 **재현 명령과 실제 에러 줄**을 넣으면 에이전트가 그것만 보고 고칠 수 있다. 실측에서 gradle 출력 꼬리 25줄을 넣었더니 `e: ...Application.kt:13:24 Initializer type mismatch` 같은 결정적인 줄이 그대로 들어갔다.

### 2026-08-27 — 터미널 테마와 Claude Code 테마는 따로 논다 — ANSI 테마로 맞춘다

cmux(Ghostty) 터미널 테마를 TokyoNight로 바꿨더니 **Claude Code 화면만 색이 겉돌았다.** Claude Code의 `theme`(`(로컬 경로)`)이 `dark`/`light`면 **터미널 팔레트와 무관한 자체 고정색**을 쓰기 때문이다 — 터미널 배경만 바뀌고 글자색은 옛 테마 기준이라 조합이 깨진다.

- 해법: `theme`을 **`dark-ansi`/`light-ansi`** 로. ANSI 테마는 터미널의 16색 ANSI 팔레트를 그대로 쓰므로 터미널 테마를 따라가고, Ghostty가 `theme = light:...,dark:...`로 시스템 모드에 따라 팔레트를 바꾸면 **Claude Code도 같이 전환된다.**
- `settings.json`을 고쳐도 **떠 있는 세션에는 적용 안 된다**(위 08-21 항목과 같은 계열). 실행 중 세션은 `/theme`에서 즉시 변경.
- cmux에서 터미널 테마는 `(로컬 경로)`가 정본이고 `cmux reload-config`로 재시작 없이 반영된다.

### 2026-08-27 — ANSI 테마의 대가: 터미널 라이트 팔레트가 저대비면 Claude Code도 같이 안 보인다

위 항목의 후속. ANSI 테마로 맞춘 뒤 **라이트 모드에서 터미널·Claude Code 둘 다 글자가 안 보였는데, 원인은 TokyoNight Day 팔레트 자체**였다 (Desktop 세션에서 Ghostty 테마 파일 실측):

- Day의 ANSI 0(검정) = `#e9e9ed` — 배경 `#e1e2e7`과 사실상 동일. 검정으로 찍히는 텍스트가 통째로 사라진다.
- ANSI 8(밝은 검정, dim/보조 텍스트) = `#a1a6c5` — 배경 대비 **약 1.8:1**. Claude Code가 보조 텍스트·테두리에 쓰는 색이라 화면 절반이 흐려진다.

**교훈: ANSI 테마는 터미널 팔레트 품질에 종속된다.** 라이트 테마를 고를 때는 `palette 0`·`palette 8`이 배경과 충분히 떨어져 있는지 테마 파일에서 직접 확인해야 한다 (테마 파일 위치: `/Applications/cmux.app/Contents/Resources/ghostty/themes/`). 교체 후보 실측: Catppuccin Latte 8=`#6c6f85`(~5:1), GitHub Light Default 8=`#57606a`(~7:1), One Half Light 8=`#4f525e`. → **라이트를 Catppuccin Latte로 교체**(`theme = light:Catppuccin Latte,dark:TokyoNight Storm`)해서 해결.

후속: 홍이 Latte도 본문이 연하다고 해서 fg `#4c4f69`→`#32364e`(대비 ~11:1)·ANSI 7/8 및 노랑·초록 계열을 진하게 만든 커스텀 테마 **`(로컬 경로)`** 를 만들어 `light:Latte Darker`로 지정. **cmux 내장 Ghostty도 `(로컬 경로)`의 커스텀 테마를 읽고 `cmux reload-config`로 반영된다** — 스크린샷으로 적용 확인(2026-08-27). 내장 테마를 통째로 덮지 말고 이렇게 커스텀 테마 파일로 라이트/다크 한쪽만 고치는 게 정석(config에 `palette`를 직접 쓰면 양쪽 모드에 다 적용돼 버린다).

최종: 홍이 "잘 튜닝된 라이트"를 원해 라이트 우선 설계 테마 4종을 실측 비교 → **GitHub Light Default 채택**. 유일하게 ANSI 16색 전부를 흰 배경용 텍스트 색으로 재조정한 팔레트다(노랑 3=`#4d2d00` 진갈색 ~10:1, 8=`#57606a` 7:1). 반면 Flexoki Light(8=`#b7b5ac`)·Selenized Light(0=`#ece3cc`, 8=`#909995`)는 라이트 우선으로 유명해도 **0·8을 배경 톤으로 쓰는 설계**라 dim 텍스트를 ANSI 8로 찍는 Claude Code에선 TokyoNight Day와 같은 방식으로 깨진다. 판별 기준은 하나: **테마 파일에서 palette 0·8이 배경과 충분히 떨어진 텍스트 색인가.**

### 2026-08-27 — "Ghostty 느낌"의 정체는 폰트 — 라틴 JetBrains Mono + 한글 D2Coding 폴백

테마 시리즈의 후속. 테마를 다 맞춘 뒤에도 홍이 "D2Coding이 별로, Ghostty 느낌으로"라고 했는데, **Ghostty 특유의 인상은 기본 폰트 JetBrains Mono에서 온다** — `font-family = "D2Coding"`으로 덮으면 테마가 같아도 느낌이 달라진다.

- 해법: Ghostty는 **`font-family`를 여러 줄 쓰면 순서대로 폴백 체인**이 된다. 1줄째 `JetBrainsMono Nerd Font`(라틴·아이콘), 2줄째 `D2Coding`(한글). JetBrains Mono에는 한글 글리프가 없어서 폴백을 안 두면 시스템 고딕(비고정폭 느낌)이 끼어든다 — 한글 코딩 폰트를 폴백에 명시하는 게 정석.
- 같은 pt에서 JetBrains Mono가 D2Coding보다 x-height가 커서 크게 보인다 — `font-size`를 17→16으로 내려 체감 크기를 맞췄다.
- 적용은 역시 `cmux reload-config`로 재시작 없이 반영.

## 참고 자료

- [Claude Code — Memory (CLAUDE.md·rules)](https://code.claude.com/docs/en/memory) — 2026-08-21 확인
- [Claude Code — Permissions](https://code.claude.com/docs/en/permissions) — 2026-08-21 확인
- [Claude Code — Skills](https://code.claude.com/docs/en/skills) — 2026-08-21 확인

### 2026-09-01 — 백그라운드 셸은 zsh다 — bash 간접 참조가 조용히 못 쓴다

- 맥락: 보험찾개냥에서 PR 리뷰 감시 Monitor 스크립트가 기동 즉시 exit 1.
- 배운 것: Bash 도구·Monitor의 스크립트는 **zsh로 실행된다**(사용자 셸 프로필 기준). bash 전용 간접 참조 `${!var}`는 zsh에서 `bad substitution`으로 즉사한다 — 변수 간접 참조 대신 평평한 변수·명시적 분기로 쓴다. `[ -n ]`·`[ -gt ]` 같은 POSIX 표현은 안전.
- 근거: 에러 원문 `(eval):12: bad substitution`, 간접 참조 제거 후 정상 기동 (2026-09-01 세션)

### 2026-09-05 — 사람만 부르는 스킬(`disable-model-invocation: true`)은 모델의 스킬 목록에 안 보인다 — `/teach`가 그 예

- 맥락: [[학습/README|학습]] 허브에 "학습선생님" 스킬을 만들려고 Matt Pocock 저장소(aihero.dev/skills)를 조사.
- 배운 것:
  - 설치된 `mattpocock-skills` 1.2.3(`(로컬 경로)`)에 **`/teach`가 이미 들어 있다** (`skills/productivity/teach/SKILL.md` + `MISSION-FORMAT.md`·`LEARNING-RECORD-FORMAT.md`·`RESOURCES-FORMAT.md`). 세션 시작 스킬 목록에는 `grilling`·`tdd`·`wizard` 등만 보이고 `teach`·`grill-me`·`handoff`가 없어서 "없다"고 오판할 뻔했다. 원인은 frontmatter `disable-model-invocation: true` — **사람이 `/이름`으로 칠 때만 열리고 모델 목록에서는 빠진다.** 스킬 존재 여부는 목록이 아니라 플러그인 `plugin.json`의 `skills` 배열이나 캐시 디렉터리로 확인해야 한다.
  - 같은 플러그인이 두 마켓플레이스(`claude-plugins-official` 1.2.3, `mattpocock` 1.2.0, 같은 SHA `2ab9580`)에 이중 설치돼 있다. 저장소 README는 "installing both leaves you with every skill twice"라고 경고한다 — 정리 대상.
  - 홍의 학습선생님은 `/teach`를 MyWiki `학습/`에 맞게 개조한 **`(로컬 경로)`**(2026-09-05 초안). 미션=주제 노트 `## 학습 계획`의 `**미션:**` 줄, 학습 기록=`## 기록`, 실험=`학습/야생학습/`. 새 폴더는 만들지 않았다.
- 근거: `(로컬 경로)`, 저장소 `.agents/invocation.md`("User-invoked — reachable only by the human typing its name"), `skills/productivity/writing-great-skills/SKILL.md`.
