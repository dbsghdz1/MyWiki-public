---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-29
updated: 2026-09-17
projects:
  - "Hermes Cloud 배포"
---

# Hermes cron 운영

한 줄 요약 — Hermes cron이 조용히 죽는 원인은 대부분 무료 LLM 쿼터다. LLM이 필요 없는 잡은 no-agent 스크립트로 바꾸는 게 유일한 영구 해결이고, 에이전트의 낡은 메모리·설정은 분석 자체를 오염시킨다.

## 핵심 정리

- **Ollama Cloud 무료 티어 주간 한도는 롤링 윈도우로 보인다** — 매일 도는 gpt-oss:120b 에이전트 잡(런당 20분~1시간, 툴콜 다수)이 예산을 계속 소진해서, 주중에 잠깐 회복돼도(08-24·08-27 성공 런) 곧바로 다시 429가 된다. 에러: `HTTP 429: you (dbsghdz1dev) have reached your weekly usage limit`. 모델을 바꿔도 한도는 계정 단위라 근본 해결이 아니고, 소비를 줄이는 것(작은 모델 핀, no-agent 전환)만 유효하다.
- **`hermes cron`의 no-agent 모드가 제로 토큰 해법** — `hermes cron edit <id> --script <name>.sh --no-agent`로 바꾸면 `(로컬 경로)`의 스크립트 stdout이 그대로 Slack에 전달된다(LLM 호출 0). 읽고-추출해-보고하는 잡(브리핑류)은 전부 이걸로 바꿀 수 있다. 아침 브리핑 `calendar_context.sh`(→`daily_context.py`)가 원형: mywiki-sync.lock + `git pull --ff-only` + 섹션 추출 + 포맷 출력.
- **`hermes fallback` 서브커맨드가 존재한다** (add/remove/list) — rate-limit·overload 시 대체 프로바이더 체인. 2026-08-29 현재 미설정. Codex를 폴백으로 넣으면 Ollama 포화 기간에 cron 전부가 Codex 쿼터를 먹어 2026-08-16 사태(대화형 429)가 재발할 수 있어 보류.
- **에이전트의 메모리·설정이 낡으면 분석이 통째로 틀어진다** — Mac Hermes는 `(로컬 경로)`와 `config.yaml` channel_prompts에 존재하지 않는 `(로컬 경로)`(+ 구 폴더 구조)를 기억하고 있었고, 그 상태로 위키를 grep하다 `Path not found`가 나자 "경계가 흐리다"며 이미 끝난 결정(생성기는 하나, 08-16)을 재논의하는 제안을 내놨다. 루틴 시스템을 바꿀 땐 관련 에이전트의 메모리·채널 프롬프트도 같은 커밋 단위로 갱신해야 한다.

- **Hermes 스크립트는 위키 경로를 하드코딩한다 — 볼트 폴더를 옮기면 같은 턴에 고쳐야 한다** (2026-09-04 실측). `daily_context.py`(35행 `relative = Path("계획") / "일간" / ...`)와 `weekly_retro_briefing.py`(58행 `weekly_rel`)가 일간·주간 파일 경로를 직접 조립한다. 09-02 계획 폴더를 `계획/일간/YYYY/MM/`로 재편했을 때 이걸 안 고쳐서 07:20 감시가 사흘 연속 "오늘 일간 파일이 원격에도 아직 없어요" 오경보를 냈다 — 파일은 매일 07:06에 정상 생성돼 있었다. 감시 장치의 오경보는 진짜 장애와 문구가 같아서 저장소(`git log -- 계획/일간`)를 먼저 봐야 구분된다.

