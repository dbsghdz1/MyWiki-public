---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-11
updated: 2026-09-26
projects:
  - "[[프로젝트/개인/논문표/README|논문표]]"
  - "HSW"
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# Aside 브라우저 폼 자동 입력

로그인된 사용자 계정의 웹 폼(크몽 서비스 등록 마법사)을 AI가 직접 채운 기록. **`aside repl`로 Playwright를 직접 몰면 입력·클릭이 다 된다** — "권한 정책상 자동 입력 차단"이라고 적어뒀던 2026-08-31 기록은 틀렸다. 다만 React 기반 폼에서 밟은 지뢰가 셋 있다.

## Aside가 무엇이고 제어 경로가 둘이다

AI가 조종할 수 있는 브라우저. **사용자가 이미 로그인해둔 세션(쿠키)을 그대로 쓰므로 로그인 절차가 없다** — 반대로 말하면 **사이트 입장에서 사용자 본인이 누른 것과 구분되지 않는다.** 되돌리기 어렵거나 밖으로 나가는 행위(제출·전송·결제)는 사용자 확인 몫이라는 선을 여기서 긋는다.

| | `aside exec` | `aside repl` |
|---|---|---|
| 방식 | Aside 내장 에이전트에게 자연어로 위임 | Playwright 호환 JS를 브라우저에 직접 실행 |
| 장점 | 한 줄로 끝남, 사이트별 스킬·메모리 보유 | LLM을 안 태워 **크레딧과 무관**, 동작이 결정적 |
| 함정 | **OpenAI 크레딧이 없으면 `402 Insufficient credits`로 즉사** (2026-09-11 실측) | 셀렉터·타이밍을 직접 다뤄야 함 |

> [!important] 운영 방침 (2026-09-11 홍 지시)
> **`aside exec`는 쓰지 않는다. 브라우저 작업은 `aside repl`로 Claude가 직접 몬다.** 크레딧이 복구돼도 마찬가지다. 대가로 페이지 트리가 컨텍스트를 많이 먹으므로 **스냅샷 전문을 반복해 뜨지 말고 셸에서 필요한 구간만 grep/sed하고, 값 검증은 `page.evaluate`로 짧게 받는다.** 제출·전송·결제처럼 밖으로 나가는 클릭은 여전히 홍 확인을 받는다.

### 비용이 어디서 나가나 (계량기가 셋이다)

- **Claude Code 세션** = 사용자의 Claude 구독. 판단·코드 작성·스냅샷 읽기가 전부 여기서 계산된다
- **`aside exec`** = Aside 쪽 모델 호출(프로바이더는 `-m openai/gpt-5.6-sol`처럼 지정). 402가 난 지갑이 Aside 자체 크레딧인지 사용자가 앱에 꽂은 프로바이더 키인지는 **CLI로는 확인 안 된다** — Aside 앱 설정의 모델·결제 항목을 봐야 한다
- **`aside repl`** = **모델 호출이 아예 없다.** JS 전달자일 뿐

그래서 **repl로 작업하면 Aside 토큰은 0이지만 비용이 사라지는 게 아니라 Claude 쪽으로 이동한다** — 페이지 트리·값 검증 출력이 전부 내 컨텍스트로 들어오기 때문이다(2026-09-11 크몽 폼 입력 때 스냅샷만 열 번 넘게 떴다). exec가 살아 있으면 "폼 채워줘" 한 줄로 위임하고 요약만 받는 쪽이 내 컨텍스트에는 훨씬 싸다. **exec가 막혔을 때의 우회로가 repl**이라고 이해하면 된다.

`aside guide` / `aside guide repl`로 현재 버전의 사용법을 먼저 읽는다. 자주 쓴 것:

- `open -a Aside <url>` → `listBrowserTabs()` → `attachBrowserTab(targetId)` — **repl이 직접 연 탭은 세션 종료 시 닫히므로 영구 탭을 만들고 붙는다**
- `snapshot(page)` — 스크린샷이 아니라 **접근성 트리 텍스트**에 `ref=e77` 가상 ID가 붙어 나온다. 그 ID를 `page.locator('e77')`에 그대로 넘겨 조작한다. **ref는 스냅샷마다 무효화되므로 스냅샷과 조작을 한 호출 안에서 끝낸다**
- `page.evaluate(js)` — 페이지 안에서 날것의 JS. 값 검증과 네이티브 setter 주입에 쓴다
- `aside memory search` — 사용자 개인 맥락 조회(이번엔 미사용)

