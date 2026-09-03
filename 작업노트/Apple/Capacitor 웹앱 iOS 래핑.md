---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-27
updated: 2026-08-27
projects:
  - "[[프로젝트/개인/한능검/README|한능검]]"
---

# Capacitor 웹앱 iOS 래핑

React/Vite로 만든 웹 코어를 Capacitor로 감싸 iOS 앱으로 내는 법. 코어 한 벌로 웹과 앱을 같이 낼 때 쓴다.

## 핵심 정리

| 항목 | 값 |
|---|---|
| 초기화 | `npx cap init "<앱이름>" "<번들ID>" --web-dir dist` |
| 플랫폼 추가 | `npx cap add ios` → `ios/App/App.xcodeproj` 생성 |
| 웹 변경 반영 | **`npm run build` 후 `npx cap sync ios`** (빌드만 하면 앱에 안 들어간다) |
| 정적 자산 | `dist/` 전체가 `ios/App/App/public/` 으로 복사돼 **번들에 포함**된다 |

## 기록

### 2026-09-04 — 위젯·iCloud KV를 붙이면 자동 서명이 App Group 등록에서 막힌다 (포털은 `aside repl`로)
- 맥락: [[프로젝트/개인/한능검/README|한능검]] 2.1.0 — 홈 위젯(`HangeomWidget`, 앱 그룹 UserDefaults 스냅샷)과 iCloud 키-값 백업(`SceneDelegate` `CloudSink` → `NSUbiquitousKeyValueStore`)을 처음 실은 릴리즈. 웹→네이티브는 `WKScriptMessageHandler`(`hangeomWidget`·`hangeomCloud`) 둘로 통했고 Capacitor 플러그인은 안 썼다(Preferences는 앱 그룹 미지원).
- 배운 것:
  - `fastlane ios release`(자동 서명 + ASC API 키)가 `Provisioning profile "iOS Team Provisioning Profile: *" doesn't support the group.com.hong.hangeom App Group` / `doesn't include the com.apple.developer.ubiquity-kvstore-identifier` / 위젯 타깃 `Authentication failed: bearer token`으로 죽었다. bundle ID 두 개(앱·위젯)에 `APP_GROUPS` capability는 켜져 있었지만 **그룹 자체가 포털에 없었고**, iCloud capability도 없었다.
  - iCloud는 ASC API로 켜진다(`POST /v1/bundleIdCapabilities` `ICLOUD`/`ICLOUD_VERSION=XCODE_6` → 201). App Group은 API가 없어 포털 — **Aside의 LLM 크레딧이 소진(OpenAI 402)돼도 `aside repl`(Playwright)로 등록·연결이 된다.** 절차와 URL은 appstore-release 스킬 플레이북에 적었다. 연결 후 재실행하면 자동 서명이 새 프로파일을 만들고 통과한다.
  - `maestro/record.sh`의 derivedData가 `/tmp`라 SPM 체크아웃이 반쯤 사라져 `Could not resolve package dependencies`(exit 74) — `SourcePackages` 삭제로 복구.
  - 인앱 버전 문자열은 `src/Settings.tsx`의 `APP_VERSION` 상수다 — pbxproj·Fastfile(`app_version` 3곳, `resubmit`의 `HANGEOM_BUILD` 기본값)과 함께 올려야 한다. 이번에 네 군데를 손으로 맞췄다.
- 근거: 릴리즈 로그 `release-2.1.0.log`(01:55 서명 실패) → 포털 등록 → `release-2.1.0-b.log` 02:14 제출. 위키 [[프로젝트/개인/한능검/App Store 심사 이력|심사 이력]] 2.1.0 절.

### 2026-09-02 — capacitor:// 웹뷰에서 유튜브 임베드는 오류 153, https 래퍼 한 장으로 풀린다