- **아침 브리핑은 마크다운 링크에서 URL을 지우고 라벨만 Slack으로 보냈다** (2026-09-13 실측·수정). `daily_context.py`의 `clean_markdown()`이 위키링크 정리와 함께 `re.sub(r"\[([^\]]+)\]\(https?://[^)]+\)", r"\1", line)`로 **링크 전체를 라벨로 치환**했다 — 일간 파일 할 일에 학습 자료 URL을 넣어도 폰에 오는 브리핑에는 제목만 남아 열 수가 없다. 치환식을 `r"\1 \2"`(URL을 캡처해 라벨 뒤에 남김)로 바꿨고, Slack은 맨 URL을 자동 링크한다. **테스트를 같이 넣었다** — `tests/test_daily_context_sync.py`의 `CleanMarkdownLinkTest` 2건(마크다운 링크·맨 URL + 위키링크 혼합). 실행: `(로컬 경로)`(venv에 pytest가 없어 unittest로 돌린다), 6/6 통과.
  - 교훈: **브리핑 스크립트의 「정리(clean)」 함수는 정보를 버리는 자리다.** 일간 파일에 무엇을 넣어도 브리핑에 안 보이면 이 함수를 먼저 본다.

- **⚠️ 정정 (2026-09-12 03:37)**: 아래 "두 호스트 동시 접속 → 재연결 폭주" 인과는 **틀렸다.**
  - Mac 게이트웨이를 03:34에 멈춘 뒤에도 서버 `Session is closed`가 분당 90건 그대로였다.
  - 에러 루프는 서버 게이트웨이 단독 문제(닫힌 aiohttp 세션 재사용)로 보이고, 재시작이 해결 후보다.
  - 게이트웨이를 한 호스트에만 두라는 결론은 **이벤트가 두 곳으로 갈리는 문제** 때문에 여전히 유효하다.
  - 교훈: 에러 수가 늘어난 시기와 설정이 겹친다고 원인으로 적지 말고, 한쪽을 끄고 분 단위로 다시 센다.
  - **해결 확인 (03:40)**: 서버 `systemctl --user restart hermes-gateway.service` 뒤 `Session is closed`가 분당 90건 → 0건이 됐다. **Socket Mode가 `RuntimeError: Session is closed` 재시도 루프에 빠지면 스스로 빠져나오지 못한다 — 재시작이 해법이다.**
  - 확인 명령: `journalctl --user --since "<시각>" | grep -c "Session is closed"`
- **Slack 앱 토큰 하나로 게이트웨이를 두 호스트(Mac launchd `ai.hermes.gateway`와 Oracle systemd user)에서 띄우면 연결이 서로 끊고 붙는다** (2026-09-12 실측, 인과는 로그 기반 추정 — 위 정정 참고).
  - 서버 journal `ERROR slack_bolt.AsyncApp: Failed to connect (error: Session is closed); Retrying...`가 하루 17k(09-01) → 43k(09-11)로 늘었다.
  - Mac 쪽은 `[Slack] Socket Mode unhealthy (transport disconnected); reconnecting`가 반복되고 `gateway.error.log`가 201MB다.
  - 한쪽에 보낸 명령을 다른 쪽이 받아 자기 호스트 경로로 처리하는 일도 생긴다.
  - **게이트웨이는 한 호스트에만 둔다.** `launchctl print gui/$(id -u)/ai.hermes.gateway`에서 `state = running`이면 Mac 쪽이 살아 있는 것이다.
- **모델을 지정하지 않은 cron은 기본 모델의 폴백 체인을 탄다.**
  - 서버 기본 `gemini-3.6-flash`가 `marking google-ai-studio exhausted (status=429)`이면 `Fallback activated: gemini-3.6-flash → gpt-5.6-sol (openai-codex)`로 넘어가 조용히 codex 쿼터를 쓴다.
  - 08-29에 인스타 잡을 `gpt-oss:20b`로 핀했는데, 09-12 `hermes cron list`에서는 모델 미지정이었다. **핀은 설정 변경 과정에서 사라질 수 있으니 목록으로 확인한다.**
  - 2026-09-12 홍 결정: 인스타 잡은 핀하지 않고 이 폴백 체인을 그대로 쓴다. codex 한도를 대화와 공유하는 위험은 감수하고, 429가 재발하면 다시 본다.
