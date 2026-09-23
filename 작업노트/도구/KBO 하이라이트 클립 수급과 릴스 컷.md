---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-23
updated: 2026-09-23
projects:
  - "[[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]]"
---

# KBO 하이라이트 클립 수급과 릴스 컷

**KBO 경기 영상은 40초 미만·비상업이면 SNS 2차 창작이 허용된다(티빙·KBO 2024~2026 계약).** 클립은 티빙스포츠 유튜브의 경기별 하이라이트에서 `yt-dlp`로 받고, 결정적 장면은 자동 자막 키워드 + 음량 피크로 좁힌 뒤 콘택트시트로 사람이(또는 LLM이) 확정한다. 인스타카드뉴스 레포의 `src/find_highlights.py` → `src/pick_moment.py` → `src/render_short.py` → `src/publish_reel.py`(오케스트레이터 `src/reel_daily.py`)가 이 순서다.

## 핵심 정리

- **정책 — 40초 미만, 수익 목적 아님.** 티빙은 2024-03 KBO 유무선 중계권(2024~2026) 계약과 함께 «40초 미만 쇼츠는 누구나 제작·SNS 공유 가능»을 발표했고, 2025-06 «2차 창작 금지» 논란 때 «규정은 계약 당시와 같다 — 유튜브·포털·SNS 경유 **수익을 목적으로 하지 않을 경우** 허용, 막는 건 불법 수익 창출»이라고 해명했다. 그래서 파이프라인은 40초를 세 군데서 막는다: `render_short.MAX_SECONDS=39.8`, `publish_reel.MAX_SECONDS=40.0`, `reel_daily`의 계획 검증 18~36초.
- **원천 채널은 `youtube.com/@TVINGSPORTS/videos`.** 경기마다 하이라이트 1편(12~17분, 1080p60)이 **경기 종료 30~60분 뒤(9/22 실측 21:30~22:35 KST)** 올라온다. 제목 형식은 클라이언트 언어에 따라 둘 중 하나다 — `[KT vs SSG] 9/22 경기 I 2026 신한 SOL KBO 리그 I 하이라이트 I TVING` 또는 `[Lotte vs. Hanwha] 9/22 Game | 2026 Shinhan SOL KBO League | Highlights | TVING`(서버에서 돌리면 영문이 온다). `find_highlights.parse_title`이 둘 다 파싱하고 영문 팀명을 한글로 되돌린다. `@KBO` 채널은 videos/shorts 탭이 없다. 티빙스포츠 **shorts 탭**은 티빙이 편집한 완성 컷(팬덤중계 등)이라 그걸 다시 올리면 재업로드에 가깝다 — 쓰지 않는다.
- **`yt-dlp`는 설치 없이 `uvx yt-dlp`.** 하이라이트는 `-f "299+140"`(1080p60 avc1 + m4a)로 16분에 약 690MB. 분석 단계는 오디오(`-f "ba[ext=m4a]"`)와 자동 자막만 받고, 본편은 후보가 정해진 뒤 **구간만** 받는다: `--download-sections "*S-E" --force-keyframes-at-cuts`(앞뒤 1초 여유를 붙여 `pick_moment.download_section`이 호출). 2026.08 버전부터 JS 런타임(deno) 없으면 경고가 뜨지만 다운로드는 된다.
- **JS 런타임을 안 주면 영상 서버가 403을 준다.** yt-dlp 2026.08부터 서명(n 파라미터) 풀이에 JS 런타임이 필요하고 기본은 deno만 켜져 있다. 없으면 `--download-sections`가 `Error opening input files: Server returned 403 Forbidden` → `ERROR: ffmpeg exited with code 8`로 죽는다(처음 몇 번은 되다가 같은 날 오후부터 막혔다). **`--js-runtimes node:(로컬 경로)`** 로 해결 — mise shim은 `env -i HOME=… PATH=/usr/bin:/bin`에서도 돈다. `pick_moment.js_runtime()`이 deno/node/mise shim 순으로 찾는다. `uvx yt-dlp@latest`로 불러 유튜브 변경 때 새 버전이 저절로 잡히게 한다(`uvx yt-dlp`는 캐시된 버전에 고정된다).
- **제목은 `경기`·`Game`에 더해 `Match` 변형도 있다** — `[Kiwoom vs KIA] 9/17 Match | … | Highlights | TVING`. 이걸 놓치면 그날 경기가 조용히 빠진다.
- **`--download-sections`는 같은 출력 이름이 있으면 새 구간을 무시하고 «already downloaded»로 옛 파일을 준다**(`--force-overwrites` 기본 꺼짐). 구간을 파일명이나 표식 파일(`.range`)에 넣고, 바뀌면 지우고 `--force-overwrites`로 받는다.
- **Oracle 서버에서는 유튜브를 못 받는다.** 채널 목록(`--flat-playlist`)은 되는데 영상 페이지는 `ERROR: [youtube] …: Video unavailable` — `/tmp`에 deno 2.9를 넣어도 같아서 IP 차단으로 판단했다(2026-09-23). 그래서 릴스 파이프라인은 **Mac launchd(`com.instacardnews.reel`, 23:45 KST)** 에서 돈다. 카드뉴스 발행 타이머는 그대로 서버다.
- **자막 키워드는 기록 잡담과 조사를 걸러야 한다.** `"홈런" in text`는 «시즌 11 홈런», «백홈런 돌파», «피홈런»에도 걸려 9/22 두산-키움에선 잡담 구간이 실제 양석환 홈런보다 1순위가 됐다 → 생중계 콜(«넘어갑니다», «홈런입니다»)이 없고 기록 표지(시즌·번째·통산·피홈런·백홈런)가 있으면 0.8점. 감탄사 «와»를 부분 문자열로 세면 «3루와 1루»의 조사가 걸린다(9/22 자막 105줄 중 36줄) → 독립된 말일 때만. 키워드 시각은 줄 시작(`tStartMs`)이 아니라 조각 시각(`tStartMs + segs[].tOffsetMs`)을 쓴다 — 줄이 최대 8.6초라 콜이 몇 초 앞당겨 찍힌다.
- **자동 자막(ko json3)이 장면 찾기의 1차 신호다.** `--write-auto-subs --sub-langs ko --sub-format json3` → `events[].tStartMs` + `segs[].utf8`. 고유명사·숫자는 자주 틀린다(«만루»→«말로», «힐리어드»→«힐리어»). **캐스터의 «홈런!» 콜은 타격 5~10초 뒤**에 나오므로 첫 강한 키워드가 말해진 시각에서 16초 앞(`PRE_ROLL`)에서 시작해야 투구·타격이 들어온다. 9/22 kt-SSG 힐리어드 만루포는 타격 834~836s, 자막 «홈런» 847.8s, HOMERUN 그래픽 845s였다.
- **음량은 `ffmpeg -af ebur128=peak=none -f null -`** 의 stderr(`t: … M: …` momentary LUFS)를 1초 버킷 평균으로 쓴다. 장면 전환은 `select='gt(scene,0.4)',showinfo`의 `pts_time` — 중계 하이라이트는 디졸브가 많아 0.4에선 컷이 드물다(852~872s에 1개, 866.4s). 끝은 그 컷에 맞추면 다음 이닝 그래픽(«9회말»)이 안 섞인다.
- **콘택트시트로 타이밍을 확정한다**: `fps=1,scale=320:-1,tile=6xN`. Homebrew ffmpeg 8.1.2에는 **`drawtext` 필터가 없다**(`No such filter: 'drawtext'`) — 시각 라벨은 PIL로 칸마다 그린다(`reel_daily.annotate_sheet`).
- **launchd는 `(로컬 경로)`을 못 읽는다** — 밤 작업은 `(로컬 경로)`에 복사한 실행본으로 돈다([[작업노트/도구/macOS 파일 접근 권한과 휴지통|macOS 파일 접근 권한]]). 레포에서 고친 뒤 `scripts/install_reel_runtime.sh`를 돌려야 반영된다.
- **문안 사실은 기사로만, 두 번 검증한다.** 자막·화면에서 읽은 아웃카운트 같은 것도 기사에 없으면 쓰지 않는다. `reel_daily.verify_copy`가 두 번째 `claude -p`로 주장을 기사와 대조하고, 고친 문안을 한 번 더 검증한다. 9/22 Sonnet 5 문안의 오류 3개(아웃카운트·«격차를 벌렸다»·#직관)를 잡았다. 헤드리스 호출은 `--model opus --setting-sources project,local --no-session-persistence`로 한다 — 사용자 설정의 훅·기본 모델을 끌고 오지 않는다(`--bare`는 OAuth·키체인을 안 읽어 구독 로그인으로는 못 쓴다).
- **렌더 레이아웃(야구1열식)**: 1080×1920, 배경은 클립을 채움·블러·어둡게, 전경은 가로 1080 맞춤(**zoom 1.15는 중계 스코어버그 «KT 4 / SSG 2»의 왼쪽이 잘린다** → 1.0), 위에 노란 킥커 칩 + Pretendard Black 제목 2줄, 클립 아래 푸터, 하단 `@yagu.3cut`. 인코딩은 Reels 사양대로 `libx264 high yuv420p -r 30 -g 60 -sc_threshold 0`, `aac 128k 48kHz`, `-movflags +faststart`. 34초에 30MB.

## 기록

### 2026-09-23 — 첫 릴스: 9/22 힐리어드 시즌 38호 만루포 (kt 8-2 SSG)

- 맥락: [[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]] — 홍 "야구 계정에 릴스를 올리고 싶어". 07-23 AI 나레이션 프로토타입(스크립트 유실)과 08-01 경기 클립 시제작(`render_short.py`·`publish_reel.py`, 미발행) 중 **경기 클립 형식**을 택했다. 결과 [[프로젝트/개인/인스타카드뉴스/릴스 발행 시작 2026-09-23|릴스 발행 시작 2026-09-23]]
- 배운 것: 위 핵심 정리 전부. 추가로 —
  - 연합뉴스 종합 기사(«힐리어드 38호 아치 만루홈런 폭발…», 22:34)가 그날의 장면을 정해 준다. 전적 기사(`[프로야구 인천전적] kt 8-2 SSG`)의 `△ 홈런 = 힐리어드 38호(9회4점·kt)` 줄이 홈런 이닝·타점의 기계 판독 원천이다. 기사 본문은 `<div class="story-news article">` 안 `<p>`이고 WebFetch는 yna.co.kr을 못 열어 `curl -A Mozilla`로 받았다.
  - `pick_moment.score_windows`가 처음엔 852.2s에서 시작하는 구간을 냈다(첫 «홈런» 자막 −5초). 콘택트시트로 보니 타격은 834s — 그래서 PRE_ROLL을 14초로, 앵커를 피크 앞 30초 안의 **가장 이른** 키워드로 바꿨다. 자막만 믿으면 홈런의 절반이 잘린다. (같은 날 리뷰 뒤 수정: 30초 안 «가장 이른» 줄은 약한 잡담 줄까지 잡아 피크가 구간 밖으로 밀렸다 — 기준점은 피크 앞 20초~뒤 5초의 **강한**(1.5점 이상) 키워드 줄로, 시각은 조각 시각으로, PRE_ROLL은 16초로 바꾸고 피크 뒤 6초는 반드시 담게 했다.)
  - 리뷰 관점의 컷: 투구 2초 전(832s) → 타구 → 관중 «힐리어드 홈런» 피켓 → HOMERUN 그래픽 → 슬로 리플레이 → 홈인·하이파이브 → «KT 8-2 SSG» 그래픽(866.4s 컷). 34.4초.
- 근거: `(로컬 경로)`(mp4·sidecar json·caption·시트), `publish_reels.json`의 `media_id 18116851507983873`, 발행물 https://www.instagram.com/reel/DdnnLy_DSLq/ (2026-09-23 14:58 KST). 정책 출처는 아래.

### 2026-09-23 — 밤 자동화 설치: 리뷰 24건 확인·수정, launchd TCC, 403

- 맥락: [[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]] 자동 파이프라인(`reel_daily.py`)을 launchd `com.instacardnews.reel`(23:45, 06:30 재시도)로 설치 — [[프로젝트/개인/인스타카드뉴스/릴스 발행 시작 2026-09-23|릴스 발행 시작]]
- 배운 것: 위 핵심 정리의 403·`Match`·구간 재사용·자막 오탐·launchd·사실 검증 항목. 추가로 —
  - 첫 헤드리스 호출은 `{"api_error_status":429,"result":"You've hit your session limit · resets 5:20pm"}`로 죽었다 — 대화 세션과 같은 구독 한도를 쓴다. 그래서 06:30 재시도 슬롯과 «같은 날 같은 알림 한 번» 장치를 넣었다. 재시도는 서버 08:00 카드 발행과 겹치지 않게 06:30.
  - `ffprobe`로 본 첫 릴스: `elst`(edit list)가 비디오·오디오 트랙에 있다. Meta 사양은 «no edit lists»지만 발행은 됐다 — 거절되면 `-use_editlist 0`부터 본다.
  - 중계 스코어버그(좌상단 `crop=640:300:0:0`)의 빨간 점 수가 아웃카운트다. 첫 릴스 캡션의 «1사 만루»는 틀렸다(실제 2사).
- 근거: `(로컬 경로)`(41개), launchd 점검 로그(yt-dlp·구간 다운로드·`claude` Opus 5.5·`hermes send`), `output/2026-09-22/reels/verify1.json`

## 참고 자료

- [디지털데일리 — 티빙, KBO 독점 공개 ② "류현진 쇼츠, SNS 업로드 OK"…40초↓ 룰 변수](https://m.ddaily.co.kr/page/view/2024030419170652732) — 2024-03-04 계약 발표와 40초 룰 (2026-09-23 확인)
- [미디어오늘 — 티빙이 프로야구 '쇼츠' 영상 금지했다? 확인해보니](https://www.mediatoday.co.kr/news/articleView.html?idxno=326980) — 2025-06 논란, «수익 목적이 아닐 경우 허용» 원문 (2026-09-23 확인)
- [경향 — 티빙 'KBO 2차 창작 금지' 논란 해명 "규정 바뀐 것 없어"](https://sports.khan.co.kr/article/202506131630003) — 2025-06-13 (2026-09-23 검색 결과에서 확인)
- [Instagram Platform — Reels 미디어 사양·REELS 컨테이너 파라미터](https://developers.facebook.com/docs/instagram-platform/instagram-graph-api/reference/ig-user/media) — MP4·H.264·23~60fps·최대 15분·300MB (2026-09-23 확인)
