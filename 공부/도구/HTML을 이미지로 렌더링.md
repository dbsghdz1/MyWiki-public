---
type: study
area: 도구
status: active
created: 2026-08-22
updated: 2026-08-22
projects:
  - "소프트웨어 마에스트로"
---

# HTML을 이미지로 렌더링

디자인 도구 없이 정확한 픽셀 크기의 홍보 이미지를 뽑는 방법 — macOS에는 헤드리스 크롬이 없어도 **Swift + WKWebView 스냅샷**이라는 경로가 이미 깔려 있다.

## 핵심 정리

- **렌더러가 없다고 포기하지 않는다.** Chrome·wkhtmltoimage·ImageMagick·Pillow가 다 없어도 macOS에는 WebKit이 있다. `swiftc` 한 번으로 `html → png` CLI가 만들어진다.
- CLI에서도 **런루프가 필요하다.** `NSApplication.shared` + `setActivationPolicy(.accessory)` + `app.run()`을 하지 않으면 `didFinish`도 `takeSnapshot` 콜백도 오지 않는다.
- **폰트는 `document.fonts.ready` 이후에 찍는다.** `didFinish`는 문서 로드지 웹폰트 적용이 아니다. 먼저 찍으면 폴백 폰트로 박제된다. 로컬 폰트·이미지를 참조하려면 `loadFileURL(_:allowingReadAccessTo:)`로 디렉터리 읽기 권한을 함께 준다.
- **배율은 두 번 곱해진다.** 디자인을 1080 CSS px로 하고 `documentElement.style.zoom = 2` + 웹뷰 프레임 2160으로 잡으면, Retina backing scale이 한 번 더 곱해져 실제 출력은 4320px가 나온다. 최종 규격은 `sips -z`로 맞추는 편이 예측 가능하다.
- **`zoom`을 쓰면 `getBoundingClientRect()`도 zoom 배율로 돌아온다.** 측정값을 CSS px 기준과 그대로 비교하면 전부 오탐이 된다. 배율로 나눈 뒤 비교한다.
- **고정 크기 캔버스의 넘침은 `scrollHeight`로 못 잡는다.** `overflow:hidden`이면 `scrollHeight == clientHeight`라 항상 "맞음"이 나온다. 자식 요소들의 `getBoundingClientRect().bottom` 최댓값을 쓰되, **일부러 캔버스 밖으로 빼는 장식(absolute 배경 블롭)은 제외**해야 한다 — 안 그러면 이번엔 전부 넘침으로 나온다.

## 기록

### 2026-08-22 — 인스타 카드뉴스를 코드로 찍으면서

- 맥락: 보험찾개냥 홍보용 인스타 카드뉴스 5장(1080×1080)과 스토리 1장(1080×1920)을 만들었다. 앱의 테마 토큰(`Client/lib/ui/core/themes/app_colors.dart`)·로고 SVG·lucide 아이콘·Jua/Pretendard 폰트를 그대로 HTML로 옮겨 브랜드를 맞췄다.
- 배운 것:
  - 이미지 렌더링 수단을 찾을 때 **설치 여부부터 확인하고 없으면 플랫폼 기본기로 내려간다.** `command -v` 몇 줄로 크롬·매직·PIL이 모두 없다는 걸 확인한 뒤 Swift/WebKit으로 갔고, 다운로드 없이 30줄짜리 도구로 끝났다.
  - **눈으로 확인하는 검증은 늦다.** 처음엔 렌더 후 이미지를 열어 보고 잘림을 발견했다. 넘침을 숫자로 뱉는 검사를 도구에 넣고 나서야 반복이 빨라졌다 — 다만 그 검사 자체가 두 번 틀렸다(장식 블롭 포함, zoom 배율 미보정). **자동 검사도 처음 나온 값은 의심한다.**
- 근거: 스크래치패드 `insta/shot.swift`(WKWebView 스냅샷 + 넘침 검사), `insta/gen.py`(카드 HTML 생성). 결과물 `(로컬 경로)`