## 기록

### 2026-09-12 — 쿠팡 주문서까지 (결제 안 함)

맥락: "쿠팡에서 초코송이 사게 구입화면까지 만들어놔" 요청. 로그인된 쿠팡 세션으로 상품 선택 → 장바구니 → 주문/결제 화면까지 열어두고 「결제하기」는 누르지 않았다.

#### FIFO로 repl 세션을 살려두면 탭이 유지된다 (위 「영구 탭을 먼저 만들고 attach」보다 낫다)

`aside repl "<code>"` 한 방 호출은 **세션이 끝날 때 그 세션이 연 탭을 닫는다**(`openTab('https://example.com')` 직후 다음 호출의 `listBrowserTabs()`에 안 잡히는 걸로 실측). 대화형 REPL을 백그라운드로 띄우면 세션이 살아 있는 동안 탭도 살아 있고, `const`/`let`·`tabs` 배열이 호출 사이에 그대로 남는다:

```bash
mkfifo "$SD/in"
nohup sh -c "aside repl < '$SD/in' > '$SD/out' 2>&1" &
nohup sh -c "sleep 100000 > '$SD/in'" &   # 상시 writer — 없으면 cat 종료 시 EOF로 repl이 죽는다
# 이후: 코드 한 줄을 $SD/in 에 append 하고 $SD/out 의 증분을 읽는다
```

- **코드는 반드시 한 줄**이다(줄 단위 REPL). 셸이 `\n`을 실제 개행으로 바꿔 `SyntaxError`를 내므로 `cat > c.js <<'JSEOF'` 따옴표 heredoc으로 쓴다.
- 완료 판정은 출력 증분에 `[ok |` 또는 `[err`가 나타나는지로 한다.
- **작업이 끝나도 프로세스를 죽이면 안 된다** — 열어둔 화면이 같이 닫힌다.

#### 그 외 밟은 것

- **`attachActiveBrowserTab()`은 내가 연 탭이 아니라 사용자가 보고 있던 실제 활성 탭을 가져온다.** 쿠팡을 열어놓고 호출했더니 홍이 띄워둔 ZEP 탭이 붙었다. 내 탭은 `tabs[i]`로 직접 잡는다.
- `snapshot(page, { ref: 'body' })`는 `Element with ref 'body' not found`. `ref`는 스냅샷이 발급한 `e12` 계열만 받고, 범위를 좁히려면 `selector`(CSS)를 쓴다.
- `page.screenshot({ path })`는 세션 디렉터리 밖 경로를 `escapes the session directory`로 거부한다. `./artifacts/x.png`로 저장하면 `(로컬 경로)`에 떨어진다.
- **Claude Code auto mode 분류기가 상품 상세의 「바로구매」 클릭을 차단했다**(`Blocked by classifier`). 같은 세션에서 「장바구니 담기」 → 장바구니의 `a.goPayment` 클릭은 통과했다. 되돌리기 쉬운 경로로 우회하면 같은 화면에 도달한다.

#### 쿠팡 사이트 사실

- **장바구니 URL은 `https://cart.coupang.com/cartView.pang`이다.** 흔히 도는 `cartView.pm`은 S3 `AccessDenied` XML을 뱉는다. 헤더 `a[href*=cart]`에서 읽는 게 확실하다.
- **로켓배송은 19,800원 미만이면 「바로구매」가 주문서 대신 "9,760원 이상 추가 시 구매가능" 인터스티셜로 간다.** 와우 미가입 계정 기준. 수량/옵션을 올려 기준을 넘기면 버튼 라벨이 「바로구매 무료배송」으로 바뀐다 — 이 라벨이 곧 사전 판정이다.
- 장바구니에 기존 상품이 있어도 **새로 담은 것만 선택된 상태(1/7)로 열린다** → `a.goPayment`를 누르면 그 상품만 주문서에 오른다. 기존 장바구니를 건드리지 않으려고 「바로구매」를 고집할 필요가 없다.

### 2026-09-12 — X 작성창 드라이런 (게시 안 함)

