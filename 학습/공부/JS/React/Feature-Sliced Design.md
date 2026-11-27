---
type: study
area: JS
audience: me
status: active
created: 2026-08-16
updated: 2026-11-28
aliases: [FSD, 피처 슬라이스 디자인]
projects:
  - "MyCryptoDiary"
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

### 루트 `app/`과 `src/`의 경계

둘은 "프론트엔드/백엔드"로 나뉜 것이 아니다. **루트 `app/`은 Next.js가 파일 위치와 이름으로 읽는 프레임워크 계약**, **`src/`는 우리가 역할과 의존 방향으로 나누는 제품 구현**이다.

| 질문 | 두는 곳 | 예시 |
|---|---|---|
| 어느 URL에서 실행되는가? | 루트 `app/` | `page.tsx`, `layout.tsx`, `route.ts` |
| 실제 화면은 어떻게 조립되는가? | `src/_pages`, `src/widgets` | `HomePage`, `TopNav` |
| 사용자가 무엇을 하는가? | `src/features` | D4의 매수 |
| 어떤 업무 명사를 다루는가? | `src/entities` | coin, account, order |
| 어느 도메인에도 속하지 않는 기반인가? | `src/shared` | DB 연결, 외부 API client, 공용 UI |

`app/`의 예약 파일은 Next가 직접 호출하므로 이름을 바꿀 수 없다. `app/market/page.tsx`는 `/market` 진입점이고, `app/(protected)/layout.tsx`는 하위 라우트를 감싼다. 반대로 `src/widgets/header/ui/TopNav.tsx` 같은 이름은 프로젝트 규칙이며, import되지 않으면 Next는 존재조차 모른다.

의존 흐름은 **`app/ → src/` 한 방향**이다. 라우트 진입점이 `src/_pages`를 부르고, `_pages → widgets → features → entities → shared`로 내려간다. 제품 구현이 특정 URL 폴더를 import하기 시작하면 재사용 가능한 매수·계좌 로직이 라우팅 구조에 묶인다.

### 같은 `api`라는 이름이 두 번 나오는 이유

| 위치 | `api`의 뜻 | URL이 생기는가? |
|---|---|---|
| `app/api/.../route.ts` | 외부 요청을 받는 HTTP 출입구 | 예: `/api/upbit/ticker` |
| `src/**/api/` | DB·Clerk·업비트처럼 바깥 데이터와 통신하는 FSD 세그먼트 | 아니오. import해서 부르는 함수 |

엄밀히 말하면 루트 `app/api`에서 특별한 이름은 `api` 폴더가 아니라 **`route.ts`**다. `api`는 URL을 `/api/...`로 만들기 위한 관례적인 경로 조각이다. 반면 `src/entities/account/api/getOrCreateAccount.ts`는 Clerk·Neon과 통신하지만 HTTP 주소를 만들지 않고, 서버 컴포넌트가 함수로 직접 호출한다.

## 기록

### 2026-11-28 — `shared/lib` 안에 폴더를 만들면 `index.ts`가 필요하다 (11-27 질문의 답 일부)

- 맥락: 에이전트가 포맷 함수를 `src/shared/lib/format/formatMoney.ts`에 넣고 `@/shared/lib/format/formatMoney`로 가져왔다. 커밋 전 검사 출력의 마지막 줄만 보고 통과로 잘못 읽었고, 리뷰 에이전트가 위반 2건을 찾았다.
- Steiger가 실제로 낸 오류:
  - `fsd/public-api`: 「This top-level folder in shared/lib is missing a public API.」
  - `fsd/no-public-api-sidestep`: 「Forbidden sidestep of public API when importing from "@/shared/lib/format/formatMoney".」
- 그래서 규칙은 이렇다: `shared/lib` 바로 아래의 **파일 하나**(`decimal.ts`, `toWonAmount.ts`)는 그 자체가 공개 지점이라 `@/shared/lib/decimal`로 가져와도 된다. `shared/lib` 아래에 **폴더**(`format/`)를 만들면 그 폴더가 한 묶음이 되어 `index.ts`가 필요하고, 밖에서는 `@/shared/lib/format`으로만 가져온다.
- 11-27 기록의 「도구마다 각자 문」은 맞고, 「도구가 폴더면 그 폴더의 `index.ts`가 문」이 덧붙는다.
- 교훈(에이전트): 검사 결과는 마지막 줄이 아니라 전체 출력을 본다.
- 근거: `src/shared/lib/format/index.ts`, 커밋 `e8bcca5`