- **발행 유닛과 미리보기 유닛이 따로면, 발행이 멈춰도 Slack은 정상처럼 보인다.**
  - `instacardnews-publish.timer`는 09-05~07 `HTTP Error 400: Bad Request`(`resolve_ig_user_id`) 뒤 09-07 23:23에 disabled됐고, 그대로 5일간 꺼져 있었다.
  - 그동안 `instacardnews-morning-preview.timer`는 매일 "08:00 발행 예정"을 보냈다.
  - 확인법: `systemctl --user list-timers --all`에 publish 행이 없으면 꺼진 것이다. 감시 스크립트는 미리보기가 아니라 **발행 결과**를 봐야 한다.
- **같은 시각에 도는 두 준비 작업은 서로 덮어쓴다.** 22:15 cron 에이전트와 systemd `insta_card_news_prepare.timer`가 같은 `output/<날짜>/` 폴더를 쓴다. 09-11에는 systemd가 승인한 `baseball-744858afb6`과 실제 미리보기 `baseball-51402e517c`가 달랐다.

## 기록

### 2026-09-17 — no-agent 감시 잡 신설: 무출력 = 무알림

- 맥락: 외부 JSON API를 하루 2번 읽어 새 항목만 Slack으로 알리는 감시 잡을 하나 추가했다(no-agent, 표준 라이브러리 Python, 같은 폴더 `seen.json`으로 중복 제거).
- 알아낸 것:
  - `hermes cron create --help` 원문: *"--no-agent … Empty stdout = silent. Classic watchdog pattern"* — **새 것이 없을 때 아무것도 print하지 않으면 Slack이 조용하다.** 감시 잡은 「없음」 메시지를 만들지 않는다.
  - `.py`를 `--script`에 바로 줄 수 있다(*".sh/.bash files run via bash, everything else via Python"*) — 기존 잡처럼 `.sh` 래퍼를 둘 필요가 없다.
  - 비대화형 ssh에서는 `hermes`가 PATH에 없다(`bash: hermes: command not found`) — `(로컬 경로)` 절대 경로로 부른다.
  - 채널 ID는 Mac `(로컬 경로)` channel_prompts의 키에서 찾을 수 있다.
  - 명령 형태: `(로컬 경로)`

### 2026-09-15 — 브리핑이 캘린더를 버리고 있었다: 스크립트는 읽는데 포맷터가 안 쓴다

- 맥락: B안 첫 적용일(09-15) 일간 파일 `## 메모`에 *"Google Calendar 커넥터가 이 세션에서 인증되지 않아(비대화형 세션이라 OAuth 진행 불가) 조회하지 못했다"* — 09-13에도 같은 줄이 있었다. **07:00 Claude 루틴의 커넥터는 비대화형에서 OAuth를 못 해 캘린더가 계속 비어 있었다.**
- 알아낸 것: `daily_context.py`의 `main()`은 `out.update(read_calendar())`로 **서버 자격증명으로 캘린더를 정상 조회해 컨텍스트에 담는다**(`calendar_read.py`, venv 자격증명). 그런데 `format_briefing()`은 그 값을 쓰지 않고 **일간 파일의 `## 오늘의 일정` 섹션만** 읽었다 — 즉 데이터는 손에 있는데 버리고 있었다. 08-22에 캘린더 조회를 Hermes → Claude 루틴으로 옮긴 뒤 이 경로가 남았다.
- 한 것: `calendar_lines(context, day)` 신설 — 컨텍스트의 이벤트 중 **오늘 날짜만** 골라 `- HH:MM–HH:MM 제목`으로 만들고, 일간 파일에 이미 있는 제목은 중복으로 넣지 않는다(종일 이벤트는 `(종일)`). 테스트 2건 추가(`CalendarFallbackTest`) — **11/11 통과.**
- 남은 것: 07:00 루틴 쪽 커넥터 인증은 여전히 안 된다. **브리핑에는 일정이 나오지만 옵시디언 일간 파일의 `## 오늘의 일정`은 `(없음)`으로 남는다.**
- **2026-09-15 후속 — 그 상태가 그대로 결론이 됐다.** 홍이 *"캘린더를 수정하지마 이미 있는것만 읽어와"*(과거에 에이전트가 캘린더에 새 일정을 써서 곤란해진 일)라고 지시해, **07:00 루틴에서 Google Calendar 커넥터를 허용 도구 목록에서 제거**했다(조회·쓰기 수단 자체 없음) + 화 11:00 「주간 일정 채우기」 루틴 `enabled: false`. **읽기는 이 스크립트 하나로 모았다** — `calendar_read.py`는 `(로컬 경로)`에 든 **비공개 iCal 피드를 `urllib`로 GET해 파싱**한다(110행, `icalendar`). MCP 커넥터와 달리 **쓰기 API가 아예 없다** — 「읽기 전용」을 권한 설정이 아니라 **도구의 능력**으로 보장하는 배치다.
  - 교훈: **외부 면(캘린더·Jira·Slack)에 쓰는 능력은 규칙으로 막지 말고 도구에서 뺀다.** 프롬프트의 금지 문장은 다음 세션이 다르게 해석할 수 있지만, 허용 목록에 없는 도구는 호출되지 않는다.
