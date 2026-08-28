---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-16
updated: 2026-08-19
projects:
  - "[[프로젝트/개인/BarStack/README|BarStack]]"
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# macOS 메뉴바와 샌드박스

App Sandbox 안에서 다른 앱의 메뉴바 아이템을 "알아내는" 유일한 통로는 window server(CGWindowList)이며, 앱 식별은 화면 기록 권한 뒤에 잠겨 있다. Accessibility API는 샌드박스에서 권한을 줘도 통째로 막힌다.

## 핵심 정리

- **샌드박스는 AX(Accessibility) IPC를 통째로 차단한다.** `AXIsProcessTrusted()`가 `true`여도 다른 프로세스에 대한 모든 `AXUIElementCopyAttributeValue`가 `kAXErrorCannotComplete(-25204)`를 돌려준다 — 메뉴바 아이템(`kAXExtrasMenuBarAttribute`)만이 아니라 Finder 창 목록(`kAXWindows`)도. 같은 바이너리를 샌드박스 entitlement만 빼고 서명하면 29개가 잡힌다. "권한 허용해도 안 되는" 것이라 재검증해도 결과는 같다.
- **`CGWindowListCopyWindowInfo`는 샌드박스·권한 없이 동작한다.** 메뉴바 아이템은 status window level(`CGWindowLevelForKey(.statusWindow)` = 25)의 창으로 열거되고 위치·폭·onscreen 여부가 나온다. macOS 26에서는 이 창들의 owner가 전부 **Control Center**라 owner PID로는 앱을 알 수 없다.
- **앱 식별은 `kCGWindowName`에만 있고, 이 필드는 화면 기록(Screen Recording) 권한이 있어야 채워진다** (10.15+). 게다가 window server가 프로세스 연결 시점의 권한을 쓰기 때문에, 실행 중에 권한을 받으면 **앱을 재실행해야** 이름이 보인다. 아무것도 캡처하지 않아도 이 권한이 필요하다.
- **진짜 창과 복제 창.** 디스플레이가 둘이면 메뉴바 행도 둘인데, 한 행은 앱의 **진짜** 상태 아이템 창(이름 = NSStatusItem autosave 이름, 대개 `Item-0`)이고 다른 행은 macOS가 만든 **복제**(이름 = 소유 앱 **번들 ID**)다. 두 행이 어느 디스플레이에 놓이는지는 활성 메뉴바에 따라 수시로 바뀐다. **단일 디스플레이(미러링으로 실측)에선 복제 행이 없어 번들 ID를 아예 얻을 수 없다** — 창 메타데이터로 앱을 식별하는 길은 멀티 디스플레이에서만 열린다. Control Center 자체 항목(Wi‑Fi·시계)은 복제 행에서 이름이 비어 있고, 인접 아이템의 순서가 두 행에서 다를 수 있다(Notion↔Macs Fan Control) — 행 간 인덱스 매칭은 폭 일치까지 확인해야 한다.
- **글리프는 진짜 창을 캡처하면 나온다.** ScreenCaptureKit `SCContentFilter(desktopIndependentWindow:)`로 **진짜(autosave 이름) 창**을 찍으면 템플릿(단색) 아이콘까지 렌더링되고, 노치 아래(onScreen=false)도 된다. **복제(번들 ID) 창**은 템플릿 아이콘이 투명으로 나온다. 어느 쪽이든 아이템이 디스플레이 밖으로 밀려나면(`x < 화면`) 캡처가 `-3811`로 실패하므로, 보일 때 찍어 캐시해야 한다.
- **합성 마우스 클릭은 진짜 창만 받는다.** 손쉬운 사용 권한이 있으면 샌드박스 앱도 `CGEvent.post(tap: .cghidEventTap)`으로 클릭을 보낼 수 있고 상태 아이템 메뉴도 열린다 — 단 **번들 ID 이름의 복제 창은 어떤 탭(hid/session/annotated/postToPid)으로 보내도 무시**한다(6/6 상관). 그리고 한 번의 이동 직후 down/up은 씹힌다: 근처로 이동 → 정확한 지점으로 한 번 더 이동(≥250ms) → down → up 순서여야 첫 클릭부터 먹는다(트래킹 영역 갱신 문제로 추정). 클릭 카운트(`mouseEventClickState=1`)도 넣어야 한다.
- **NSStatusItem 길이 보간은 macOS 26에서 슬라이드가 아니다.** 스페이서 아이템의 `length`를 16ms 간격으로 늘려도 macOS는 중간값마다 바 전체를 재배치할 뿐 슬라이드로 그리지 않는다 — 35ms 프레임 캡처로 보면 펼침은 한 프레임에 나타나고, 접힘은 ~150ms 동안 항상 보이는 아이템까지 사라졌다 돌아오며 늘어나는 아이템 안의 이미지(‹)가 중앙에 떠다닌다. 즉시 점프가 오히려 깨끗하다. 또 길이 변경 직후 ~수백 ms는 창 서버 좌표가 과도기라 그 사이 CGWindowList로 위치를 읽으면 오탐이 난다.
- **IORegistry 읽기는 샌드박스에서 막히지 않는다.** AX와 달리 `IOServiceGetMatchingService` + `IORegistryEntryCreateCFProperties`는 entitlement 없이도 그대로 동작한다 — `AppleSmartBattery`의 58개 프로퍼티(사이클 수·설계 용량·온도·전압·전류)가 샌드박스 안팎에서 동일하게 읽혔다. 샌드박스가 막는 것은 **다른 프로세스에 대한 IPC**(AX·Apple Events)이지 커널 디바이스 트리 조회가 아니다. IOKit이라도 `IOServiceOpen`으로 드라이버에 연결하는 것은 별개이며 여기선 필요 없다.
- **샌드박스 여부를 실기로 검증하는 최소 절차**: 테스트 바이너리를 `.app` 껍데기(Info.plist 포함)에 넣고 `codesign --force --sign - --entitlements sb.entitlements`로 **애드혹 서명**하면 샌드박스가 실제로 켜진다. 팀 서명 없이도 되므로 "이 API가 MAS에서 되나?"를 5분 안에 답할 수 있다. 단 팀 ID 접두 App Group 같은 항목은 애드혹으로 검증되지 않으니 빼고 최소 entitlement만 넣는다.
- **TCC 프롬프트가 아니라 시스템 설정 + 재실행.** 화면 기록·손쉬운 사용 모두 시스템 프롬프트는 앱당 한 번만 뜨고, 그 뒤엔 설정 패널을 직접 열어줘야 한다(`x-apple.systempreferences:com.apple.preference.security?Privacy_ScreenCapture` / `?Privacy_Accessibility`).
- **TCC의 "책임 프로세스" 함정.** 터미널에서 직접 실행한 바이너리는 터미널의 권한을 물려받아 `AXIsProcessTrusted`가 true로 나온다. 앱 자체의 권한을 재려면 `open`(LaunchServices)으로 띄워야 한다.

