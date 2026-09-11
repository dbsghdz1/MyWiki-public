---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-11
updated: 2026-09-11
projects:
  - "HSW"
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
