---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-25
updated: 2026-08-25
projects:
  - "[[프로젝트/개인/즉석카메라/README|Fadeo(즉석카메라)]]"
---

# 시뮬레이터로 UI 검증하기

iOS 앱을 만들면서 **화면을 눈으로 확인하는 방법**에 대한 것. 시뮬레이터에는 구멍이 세 개 있고, 그 구멍을 우회하는 정답은 **앱 안에 DEBUG 데모 모드를 넣는 것**이다.

## 핵심 정리

### 구멍 셋 — 시뮬레이터로는 못 보는 것

| 막히는 것 | 이유 |
|---|---|
| **탭·스와이프·스크롤·흔들기** | `simctl`에 입력 명령이 **아예 없다.** `io ... screenshot`은 있어도 tap이 없다 |
| **카메라** | 하드웨어가 없다. `AVCaptureSession`에 입력이 안 붙고 프리뷰는 검다 |
| **권한 다이얼로그** | 넘길 방법이 없다. `simctl privacy grant camera`를 해도 **카메라 부재는 못 고친다** |

`simctl privacy grant`는 TCC 레코드만 손대므로 하드웨어가 필요한 권한(카메라)에는 무력하다. 사진 라이브러리·알림처럼 순수 권한은 통한다.

### 권한 다이얼로그는 고착된다 — `erase`만 듣는다 ★

한 번 뜬 권한 다이얼로그는 **`terminate` → `launch`로도, 앱을 다시 설치해도 사라지지 않는다.** 화면 위에 그대로 남아 아래 UI를 가린다.

- 듣는 것: `xcrun simctl erase <device>` (부팅 상태면 `shutdown` 먼저)
- **이게 오진의 원인이 된다.** 코드를 고치고 재실행했는데 옛 다이얼로그가 그대로 있으면, 고친 코드가 안 먹은 것으로 읽는다. 실제로 [[작업노트/Apple/SwiftUI|SwiftUI]]에 "TabView가 안 보이는 탭도 미리 로드해 권한을 묻는다"는 **틀린 결론**을 남긴 적이 있다 — 로그를 찍어 보니 그 `.task`는 아예 실행되지 않았다.
- 규칙: **화면이 예상과 다르면 코드를 의심하기 전에 `erase`부터 한 번 한다.**

### 답 — 앱 안에 DEBUG 데모 모드를 넣는다 ★

바깥에서 시뮬레이터를 조종할 수 없으면 **안에서 앱이 스스로 움직이게** 한다.

```swift
enum DemoMode {
    #if DEBUG
    static let isOn = ProcessInfo.processInfo.environment["APP_DEMO"] == "1"
    static var autoShootDelay: TimeInterval? { ... }   // 탭 대신 스스로 촬영
    static var startsInAlbum: Bool { ... }             // 탭 전환 대신 그 화면에서 시작
    static var seedCount: Int? { ... }                 // 목록 화면을 채울 더미 데이터
    #else
    static let isOn = false
    #endif
}
```

환경변수는 `SIMCTL_CHILD_` 접두사로 넘긴다 — **`simctl launch`가 접두사를 떼고 앱 프로세스에 넘겨준다.**

```bash
SIMCTL_CHILD_APP_DEMO=1 SIMCTL_CHILD_APP_DEMO_SEED=6 \
  xcrun simctl launch "$DEVICE" com.example.app
```

이렇게 열리는 것:

1. **카메라 자리에 번들 사진을 흘린다** → 프리뷰·촬영 경로를 시뮬레이터에서 그대로 본다
2. **권한 요청을 건너뛴다** → 다이얼로그가 안 뜨니 화면이 안 가려진다
3. **자동으로 한 번 동작시킨다**(촬영·흔들기 등) → 애니메이션을 프레임 단위로 찍는다
4. **더미 데이터를 심는다** → 빈 목록 말고 실제 사용 중인 화면을 본다
5. **특정 화면에서 시작한다** → 탭할 수 없는 화면에 도달한다

부수 효과가 하나 더 있다. **스토어 스크린샷을 실제 앱에서 뽑을 수 있다.** 그 전에는 앱 렌더 경로를 스크립트로 재구현해 PNG를 만들고 있었는데, 재구현은 실제와 어긋날 수 있고 어긋나면 스토어 이미지가 거짓말이 된다.

### 애니메이션을 프레임으로 찍기

