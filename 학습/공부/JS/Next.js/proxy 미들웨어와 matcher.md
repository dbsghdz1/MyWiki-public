---
type: study
area: JS
audience: me
status: active
created: 2026-08-28
updated: 2026-09-16
aliases: [proxy.ts, middleware, matcher, 미들웨어, Route Group, 인증 경계]
projects:
  - "[[프로젝트/개인/MyCryptoDiary/README|MyCryptoDiary]]"
---

# Next.js proxy 미들웨어와 matcher

**proxy(구 middleware)는 페이지마다가 아니라 요청마다 돈다.** matcher는 뺄 것을 나열하고 나머지를 잡는 부정 look-ahead로 쓰고, 빈 배열은 「아무 데서도 안 돈다」다 — 빌드 출력의 `ƒ Proxy (Middleware)` 줄이 실행 여부의 증거다. (2026-09-16 [[학습/공부/JS/Next.js/Next.js 서버와 캐싱|Next.js 서버와 캐싱]]에서 분리)

## 핵심 정리


Next 16에서 `middleware.ts` → **`proxy.ts`**로 이름이 바뀌었다. 프로젝트 루트에 두고 `config.matcher`로 어느 경로에서 돌지 정한다.

```
matcher 생략    → 모든 요청에서 실행 (정적 파일 포함)
matcher: ['/a'] → /a 에서만 실행
matcher: []     → 아무 데서도 실행 안 됨 (파일은 있는데 하는 일이 없다)
```

**핵심은 "페이지마다"가 아니라 "요청마다" 돈다는 것이다.** 화면 하나를 여는 데 HTML·CSS·JS 번들·폰트·아이콘까지 수십 개의 요청이 나간다. 정적 파일을 안 빼면 홈 한 번 여는 데 미들웨어가 수십 번 실행되고, 인증 미들웨어라면 매번 쿠키를 까고 JWT 서명을 검증한다. 폰트 파일에는 "로그인한 사람용 폰트"가 없으므로 그 판정은 전부 버려진다.

**그런데 진짜 문제는 성능이 아니라 파손이다.** 인증 미들웨어는 보통 "로그인 안 했으면 `/sign-in`으로 리다이렉트"인데, 이게 CSS 요청에 걸리면 브라우저는 CSS를 기대하고 **로그인 페이지 HTML을 받는다.** 스타일이 통째로 안 먹어 화면이 무너지고, 심하면 로그인 페이지가 자기 CSS를 못 불러와 로그인조차 못 한다. Next 문서도 이 순서로 경고한다 — *"auth logic or redirects can unintentionally block CSS, JS, or images from loading."*

그래서 matcher는 **뺄 것을 나열하고 나머지를 다 잡는** 부정 look-ahead `(?!...)`로 쓰고, 반대로 반드시 포함할 것(API 등)은 따로 한 줄 더 둔다.

```js
matcher: [
  // _next 내부와 정적 확장자를 제외한 나머지 전부
  '/((?!_next|[^?]*\\.(?:css|js(?!on)|png|svg|woff2?|ico)).*)',
  '/(api|trpc)(.*)',   // API는 반대로 반드시 포함
]
```

### Route Group layout — 인증 경계를 URL 없이 묶는다

`(protected)`처럼 괄호로 감싼 폴더는 파일을 묶지만 URL에는 나타나지 않는다. `app/(protected)/market/page.tsx`는 `/market` 그대로이면서 공통 `layout.tsx` 하나로 보호된다. 요청 전처리(`proxy.ts`)는 세션 연결만 맡기고, **자원 보호는 layout의 `auth()`**에 두는 분담이 경로 문자열 목록(`createRouteMatcher`)보다 라우트 이동에 덜 어긋난다.

## 기록

### 2026-08-28 (2) — proxy.ts matcher (D3 블록 2)

맥락: MyCryptoDiary D3에서 Clerk를 붙이며 `proxy.ts`를 처음 만들었다. 처음엔 `matcher: []`로 두었는데 **빌드도 통과하고 화면도 멀쩡해서 "잘 된다"고 판단했다.** 실제로는 빈 배열이 "돌릴 경로가 하나도 없다"는 뜻이라 미들웨어가 **한 번도 실행되지 않고 있었다.**

확인 방법을 찾은 것이 수확이다 — `npm run build` 출력 맨 아래 **`ƒ Proxy (Middleware)`** 줄이 뜨는지 본다. matcher가 비어 있으면 이 줄이 없고, 채우면 나타난다. 파일 존재·빌드 통과·화면 정상은 전부 "실행된다"의 증거가 아니었다.

> 같은 함정이 이 프로젝트에서 두 번째다. 앞서 `db:seed`로 드라이버 교체를 검증하려 했을 때도 seed가 바뀐 코드를 아예 지나가지 않아 초록불이 아무것도 증명하지 못했다. **"통과했다"가 아니라 "무엇이 통과했다"를 봐야 한다.**

그리고 왜 정적 파일을 빼야 하는지 — 성능이 아니라 **파손**이 이유라는 것(위 절).

### 2026-08-29 — Route Group layout으로 인증 경계 묶기 (D3 블록 6)

맥락: [[프로젝트/개인/MyCryptoDiary/README|CoinPilot]] D3에서 비로그인 사용자는 홈만 보고 나머지 UI는 로그인하도록 보호했다([[프로젝트/개인/MyCryptoDiary/Clerk 인증 D3 2026-08-29|작업 기록]]).

- `(protected)`처럼 괄호로 감싼 폴더는 파일을 묶지만 URL에는 나타나지 않는다. 그래서 `app/(protected)/market/page.tsx`를 실제 `/market` 주소 그대로 유지하면서 공통 layout 하나로 보호할 수 있다.
- 설치된 Clerk v7 타입에서 `createRouteMatcher`는 deprecated였다. 경로 문자열 목록은 실제 라우트 이동과 어긋날 수 있으므로, 요청 전처리용 `proxy.ts`는 Clerk 세션 연결만 맡기고 자원 보호는 Route Group layout의 `auth()`에 뒀다.
- TypeScript는 `layout.tsx`가 default export여야 한다는 Next 파일 규약을 모른다. named export도 `tsc`를 통과했으므로 `next build`와 로그아웃 직접 접근으로 검증했다.
- 근거: MyCryptoDiary 커밋 `6f47e54`, PR #7. 로그아웃 `/market`·`/settings`는 로그인으로 이동하고 `/`는 공개, 로그인 후 보호 화면 접근을 확인했다.

## 참고 자료

- [Next.js — proxy.js](https://nextjs.org/docs/app/api-reference/file-conventions/proxy) — *"Without a `matcher`, Proxy runs on every request, including static files … otherwise auth logic or redirects can unintentionally block CSS, JS, or images from loading."* middleware → proxy 개명 이유(v16), 부정 매칭 예제, `_next/data`는 제외해도 돈다는 주의 (2026-09-16 확인)
- [Next.js — Route Groups](https://nextjs.org/docs/app/api-reference/file-conventions/route-groups) — `(folderName)`은 URL에 포함되지 않고, 특정 세그먼트만 layout을 공유시키는 용도. 다른 그룹이 같은 URL로 풀리면 에러 (2026-09-16 확인)
