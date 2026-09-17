---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-09
updated: 2026-09-09
projects:
  - "소프트웨어 마에스트로"
  - "[[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]]"
  - "홍보 자동화"
---

# Instagram 콘텐츠 발행 API

**게시물 자동 발행은 된다 — 이미 하고 있다.** `graph.instagram.com`(Instagram API with Instagram Login) 3단계 컨테이너 발행이고, 페이스북 페이지 없이 액세스 토큰 하나로 돈다.

## 핵심 정리

- **엔드포인트 3개가 전부다.** `POST /{ig-user-id}/media`(컨테이너 생성) → `GET /{container-id}?fields=status_code`가 `FINISHED`가 될 때까지 폴링 → `POST /{ig-user-id}/media_publish?creation_id=`. 캐러셀은 자식마다 `is_carousel_item=true`로 컨테이너를 만들고, 부모를 `media_type=CAROUSEL` + `children=<쉼표 목록>` + `caption=`으로 한 번 더 만든다.
- **로컬 파일을 못 올린다 — multipart 업로드가 없고 `image_url`(공개 HTTPS URL)만 받는다.** 그래서 소마·인스타카드뉴스 둘 다 렌더한 PNG를 **imgbb에 먼저 올려 URL을 얻고** 그 URL을 넘긴다. 이미지 호스팅이 파이프라인의 필수 부품이다.
- **페이스북 페이지는 필요 없다.** `graph.instagram.com`(Instagram Login, 권한 `instagram_business_content_publish`, 옛 `business_content_publish`는 폐기)은 인스타 프로페셔널(비즈니스·크리에이터) 계정만 있으면 된다. `graph.facebook.com` 경로(FB 페이지 연결 + `instagram_content_publish`)와 혼동하면 안 된다 — 홍보 자동화 08-12 채널 조사표의 "비즈니스 계정 + FB 페이지 + 심사 2~4주"는 후자 기준이고, 실제로 돌고 있는 건 전자다.
- **한도는 24시간 이동 창 100건**(과거 25 → 50 → 100으로 올라왔다), **캐러셀은 몇 장이든 1건**. 남은 수량은 `GET /{ig-user-id}/content_publishing_limit`으로 확인한다.
- **한도와 별개로 "API access blocked"가 뜬다.** 인스타카드뉴스가 2026-07-30~08-01에 임시 차단을 겪었고, 발행 간격 옵션(`--interval`, 기본 600초)으로 대응했다. 연속 발행은 텀을 둔다.
- **발행만 된다 — 프로필·하이라이트는 API로 못 건드린다.** 계정 필드(`biography`·`name`·`website`·`profile_picture_url`)는 **읽기 전용**이고 쓰기 엔드포인트가 없다. 스토리 하이라이트는 생성·편집 API 자체가 없다(스토리는 `media_type=STORIES`로 발행만 되고 하이라이트에 담는 건 앱에서 손으로). 검색에 나오는 «Instagram Highlights API»들은 전부 서드파티 읽기(스크래핑)이고, 프로필 편집을 파는 도구는 비공식 private API라 계정 리스크다.
- 컨테이너는 만들자마자 발행하면 실패한다 — 자식은 `status_code=FINISHED` 폴링, 부모 캐러셀도 몇 초 대기 후 `media_publish`.

## 기록

### 2026-09-09 — 소마: "인스타 API로 게시물 업로드 되나?" → 이미 되고 있었다

