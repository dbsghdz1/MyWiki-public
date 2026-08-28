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
