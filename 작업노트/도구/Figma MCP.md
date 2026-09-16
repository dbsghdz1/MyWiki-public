---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-04
updated: 2026-09-16
projects:
  - "보험찾개냥"
  - "[[프로젝트/개인/약국맵/README|약국맵]]"
---

# Figma MCP

`use_figma`(Plugin API JS 실행)·`get_design_context`·`get_metadata`를 쓰며 알게 된 것.

## 핵심 정리

- **`use_figma` 스크립트는 전체가 한 트랜잭션이다.** 중간에 예외가 나면 그 앞의 mutation까지 전부 롤백된다 — "앞부분은 적용됐겠지"로 이어 쓰면 안 되고, 에러 뒤에는 캔버스를 읽어 확인한다. 실측: 프레임 이동·복제·appendChild 20여 개를 한 뒤 마지막 줄 `query()`가 던졌더니 아무것도 남지 않았다.
- **`node.query()` 셀렉터 값에 `/`가 들어가면 파서 에러**(`Invalid selector: unexpected character '/'`). `icon/chevron-right`처럼 슬래시 이름이 흔한 아이콘 컴포넌트는 `page.findAllWithCriteria({ types: ['INSTANCE'] })`로 인스턴스를 찾아 `mainComponent.id`를 쓴다.
- `get_metadata`는 프레임이 있는 페이지를 **자동으로 못 찾는다** — 페이지 목록(`nodeId` 없이)이 첫 페이지만 돌려주는 경우가 있었다. `use_figma`로 `figma.root.children`을 읽고 `getNodeByIdAsync(id)`의 부모를 따라 올라가면 확실하다.
- 오버레이(시트·다이얼로그) 화면은 **기존 오버레이 화면의 `LOCAL/Scrim`·시트 프레임을 `clone()`해 새 프레임에 `appendChild`**하는 게 가장 싸다 — 토큰 바인딩(scrim 변수·radius·fill)이 따라온다. auto-layout 부모에 넣은 뒤 `layoutPositioning = 'ABSOLUTE'` + `constraints`로 앵커.

- **컴포넌트 상태(hover/disabled)의 실제 색은 `get_variable_defs`로 안 나온다.** 그 도구는 프레임이 쓰는 토큰 목록만 주고 "어느 변형에 어느 토큰"인지는 빠진다. `get_screenshot`으로 PNG를 받아 각 스와치를 픽셀 샘플링하면 변형별 값이 바로 갈린다(탭탭 Btn 컴포넌트셋 실측: primary `#6251FB→#5345D5`, gray `n30→n40`, white `n10→n30`, 채워진 danger는 변화 없음). 스와치 좌표는 `get_metadata`의 심볼 x/y/width/height에서 계산하되 **프레임 원점을 빼야** 이미지 좌표가 된다.
- `get_screenshot`은 이미지 대신 **짧은 수명의 URL**을 준다 — `curl -L -o`로 받아 두고 로컬에서 샘플링하는 편이 컨텍스트도 아끼고 재확인도 쉽다.
- **플러그인 환경의 폰트 목록은 파일이 쓰는 폰트와 다르다.** 시안 전체가 Pretendard인데 `loadFontAsync({family:"Pretendard"})`가 `The font family "Pretendard" does not exist`로 죽었다(에러 본문이 대안 폰트를 같이 알려준다). Figma 렌더러/로컬에는 있어도 `use_figma` 샌드박스에는 없을 수 있다 — **새로 만드는 텍스트는 `listAvailableFontsAsync()`로 실재를 확인한 계열로 짜고**(한글은 `Noto Sans KR`: Black/Bold/DemiLight/Light/Medium/Regular/Thin, SemiBold 없음), 나중에 사람이 select-all로 원래 폰트로 바꾸게 둔다. **기존 노드를 `clone()`해도 `characters`를 고치려면 그 폰트를 로드해야 하므로 우회가 안 된다.**
- **대시 테두리 속성명은 `dashPattern`이다.** `strokeDashPattern`은 `no such property 'strokeDashPattern' on FRAME node`로 죽는다.
- **`layoutPositioning = 'ABSOLUTE'`는 부모가 auto-layout일 때만 된다** — 일반 프레임에 넣고 설정하면 `Can only set layoutPositioning = ABSOLUTE if the parent node has layoutMode !== NONE`. 일반 프레임 자식은 그냥 `x`/`y`를 쓴다.
- **`get_screenshot`은 프레임 크기가 아니라 그림자까지 포함한 크기를 준다.** 응답의 `original_width/height`가 프레임보다 큰 만큼이 좌우·상하 대칭 패딩이다(예: 374px 프레임 → 430px). 다른 배경에 합성할 때 **`(original - frame)/2`를 빼야** 좌표가 맞는다. `contentsOnly: true`로 떠도 패딩은 남는다.
- **`upload_assets`는 2단계다.** `count`·`nodeIds`로 업로드 URL을 받고 → 그 URL에 `curl -F "file=@...;filename=...;type=image/png"`로 POST하면 그 노드의 이미지 fill로 바로 꽂힌다(응답에 `placedOnNodeId`). 같은 노드에 다시 올리면 fill이 교체되므로 **배경 이미지를 갈아끼우는 데 쓸 수 있다** — 위에 얹은 레이어는 그대로 남는다.
- **`node.query()`의 속성값에 공백이 있으면 안 잡힌다.** `query('FRAME[name=3D slots]')`가 `null`을 돌려줘 `children.find(c => c.name === '...')`로 바꿔야 했다.