### 2026-11-27 — 왜 `shared/lib`는 `index.ts`를 거치지 않나 (다시 물어볼 것)

- 맥락: [MyCryptoDiary](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/MyCryptoDiary/README.md) 내 자산 화면. 매도의 `calculateProceeds`를 `src/shared/lib/toWonAmount.ts`로 내리면서 홍이 물었다: 슬라이스는 `index.ts`로만 여는데 이건 왜 `@/shared/lib/toWonAmount`로 바로 가져오나. 홍이 「나중에 다시 물어볼 것」으로 기록을 요청했다.
- 답(짧게 들은 것):
  - `index.ts`는 **슬라이스의 안과 밖을 가르는 문**이다. `entities/holding`처럼 한 덩어리 안의 `api/`, `model/`, `ui/`는 내부 사정이라 밖에서 직접 건드리지 않는다. 안을 옮겨도 문만 같으면 바깥이 안 깨진다.
  - `shared`에는 슬라이스가 없다. `decimal.ts`와 `toWonAmount.ts`는 한 덩어리의 부품이 아니라 서로 무관한 독립 도구라 지킬 내부 사정이 없다. 그래서 **도구 하나가 곧 공개 지점**이다.
  - `shared/lib/index.ts` 하나로 전부 모으지 않는 이유: 하나만 필요해도 모은 파일이 가리키는 모듈을 다 읽게 되고, 서버 전용 모듈이 클라이언트로 딸려 갈 위험이 생긴다.
  - 정리: 슬라이스는 문 하나, `shared`는 도구마다 각자 문.
- 아직 남은 것: CLAUDE.md의 「`shared`는 세그먼트가 문 (`@/shared/ui`)」 문구와 실제 사용(`@/shared/lib/decimal`, 파일 단위)이 어긋나 보인다. 이번 피처 code-review에서 짚는다. 「공식 문서가 도구별 공개 지점을 권한다」는 설명은 에이전트가 기억으로 한 말이라, 다시 볼 때 아래 Public API 문서에서 확인한다.
- 근거: `src/shared/lib/toWonAmount.ts`, `src/features/market-sell/api/placeSellOrder.ts`의 `@/shared/lib/decimal` import

### 2026-08-29 — `app/`과 `src/`, 두 `api`의 경계

- 맥락: [CoinPilot](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/MyCryptoDiary/README.md) D4를 시작하기 전에 "왜 `src`와 `app`을 나눴는가, `app/api`와 `src/**/api`는 왜 둘 다 api인가"를 질문하며 현재 구조를 다시 읽었다.
- 배운 것:
  - `app/`은 URL과 Next 예약 파일을 연결하는 얇은 어댑터이고, `src/`는 화면·행동·업무 명사·기반 코드의 실제 소유권을 나누는 곳이다.
  - HTTP로 공개할 필요가 없는 서버 로직은 `app/api`를 만들지 않는다. `getOrCreateAccount()`는 홈 서버 컴포넌트가 직접 호출하므로 `src/entities/account/api`만 있으면 된다.
  - 같은 업비트 조회도 서버 컴포넌트는 `src` 함수를 직접 호출하고, 브라우저는 그 파일을 실행할 수 없어 `app/api/.../route.ts`를 HTTP로 호출한다.
- 근거: `app/page.tsx`, `app/api/upbit/ticker/route.ts`, `src/_pages/home/ui/HomePage.tsx`, `src/entities/account/api/getOrCreateAccount.ts`, `src/entities/coin/api/getCoins.ts`; D3 PR #7.

### 2026-08-16 — MyCryptoDiary FSD 이사