- 맥락: 보험찾개냥 홍보(접수 창구를 앱 → 인스타 DM으로 내린 뒤)에서 카드뉴스 발행 자동화 가능 여부를 물어봄
- 배운 것:
  - `(로컬 경로)`가 이미 캐러셀 3장을 **`https://graph.instagram.com/v23.0`** 로 발행하고 있다. 흐름: `GET /me?fields=user_id,username` → imgbb 업로드 → 자식 `POST /{user_id}/media -d image_url= -d is_carousel_item=true` → `status_code` 폴링(3초 × 20) → 부모 `media_type=CAROUSEL&children=&caption=` → `POST /{user_id}/media_publish -d creation_id=`
  - 인증은 `IG_ACCESS_TOKEN`(`IGAA…` 로 시작하는 Instagram Login 토큰) 환경변수 하나. imgbb 키는 `(로컬 경로)`의 `IMGBB_API_KEY`를 재사용한다 — **소마 발행 스크립트가 인스타카드뉴스 레포의 .env에 의존한다**(그 파일이 없으면 실패)
  - 캡션은 `caption.txt`에서 읽고 `--data-urlencode`로 넘긴다. 게시물 폴더 규약은 `posts/YYYY-MM-DD-주제/{caption.txt,card1..3.png,render_cards.py}`이고 `./publish.sh posts/2026-09-08-DM접수시작` 처럼 폴더를 인자로 준다
  - 릴스·스토리도 같은 컨테이너 방식(`media_type=REELS`/`STORIES`)이지만 **소마·인스타카드뉴스 어디에도 연동돼 있지 않다**(인스타카드뉴스는 AI 릴스 프로토타입까지만)
- 근거: `(로컬 경로)` (2026-09-02 작성), `posts/` 3개 폴더(09-02·09-03·09-08), [[프로젝트/개인/인스타카드뉴스/README|인스타카드뉴스]] "API access blocked" 대응 기록, 한도 수치는 Meta 콘텐츠 발행 문서 정리 글(2026-09-09 확인)

### 2026-09-09 — 프로필·하이라이트 자동 설정은 안 된다 (문안_프로필_DM.md 1절)

- 맥락: 보험찾개냥 `(로컬 경로)`(이름 필드·소개글 150자·링크·하이라이트 5개·DM 문안)를 API로 적용할 수 있는지 물어봄
- 배운 것:
  - **프로필 편집 엔드포인트가 없다.** Instagram Platform 기능은 메시징 · 미디어 발행/댓글 · 멘션·해시태그 · 인사이트 · 공유 · 임베드뿐이고 **계정 프로필 관리는 목록에 없다**. `biography`·`name`·`username`·`website`·`profile_picture_url`은 조회 필드일 뿐이다
  - **스토리 하이라이트는 생성·편집 API가 없다.** `media_type=STORIES`로 스토리 발행까지는 되지만, 그 스토리를 하이라이트에 넣는 동작은 공식 API에 없다 → 하이라이트 커버는 렌더해두고 **앱에서 손으로** 만든다(반자동이 상한)
  - **무작위 DM 발송(SSH-521)은 API로 못 한다.** Instagram Messaging API는 *사용자가 먼저 말을 건* 대화에만 응답할 수 있고(24시간 표준 메시징 창), 비요청 DM은 정책상 막혀 있다 — 문안 4절은 손으로 보내는 작업이다