## 기록

### 2026-09-04 — 07a-1 위임 동의 시트 신설·07b 동의 제거 (보험찾개냥 SSH-457)

- 07a 프레임 복제 → 852 고정(Body `FILL`), 04a의 `LOCAL/Scrim`·`LOCAL/Source Sheet` 복제 → 시트 내용을 제목·묶음 행·동의 3행·`App / Button`으로 교체. 텍스트는 `setTextStyleIdAsync` + `setBoundVariableForPaint`, 간격은 `setBoundVariable('paddingTop', …)`. 레포 `figma-design` 스킬의 토큰 감사 스크립트로 `unbound: 0` 확인.
- 첫 시도가 `query('INSTANCE[name=icon/chevron-right]')`에서 죽어 전체 롤백 — 읽기 스크립트로 확인 후 재실행.

### 2026-09-16 — 약국맵 v2 섹션을 Figma에 새로 만들며 (약국맵)

- 맥락: [[프로젝트/개인/약국맵/README|약국맵]] 시안을 카카오맵 대응으로 보완. **기존 섹션을 고치지 않고 아래에 v2 섹션을 새로 만들어** 비교하고 버릴 수 있게 했다(`figma.createSection()` → `x/y` + `resizeWithoutConstraints`).
- 배운 것:
  - 위 「핵심 정리」의 6개 — 특히 **Pretendard 부재**와 **`dashPattern`**, **`get_screenshot`의 그림자 패딩**에서 시간을 썼다.
  - **판정판은 이미지로 굽지 말고 Figma 안에서 조립하는 게 낫다.** 배경만 `upload_assets`로 꽂고 패널·핀은 기존 노드를 `clone()`해 절대 좌표로 얹으면, 핀을 끌어 옮겨가며 볼 수 있고 배경 교체도 fill 재업로드 한 번이다. PIL로 합성한 첫 버전은 배경을 바꿀 때마다 전부 다시 구워야 했다.
  - **v1 좌표를 그대로 재사용하면 before/after가 정직해진다** — `get_metadata`가 주는 프레임 x/y/w/h를 그대로 써서 같은 자리에 v1 패널과 v2 핀을 각각 얹었다.
- 근거: Figma `MyCryptoDiary` 섹션 「약국맵 v2 — 카카오맵 대응 · 3D 아이콘」(`464:467`) / 실패 로그: `Pretendard does not exist`, `no such property 'strokeDashPattern'`, `Can only set layoutPositioning = ABSOLUTE …`