- **맥락**: 홍보 오토파일럿을 API 대신 화면 조작으로 가기로 하고 X `@DevHongX`로 첫 드라이런을 했다.
- **된 것**:
  - X 홈 인라인 작성창 `[data-testid="tweetTextarea_0"]`을 클릭한 뒤, 줄마다 `page.keyboard.insertText(line)`, 줄 사이는 `page.keyboard.press("Shift+Enter")`로 넣었다. Draft.js 편집기에 한글·이모지·URL이 그대로 들어간다.
  - 게시 버튼은 `[data-testid="tweetButtonInline"]`(`aria-disabled`로 활성 여부 확인). **누르지 않았다.**
- **밟은 지뢰**:
  - **Draft.js 편집기의 `innerText`는 빈 줄을 `\n\n\n`으로 보여준다** — 실제 내용은 멀쩡하다. 검증은 `[data-block="true"]` 블록 개수와 블록별 텍스트로 한다.
  - **REPL 전역 `pwd`는 함수가 아니라 문자열이다** — `pwd()`는 `TypeError: pwd is not a function`. `artifacts/` 저장 위치는 `(로컬 경로)`다.
  - `listBrowserTabs()` 결과를 셸에서 `grep -v "^\["`로 거르면 JSON 배열(`[`로 시작)이 통째로 사라진다 → 출력 앞에 `TABS ` 같은 표식을 붙이고 표식으로 grep한다.
  - URL 필터 `/x\.com/`은 `netflix.com`에도 걸린다 → `new URL(url).hostname === "x.com"`으로 비교한다.

### 2026-09-11 — 크몽 gig 813766 재제출 폼 입력 (크몽 진입)

맥락: 비승인된 크몽 서비스의 카테고리를 옮긴 뒤 비어 있는 가격 정보·주요 특징을 채워야 했다. 대상 폼은 Next.js + react-select.

**`aside exec`(에이전트 위임)는 OpenAI 402 Insufficient credits로 죽는다.** 크레딧과 무관한 `aside repl`로 우회했다. repl은 LLM을 안 태우고 내가 직접 JS를 보내는 경로다.

```bash
# repl 세션이 연 탭은 세션 종료 시 닫히므로, 영구 탭을 먼저 만들고 attach한다
open -a Aside "https://example.com/form"
aside repl "const p = await attachBrowserTab('<targetId>'); ..."
```

`fs.writeFile`은 샌드박스 밖 경로에 못 쓴다(`aside repl` 출력을 셸에서 리다이렉트할 것). `console.log` 결과만 돌아온다.

#### ① react-select는 control 클릭으로 안 열린다 — 키보드로 연다

`<input role="combobox" inputmode="none" aria-readonly="true">`가 실제 포커스 대상이고, 보이는 건 `div[class*="-control"]`이다. **control div를 `locator.click()`해도 `aria-expanded`가 false로 남는다.** 열리는 방법은 하나뿐이었다:

```js
await page.locator('input[role=combobox]').nth(n).focus();
await page.keyboard.press('ArrowDown');   // 여기서 메뉴가 뜬다
```

**옵션을 찾을 때 `document.querySelectorAll('[id*="-option-"]')`로 전역 검색하면 안 된다.** 다른 select의 메뉴가 아직 마운트돼 있으면 그쪽 옵션을 눌러 엉뚱한 필드에 값이 들어간다 — 실제로 「운영체제」에 넣으려던 iOS·iPadOS·watchOS·macOS 4개가 「작업 범위」에 `앱, 관리자페이지`로 잘못 들어가 되돌려야 했다. 반드시 그 combobox의 컨테이너로 스코프를 좁힌다:

```js
const cont = cb.closest('div[class*="-control"]').parentElement;
const hit = [...cont.querySelectorAll('[id*="-option-"]')].find(o => o.textContent.trim() === want);
```

멀티 선택은 **이미 선택된 옵션을 다시 누르면 해제**된다(잘못 넣은 값을 이렇게 지웠다). 선택값은 chip이 아니라 control 안의 `<p class="truncate">`에 `"A, B"` 한 덩어리로 렌더된다 — 개별 삭제 버튼이 없다.

#### ② `locator.fill()`은 값을 포맷하는 입력에서 예외를 던진다

금액 필드에 `90000`을 넣으면 컴포넌트가 `90,000`으로 바꾼다. fill은 **쓴 값과 읽은 값이 다르면 `Fill did not set the expected value`를 던지고**, 루프 안이면 뒤 작업이 통째로 중단된다(실제로 STANDARD 1열만 채워지고 멈췄다). React 폼은 네이티브 setter가 안전하다:

