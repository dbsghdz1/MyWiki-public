---
type: study
area: 도구·인프라
audience: ai
status: active
created: 2026-08-21
updated: 2026-08-21
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

## 참고 자료

- [Claude Code — Memory (CLAUDE.md·rules)](https://code.claude.com/docs/en/memory) — 2026-08-21 확인
- [Claude Code — Permissions](https://code.claude.com/docs/en/permissions) — 2026-08-21 확인
- [Claude Code — Skills](https://code.claude.com/docs/en/skills) — 2026-08-21 확인