## 기록

### 2026-08-16 — 숨긴 아이콘을 팝오버에 보여주기 위한 재검증과 구현
- 맥락: [[프로젝트/개인/BarStack/README|BarStack]]에서 "팝오버에 어떤 앱이 숨겨져 있는지 보여달라"는 요구. wiki엔 "샌드박스가 AX를 차단"이라고 실측돼 있었고, 나는 Magnet(MAS·샌드박스·AX 사용)을 반례로 들어 재검증을 제안했다가 **틀렸다** — 샌드박스는 AX를 정말 통째로 막는다.
- 배운 것:
  - 샌드박스에서 다른 앱 메뉴바를 읽는 길은 CGWindowList + 화면 기록 권한(이름 해금)뿐. 클릭 실행은 불가(AX).
  - 화면 기록 권한은 재실행 후 반영, 디스플레이별 행·명명 규칙 차이, 행 간 순서 불일치 — 위 핵심 정리.
- 근거: 테스트 앱 `AXProbeSandboxed/Plain`(같은 바이너리, sandbox entitlement만 차이)을 `open`으로 띄워 손쉬운 사용 허용 후 비교 — 샌드박스 0개(전부 -25204) / 비샌드박스 29개. `SRProbeSandboxed`로 화면 기록 허용 후 재실행 → 번들 ID 30개. 구현 커밋 `fdff799`(CollectionTopBar `HiddenItemsScanner.swift`), 작업 기록 [[프로젝트/개인/BarStack/BarStack 개발 기록 2026-08-16|BarStack 개발 기록 2026-08-16]].