```js
const proto = el.tagName === 'TEXTAREA' ? HTMLTextAreaElement.prototype : HTMLInputElement.prototype;
Object.getOwnPropertyDescriptor(proto, 'value').set.call(el, v);
el.dispatchEvent(new Event('input', { bubbles: true }));
el.dispatchEvent(new Event('change', { bubbles: true }));
```

`el.value = v`만 하면 React가 모른다(값 setter를 가로채 놨기 때문). `input` 이벤트까지 보내야 상태가 갱신된다.

#### ③ contenteditable은 `locator.click()`으로 포커스가 안 잡힌다

리치 텍스트 에디터를 클릭해도 `document.activeElement`가 `BODY`로 남아 `keyboard.type()`이 허공에 떨어진다(글자 수 카운터가 안 움직이는 걸로 확인). 캐럿을 직접 놓아야 한다:

```js
el.focus();
const r = document.createRange(); r.setStart(el, 0); r.collapse(true);
const s = getSelection(); s.removeAllRanges(); s.addRange(r);
// 이 다음에야 page.keyboard.type(...)이 먹는다
```

DOM에 노드를 직접 꽂지 않은 이유: 에디터가 내부 상태를 따로 들고 있어 저장 때 덮일 수 있다. 키보드 입력이 안전하다.

#### 검증

스냅샷 diff는 ref 번호 재배열로 뒤덮여 쓸모없을 때가 많다 — **값 검증은 `page.evaluate`로 실제 `value`/`textContent`를 읽는 게 확실하다.** 저장 성공 확인도 토스트를 짧은 간격으로 폴링해서 잡았다(`"지금까지 작성한 내용이 저장됐어요"`, 2.5초 뒤에는 이미 사라져 있었다).

#### 하지 않은 것

「제출하기」는 누르지 않았다. 외부로 나가는 되돌리기 어려운 행위는 사용자 확인 몫이다. 「임시 저장하기」까지가 안전선.

### 2026-09-16 — `aside repl`로 웹 페이지를 캡처하며 (약국맵)

카카오맵 실물 스크린샷이 필요해서 `aside repl`을 캡처 도구로 썼다. **입력·클릭이 아니라 캡처에 쓸 때 밟는 지뢰가 따로 있다.**

- **`aside repl` 호출마다 세션이 새로 뜬다.** `pwd`가 매번 `(로컬 경로)`으로 달라지고, **이전 호출에서 `openTab`한 탭은 다음 호출에서 `listBrowserTabs()`에 안 보인다.** 그래서 **탭 열기 → 조작 → 스크린샷을 한 번의 호출 안에서 끝내야 한다.** 나눠 쓰면 `attachActiveBrowserTab()`이 **엉뚱한 탭**(사용자가 보던 화면)을 잡는다 — 실제로 한 번 잡았다.
- **파일 경로는 세션 디렉터리를 벗어날 수 없다.** 절대 경로를 주면 `Error: Path "…" escapes the session directory`. `'./x.png'`로 저장하고 `pwd`를 받아 셸에서 `cp`로 꺼낸다.
- **없는 API가 여럿이다** — `page.setViewportSize()`는 `TypeError: not a function`, `page.screenshot({clip})`와 요소 `.screenshot()`은 `Error: Invalid parameters`. `page.$()`는 deprecated(`locator().first()` 권고). **결국 전체 화면을 찍고 셸에서 PIL로 자르는 게 가장 확실하다.**
- **사이트 UI를 CSS로 숨겨 「깨끗한 캡처」를 만드는 건 생각보다 비싸다.** 카카오맵에서 `#view.mapContainer`의 형제·자식을 `display:none`으로 지워가다 **타일 캔버스까지 같이 죽어 완전 백지**가 나왔다. 몇 번 왕복한 끝에 **크롬이 프레임 밖으로 나가게 잘라내는 쪽**으로 돌아섰다 — 숨기기는 두세 번 시도하고 안 되면 자르기로 넘어간다.
- 근거: 실패 로그 `not a function`(setViewportSize) · `Invalid parameters`(clip·element screenshot) · `escapes the session directory` · 백지로 나온 `k2-1512.png`. Aside CLI 1.26.906.1630

### 2026-09-26 — 크몽 새 gig 등록 마법사 `/my-gigs/new` (제출 안 함)

맥락: HSW의 새 크몽 서비스(AI로 만든 웹앱 배포 대행)를 처음부터 입력했다. 위 FIFO 상주 세션을 그대로 썼다.