- 맥락: 한능검 강의 임베드. 썸네일·플레이어 UI는 떠서 멀쩡해 보였는데 **재생을 누르면 오류 153("동영상 플레이어 구성 오류")**. 시뮬레이터·실기기 동일.
- 원인: 유튜브는 재생 시점에 임베드한 쪽의 HTTP Referer(https 원점)를 요구한다. capacitor:// 커스텀 스킴 웹뷰는 Referer가 없다. iOS는 `iosScheme`을 http/https로 바꿀 수 없어 앱 쪽에서 해결 불가.
- 해법: **GitHub Pages 정적 래퍼** — `player.html?v=<id>`가 유튜브를 임베드하고, 앱은 그 https 페이지를 iframe으로 문다. 유튜브 입장에선 정상 웹사이트의 임베드. 재생 실측 통과. 중첩 iframe이므로 allowfullscreen을 양쪽에 줘야 전체화면이 된다.
- 검증 교훈: `playableInEmbed` 같은 서버 판정과 썸네일 렌더는 재생 보장이 아니다 — **재생 버튼까지 누르는 것**이 임베드 검증의 끝이다.
- 근거: `dbsghdz1/hangeom-docs`, `web/src/Lecture.tsx`(PLAYER_URL 분기), 재현 스크린샷 오류153 → 재생 중 프레임.

### 2026-09-02 — disabled 버튼은 터치 사각지대다 (스와이프가 안 먹는 영역의 정체)

- 맥락: 한능검, 홍 QA "앞뒤로 넘어가는 제스처가 안 되는 영역이 있다, 하단으로 갈수록". 좌우 스와이프 핸들러는 컨테이너(`.wrap.q`)의 touchstart/touchend에 걸려 있었다.
- 원인: **WebKit은 `disabled` 폼 요소 위에서 터치/마우스 이벤트를 아예 발생시키지 않는다**(버블링될 이벤트 자체가 없음). 채점 후 선지 `<button disabled>`가 화면의 큰 영역을 차지해 그 위에서 시작한 스와이프가 통째로 죽었다. Maestro 스토어 플로우의 "채점 직후 중앙 스와이프 무시"도 같은 원인(중앙이 disabled 선지 위) — 당시엔 좌표 지정으로 우회했었다.
- 해법: `disabled` 대신 `aria-disabled` + 클릭 가드(`onClick={() => !answered && onPick(n)}`). CSS는 `:disabled` 셀렉터를 `[aria-disabled="true"]`로. 터치가 정상 전파되고 접근성 의미도 유지된다.
- 참고: YouTube 임베드 iframe 위도 여전히 사각지대다(iframe 문서로 이벤트가 들어감) — 이건 구조상 수용.
- 근거: `web/src/Question.tsx`, `web/src/index.css`, 재현 `shots/20260902-*-store` step-013.

### 2026-09-01 — `contentInset: 'always'` + CSS `env(safe-area-inset-top)`는 상단 여백을 두 번 준다

- 맥락: 한능검 2.0, 홍 QA "문제 화면 UI가 이상하다" — 상단 유리 캡슐 위 빈 공간이 화면마다 다르게(91~134pt) 벌어져 있었다.
- 원인: `capacitor.config.ts` `ios.contentInset: 'always'`는 WKWebView 스크롤뷰의 `contentInsetAdjustmentBehavior = .always`라 콘텐츠 전체를 상태 표시줄만큼 내리는데, CSS `.top { padding-top: calc(env(safe-area-inset-top) + 8px) }`도 같은 값을 더한다. `position: sticky; top: 0`은 스크롤하면 스크롤뷰 인셋 기준으로 붙어 스크롤 전후 위치가 달라 보였다.
- 해법: `contentInset: 'never'`. `StatusBar.setOverlaysWebView({overlay:true})`와 CSS env()만으로 처리한다. env() 값은 인셋 동작과 무관하게 safeAreaInsets를 그대로 준다 — 적용 후 캡슐이 안전영역+8px(72pt)에 고정됐다.
- 근거: `web/capacitor.config.ts`, 스크린샷 `web/maestro/shots/20260901-2133-…/…/02-solve.png`(전) vs `20260901-2136-v2.0-question-redesign/02-solve.png`(후).

