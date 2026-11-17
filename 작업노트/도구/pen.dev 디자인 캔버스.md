---
type: study
area: 도구
audience: ai
status: active
created: 2026-11-17
updated: 2026-11-17
projects:
  - "CoinPilot (MyCryptoDiary)"
---

# pen.dev 디자인 캔버스

**pen.dev(옛 pencil.dev)는 `.pen` 문서가 앱에 열려 있을 때만 MCP로 그릴 수 있다.** 세션에 도구가 안 붙었으면 MCP 서버 바이너리에 stdio로 직접 연결해도 된다.

## 핵심 정리

- pencil.dev가 pen.dev로 이름이 바뀌었다. 맥 앱은 `/Applications/Pen.app`이고 MCP 서버 이름은 여전히 `pencil`이다. 서버 바이너리는 `Pen.app/Contents/Resources/app.asar.unpacked/out/mcp-server-darwin-arm64`, 인자는 `--app desktop --agent claudeCodeCLI`다.
- 앱이 떠 있고 `.pen` 문서가 열려 있어야 한다. 문서가 없으면 `Failed to access document "". A document needs to be open in the editor to perform this action.`이 난다.
- 빈 문서는 `{"version":"2.22","children":[]}` 한 줄을 `.pen` 파일로 저장하고 `open -a /Applications/Pen.app <파일>`로 열면 된다. `save` 도구로 저장한 결과도 일반 JSON이었다(1.2.23 기준). MCP 안내문은 암호화 파일이라 읽지 말라고 하므로 읽기와 수정은 MCP로만 한다.
- 문서 URI는 `get_app_state`가 돌려준 값을 그대로 쓴다. 한글 경로는 자모가 풀린(NFD) 형태로 퍼센트 인코딩되어 있어 직접 만들면 어긋난다.
- 로컬에 설치한 글꼴은 그려지지 않는다. `fontFamily: "Pretendard"`는 오류 없이 대체 글꼴로 그려져서 내보낸 이미지를 비교해야 알 수 있다. Google 폰트만 쓸 수 있어 Inter로 대신했다.
- lucide 아이콘 이름은 `chart-candlestick`이다. `candlestick-chart`는 오류 없이 물음표 아이콘으로 그려진다.
- `Get` 방문자의 `ctx.bounds`는 호출 시점의 레이아웃이다. 형제 노드를 다 넣기 전에 재면 `fill_container` 폭이 크게 나온다. 고정 크기가 필요한 `path`는 화면을 다 만든 뒤에 재서 넣는다.
- 확인은 `Export([id], "png", <절대 경로>, {scale: 1})`로 한다. `TakeScreenshot`은 폭 400px 정도로 줄어 글자를 읽을 수 없다. `ctx.problems`의 `partially clipped`는 정상 화면에서도 나왔다.
- `Get(document, {depth: 1})`은 거부된다. 최상위 목록은 `Get((n, c) => c.depth === 0 ? [n.id, n.name] : undefined)`로 읽는다.
- 실패한 `execute`는 응답의 `editId`와 `edits`(find, replace)로 고쳐 다시 실행한다. 실패한 호출의 변경은 모두 되돌려진다.

## 기록

### 2026-11-17 — CoinPilot 5개 화면 시안

- 맥락: [CoinPilot 화면 시안](../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/MyCryptoDiary/README.md#화면-시안-2026-11-17). 저장소 커밋 `7297117`의 `design/coinpilot.pen`.
- 설치: 공식 주소 `pen.dev/download/Pen-mac-arm64.dmg`(약 518MB)를 받았다. `spctl -a -vv` 결과는 `Notarized Developer ID`, 서명은 `Developer ID Application: High Agency Inc. (N438VX24T8)`였다.
- 세션에 도구가 안 붙은 경우: 세션을 시작할 때 앱이 없으면 `pencil (ENOENT)`로 실패하고, 앱을 설치한 뒤에도 그 세션에는 도구가 생기지 않았다. MCP 바이너리를 직접 띄워 stdio JSON-RPC(`initialize`, `notifications/initialized`, `tools/call`)로 호출해 작업했다.
- 직접 연결할 때의 함정: 처음에는 `get_app_state`가 60초 넘게 멈췄다가 `failed to execute tool call. you are probably referencing the wrong .pen file`로 끝났고 앱 로그(`(로컬 경로)`)에 `Invalid request: missing request_id or client_id`가 찍혔다. 실행 인자에 `--conversation_id <값>`을 넣고 `tools/call`의 `params._meta.progressToken`을 함께 넣은 뒤 정상 응답했다. 둘 중 어느 쪽이 원인인지는 나눠서 확인하지 않았다.
- 차트 폭: 거래소 화면에서 주문 폼을 넣기 전에 차트 영역을 재서 폭 730이 나왔고 선이 옆 카드 위로 넘쳤다. 화면을 다 만든 뒤 다시 재니 394였다.
- 글꼴: 같은 문장을 Pretendard, Noto Sans KR, Inter, Open Sans로 그려 내보냈더니 Pretendard 줄이 Open Sans 줄과 같은 모양이었다.
- 미확인: 무료 요금제의 「에이전트와 디자인하는 날 월 5일」 제한이 MCP 연결에도 적용되는지.
