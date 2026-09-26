---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-27
updated: 2026-09-27
projects:
  - "[[프로젝트/개인/오늘 본 장면/README|오늘 본 장면]]"
---

# 유튜브 쇼츠 CC 소싱과 자동 업로드

CC BY 원본으로 쇼츠를 자동 제작·게시할 때의 결론: **CC 표시만 믿지 말고 원 저작자 채널 화이트리스트 + Claude 판정으로 거른다**, 자막은 **쇼츠 UI 안전 영역(위 13%·아래 25%·오른쪽 13% 제외)** 안에, 업로드는 **Data API가 아니라 Aside REPL로 스튜디오**.

## 핵심 정리

- **CC 필터 검색은 yt-dlp로 된다(API 키 불필요).** 검색 URL `https://www.youtube.com/results?search_query=<q>&sp=<base64>`의 `sp`는 protobuf: `08 03`(정렬=조회수) + `12 06 [08 0N 18 0M 30 01]`(업로드일 N: 3=이번 주·4=이번 달·5=올해 / 길이 M: 1=4분 미만·2=20분 초과·3=4~20분 / 필드6=1 크리에이티브 커먼즈). 예: 조회수순·올해·4분 미만·CC = `base64(bytes([8,3,18,6,8,5,24,1,48,1]))`. `yt-dlp --flat-playlist -J`로 받는다. 유튜브의 CC는 CC BY 하나뿐이다.
- **CC 표시 ≠ 쓸 수 있음.** 09-26 검색 193개 중 절반이 ① 남의 영상(AGT·유명인·방송)을 CC로 다시 올린 채널 ② AI 생성 동물 영상 채널이었다. 원 저작자가 아니면 CC 표시는 무효. 개별 검사는 `yt-dlp -J`의 `license` 필드(`Creative Commons Attribution license (reuse allowed)`)지만, 재업로드 여부는 사람·LLM 판정이 필요하다.
- **Claude(Opus) 판정이 잘 잡은 것**: AI 생성(표지판 글씨가 컷마다 바뀜, 사람과 목재 모핑 잔상, 신발이 갑자기 맨발), 재업로드(방송 로고·다른 채널 워터마크·3인칭 내레이션), 원본 자막 박힘(화면 중앙 영어 대사 자막), 아동 곤란 장면.
- **원본 자막이 박힌 채널은 통째로 쓸모없다.** KarmaBiker는 전 영상에 `Biker:/Cop:` 영어 자막이 박혀 시도 18회 전부 거절 → 채널 단위로 빼고, 한 채널이 시도를 다 먹지 않게 라운드로빈 + 채널당 시도 상한.
- **쇼츠 UI 안전 영역(1080×1920 기준, 2026-09-27 iPhone 캡처)**: 하단 ~1440px부터 채널명·제목·설명, 오른쪽 x>940 · y>1190에 좋아요·댓글·공유, 상단 y<260 좌우 끝에 뒤로가기·검색. 자막을 y=1284(67%)에 두면 `@핸들` 줄과 정확히 겹친다 → y≈1050 또는 560, 글자 폭 900 이하.
- **업로드**: YouTube Data API `videos.insert`는 2020-07-28 이후 만든 미심사 API 프로젝트에서 올린 영상을 비공개로 잠근다 → 스튜디오 화면을 Aside REPL로 조작한다. 흐름은 `studio.youtube.com/channel/<UC…>/videos/upload?d=ud` → `input[type=file]` → 제목 textbox `동영상을 설명하는 제목…` → 설명 textbox `시청자에게…` → radio `아니요, 아동용이 아닙니다` → `다음` 3번 → radio `공개` → `게시`. 첫 방문엔 `YouTube 스튜디오에 오신 것을 환영합니다` 대화상자의 `계속`을 먼저 누른다. 쇼츠 링크(`https://youtube.com/shorts/<id>`)는 세부정보 단계 스냅숏에 이미 있다 — 게시 후 확인 화면 문구는 못 읽을 때가 있어 링크는 앞에서 챙긴다.
- **새 채널 만들기**: `https://www.youtube.com/create_channel` 대화상자의 textbox `이름`·`핸들`. 핸들이 쓰이고 있으면 `이 핸들은 사용할 수 없습니다`와 `채널 만들기` [disabled]. 맞춤설정(`/editing/profile`)의 `input[type=file]`은 배너·프로필·워터마크 순서(`nth(1)`이 프로필), 업로드 뒤 자르기 대화상자 `완료` → `게시`. 핸들·이름은 14일에 2회까지 바꿀 수 있다.

## 기록

### 2026-09-27 — 하루 10개 제작·게시 자동화 첫날
- 맥락: [[프로젝트/개인/오늘 본 장면/README|오늘 본 장면]] 채널 개설, `(로컬 경로)`(`src/factory.py`·`render.py`·`uploader.py`) 작성
- 배운 것:
  - Homebrew ffmpeg에는 `drawtext`·`ass`·`subtitles` 필터가 없다(`ffmpeg -filters`에 안 보임) → 자막은 Pillow로 전체 화면 RGBA PNG를 글자 단위로 만들고 concat demuxer(`file`/`duration`)로 슬라이드쇼 스트림을 만들어 `overlay`한다. 타이핑 중 글자 크기가 흔들리지 않게 완성 문장 기준으로 폰트 크기를 고정한다.
  - `mlx-whisper`(`mlx-community/whisper-large-v3-turbo`)는 Apple Silicon에서 36분 영상을 몇 분에 전사하지만, 음악·무음 구간에서 `over and over…` 같은 반복 환각과 0초 세그먼트 수십 개를 낸다 → 단어 다양성이 1/4 미만인 세그먼트와 연속 중복을 버린다.
  - `claude -p`에 이미지를 주려면 프롬프트에 절대 경로를 적고 `--allowedTools "Read(//<작업폴더>/**)"`만 연다(맨 앞 `//`가 절대 경로). 1편 기획 20~60초.
  - 기획 JSON의 자막 시간은 **원본 기준 초**로 받고 렌더러가 구간·배속을 따라 출력 시간으로 옮긴다 — LLM에게 출력 시간 계산을 시키면 배속 구간에서 틀린다.
  - 원 저작자 CC 채널(09-27): AMomWithDogs(골든리트리버 연출, 영상 400+·쇼츠 650+), KristinaFam, Hsinwei(가족 브이로그, 쇼츠 9,500+), 민아연이네(아기, 한글 자막 박힌 편 많음). 뺀 채널: KarmaBiker(자막 박힘·재업로드 추정), BuildLapse(AI 생성).
- 근거: 09-26 배치 로그(1차 2/10 → 수정 후 10/10, 시도 21), 09-27 배치 9/10 시도 40 · 첫 자동 게시 https://youtube.com/shorts/9RyhIsaKIyk · 로컬 커밋 `9047ced`·`aed820d`