- 배운 것: **조회 실패와 표시 누락은 다른 문제다.** 두 경로(루틴·Hermes)가 같은 데이터를 각자 읽는 구조에서는, 한쪽이 실패해도 다른 쪽 값을 쓰면 된다 — 그 연결이 빠져 있었던 것이 이번 원인이다.


### 2026-09-14 — 아침 브리핑을 「할 일 / 개념 / 링크」 세 갈래로 재조립 (B안)

- 맥락: 09-13에 링크 보존을 고친 뒤 홍이 브리핑 형식을 골랐다. 두 안을 Slack으로 보내 비교 — **A**(할 일마다 한 덩어리) vs **B**(종류별 분리). 홍: *"나는 B가 훨씬 좋아"*. 내 추천은 A였는데(항목이 2개뿐이라 헤더가 내용보다 길어진다) **홍의 선택이 B**여서 B로 구현했다.
- 한 것 (`(로컬 경로)`):
  - `extract_section(content, heading, keep_indent=False)` — **들여쓰기 보존 옵션 추가.** 기존엔 `line.strip()`으로 계층을 버려서 자식 줄을 구분할 수 없었다.
  - `parse_tasks(raw_lines)` 신설 — 부모 `- [ ]` 줄 하나에 들여쓴 `- 개념:`·`- 링크:` 자식을 묶는다. 접두어 없는 자식은 개념으로 본다.
  - `task_label()` — `- [ ] ` 제거 + `(아침 45분)` → `(45분)`(브리핑 자체가 아침이라 중복).
  - `weather_line()` 분리 + 한 줄 압축(`맑음 19.4° (17.8~25.7) · 비 없음`), `문제: 없음` 트레일러 삭제, 일정 없으면 `일정 · 없음` 한 줄, **빈 갈래는 섹션 자체를 만들지 않는다**(평일은 링크 섹션이 없다).
  - 비 예보 시 `이불빨래` 제외는 유지 — 이제 **딸린 자식 줄까지 함께** 빠진다(테스트로 고정).
- 테스트: `tests/test_daily_context_sync.py`에 `test_formats_briefing_in_three_buckets`·`test_omits_empty_concept_and_link_sections`·`ParseTasksTest` 2건 추가, 기존 포맷 단언은 B안으로 교체. **`(로컬 경로)` 9/9 통과.** 실제 스크립트에 가짜 일간 파일을 물려 토요일·평일 두 장을 렌더해 Slack으로 확인.
- 짝 변경: 일간 파일이 **「부모 + 들여쓴 자식」** 형식이 됐다 — 07:00 루틴 프롬프트와 계획 「일간 슬롯 도출」에 형식을 박았다. **체크박스는 부모 줄에만** 둔다(자식에 넣으면 일요일 회고의 슬롯 집계 분모가 틀어진다).
- 배운 것: **브리핑 형식을 바꾸려면 스크립트만 고쳐선 안 되고 「일간 파일이 무엇을 담는가」를 같이 바꿔야 한다.** 스크립트는 재조립만 할 수 있고, 없는 정보(개념·링크)는 만들어내지 못한다.


