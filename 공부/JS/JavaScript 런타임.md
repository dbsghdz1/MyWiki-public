---
type: study
area: 언어·프레임워크
status: active
created: 2026-08-18
updated: 2026-08-18
aliases: [Node.js, V8, 자바스크립트 엔진]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# JavaScript 런타임

JS 파일은 그냥 텍스트고, 이걸 읽어 실행해주는 **엔진**이 있어야 돈다. 같은 언어가 **브라우저**와 **Node** 두 곳에서 도는데 **할 수 있는 일이 서로 다르다** — 이 차이가 서버 컴포넌트/클라이언트 컴포넌트, API 키를 어디 두는가, CORS 같은 문제의 뿌리다.

## 핵심 정리

### Node = V8을 크롬에서 꺼내 온 것

크롬 안에는 **V8**이라는 JS 엔진이 들어 있고, 웹페이지의 JS는 V8이 실행한다. 2009년 이전엔 엔진이 브라우저 안에만 있어서 JS를 실행하려면 웹페이지를 열어야 했다. Node.js는 **V8만 떼어다가 브라우저 기능 대신 운영체제 기능을 붙인** 독립 프로그램이다.

```
크롬  = V8 + 브라우저 기능 (DOM, window, 화면)
Node = V8 + OS 기능 (파일 읽기/쓰기, 포트 열기, 프로세스)
```

직접 확인한 것 (크롬을 켜지 않고 터미널에서):

```bash
node -e "console.log(typeof window)"   # undefined  ← 브라우저 기능 없음
node -e "require('fs').readFileSync('package.json','utf8')"   # 파일 읽힘 ← 브라우저에선 불가
```

브라우저가 파일 읽기를 막는 건 성능 문제가 아니라 **보안 설계**다. 웹페이지가 하드디스크를 마음대로 읽으면 안 되니까. 반대로 Node는 OS 권한이 있어서 **포트를 열고 요청을 기다릴 수 있고**, 그래서 서버가 된다.

### 두 환경의 능력 차이

| | Node (서버) | 브라우저 |
|---|---|---|
| 파일·DB 접근 | ✅ | ❌ |
| 비밀키·API 키 | ✅ (유저가 못 봄) | ❌ (개발자도구에 다 보임) |
| 포트 열기 → 서버 되기 | ✅ | ❌ |
| `window`, DOM, 클릭 이벤트, `useState` | ❌ | ✅ |
| Next.js에서 | 서버 컴포넌트 (기본) | `'use client'` 붙인 것 |

Next.js에서 환경 변수 중 `NEXT_PUBLIC_` 접두사가 붙은 것만 클라이언트 번들에 들어가는 것도 같은 이유다. 실수로 서버 전용 코드를 클라이언트에서 import 하는 걸 막으려면 `server-only` 패키지를 쓴다.

## 기록

### 2026-08-18 — "브라우저 없이 JS를 실행한다"가 무슨 뜻인지

- 맥락: [[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]] D1 업비트 연동 중, `npm run dev`가 띄우는 게 무엇인지 물으며 ([[프로젝트/개인/MyCryptoDiary/모의투자 전환 D1 2026-08-16|작업 기록]])
- 배운 것: 위 핵심 정리 전체. "브라우저 = 크롬"이라는 인식에서 출발해, **엔진(V8)과 브라우저는 다른 것**이고 Node가 엔진만 떼어 온 것임을 알게 됨.
- 근거: 로컬에서 `node -e`로 `typeof window === 'undefined'`와 `fs.readFileSync` 성공을 직접 확인. `node -v` v24.18.0

## 참고 자료

- [Node.js — Introduction](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs) — Node가 V8 위에 만들어졌다는 공식 설명 (2026-08-18 확인)
- [MDN — JavaScript 개요](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction) — 언어와 실행 환경(브라우저 API)의 구분 (2026-08-18 확인)