- 근거: [Instagram Platform 개요](https://developers.facebook.com/docs/instagram-platform) 기능 목록 WebFetch 확인(2026-09-09), `문안_프로필_DM.md:35`(하이라이트 절)·`:127`(무작위 DM)

### 2026-09-17 — 참고 릴스를 로그인 없이 받아 프레임으로 분석하기

- 맥락: 보험찾개냥 홍이 라이프캐치 릴스 `instagram.com/reel/DdSpF5SALXj`(2026-09-15 게시)를 주며 「이렇게 만들고 싶다」고 함. 결과물 `(로컬 경로)`
- 배운 것:
  - **공개 릴스는 `uvx yt-dlp -o "ref.%(ext)s" --write-info-json <릴스 URL>`로 로그인·설치 없이 받아진다**(`yt-dlp`가 PATH에 없어도 `uvx`가 임시 실행). `ref.info.json`에 캡션 전문(`description`)·`like_count`·`comment_count`·`upload_date`가 들어온다. `duration`·`view_count`는 `None`으로 온다 — 길이는 `ffprobe`로 본다
  - **영상 내용은 콘택트시트 한 장으로 읽는다**: `ffmpeg -i ref.mp4 -vf "fps=1,scale=270:-1,tile=6x4" -frames:v 1 sheet.jpg` — 1초 간격 프레임을 한 이미지로 묶으면 컷 구성·자막 교체 시점이 한 번에 보인다
  - `aside exec`는 이날 `OpenAI API error (402): "Insufficient credits"`로 세션 생성 직후 죽었다 — Aside 에이전트 크레딧이 없으면 브라우저 위임 경로가 통째로 막힌다. 읽기 전용 분석이면 위 yt-dlp 경로가 더 싸다
  - **제너레이터가 같은 PIL 이미지를 매번 `yield`하면 호출부의 `alpha_composite`가 프레임마다 누적된다** — 자리표시 배경에서 카드 그림자가 검은 테두리로 굳어 보였다. `yield ph.copy()`로 해결(`render_reel.py` `bg_frames`)
- 근거: 실제 실행 — 666KB `ref.mp4`(15.02초, 1080×1920)·`ref.info.json` 수신, 콘택트시트로 구조 확인(캐릭터 한 장면 고정 + 상단 대화 카드 4장 + 숫자 엔딩 3.6초)

### 2026-09-17 — Kling 5초 클립 두 개를 목소리 그대로 이어 릴스로 합성

- 맥락: 보험찾개냥 `(로컬 경로)` — 홍이 Kling 3.0 무료(5초 상한)로 뽑은 `clips/01.mp4`·`clips/02.mp4`를 이어 12.3초 `reel.mp4`로
- 배운 것:
  - **영상 AI 프롬프트에 「엔딩 화면·버튼 문구」를 넣으면 깨진 한글이 화면에 박힌다** — 01은 4.2초부터 「영수중 사 한연…」 카드가 덮여 대사 3이 잘렸다. 글자는 프롬프트에서 금지하고(「글자·자막·로고·엔딩 화면 금지」) 자막·엔딩은 `render_reel.py`가 얹는다. 02는 그렇게 해서 깨끗했다
  - **끊을 지점은 받아쓰기 타임스탬프로 정한다**: `uv run --with faster-whisper python3 -c "WhisperModel('small',compute_type='int8').transcribe(wav,language='ko',word_timestamps=True)"` — 설치 없이 돈다. 고유명사는 틀리지만(「펫보험」→「폐포음」) 대사 경계 시각(1.95 / 3.58초)은 쓸 만했다. 대사 2가 끝나는 3.54초에서 자르고 **그 프레임을 다음 클립의 Image-to-Video 시작 이미지로** 주면 다른 계정에서 뽑아도 캐릭터·구도가 이어진다
  - 새 계정은 앞 클립을 모른다 — 프롬프트에 캐릭터 생김새와 **목소리 묘사**(높은 아이 목소리·반말 / 차분한 청년·존댓말)를 다시 적어야 한다
  - 가로(16:9) 클립을 세로 릴스에 넣기: 같은 영상을 `scale=-2:1920,crop,boxblur=40:2,drawbox=c=0xFFF6EA@0.62:t=fill`로 깔고 위에 `scale=1210:-2,crop=1080:680`(1.12배 줌)을 `overlay=0:760`. 두 클립은 해상도(1280×720 / 1920×1080)·샘플레이트가 달라 `scale`·`aresample=48000`·`loudnorm`으로 맞춘 뒤 `concat`
  - 무료 Kling 워터마크는 클립마다 위치가 다르다(01 우하단, 02 하단 중앙) — 가리지 않고 그대로 뒀다
- 근거: 실제 렌더 368프레임·12.27초·오디오 트랙 포함, 프레임 시트로 카드 4장이 대사 시각과 맞는 것 확인

## 참고 자료

- [Instagram Platform — Content Publishing](https://developers.facebook.com/docs/instagram-platform/content-publishing) — 컨테이너 3단계·`content_publishing_limit`·미디어 사양 원전 (2026-09-09 확인)
- [Instagram API with Instagram Login](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login) — FB 페이지 없이 쓰는 경로와 `instagram_business_content_publish` 권한 (2026-09-09 확인)