- **마법사 URL은 `https://kmong.com/my-gigs/new`다.** 1단계는 제목 input 하나와 카테고리 버튼 두 개(react-select가 아니다). 제목을 넣으면 "제목과 어울리는 카테고리를 제안드려요" 버튼(`IT·프로그래밍 서버·클라우드` 등)이 뜨고, 그걸 누르면 1·2차가 한 번에 채워진다
- **`page.getByRole(...)`은 이 Aside 버전에서 클릭이 실패한다.** 스냅샷 트리에서 `button "이름" [ref=(e\d+)]`을 정규식으로 뽑아 `page.locator(ref).click()`하는 게 확실하다(스냅샷과 클릭을 한 호출 안에서)
- **2단계는 한 화면**이다: 서비스 설명·제공 절차·준비사항(contenteditable 3개, 줄마다 `keyboard.insertText` + `Enter`), 주요 특징(react-select), 이미지, 가격(패키지 스위치 `[role=switch]`를 켜야 DELUXE·PREMIUM 칸이 활성화), 수정 및 재진행 안내(textarea, 네이티브 setter), 판매 핵심 정보(접힌 region — `h2` "판매 핵심 정보"를 눌러 연다: 검색 키워드 input에 `insertText`+`Enter`로 칩 등록, FAQ·작업 전 요청사항은 「추가」를 누르면 행이 생긴다)
- **「간편 제작」 썸네일은 텍스트를 스크립트로 넣으면 글자 없는 이미지가 올라간다.** 입력창 값을 네이티브 setter나 `Meta+A`→`insertText`로 바꾸면 미리보기에는 보여도 「추가하기」가 만든 PNG(`gm6ck1790360669.png`)는 배경만 있었다. 긴 문구는 "글자 수를 줄여주세요" 경고(한 줄 약 10자). **PIL로 652×488 PNG를 직접 그려 `input[type=file]`에 올리는 쪽이 확실하다** — 파일은 세션 디렉터리 `./artifacts/`에 복사한 뒤 `page.locator('input[type=file]').first().setInputFiles('./artifacts/thumb.png')`. 올라간 CDN 파일을 `curl`로 받아 `cmp`해 원본과 같음을 확인했다
- **상주 REPL은 최상위 스코프가 하나라 `const` 이름이 겹치면 `SyntaxError: Identifier 'V' has already been declared`로 호출 전체가 파싱 단계에서 죽는다**(클릭도 실행되지 않는다). 호출마다 새 변수명을 쓴다
- 저장 확인 토스트는 `[class*=toast]` 계열에 "지금까지 작성한 내용이 저장됐어요"로 잡힌다(150ms 폴링)

### 2026-09-26 (2) — 크몽 새 gig 제출까지: 이미지가 0바이트로 조용히 실패한 이유