### 2026-09-13 — 아침 브리핑이 학습 링크를 지우고 있었다: clean_markdown 치환식

- 맥락: [[학습/공부/README|공부]] 9·10월 CS 1획독 계획에서 **토요일 「영어 원문 학습 세션」의 자료 URL을 일간 할 일에 실어 아침 Slack 브리핑으로 받자**는 요구. 홍 지시: "너가 링크도 작성해서 옵시디언에 기록하고 헤르메스가 링크를 나한테 보내게 만들어야해"
- 확인한 것:
  - 07:20 잡 `54d6f3817dc6`(이름은 「MyWiki 아침 브리핑·일간 감시」)은 **no-agent** 모드라 `calendar_context.sh` → `daily_context.py` **stdout이 그대로** Slack `C0BMNRH3GV6`로 간다. 즉 브리핑 문구는 LLM이 아니라 스크립트가 정한다.
  - `format_briefing()`은 `## 할 일`에서 `- [`로 시작하는 줄만 골라 `clean_markdown()`을 거쳐 붙인다. 그 함수가 **URL을 버리고 있었다**(위 핵심 정리).
  - 비 예보(`rain_windows` 또는 강수확률 ≥35%)면 `이불빨래` 줄을 빼는 필터도 같은 함수 옆에 있다 — 브리핑은 단순 전달이 아니라 **판단이 들어간 산출물**이다.
- 한 것: `clean_markdown` 치환식 수정 + `CleanMarkdownLinkTest` 2건 추가 + `unittest` 6/6 통과 + `calendar_context.sh` 수동 실행으로 출력 확인(일요일이라 할 일 0건 — "없음"). 백업은 서버 관례대로 `daily_context.py.bak-<UTC타임스탬프>`.
- 근거: `ubuntu@129.225.204.121:(로컬 경로)` 191~192행, `(로컬 경로)` 말미. 접속은 `ssh -i (로컬 경로) ubuntu@129.225.204.121`(Security List가 현재 클라이언트 IP /32만 허용 — IP가 바뀌면 먼저 그걸 의심한다).
- 배운 것: [[학습/공부/CS/네트워크/네트워크|네트워크]] 학습 계획의 URL이 실제로 폰까지 도달하는 경로는 **일간 파일 → git push → Hermes가 pull → 스크립트 포맷 → Slack**이다. 중간에 링크를 지우는 단계가 있으면 계획 파일을 아무리 잘 써도 안 온다.


### 2026-09-12 — 오토파일럿 설계 전 실측: 게이트웨이 이중 실행·발행 타이머 정지·폴백 체인

- **맥락**: 홍이 앱 홍보·인스타 운영을 맥 없이 맡기고 싶다고 요청했다. 인프라 실측(읽기 전용)을 거쳐 오토파일럿 설계안을 썼다.
- **배운 것**: 위 핵심 정리의 2026-09-12 항목 4개. 위키 기록과 실제가 다른 곳이 5곳이었다 — Mac 게이트웨이 실행 중, Zappy 초안 cron active, 인스타 cron 모델 미지정, 서버 기본 모델 gemini, 인스타 발행 정지.
- **근거**:
  - 서버 `systemctl --user list-unit-files`: `instacardnews-publish.timer disabled`
  - `journalctl --user -u instacardnews-publish.service`: `Sep 07 23:00:04 … status=1/FAILURE`
  - Mac `launchctl print gui/501/ai.hermes.gateway`: `state = running`
  - 서버 `hermes cron list`
  - **수정은 하지 않았다** — 명령은 인프라 실측 문서에 있고 홍이 실행한다.