- 맥락: [MyCryptoDiary](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/MyCryptoDiary/README.md) 모의투자 전환 D1, `src/components/{ui,layout,home}` → FSD 계층으로 재배치 ([작업 기록](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/MyCryptoDiary/%EB%AA%A8%EC%9D%98%ED%88%AC%EC%9E%90%20%EC%A0%84%ED%99%98%20D1%202026-08-16.md))
- 배운 것:
  - `index.ts`가 뭔지 처음엔 잡히지 않았다. "슬라이스의 **문**"으로 이해하니 풀림 — 밖에서는 문으로만 들어오고, 안의 파일 구조가 바뀌어도 문의 주소(`@/widgets/header`)는 그대로. 규모가 작을 땐 효과가 안 느껴지는 게 정상.
  - `shared/ui/index.ts` vs `widgets/header/index.ts` — 문의 위치가 계층마다 다른 이유는 shared/app만 슬라이스 없이 세그먼트가 바로 오기 때문.
  - `_app`과 Next `app/`을 헷갈렸다. `app/page.tsx`는 URL `/`에 대응하는 Next 규약 위치라 못 옮기고, 내용은 `export { HomePage as default } from '@/_pages/home'` 한 줄로 `_pages`에 위임한다.
  - `next/font/local`의 `src`는 **호출 파일 기준 상대경로**만 받는다. 폰트 파일을 `src/_app/fonts/`로 옮기고 `app/layout.tsx`에서 부르면 `../src/_app/fonts/...`가 되어 어색 → 폰트 로드 코드 자체를 `_app`으로 옮겨야 한다. (`import localFont from 'next/font/local'`은 로더 함수 import이고 `src:`가 파일 경로 — 둘을 섞어 고쳐서 500이 났다.)
  - **Steiger 0.6.0 버그**: `getLayers`는 `_` 접두사를 벗겨 `_app`을 app 계층으로 인식하지만 `isSliced`는 `basename` 그대로 `["shared","app"]`과 비교해서 `_app`을 슬라이스 있는 계층으로 취급한다. 그래서 `_app/fonts/`가 "세그먼트 없는 슬라이스"로 잡힌다. `_app` 한정으로 `fsd/no-segmentless-slices`를 껐다. `typo-in-layer-name`도 `_app`/`_pages`를 오타로 보므로 끔(공식 Next.js 가이드가 이 이름을 권장). `insignificant-slice`는 "한 곳에서만 쓰이는 슬라이스는 합치라"는 휴리스틱이라 페이지가 하나뿐인 초기엔 전부 걸림 → 재사용 계획이 있는 위젯이라 끄고 이유를 config 주석에 남겼다.
- 근거: MyCryptoDiary 커밋 `a053007`, `e7751b8`; `steiger.config.ts`; `node_modules/@feature-sliced/filesystem/dist/index.js`의 `getLayers`/`isSliced`

### 2026-08-20 — 규칙 2번("같은 계층 import 금지")이 실제로 작동한 순간

- 맥락: [MyCryptoDiary](../../../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/MyCryptoDiary/README.md) D1 마무리. `widgets/market-board`와 `widgets/watchlist`가 똑같은 로직(시세 받기 → 마켓 목록에서 이름 찾기 → 변환)을 쓰게 됨
- 배운 것:
  - 위젯끼리 import는 규칙 2번 위반. 규칙이 제시하는 해법 그대로 **아래 계층으로 내렸다** — `entities/coin/api/getCoins.ts`. "시세를 받아 Coin 배열을 만든다"는 코인 도메인의 일이라 entities가 맞고, entities는 두 위젯보다 아래라 양쪽이 쓸 수 있다.
  - **내리는 타이밍**이 핵심이다. 계획서의 "애매하면 widget에 통째로, 재사용이 실제로 생길 때 내린다"대로, 첫 위젯을 만들 땐 위젯 안에 두고 두 번째 소비자가 나타난 시점에 내렸다. 미리 내렸으면 조기 추상화였다.
  - 세그먼트 선택: 외부와 통신하니 `api`, 타입·순수 계산은 `model`, 상수는 `config`. `widgets/market-board/config/coins.ts`(어떤 코인을 보여줄지)가 config의 예.
- 근거: 커밋 `1fa0732`, `src/entities/coin/api/getCoins.ts`

## 참고 자료

- [FSD Overview](https://feature-sliced.design/docs/get-started/overview) — 계층·슬라이스·세그먼트 공식 정의 (2026-08-16 확인)
- [FSD Public API](https://feature-sliced.design/docs/reference/public-api) — index.ts를 "슬라이스와 사용자 사이의 계약"으로 설명 (2026-08-16 확인)
- [FSD with Next.js](https://feature-sliced.design/docs/guides/tech/with-nextjs) — `_app`/`_pages` 개명과 루트 `app/` 분리의 원본 (2026-08-16 확인)
