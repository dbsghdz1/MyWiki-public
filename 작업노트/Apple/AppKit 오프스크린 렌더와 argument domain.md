---
type: study
area: Apple
audience: ai
status: active
created: 2026-09-01
updated: 2026-10-01
projects:
  - "Zappy"
---

# AppKit 오프스크린 렌더와 argument domain

앱 UI를 **창 없이**(CLI·`-renderShots`) 그려서 PNG로 뽑거나, 실제 사용자 설정을 건드리지 않고 특정 상태를 강제해 확인할 때 걸리는 것들. Zappy 1.13에서 스토어 스크린샷 생성기(`docs/store-screenshots/generate.swift`)와 온보딩·NEW 배지 검증을 만들며 확인했다.

## 핵심 정리

- **창 없는 NSView는 시스템 외관을 물려받는다.** `NSView`를 윈도우에 붙이지 않고 `displayIgnoringOpacity(_:in:)`로 그리면 `effectiveAppearance`가 `NSApp`(없으면 시스템)에서 오므로, **다크 모드 Mac에서는 `labelColor`가 흰색**이 되어 밝은 배경에 그린 텍스트·틴트 아이콘이 통째로 사라진다. `NSAppearance.performAsCurrentDrawingAppearance {}`로 호출을 감싸도 소용없다 — 뷰가 display 시점에 자기 `effectiveAppearance`를 다시 current로 세운다. 답은 **`view.appearance = NSAppearance(named: .aqua)`** 처럼 뷰 자체에 박는 것. 앱 안의 `-renderShots`도 같은 이유로 `view.appearance`·`window.appearance`를 aqua로 고정한다.
- **벡터 재렌더는 `NSImage(size:flipped:drawingHandler:)` 안에서 뷰를 그리면 된다.** `bitmapImageRepForCachingDisplay`는 그 시점 배율(창 없으면 1x)로 굽지만, drawingHandler는 그릴 때마다 재호출되므로 `image.draw(in: 큰 rect)`로 확대해도 선명하다. flipped 뷰(`isFlipped = true`)는 handler도 `flipped: true`로 맞춘다.
- **argument domain으로 실제 설정을 안 건드리고 상태를 강제한다.** `open App.app --args -key value`(또는 CLI 바이너리에 `-key value`)는 `UserDefaults.standard` 읽기에서 최우선이고 프로세스가 사는 동안만 유효하다. `set()`으로 써도 읽기는 계속 인자 값을 돌려주므로 "첫 실행" 같은 상태를 반복 재현하기 좋다.
  - **`-AppleLanguages "(en)"`** — `Locale.preferredLanguages`가 이걸 따르므로 앱의 수동 L10n(`Locale.preferredLanguages` 기반)이 그대로 전환된다. 앱(`open --args`)과 swiftc로 만든 CLI 렌더러 모두 동작.
  - **배열 값은 old-style plist**로 파싱된다 — ASCII면 `"(풍선,나무)"` 처럼 따옴표 없이 되지만 **한글·비ASCII 원소는 따옴표가 없으면 파싱에 실패해 문자열로 취급**되고 `stringArray(forKey:)`는 nil → 아무 일도 안 일어난다. `'("풍선","나무")'`로 쓴다.
  - Bool은 `-onboarded NO`처럼 YES/NO.
- **`-theme <값>`·`-forceLevel N`으로 띄운 테스트 인스턴스는 메뉴에서 테마를 골라도 바뀌지 않는다** — 인자 도메인이 `UserDefaults` 읽기에서 최우선이라 `set()`은 되지만 읽기가 계속 인자 값을 돌려준다. 사용자에게 "선택해도 안 바뀐다"로 보이니 **사용자가 만질 인스턴스는 `-devFullAccess YES`만 붙여 띄운다**. 그리고 `open Zappy.app`은 이미 떠 있는 인스턴스를 재사용하므로 새 빌드를 보려면 먼저 `kill <pid>`.
- **`NSImage.lockFocus()`로 그린 PNG는 Retina에서 2x(840×528)로 구워진다.** 정해진 픽셀 크기(랜딩 카드 420×264)가 필요하면 `NSBitmapImageRep(bitmapDataPlanes:nil, pixelsWide:…)`를 만들고 `NSGraphicsContext(bitmapImageRep:)`를 current로 세워 직접 그린다.
- **레퍼런스 PNG/GIF를 전체 재생성하면 손대지 않은 테마 파일도 바이트가 바뀐다**(`CGImageDestination` 인코더 노이즈, 69/95 파일). 커밋 전 `git status --short docs/theme-reference | grep "^ M" | grep -v <바꾼 테마> | xargs git checkout --`로 되돌린다. 단 `??`(신규) 파일이 섞이면 xargs가 pathspec 에러로 멈추니 `^ M`만 고른다.
- 실행 파일명과 앱 이름이 다르면(`CuteBattery` ↔ Zappy.app) 개발 인스턴스만 죽일 때는 `pkill -x`가 스토어 설치본까지 잡는다 — `pgrep -f "<경로>/Zappy.app/Contents/MacOS"`로 PID를 골라 죽인다.

