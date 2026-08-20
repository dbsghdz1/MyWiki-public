---
type: synthesis
status: needs-review
aliases:
  - Claude Code 디자인 스킬
  - AI design skills
  - Figma MCP
  - Blender MCP
created: 2026-07-31
updated: 2026-08-18
sources:
  - "[[_wiki/Sources/2026/07/2026-07-31-ai-design-skills-웹-조사]]"
---

# AI 디자인 스킬

Claude Code 등 코딩 에이전트에 설치해 쓰는 스킬 중 디자인 관련으로 가장 유명한 것들의 지형도다. 2026-07-30 웹 조사 기준이며, 설치 수는 집계 사이트·블로그가 보고한 2차 수치다. [[_wiki/Sources/2026/07/2026-07-31-ai-design-skills-웹-조사|AI 디자인 스킬 웹 조사]]

> [!note] 근거 상태
> 설치 수·스타 수를 각 스킬의 원 저장소에서 직접 확인하지 않아 `needs-review`다. 이 수치들은 빠르게 변하므로 인용 시 기준일을 함께 표기한다.

## 양대 스킬 (2026-07-30 기준)

- **frontend-design** (Anthropic 공식) — 압도적 1위. 흔한 폰트(Inter·Roboto·Arial·Space Grotesk)를 코드 작성 전에 금지하고, 명시적 미학 방향(브루탈리즘·에디토리얼 등)을 먼저 확정하게 강제해 "AI 슬롭" 디자인을 차단한다.
- **ui-ux-pro-max** — 커뮤니티 1위. 84개 UI 스타일, 192개 컬러 팔레트, 74개 폰트 페어링 데이터베이스를 갖춘 디자인 시스템 자동 생성기.

> [!warning] 근거 충돌
> - frontend-design 설치 수 277,000+ (2026-03 기준) — Composio 기사
> - frontend-design 설치 수 686.9K (2026-07-30 조회) — claudeskills.info
> - 현재 판단: 기준일이 4개월 다르므로 성장으로 설명될 수 있으나, 집계 방식 차이 가능성도 있어 미해결
> - 검토일: 2026-07-31

## 설치 수 상위 목록 (claudeskills.info, 2026-07-30 조회)

| 스킬 | 설치 수 | 역할 |
|---|---|---|
| frontend-design | 686.9K | 안티슬롭 비주얼 디자인 (Anthropic 공식) |
| ui-ux-pro-max | 276.9K | 스타일·팔레트·폰트 DB 기반 디자인 시스템 생성 |
| design-taste-frontend | 274.7K | 랜딩·포트폴리오 템플릿화 방지 |
| shadcn | 244.3K | shadcn/ui 컴포넌트 설치·조합·디버깅 |
| high-end-visual-design | 213.4K | 에이전시급 타이포·간격·그림자·애니메이션 원칙 |
| redesign-existing-projects | 210.1K | 기존 사이트·앱 감사 후 프리미엄 리디자인 |
| emil-design-eng | 151.4K | UI 폴리시·마이크로 디테일 철학 |

## 용도별 보조 스킬

- **접근성**: accesslint(WCAG·대비 검사), web-accessibility-website-audit, Vercel Web Design Guidelines
- **디자인↔코드 브리지**: Figma implement design, playwright/webapp-testing(스크린샷 비교 검증)
- **토큰·팔레트**: theme-factory, color-palette
- **비주얼 산출물**: canvas-design(PNG·PDF 아트), Excalidraw diagram, markdown-slides
- **워크플로**: Design process pack(요구사항→리뷰 7단계), designer skills collection(63개 스킬)

큐레이션 모음은 travisvn/awesome-claude-skills, ComposioHQ/awesome-claude-skills, awesome-claude-design(9개 미학 계열 DESIGN.md 모음)이 대표적이다.

## Figma MCP 실전 노하우 (1차 경험, 2026-08-09 기준)

[[프로젝트/개인/DayTune/README|DayTune]] 디자인 작업에서 검증한 에이전트 Figma 편집(`use_figma` Plugin API) 패턴. 출처: Claude 대화, 2026-08-09.

