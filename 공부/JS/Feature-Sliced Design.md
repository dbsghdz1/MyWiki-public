---
type: study
area: JS
audience: me
status: active
created: 2026-08-16
updated: 2026-08-20
aliases: [FSD, 피처 슬라이스 디자인]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# Feature-Sliced Design

프론트엔드 프로젝트를 **계층(layer) → 슬라이스(slice) → 세그먼트(segment)** 3단으로 나누고, "위 계층은 아래만 import" · "같은 계층 슬라이스끼리 import 금지" · "슬라이스 접근은 public API(`index.ts`)로만" 세 규칙으로 의존 방향을 강제하는 아키텍처 방법론. 규칙은 언어가 아니라 린터(Steiger)가 지킨다.

## 핵심 정리

- **계층**은 "무엇이 무엇을 알아도 되는가"의 순서다. `shared`(도메인 지식 0) → `entities`(명사) → `features`(유저 행동, 동사) → `widgets`(화면 덩어리) → `pages` → `app`(전역 설정). 위로 갈수록 아는 게 많다.
- **슬라이스 판별**: 명사면 entity, 유저가 하는 행동이면 feature, 화면 덩어리면 widget. **애매하면 widget에 통째로** 두고 재사용이 실제로 생길 때 내린다 — 조기 추상화가 FSD 최대 실패 요인.
- 슬라이스 이름은 컴포넌트명이 아니라 **역할**로 짓는다 (`TopNav` → `header`). "어디에 보이는가"(`components/home/`)로 묶는 것은 배치이지 성질이 아니다.
- **`index.ts`(public API)의 위치는 "밖에서 접근하는 단위"의 루트다.** widgets/features/entities는 슬라이스 루트(`widgets/header/index.ts`), `shared`와 `app`은 슬라이스가 없어 세그먼트 루트(`shared/ui/index.ts`). 그래서 import 경로도 `@/widgets/header` vs `@/shared/ui`로 깊이가 다르다.
- `export *` 금지 — index.ts는 "공개 목록이 눈에 보이는 것"이 존재 이유다.
- **Next.js App Router와 같이 쓸 때**: FSD의 `app`/`pages`를 `_app`/`_pages`로 개명하고, Next 라우팅은 루트 `app/`(한 줄 re-export만), FSD는 `src/`. `app/`은 Next에게 보여주는 얇은 껍데기, `src/_app`은 FSD의 전역 설정 계층 — 이름은 비슷하지만 주인이 다르다.

## 기록

### 2026-08-16 — MyCryptoDiary FSD 이사

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] 모의투자 전환 D1, `src/components/{ui,layout,home}` → FSD 계층으로 재배치 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]])
- 배운 것:
  - `index.ts`가 뭔지 처음엔 잡히지 않았다. "슬라이스의 **문**"으로 이해하니 풀림 — 밖에서는 문으로만 들어오고, 안의 파일 구조가 바뀌어도 문의 주소(`@/widgets/header`)는 그대로. 규모가 작을 땐 효과가 안 느껴지는 게 정상.
  - `shared/ui/index.ts` vs `widgets/header/index.ts` — 문의 위치가 계층마다 다른 이유는 shared/app만 슬라이스 없이 세그먼트가 바로 오기 때문.
  - `_app`과 Next `app/`을 헷갈렸다. `app/page.tsx`는 URL `/`에 대응하는 Next 규약 위치라 못 옮기고, 내용은 `export { HomePage as default } from '@/_pages/home'` 한 줄로 `_pages`에 위임한다.
  - `next/font/local`의 `src`는 **호출 파일 기준 상대경로**만 받는다. 폰트 파일을 `src/_app/fonts/`로 옮기고 `app/layout.tsx`에서 부르면 `../src/_app/fonts/...`가 되어 어색 → 폰트 로드 코드 자체를 `_app`으로 옮겨야 한다. (`import localFont from 'next/font/local'`은 로더 함수 import이고 `src:`가 파일 경로 — 둘을 섞어 고쳐서 500이 났다.)
  - **Steiger 0.6.0 버그**: `getLayers`는 `_` 접두사를 벗겨 `_app`을 app 계층으로 인식하지만 `isSliced`는 `basename` 그대로 `["shared","app"]`과 비교해서 `_app`을 슬라이스 있는 계층으로 취급한다. 그래서 `_app/fonts/`가 "세그먼트 없는 슬라이스"로 잡힌다. `_app` 한정으로 `fsd/no-segmentless-slices`를 껐다. `typo-in-layer-name`도 `_app`/`_pages`를 오타로 보므로 끔(공식 Next.js 가이드가 이 이름을 권장). `insignificant-slice`는 "한 곳에서만 쓰이는 슬라이스는 합치라"는 휴리스틱이라 페이지가 하나뿐인 초기엔 전부 걸림 → 재사용 계획이 있는 위젯이라 끄고 이유를 config 주석에 남겼다.
- 근거: MyCryptoDiary 커밋 `a053007`, `e7751b8`; `steiger.config.ts`; `node_modules/@feature-sliced/filesystem/dist/index.js`의 `getLayers`/`isSliced`

### 2026-08-20 — 규칙 2번("같은 계층 import 금지")이 실제로 작동한 순간

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1 마무리. `widgets/market-board`와 `widgets/watchlist`가 똑같은 로직(시세 받기 → 마켓 목록에서 이름 찾기 → 변환)을 쓰게 됨
- 배운 것:
  - 위젯끼리 import는 규칙 2번 위반. 규칙이 제시하는 해법 그대로 **아래 계층으로 내렸다** — `entities/coin/api/getCoins.ts`. "시세를 받아 Coin 배열을 만든다"는 코인 도메인의 일이라 entities가 맞고, entities는 두 위젯보다 아래라 양쪽이 쓸 수 있다.
  - **내리는 타이밍**이 핵심이다. 계획서의 "애매하면 widget에 통째로, 재사용이 실제로 생길 때 내린다"대로, 첫 위젯을 만들 땐 위젯 안에 두고 두 번째 소비자가 나타난 시점에 내렸다. 미리 내렸으면 조기 추상화였다.
  - 세그먼트 선택: 외부와 통신하니 `api`, 타입·순수 계산은 `model`, 상수는 `config`. `widgets/market-board/config/coins.ts`(어떤 코인을 보여줄지)가 config의 예.
- 근거: 커밋 `1fa0732`, `src/entities/coin/api/getCoins.ts`

## 참고 자료

- [FSD Overview](https://feature-sliced.design/docs/get-started/overview) — 계층·슬라이스·세그먼트 공식 정의 (2026-08-16 확인)
- [FSD Public API](https://feature-sliced.design/docs/reference/public-api) — index.ts를 "슬라이스와 사용자 사이의 계약"으로 설명 (2026-08-16 확인)
- [FSD with Next.js](https://feature-sliced.design/docs/guides/tech/with-nextjs) — `_app`/`_pages` 개명과 루트 `app/` 분리의 원본 (2026-08-16 확인)