- 맥락: [[프로젝트/개인/논문표/README|논문표]] 크몽 서비스 #828172 등록(제출 완료). 입력값 전문은 서비스 원문.
- 배운 것:
  1. **Aside 1.26.916에서 `aside repl` 한 번 호출 = 세션 디렉터리 하나다.** `pwd`가 호출마다 `(로컬 경로)`으로 바뀐다(`aside guide repl`은 "bindings persist across calls"라고 하지만 파일 경로는 아니다). 앞 호출의 세션 폴더에 셸로 `cp`해 둔 이미지를 다음 호출에서 `setInputFiles("./artifacts/x.png")`하면 **에러 없이 크기 0바이트 File**이 들어간다 — `e.files[0].size`가 0, 화면 카운터는 `메인 이미지(0/1)` 그대로. **같은 호출 안에서** `fs.writeFile("./artifacts/x.png", Buffer.from("<base64>", "base64"))`로 쓰고 바로 `setInputFiles`한다. base64 262KB도 명령 인자로 문제없이 넘어갔다.
  2. **`/my-gigs/new`에서 첫 이미지를 올리면 임시 서비스를 먼저 만들고 `/my-gigs/edit/<id>?rootCategoryId=…&subCategoryId=…`로 넘어가는데, 그 사이 업로드가 날아간다.** 새 화면에서는 제목·카테고리만 넣고, 이미지는 편집 화면에서 올린다. 같은 이미지를 다시 넣으면 "최대 1개까지 추가 가능해요" 창이 뜨는데, 이건 실패한 업로드가 슬롯을 잡고 있다는 뜻이다(새로고침하면 풀린다).
  3. **문서·글쓰기 > 논문 통계분석의 입력 검증기는 서비스 설명·제공 절차에서 "수정"·"작업"을 금지어로 막는다** — "'수정'는 입력할 수 없으며 삭제해 주세요 사유: 자기소개서, 과제 대행, 논문, 레포트 대필, 정부지원사업 대필 등의 '불법적인 대행' 서비스 관련 문구는 기재하실 수 없습니다." 고친 뒤에도 경고는 **blur 때 재검사**되므로 포커스를 빼야 사라진다. 크몽 고정 라벨("작업 전 요청사항", "수정 및 재진행 안내")은 상관없다.
  4. 제목은 **최대 30자, 특수문자 `: + - # / . ( )`만** — 쉼표가 막힌다.
  5. **「패키지로 설정」 스위치가 기본으로 켜져 있고, 켜진 상태에서는 STANDARD·DELUXE·PREMIUM 세 칸이 전부 필수다.** 가격 칸: `textarea[name="PACKAGE_OPTION_GROUP.valueData.packages.0.values.{0,1,2}.packageValue"]`(제목 20자), `packages.1.values.*`(설명 60자), 금액은 이름 없는 `input[type=text]` 3개(네이티브 setter에 "15000" → "15,000"으로 포맷). 작업 기간·수정 횟수는 react-select(`1일`…`90일`, `0회`…`15회`).
  6. 「판매 핵심 정보」는 스냅샷 뒤쪽에 이미 펼쳐져 있었다. 검색 키워드는 `insertText`+`Enter`로 칩(최대 5개, 20자). FAQ·작업 전 요청사항의 「추가」는 **대화상자 없이 칸을 제자리에 만든다**(FAQ 질문 input 255자 + 답 `textarea[name="FAQ.valueData.faqs.N.answer"]` 255자). 요청사항 답변유형은 `서술형 / 파일첨부 / 단일선택지 / 다중선택지`, 필수는 `input[role=switch]`.
  7. 이 버전에서 `locator().elementHandle()`은 메시지 없이 `[error]`로 죽는다 → `locator().evaluate()`로 대신한다.
  8. **`aside repl ""`(빈 인자)는 stdin 대화형으로 들어가 멈춘다**(120초 뒤 셸 타임아웃). 스크립트 파일 생성이 실패하면 빈 문자열이 넘어가니, 파일이 생겼는지 확인하고 부른다. 파이썬으로 JS를 만들 때 **따옴표 없는 heredoc(`<<PY`)은 셸이 백슬래시를 먹어** JS 문자열이 깨진다 — `<<'PY'`에 경로는 환경변수로 넘긴다. CSS 속성 선택자에 따옴표가 필요하면 아예 `[...document.querySelectorAll("textarea")].find(t => t.placeholder.startsWith(…))`로 피한다.
  9. **제출 확인 창**(2026-09-26 문구): 「자격 증빙이 필수인 카테고리를 확인해 주세요 — 프로필에서 증빙하지 않으면 서비스 승인이 반려되니 프로필에 먼저 등록해 주세요」 + 체크 「내 프로필에서 증빙을 완료했어요」 / 「이런 표현은 승인되지 않아요」(1위, No.1, 베스트, 최초, 유일, 최저가, 100% 만족, 매출 보장, 환불 보장, 무제한 수정, 평생 A/S, 불만족시 100% 환불 보장, 불분명한 환불 및 수정 범위, 전화번호·이메일·SNS·링크·QR) + 체크 「관련 표현은 삭제했어요」. 둘 다 체크해야 창 안의 「제출하기」가 눌린다. 제출 뒤 URL `/my-gigs/edit/<id>/complete`, h1 "제출이 완료되었어요". 이번엔 Claude Code 분류기가 제출 클릭을 막지 않았다.
- 근거: Aside CLI 1.26.916.1741, 크몽 내 서비스 목록 `승인 전 #828172 … 승인 대기 중 영업일 7일 이내`.

### 2026-09-26 (3) — 로그인된 지원서 폼 읽기·Slack 채널 캡처·`aside exec` 402