- **절대좌표 파일은 편집 누적에 취약하다.** 수정할수록 정렬이 어긋나 감사→수정 루프가 끝나지 않는다. 톤·컬러만 유지하고 오토레이아웃 + 로컬 컴포넌트로 재건축하는 편이 총비용이 낮았다 (13개 화면 기준).
- **재건축 순서**: 새 페이지 → 공용 컴포넌트(StatusBar·TabBar variants·Button) → 화면당 `use_figma` 1회 + `node.screenshot()` 인라인 검증 → 구버전을 클론으로 교체. 스크립트가 원자적이라 실패 복구가 쉽다.
- **앵귤러 그라디언트 시작점 회전**: `GRADIENT_ANGULAR`는 3시 방향 시작이 기본. 12시 시작은 `gradientTransform: [[0, -1, 1], [1, 0, 0]]` — 반대 부호(`[[0,1,0],[-1,0,1]]`)를 쓰면 시임이 6시에 나타난다. 스크린샷으로 확인 후 부호를 뒤집는 게 빠르다.
- **도넛/프로그레스 링은 `arcData`로**: 원 3개 겹치기 대신 `ellipse.arcData = { startingAngle: -π/2, endingAngle: -π/2 + 2π·progress, innerRadius: 0.8 }`. 라운드 캡은 양 끝 좌표(`cx + midR·cos θ`)에 작은 원을 얹어 근사.
- **아이콘·실루엣은 `createNodeFromSvg`**: 회전 프리미티브 조합보다 SVG 문자열 임포트가 신뢰성 높다 (스킬 문서 권고와 실전 일치).
- **오토레이아웃 안 장식 요소**: 글로우·스파클은 `layoutPositioning = "ABSOLUTE"`로 얹으면 레이아웃 흐름을 해치지 않는다.
- **텍스트 강조는 `setRangeFills`**: 한글 인덱스는 개행 포함 문자 수로 세야 한다 — 한 글자 어긋나는 실수가 잦으니 `getStyledTextSegments`로 검증.


### 디자인 QA — px 대조 (탭탭, 2026-08-09 기준)

탭탭 macOS 앱 디자인 QA에서 검증한 읽기 전용 워크플로우 (편집이 아니라 구현↔시안 검증 용도). 출처: Claude 대화, 2026-08-09.

- **`get_metadata`가 px 대조의 핵심**: 노드별 x/y/width/height가 나오므로 패딩·간격·정렬을 계산해 코드의 `.padding`/`.frame` 값과 표로 대조한다. 스크린샷 육안 비교보다 신뢰도가 높고, "1pt 차이" 검증은 스크린샷(@2x 렌더)보다 코드 값 대조가 정확하다.
- **색상 3단 확인**: ① `get_variable_defs`(디자인 토큰 있으면 토큰명째) ② 토큰이 없는 색은 `get_screenshot` PNG를 받아 픽셀 샘플링 ③ PIL 없는 맥에서는 zlib로 PNG를 직접 디코드하는 파이썬 스니펫으로 해결 가능.
- **일러스트 추출**: `download_assets`의 svgAssets는 레이어 조각(수십 개)이라 재조립이 어려움 — 전체 노드를 `defaultScale: 2` PNG로 export한 뒤 metadata 좌표로 crop(`sips --cropToHeightWidth h w --cropOffset y x`)이 실용적. 단 **export에는 그림자 여백이 포함**되므로 (렌더 크기 − 노드 크기)/2 만큼 오프셋 보정 필요.
- **인스턴스 내부 노드 접근**: `get_metadata`는 `I13866:26513;0:17` 형식(I-prefix)을 받지만 `get_screenshot`/`download_assets`/`get_variable_defs`는 플레인 id만 받는다 — 인스턴스 내부만 필요하면 부모를 export해 crop.
- **변형(variant) 상태 대조**: 호버·선택 상태는 프레임 이름("...버튼 호버")이 어느 상태의 시안인지 알려준다 — 이름을 근거로 어떤 인터랙션 상태에 어떤 값이 적용되는지 판단하고, 없는 상태(비호버 등)는 추정임을 밝히고 사용자에게 확인.
- **아이콘 굵기가 제각각일 때 — outlined PDF 진단** (2026-08-18 추가): 피그마에서 export한 아이콘 PDF는 스트로크가 면으로 펴져 있어 코드(`renderingMode(.template)`·`frame`)로는 굵기를 못 맞춘다. 굵기는 PDF 콘텐츠 스트림을 zlib로 풀고 `m→l` 짧은 세그먼트 길이 분포(=스트로크 반폭)나 라운드 캡의 베지어 반지름으로 역산할 수 있다 — 탭탭 팝업 아이콘 4종이 0.8/1.2pt 혼재로 확인돼 디자이너에게 통일 재export 요청. 원인 진단은 코드가 아니라 에셋 쪽으로 보내야 한다.

