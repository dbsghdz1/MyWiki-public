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

## 참고 자료

- [Instagram Platform — Content Publishing](https://developers.facebook.com/docs/instagram-platform/content-publishing) — 컨테이너 3단계·`content_publishing_limit`·미디어 사양 원전 (2026-09-09 확인)
- [Instagram API with Instagram Login](https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login) — FB 페이지 없이 쓰는 경로와 `instagram_business_content_publish` 권한 (2026-09-09 확인)
