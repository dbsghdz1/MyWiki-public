---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-01
updated: 2026-09-01
projects: []
---

# Notion MCP

**claude.ai Notion 커넥터(`mcp__claude_ai_Notion__*`)에는 페이지 삭제 도구가 없다.** DB 행을 지우려면 보관용 페이지를 만들고 `notion-move-pages`로 옮겨 DB에서 빼낸 뒤 사용자가 그 페이지를 통째로 휴지통에 넣게 한다. 검색 정렬·필터, 수식 본문 읽기도 안 된다 — 아래 우회책을 쓴다.

## 핵심 정리

- **삭제 없음.** `notion-update-page`에 archive/in_trash 인자가 없고, `notion-update-data-source`의 `in_trash`는 **DB 통째**다. 행 정리 = 보관 페이지 생성(`notion-create-pages` parent `page_id`) → `notion-move-pages`(한 번에 100개, `new_parent: {type: page_id}`) → 사용자에게 "이 페이지 휴지통에" 안내. 실측: DB 행 22개를 일반 페이지 밑으로 옮기면 DB 속성은 사라지고 페이지로만 남는다.
- **검색은 basic만.** `notion-search`에 `sort: last_edited`나 `filters`를 주면 `Some requested search filters aren't available for this connection. Retry with a basic search.` — 개인(무료) 플랜 연결. 빈 `query`는 filters 없이는 거부되므로 흔한 단어 하나("페이지")로 검색하거나, 공유 범위 파악은 `notion-list-recent-pages`가 더 빠르다.
- **수식 본문은 못 읽는다.** 스키마의 `formulaCode://<ds>/<id>`를 `notion-fetch`에 넣으면 `URL type formulaCode not currently supported for fetch tool`(validation_error 400). 수식은 `ALTER COLUMN "X" SET FORMULA('…')`로 **덮어쓰기만** 가능하니, 원본을 모르면 손대지 않는다.
- **`ALTER COLUMN … SET SELECT(...)`는 기존 옵션을 보존한다.** 기존 옵션명을 같은 색으로 다시 나열하고 뒤에 새 옵션을 붙이면 기존 옵션의 `collectionPropertyOption://…` URL이 그대로 유지됐다(1~4일차 유지, 5~9일차 추가). `SET NUMBER FORMAT 'won'`은 표시 형식만 바꾸고 값·수식은 그대로.
- **템플릿(마켓플레이스 다운로드) DB는 관계가 반쯤 끊겨 온다.** 일정 DB의 relation `지출목록`은 워크스페이스에 없는 collection(`c8b9df30-…`)을 가리키고, 지출 DB의 `관련 일정 연결`만 실제 일정 DB를 가리켰다. **관계는 풀리는 쪽(지출→일정)에서만 설정**한다. 샘플 행(도쿄 4월 일정 11개·엔화 지출 11개)은 내용으로만 구분된다 — 실데이터 넣기 전에 먼저 걷어낸다.
- **날짜 속성은 오프셋 포함 ISO를 그대로 받는다.** `date:시간:start: "2026-10-30T18:10:00+09:00"`, `end: "…T00:10:00+08:00"`처럼 출발·도착 시간대가 달라도 저장되고, SQL 조회는 UTC(`2026-10-30 09:10:00Z`)로 돌려준다. 항공편은 이 방식이 시차 계산을 안 하게 해준다.
- **Gmail 커넥터 `get_thread`의 `PLAIN_TEXT`는 마케팅 메일이면 HTML 원문을 준다.** `plaintextBody`에 `<!doctype html>`부터 통째로 들어오고 5만 자를 넘으면 tool-results 파일로 떨어진다. python `re.sub(r"<style.*?</style>|<!--.*?-->|<[^>]+>", …)` + `html.unescape`로 걷어내면 예약번호·편명·금액이 줄 단위로 남는다. `search_threads` 쿼리는 Gmail 연산자(`from:`, `newer_than:`, OR 괄호)를 그대로 받는다.

## 기록

### 2026-09-01 — 여행 템플릿 페이지를 Gmail 예약 메일로 채우기

- 맥락: 홍 개인 Notion의 여행 일정 템플릿 페이지(마켓플레이스 "여행 일정 & 경비 관리 All in One")에 Gmail의 Mytrip 항공권 확정 메일·앱 캡처를 옮겨 일정 DB·지출 DB를 채우고, 템플릿 샘플 행을 걷어냈다. 위키 내 맥락 문서 없음(프로젝트 아님).
- 배운 것:
  - 삭제 도구 부재 → 보관 페이지 + `notion-move-pages` 22개 일괄 이동으로 해결.
  - `notion-search` sort/filters 거부, `formulaCode://` fetch 거부 — 위 핵심 정리.
  - SELECT 옵션 확장·NUMBER FORMAT 변경은 데이터 손실 없이 됐다.
  - 템플릿 DB의 relation 한쪽이 존재하지 않는 collection을 가리켰다.
- 근거: 세션 내 도구 응답 원문 — `Some requested search filters aren't available for this connection.`, `URL type formulaCode not currently supported for fetch tool.`, `notion-move-pages` → `Successfully moved 22 items`. 대상 DB `collection://4389699b-9238-82ae-b959-07e2db2f1978`(일정)·`collection://69d9699b-9238-82d9-b7ff-0743723f58ca`(지출).
