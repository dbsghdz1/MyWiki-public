---
type: study
area: 언어·프레임워크
status: active
created: 2026-08-16
updated: 2026-08-16
aliases: [ES Modules, export/import, 모듈 재수출]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# JavaScript 모듈 시스템

파일 하나가 모듈이고, `export`가 붙은 것만 밖에서 보인다. 가시성은 **export 있음/없음** 두 상태뿐이라 Swift의 `internal`/`private` 같은 중간 단계가 없다 — 모듈 경계 위의 "패키지 경계"는 언어가 아니라 도구(린터)와 관례가 지킨다.

## 핵심 정리

- **default export vs named export**: `export default function X`는 이름 없이 나간다(가져오는 쪽이 아무 이름이나 붙임). `export const x`, `export type T`는 이름 달고 나간다(`{ x }`로 그 이름으로만 받음).
- **재수출(re-export)**: `export { default as X } from './X'` — 옆 파일의 default를 받아 `X`라는 이름으로 즉시 내보낸다. import 문 없이 한 줄로 통과시키는 것이라 index 파일에는 보통 import가 없다. named는 `export { x } from './x'`, 타입은 `export type { T } from './t'`(런타임에 사라짐을 명시).
- `as` 뒤 이름은 자유라서 **컴포넌트 이름과 다르게 붙이면 함정**이 된다(`InvestorTrendCard`라는 이름으로 `TrendingIndustryStrip`을 내보내는 식). 관례는 원래 이름 그대로.
- 폴더 경로로 import 하면(`@/shared/ui`) 그 안의 `index.ts`를 자동으로 찾는다. 그래서 index.ts가 "폴더 = 모듈"처럼 느껴진다.
- **접근제어자가 없다.** export = 그 파일 밖 누구나(Swift `public`), export 없음 = 그 파일 안에서만(Swift `fileprivate`). "index.ts에는 보이고 다른 파일엔 안 보이게"는 언어 차원에 없다. index.ts도 그냥 옆에 있는 파일이라, 원본이 export 하지 않은 건 index.ts에게도 안 보인다. (`class`의 `private`은 TS가 검사하지만 그건 클래스 멤버 얘기.)

## 기록

### 2026-08-16 — FSD public API를 만들며

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] FSD 이사에서 슬라이스마다 `index.ts`를 쓰다가 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]])
- 배운 것:
  - `export { GlassCard } from ...`은 named export를 찾는 문법이라 `export default function GlassCard`를 잡지 못한다 → `export { default as GlassCard } from './GlassCard'`.
  - 상대경로 `./`는 "내가 있는 폴더". `shared/ui/index.ts`에서 `./ui/...`나 `./index`를 쓰면 자기 폴더 아래/자기 자신을 가리키는 순환이 된다.
  - iOS 배경에서 "GlassCard를 private으로 하면 index.ts 때문에 외부 접근이 되나?"라고 물었는데, 답은 "JS엔 그 개념이 없다". FSD의 "문으로만 들어와라"는 Steiger가 검사하는 규칙이지 컴파일러가 막는 게 아니다.
  - Next 프로젝트가 컴포넌트를 `export default`로 통일한 이유는 강한 기술 근거가 아니라 관례 통일 — `page.tsx`/`layout.tsx`가 default export를 강제해서 라우트 파일과 컴포넌트를 한 방식으로 맞춘 것. named로 통일하는 반대편 주장(리팩터링·자동완성 유리)도 유효하며 한 프로젝트 안에서 한 가지로 통일하는 게 요점.
- 근거: MyCryptoDiary 커밋 `a053007`의 `src/**/index.ts` 8개; `tsc` 에러 `TS2614: Module '.../ui/InvestorTrendCard' has no exported member 'InvestorTrendCard'`(깊은 경로로 named import를 시도했을 때)

## 참고 자료

- [MDN — export](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) — `export { default as name1 } from "bar.js"` 재수출 문법 원본 (2026-08-16 확인)
- [MDN — JavaScript modules 가이드](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules) — default/named export, 재수출, 순환 import까지 (2026-08-16 확인)