`simctl`에 녹화(`io recordVideo`)가 있지만 판정에는 **스트립이 낫다** — 한 화면에 놓고 비교할 수 있다.

```bash
S=$(python3 -c "import time;print(time.time())")
for t in 3.4 4.4 5.4 6.4; do
  python3 -c "import time;d=$S+$t-time.time()
if d>0: time.sleep(d)"
  xcrun simctl io "$D" screenshot out/$t.png
done
```

**절대 시각 기준으로 기다린다.** `sleep 1`을 반복하면 스크린샷 자체가 0.3~0.5초씩 먹어 뒤로 갈수록 어긋난다.

가로로 붙이는 건 `sips`(리사이즈) + 짧은 Swift/AppKit 스크립트로 충분하다. macOS에 ImageMagick도 PIL도 기본 설치돼 있지 않다.

### 화면을 지우고 다시 하는 표준 절차

```bash
xcrun simctl shutdown "$D"; xcrun simctl erase "$D"; xcrun simctl boot "$D"
xcrun simctl bootstatus "$D" -b >/dev/null
xcrun simctl install "$D" path/to/App.app
SIMCTL_CHILD_... xcrun simctl launch "$D" com.example.app
```

`bootstatus -b`가 있어야 부팅 완료를 기다린다. 이게 없으면 `install`이 간헐적으로 실패한다.

### 앱 안이 만든 파일을 꺼내 보기

내보내기·합성 결과처럼 **화면 밖으로 나가는 그림**은 스크린샷으로 확인이 안 된다. 데모 모드에서 Documents에 떨어뜨리고 컨테이너에서 꺼낸다.

```bash
C=$(xcrun simctl get_app_container "$D" com.example.app data)
cp "$C/Documents/export-check.png" /tmp/
```

## 기록

### 2026-08-25 — Fadeo 촬영·앨범 화면을 다시 짜면서

- 맥락: [[프로젝트/개인/즉석카메라/README|Fadeo]]의 카메라 화면을 "카메라 본체"로 다시 만드는 작업. 레이아웃·배출 애니메이션·앨범을 눈으로 봐야 하는데 시뮬레이터에서 **첫 화면조차 권한 다이얼로그에 가려 안 보였다.**
- 배운 것: 위 「핵심 정리」 전부. 특히 ① `simctl`에 탭이 없다 ② 권한 다이얼로그는 `erase`만 듣고 그 고착이 **오진을 만든다** ③ 답은 바깥 조종이 아니라 **앱 안의 DEBUG 데모 모드**.
- 근거: `(로컬 경로)` 커밋 `de78e07`(`Sources/App/DemoMode.swift` 신설). 데모 모드 도입 전에는 배출 애니메이션·앨범·상세 화면을 한 번도 못 봤고, 도입 후 같은 세션에서 전부 확인하고 레이아웃 결함 네 개를 고쳤다.

### 스크린샷을 찍기 전에 상태 표시줄을 고정한다

`simctl status_bar`로 시각·통신사·배터리를 덮어쓸 수 있다. 스토어 스크린샷에 실제 시각과 반쯤 닳은 배터리가 찍히는 걸 막는다.

```bash
xcrun simctl status_bar "$D" override   --time "9:41" --cellularMode active --cellularBars 4 --batteryState charged --batteryLevel 100
```

`xcrun simctl status_bar "$D" clear`로 되돌린다.

## 참고 자료

- `xcrun simctl help launch` — **`SIMCTL_CHILD_` 규약의 1차 출처.** *"If you want to set environment variables in the resulting environment, set them in the calling environment with a SIMCTL_CHILD_ prefix."* (2026-08-25 로컬 확인)
- `xcrun simctl help` — 하위 명령 전체 목록. **tap·swipe·shake가 없다는 것이 여기서 확정된다** (2026-08-25 로컬 확인). 대신 알아둘 만한 것: `status_bar`(상태 표시줄 덮어쓰기) · `push`(가짜 푸시 전송) · `addmedia`(사진 라이브러리에 넣기) · `get_app_container`(컨테이너 경로)
- [Running your app on simulated or physical devices](https://developer.apple.com/documentation/xcode/running-your-app-in-simulator-or-on-a-device) — Apple 공식 개요. **`SIMCTL_CHILD_` 언급은 이 페이지에 없다**(2026-08-25 WebFetch로 확인) — 그 규약은 `simctl help`에만 있다