### 2026-08-16 (밤) — 클릭 실행 시도와 되돌림, 단일 디스플레이 실측
- 맥락: [[프로젝트/개인/BarStack/README|BarStack]] 팝오버의 숨긴 아이콘을 눌러 그 앱 메뉴를 열게 하려다가, 클릭이 계속 씹히는 원인을 쫓다 보니 **번들 ID 이름 창이 복제본**이란 걸 알게 됐고, 미러링으로 단일 디스플레이를 흉내 내 보니 **번들 ID 자체가 사라져** 낮에 배포한 목록이 노치 맥북 단독에선 빈 목록이었다.
- 배운 것: 위 핵심 정리의 "진짜 창과 복제 창", "글리프는 진짜 창을 캡처", "합성 클릭은 진짜 창만" 세 항목. 결국 목록은 **글리프 캡처**로 다시 만들었고(진짜 행은 우리 아이템의 autosave 이름 `BarStack.divider`로 찾음), 클릭 실행은 커서 점프·노치 아래 아이템 문제로 사용자 판단에 따라 되돌렸다.
- 근거: 커밋 `1c88fbb`(글리프+클릭) → `e8bdbe6`(클릭 제거). 상관 실험 로그는 [[프로젝트/개인/BarStack/BarStack 개발 기록 2026-08-16|개발 기록]]. 미러링 토글은 `CGConfigureDisplayMirrorOfDisplay`(세션 범위)로 했고, `CGGetActiveDisplayList`는 미러링 중 보조 디스플레이를 빼므로 되돌릴 땐 `CGGetOnlineDisplayList`를 써야 한다.


### 2026-08-19 — 샌드박스에서 배터리 상세를 읽을 수 있나 (IORegistry)
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] 1.7의 Zappy+ 기능으로 "배터리 리포트"(사이클 수·최대 용량·온도·전력)를 넣기로 했는데, 공개 API인 `IOPSCopyPowerSourcesInfo`로는 잔량·남은 시간까지만 나온다. 사이클 수는 `AppleSmartBattery` IORegistry 노드에 있어서 **MAS 샌드박스에서 읽히는지**가 기능 자체의 성립 조건이었다.
- 배운 것:
  - IORegistry 조회는 샌드박스에서 그대로 된다 — entitlement 추가도, 권한 프롬프트도 없다. 위 핵심 정리 두 항목.
  - `MaxCapacity`는 기종에 따라 퍼센트(100)로 오므로 mAh가 필요하면 `AppleRawMaxCapacity`(없으면 `NominalChargeCapacity`)를 쓴다. `Temperature`는 1/100 ℃, `Voltage`는 mV, `Amperage`는 mA(방전 시 음수)라 전력(W)은 `V × A / 1000`.
  - 새 배터리는 완충 용량이 설계 용량을 넘겨 보고된다(6360 / 6249 = 101.8%). 시스템 설정과 같게 보이려면 100%로 클램프해야 한다.
- 근거: 애드혹 서명한 `BattTest.app`(sandbox entitlement만)으로 샌드박스 안/밖 동일 출력 확인 — 사이클 19, 설계 6249mAh, 완충 6360mAh, 30.84℃. 구현은 `Sources/CuteBattery/BatteryHealth.swift`, 커밋 `a791917`.