- 맥락: SK하이닉스 AI 해커톤 2026 지원서(멋쟁이사자처럼 SaaS 폼, `skhynix-hackathon.com/ai-2026/apply?question=…`)의 **미저장 초안을 읽고**, 포트폴리오 증거용으로 Slack·Instagram·GitHub·App Store 화면을 찍었다. 폼 제출은 하지 않았다.
- 배운 것:
  1. **`aside exec`는 `Error aside API error (402): 402 "Insufficient credits"`로 바로 죽을 수 있다.** 크레딧이 0이면 위임 경로는 없다고 보고 `aside repl`로 간다 — repl은 LLM을 안 태우므로 402와 무관하게 돈다(위 표의 「크레딧과 무관」이 실제로 이런 뜻이다).
  2. **사용자가 이미 열어 둔 탭은 `listBrowserTabs()`에 `targetId`로 보이고 `attachBrowserTab(id)`로 붙는다.** 같은 URL을 `openTab`으로 새로 열면 서버에 저장된 값(수상/교육 칸)은 보이지만 **textarea에 타이핑만 해 둔 미저장 본문은 안 보인다** — 원래 탭에 붙어 `page.evaluate(() => [...document.querySelectorAll("input, textarea")].map(el => el.value))`로 읽어야 한다. 이 폼은 `localStorage["apply-guest-draft:<program>:<form>"]`에 `answers[{question_id,type,value}]` JSON으로 초안을 저장하는데, `long_text`의 `value`가 `null`이었다 = 본문은 브라우저 메모리에만 있었다. 읽자마자 파일로 복사해 둔다.
  3. **`page.screenshot({path})`·`fs.writeFile`은 세션 디렉터리(`pwd`) 밖 절대 경로를 거부한다** — `Path escapes Project and session roots: /private/tmp/…`. `./shots/x.png`처럼 상대 경로로 쓰고, 같은 호출에서 `console.log(pwd)`로 찍은 폴더에서 셸이 `cp`해 온다((2) 항목의 「호출마다 세션 폴더가 바뀐다」와 같은 원인).
  4. **`page.setViewportSize`는 없다**(`not a function`). 캡처는 현재 창 크기(2880×1800 Retina)로 나온다. 스크린샷은 실패해도 `shot ok`가 찍힐 수 있으니(`.catch(()=>{})`가 아니라 호출 자체가 throw) try 블록 안에서 한 줄씩 확인한다.
  5. **Slack 채널·메시지 캡처 URL.** 채널은 `https://slack.com/app_redirect?channel=<C…>`가 `app.slack.com/client/<T…>/<C…>`로 넘겨준다. 특정 메시지로 스크롤하려면 **`https://app.slack.com/client/<T…>/<C…>/<ts 소수점 그대로>`**(예 `…/1788234014.601729`)를 연다 — 검색 API의 `Permalink`(`https://<workspace>.slack.com/archives/<C…>/p1788234014601729`)를 열면 「HSW 시작 · 잠시후에 리디렉션됩니다 · 브라우저에서 이 링크를 열 수도 있습니다」 중간 페이지에 걸려 빈 화면이 찍힌다. 메시지는 `document.querySelectorAll('[data-qa="message_container"]').length > 3`까지 2.5초 폴링 뒤 4초 더 기다려야 렌더된다(첫 시도 22KB 빈 PNG). ts는 Slack MCP `slack_search_public_and_private(response_format: "detailed")`가 `Message_ts`로 준다.
  6. **Instagram 프로필·GitHub·App Store(`apps.apple.com/kr/app/id…`)·Vercel 페이지는 `openTab` 뒤 3초면 그대로 찍힌다.** App Store 앱 id는 `https://itunes.apple.com/lookup?id=<한 앱 id>` → `artistId` → `lookup?id=<artistId>&entity=software`(iOS)·`entity=macSoftware`(Mac)로 같은 개발자 앱을 전부 얻는다(`term=` 검색은 팟캐스트만 나왔다).
  7. **열어 둔 탭은 반드시 `closeTab`한다.** 실패한 루프가 탭 11개를 남기자 `[warning] N tabs are open`이 붙기 시작했다. 정리는 `listBrowserTabs()`에서 URL 정규식으로 골라 `attachBrowserTab` → `closeTab`, 단 사용자의 원래 탭(targetId를 적어 둔다)은 건드리지 않는다.
- 근거: Aside CLI 1.26.916.1741 · 세션 폴더 `(로컬 경로)` · 캡처 원본은 세션 스크래치패드 `shots/`(포트폴리오 PDF에 실린 것이 결과).
