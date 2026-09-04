---
type: study
area: 도구
audience: ai
status: active
created: 2026-09-04
updated: 2026-09-04
projects:
  - "보험찾개냥"
---

# Figma MCP

`use_figma`(Plugin API JS 실행)·`get_design_context`·`get_metadata`를 쓰며 알게 된 것.

## 핵심 정리

- **`use_figma` 스크립트는 전체가 한 트랜잭션이다.** 중간에 예외가 나면 그 앞의 mutation까지 전부 롤백된다 — "앞부분은 적용됐겠지"로 이어 쓰면 안 되고, 에러 뒤에는 캔버스를 읽어 확인한다. 실측: 프레임 이동·복제·appendChild 20여 개를 한 뒤 마지막 줄 `query()`가 던졌더니 아무것도 남지 않았다.
- **`node.query()` 셀렉터 값에 `/`가 들어가면 파서 에러**(`Invalid selector: unexpected character '/'`). `icon/chevron-right`처럼 슬래시 이름이 흔한 아이콘 컴포넌트는 `page.findAllWithCriteria({ types: ['INSTANCE'] })`로 인스턴스를 찾아 `mainComponent.id`를 쓴다.
- `get_metadata`는 프레임이 있는 페이지를 **자동으로 못 찾는다** — 페이지 목록(`nodeId` 없이)이 첫 페이지만 돌려주는 경우가 있었다. `use_figma`로 `figma.root.children`을 읽고 `getNodeByIdAsync(id)`의 부모를 따라 올라가면 확실하다.
- 오버레이(시트·다이얼로그) 화면은 **기존 오버레이 화면의 `LOCAL/Scrim`·시트 프레임을 `clone()`해 새 프레임에 `appendChild`**하는 게 가장 싸다 — 토큰 바인딩(scrim 변수·radius·fill)이 따라온다. auto-layout 부모에 넣은 뒤 `layoutPositioning = 'ABSOLUTE'` + `constraints`로 앵커.

## 기록

### 2026-09-04 — 07a-1 위임 동의 시트 신설·07b 동의 제거 (보험찾개냥 SSH-457)

- 07a 프레임 복제 → 852 고정(Body `FILL`), 04a의 `LOCAL/Scrim`·`LOCAL/Source Sheet` 복제 → 시트 내용을 제목·묶음 행·동의 3행·`App / Button`으로 교체. 텍스트는 `setTextStyleIdAsync` + `setBoundVariableForPaint`, 간격은 `setBoundVariable('paddingTop', …)`. 레포 `figma-design` 스킬의 토큰 감사 스크립트로 `unbound: 0` 확인.
- 첫 시도가 `query('INSTANCE[name=icon/chevron-right]')`에서 죽어 전체 롤백 — 읽기 스크립트로 확인 후 재실행.
