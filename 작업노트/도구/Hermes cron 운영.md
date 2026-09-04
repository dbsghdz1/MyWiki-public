---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-29
updated: 2026-09-04
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

## 기록

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