### 2026-08-27 — 상태 표시줄은 `overlay:true` + CSS safe-area 가 정답이다

- 맥락: [[프로젝트/개인/한능검/README|한능검]] 앱을 Capacitor로 래핑해 시뮬레이터에서 처음 띄웠더니 **헤더가 상태 표시줄과 겹쳐** 앱 제목과 시계가 뒤엉켰다.
- 배운 것:
  1. **`StatusBar.setOverlaysWebView({ overlay: false })`는 해결이 아니다.** 겹침은 사라지지만 웹뷰가 상태 표시줄 아래에서 시작하고 **그 자리를 아무도 칠하지 않아 검은 띠**가 남는다. 배경색을 따로 지정해야 하는데, 다크 모드까지 따라가려면 관리 지점이 늘어난다.
  2. **정답은 `overlay: true` + CSS `env(safe-area-inset-top)`이다.** 웹뷰가 화면 전체를 덮게 두고(iOS 표준) 겹침은 CSS가 피한다. 스크롤할 때 콘텐츠가 상태 표시줄 아래로 흐르는 자연스러운 동작이 덤으로 온다.
  3. **`env()`가 동작하려면 `<meta name="viewport" ... viewport-fit=cover>`가 있어야 한다.** 이게 빠지면 safe-area 값이 전부 0이라, CSS는 맞게 썼는데 왜 안 되는지 한참 헤맨다.
  4. 좌우도 잊지 말 것 — `padding-left: max(16px, env(safe-area-inset-left))`. 가로 모드와 노치 기기에서 콘텐츠가 잘린다.
- 근거: `(로컬 경로)`, `src/index.css`. 정리는 [[프로젝트/개인/한능검/iOS 앱 2026-08-27|iOS 앱]].

### 2026-08-27 — 정적 데이터를 번들에 넣으면 App Store 4.2 방어가 된다

- 맥락: Capacitor 래핑은 *"웹사이트를 감싼 앱"*으로 **App Store 4.2 Minimum Functionality** 거절 대상이 될 수 있다. 실제로 착수 전부터 리스크로 적어뒀던 항목이다.
- 배운 것:
  1. **`dist/` 안의 것은 전부 앱 번들로 들어간다.** 문제 데이터 1.6MB(24개 JSON)를 `public/data/`에 두었더니 그대로 `ios/App/App/public/data/`에 복사됐고, **네트워크 요청 없이 전 기능이 동작**한다.
  2. 그래서 *"서버를 두지 않는다"*는 설계가 그대로 **심사 방어 논리**가 된다 — 웹사이트 래퍼가 아니라 오프라인 앱이다. 로컬 우선 저장(계정·서버 없음)과 결이 같아서 설명도 일관된다.
  3. 확인 방법: `xcrun simctl get_app_container <udid> <bundleId> app` 으로 설치된 번들 경로를 얻어 `public/` 내용을 직접 세면 된다. 빌드 로그만 봐서는 자산이 들어갔는지 알 수 없다.
- 한계: 번들이 커지면 앱 용량이 그대로 늘어난다. 수십 MB 규모면 온디맨드 리소스나 최초 실행 시 다운로드를 고려해야 한다.

## 참고 자료

- [Capacitor — iOS Documentation](https://capacitorjs.com/docs/ios) — 플랫폼 추가·동기화·설정 (2026-08-27 확인)
- [Capacitor — Status Bar 플러그인](https://capacitorjs.com/docs/apis/status-bar) — `setOverlaysWebView`·`setStyle` (2026-08-27 확인)
- [App Store Review Guidelines — 4.2 Minimum Functionality](https://developer.apple.com/app-store/review/guidelines/) — *"Your app should include features, content, and UI that elevate it beyond a **repackaged website**"*. 4.2.2는 *"web clippings, content aggregators, or a collection of links"*를 금지한다 (2026-08-27 확인)
