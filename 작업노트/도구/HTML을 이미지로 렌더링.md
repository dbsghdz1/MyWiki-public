---
type: study
area: 도구
audience: ai
status: active
created: 2026-08-22
updated: 2026-09-04
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

### 2026-09-02 — 같은 카드뉴스를 Pillow로 찍으면서 (폰트 함정 둘)

- 맥락: 보험찾개냥 인스타에 「보험사마다 다른 청구 서류」 캐러셀 3장(1080×1350)을 추가로 만들었다. 08-02 런칭 캐러셀이 이미 **Pillow**로 짜여 있어(`(로컬 경로)`) 그 시각 체계를 그대로 이어썼다 — 08-22에 "PIL이 없다"고 적었지만 지금 환경에는 있다.
- 배운 것:
  - **Jua에는 가운뎃점(`·`, U+00B7) 글리프가 없다.** 제목에 `동물등록증 · 사진`을 넣었더니 두부(□)로 렌더됐다. Pillow는 없는 글리프를 조용히 `.notdef`로 그리고 **에러도 경고도 없다.** 같은 문자를 Pretendard 본문에 쓰면 멀쩡해서, 폰트를 섞어 쓰는 레이아웃에서는 **어느 폰트로 그린 줄인지**가 곧 안전 여부다. 디스플레이 폰트에는 한글·숫자·기본 문장부호 밖으로 나가지 않는 편이 싸다.
  - **Jua는 시스템 폰트에 없다.** `(로컬 경로)`에 Pretendard 9종은 있는데 `Jua-Regular.ttf`는 없고, 앱 저장소 `Boheomgaenyang/Client/assets/fonts/`에만 있다. 08-02 스크립트가 `(로컬 경로)`를 가리키고 있어 그대로 돌리면 죽는다 — **앱이 번들하는 폰트를 마케팅 이미지에도 쓸 거면 저장소 쪽을 원본으로 참조한다.** 시스템 설치는 기기마다 다르다.
  - **고정 높이 카드에서 겹침은 렌더 결과를 봐야 잡힌다.** 항목 카드 3개(240px)를 280 간격으로 깔았더니 마지막 카드가 하단 각주를 덮었는데, 스크립트는 정상 종료했다. 08-22에 적은 넘침 검사가 Pillow 경로에는 없다 — **좌표를 손으로 계산하는 렌더러는 눈으로 한 번 보기 전까지 끝난 게 아니다.**
- 근거: `(로컬 경로)`, 카드 내용 출처는 `Server/src/main/resources/db/migration/V2__create_document_rule.sql`(원본 노션 「보험사 서류 정리」 2026-07-28 기준).

### 2026-09-04 — 메타 광고 소재(9:16·4:5·영상 오버레이)를 Pillow로 찍으면서

- 맥락: 보험찾개냥 전수열 멘토링 미션 "비라이브 광고 3개". 정지 소재 4장(A·C × 9:16·4:5), 영상용 투명 자막 오버레이 3장, 타이밍 미리보기 mp4를 만들었다 → 메타 광고 소재 기획.
- 배운 것:
  - **Dia.app은 헤드리스 렌더러가 아니다.** 크로미움 기반이라 `/Applications/Dia.app/Contents/MacOS/Dia --headless=new --screenshot=… --user-data-dir=…`를 시도했지만 60초 넘게 돌아가다 exit 144, PNG 없음. `/Applications`에 Chrome·Chromium이 없으면 이 경로는 보지 말고 바로 Pillow(있음)나 WKWebView(08-22)로 간다.
  - **Jua 폰트는 Google Fonts에서 직접 받아진다** — `curl -A "Mozilla/5.0" "https://fonts.googleapis.com/css2?family=Jua"`로 CSS를 받으면 `https://fonts.gstatic.com/s/jua/v18/co3KmW9ljjAjcw.ttf`가 나오고 그 TTF를 Pillow가 그대로 연다. 다만 09-02 기록대로 **원본은 앱 저장소 `Client/assets/fonts/Jua-Regular.ttf`** 라 `render.py`는 저장소 파일을 우선하고 gstatic 사본은 폴백으로 뒀다.
  - **4:5는 9:16을 줄여서 못 만든다.** 같은 레이아웃 함수에 높이만 1350을 넣었더니 CTA가 잘렸다. 카드 내부 여백(`compact=True`: pad 36, 행 간격 56)·스텝 카드 높이(82)·소제목 생략까지 별도 분기가 필요했다. 9:16은 상단 250px·하단 340px가 릴스 UI에 가리므로 본문은 그 안쪽에 둔다.
  - **발바닥 아이콘은 발가락 비율이 전부다.** 큰 원 3개(r=0.15s)로 그리면 미키마우스로 읽힌다. 납작한 패드(0.72×0.52s) + 작은 발가락 4개(r=0.115s)를 호 모양으로 놓아야 발바닥이 된다. og.png의 실제 아이콘을 보고 고쳤다.
  - **도장(회전 텍스트)은 임시 RGBA에 그려 `rotate(expand=True)` 후 `alpha_composite`.** 영수증 카드 위에 얹을 때 금액 줄을 덮지 않도록 카드 높이를 늘려(480) 빈 띠에 넣는 편이 "도장이 숫자를 가리는" 것보다 낫다.
  - **ffmpeg 슬라이드쇼는 `xfade` 체인으로 한 줄.** `-loop 1 -t 4` 입력 3개에 `[0][1]xfade=transition=fade:duration=0.5:offset=3.5[v1];[v1][2]xfade=…:offset=7[v2]` — 두 번째 offset은 앞 구간 합(4+4)에서 전환 길이·앞 전환 겹침을 뺀 값(7). 결과 12초·1080×1920·yuv420p, 애드 매니저 업로드 규격에 맞다.
- 근거: `(로컬 경로)`(같은 폴더에 산출물 14개), 세션 스크래치패드 `ads/`.