### 2026-08-29 — 죽은 cron 4개 정리: no-agent 전환 + 모델 다운핀
- 맥락: Hermes Cloud 배포 — Oracle Hermes cron 5개 중 4개가 08-23부터 Ollama 429로 전멸(아침 브리핑만 no-agent라 생존). 최소 호출(`hermes -z "OK" -m gpt-oss:120b`)도 429인 것을 실측 확인.
- 배운 것:
  - 주간 회고 브리핑(`adb71824240d`)을 no-agent로 전환 — `(로컬 경로)`(+`.sh`) 신규 작성. `### 완료 현황` 추출·전달 + 고정 회고 질문 3개, 완료 현황이 비어 있으면 Claude 회고 루틴 장애로 경고(감시 역할 유지).
  - Instagram 잡 2개(`d097a778d5da`·`67c8b4ab98a1`)는 `gpt-oss:20b`로 핀 변경 — 예산 소비 축소 목적. Zappy 주간 점검(`2f2f22fce9f4`)은 주 1회라 120b 유지.
  - Mac `(로컬 경로)` channel_prompts 12곳·`memories/MEMORY.md` 라우팅 항목의 `(로컬 경로)` → 실제 iCloud 볼트 경로로 수정(백업: `config.yaml.bak-path-fix-20260829`).
- 근거: 서버 `hermes cron list` 실측(429 ref 다수, 예: `203b316f`), `/home/ubuntu/.hermes/sessions/request_dump_cron_*` 타임스탬프(08-24 20분 성공 런 vs 08-28 10초 429 실패), 스크립트 드라이런 출력.

### 2026-09-04 — 07:20 감시 오경보 사흘: 계획 연·월 폴더 재편 뒤 스크립트 경로 미수정
- 맥락: 홍 "오늘 또 왜 생성못했어". `#00-헤르메스-비서`에 09-02·03·04 07:21 "오늘 일간 파일이 원격에도 아직 없어요. 07:00 생성 루틴을 확인해야 합니다." 그러나 `계획/일간/2026/09/일간 2026-09-04.md`는 Claude 루틴이 07:06 KST에 커밋(`dcb82e5`)·push했고 내용도 정상. 09-01까지는 브리핑이 정상 왔다.
- 원인: 09-02 04:24 `f8b0bd5` "계획 연·월 폴더 재편"(주간 5·일간 23 이동) 이후 Oracle `(로컬 경로)` 35행이 여전히 `계획/일간/일간 {day}.md` 평면 경로를 찾음. `git pull --ff-only`는 성공 → `target.is_file()` False → `cat-file -e origin/main:<옛 경로>`도 False → `file_missing_remote`. `weekly_retro_briefing.py` 58행도 같은 평면 경로였고(일 20:20, 첫 실행 예정 09-06) 미리 고쳤다.
- 조치: 세 파일을 `.bak-<UTC>`로 백업 후 한 줄씩 수정 — 일간 `Path("계획")/"일간"/day[:4]/day[5:7]/f"일간 {day}.md"`, 주간 `f"{tuesday:%Y}"/f"{tuesday:%m}"`, `tests/test_daily_context_sync.py` 42행 동일. `python3 -m unittest tests.test_daily_context_sync` 4개 OK, `python3 daily_context.py` 실행 시 "오늘 브리핑 — 9월 4일 금요일" 정상 출력. 홍이 터미널에서 직접 실행(Claude auto 모드가 원격 파일 수정을 차단).
- 덤: Claude 루틴 "MyWiki 일간 계획 생성"(`trig_014hB2B75srhp5a8HGpmz5qt`) 프롬프트도 옛 경로 `계획/일간/일간 ...`으로 쓰여 있는데, "프롬프트와 위키가 어긋나면 위키 우선" 규칙 덕에 `계획/README.md`를 읽고 새 경로에 쓰고 있다. 프롬프트는 미수정(동작엔 문제 없음).