## 기록

### 2026-09-01 — Zappy 1.13 스토어 스크린샷 생성기 · 온보딩/NEW 배지 검증
- 맥락: 5로케일 스크린샷을 Figma 대신 코드로 만들며 4번째 장에 실제 `ThemeGridView`를 넣었다. 첫 렌더에서 그리드 라벨·모노 아이콘이 전부 흰색(투명해 보임) — 홍의 Mac이 다크 모드라 `labelColor`가 흰색으로 해석된 것. `performAsCurrentDrawingAppearance`는 효과 없었고 `grid.appearance = NSAppearance(named: .aqua)`로 해결.
- NEW 배지 확인용 `-newThemes "(풍선,나무)"`는 무동작 → `'("풍선","나무")'`로 바꾸자 배지가 떴다.
- 근거: `docs/store-screenshots/generate.swift`(쇼트 4의 `grid.appearance` 주석), 레포 CLAUDE.md 개발 플래그 절, 개발 기록 [Zappy 개발 기록 2026-09-01](../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/Zappy/Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-09-01.md).

### 2026-09-04 — Zappy 1.14 테마 다듬기 · 랜딩 카드 · 테스트 인스턴스
- 맥락: 날씨·사과·선인장·해파리 4종을 다듬고(`WeatherGeom`·`AppleGeom` 신설) 렌더 하네스로 검증하던 중. 홍이 테스트 앱 메뉴에서 "다른 캐릭터로 안 바뀌네" — `ps`로 보니 `-theme 사과 -forceLevel 60` 인자로 떠 있던 인스턴스(PID 18874). 죽이고 `-devFullAccess YES`만으로 재실행하니 정상. 그 뒤 `package-app.sh` 후 `open`했는데 옛 프로세스(47367)가 그대로 살아 있어 새 빌드가 안 뜸 → `kill` 후 `open`.
- 랜딩 카드 재생성: `lockFocus` 결과가 840×528 → `NSBitmapImageRep` 직접 그리기로 420×264.
- `docs/theme-reference` 재생성에서 69개 노이즈 파일 되돌림; 풍선 레퍼런스 6장이 그동안 빠져 있었던 것도 이때 발견해 추가.
- 근거: [Zappy 개발 기록 2026-09-04](../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/Zappy/Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-09-04.md), 커밋 `6cda9c7`·`45a171a`·`e4a2b1b`.

### 2026-10-01 — `-renderShots`는 폴더가 없으면 조용히 아무것도 안 남긴다 · `screencapture`는 권한 없이 좌표 에러처럼 죽는다 (Zappy 1.17)

- `open Zappy.app --args -renderShots /tmp/x`에서 `/tmp/x`가 없으면 `try? ... write(to:)`가 실패하고 앱은 렌더 뒤 그대로 종료 — 폴더도 PNG도 안 생긴다. **`mkdir -p` 먼저.** 생기는 파일: `grid.png`·`paywall.png`·`report.png`·`onboarding.png`·`onboarding-2.png`(라이트 외관 고정). 온보딩 2페이지에 체크박스를 추가한 뒤 이걸로 영어 설명이 2줄을 넘는 걸 잡았다.
- `screencapture -x -R x,y,w,h`가 **"could not create image from rect"** 로 실패하면 좌표 문제가 아니라 터미널(auto 모드 세션)에 화면 기록 권한이 없는 것이다. `-R`이 화면 밖이면 메시지가 다르다("does not intersect any displays" — Finder `bounds of window of desktop`은 여러 모니터 합집합이라 메인 화면 폭으로 쓰면 안 된다; `swift -e 'import AppKit; print(NSScreen.main!.frame)'`로 잰다). 메뉴바 라벨은 캡처 대신 스크린샷 생성기가 같은 L10n 함수로 그린 장으로 확인했다.
- 맥락: [Zappy](../../%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8/%EA%B0%9C%EC%9D%B8/Zappy/README.md) 1.17 아이콘 옆 「남은 시각」 라벨 검증.