## Blender MCP 실전 노하우 (1차 경험, 2026-08-09 기준)

[[프로젝트/개인/Zappy/README|Zappy]] 눈사람 3D화에서 검증한 에이전트 Blender 조작(`execute_blender_code`) 패턴. 출처: Claude 대화, 2026-08-09.

- **콘 프리미티브의 팁은 로컬 +Z, 밑면은 -Z.** X축 회전 부호를 잘못 잡으면 팁이 오브젝트 안으로 박히고 밑면이 바깥을 본다(당근코가 뒤집힌 원인). 회전 후 `matrix_world`로 밑면 위치를 검산하는 게 안전.
- **카메라 정면에서 앞으로 향한 콘은 원판으로 보인다.** 돌출부가 있는 캐릭터는 3/4 앵글이 기본값 — 정면샷 고집하며 콘을 키우는 것보다 카메라를 트는 게 빠르다.
- **뷰포트 스크린샷 ≠ 렌더.** `view3d.view_camera`는 컨텍스트에 따라 안 먹힐 수 있어 `region_3d.view_perspective = 'CAMERA'` 직접 대입이 확실. 최종 확인은 `render.render(write_still=True)`로 파일 렌더 후 파일을 읽는다.
- **애니메이션은 키프레임 대신 파라메트릭 함수 + 프레임 루프.** 상태 함수(`apply_melt(level)`)를 헬퍼 `.py`로 저장하고 매 호출 `exec(open(...).read())`로 로드 — MCP 호출 간 네임스페이스가 유지되지 않는 문제를 우회한다. 렌더는 20~30프레임 배치로 나눠 타임아웃 방지.
- **join한 메시의 트윙클(scale 애니메이션)은 `transform_apply(scale=True)` 필수.** 오브젝트 스케일에 형태가 들어있으면 `o.scale=(s,s,s)` 덮어쓰기가 형태를 파괴한다.
- **오브젝트가 렌더에 안 보이면 프레임 밖·표면 매몰부터 의심.** 카메라 픽셀 각도 계산까지 갈 것 없이, 대상을 화면 중앙 쪽으로 옮겨 재렌더하는 이분 탐색이 빠르다 (떨어진 당근코가 하단 프레임 밖이었던 사례).
- **조립은 ffmpeg**: 스틸 시퀀스 → `palettegen/paletteuse`(128색, bayer 디더)로 GIF, `libx264 -pix_fmt yuv420p`로 MP4. macOS에 Homebrew ffmpeg 설치돼 있음 (2026-08-09 확인).

## 이 vault에의 적용 (종합)

- 자주 언급되는 시작 조합은 frontend-design + Vercel Web Design Guidelines + Vercel React Best Practices다. 이 Mac의 Claude Code에는 이미 `vercel:shadcn`, `vercel:react-best-practices`, `dataviz`가 설치되어 있다 (2026-07-30 확인).
- 적용 1순위 후보는 [[프로젝트/개인/math-sprint/README|math-sprint]] — React 기반 미니게임 UI라 frontend-design류 안티슬롭 스킬의 효과가 즉시 보이는 프로젝트다.
