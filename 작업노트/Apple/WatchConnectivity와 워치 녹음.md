---
type: study
area: Apple
audience: ai
status: active
created: 2026-09-01
updated: 2026-09-01
projects:
  - "[[프로젝트/개인/WristNote/README|WristNote]]"
---

# WatchConnectivity와 워치 녹음

watchOS 앱에서 `AVAudioRecorder`로 녹음하고 `WCSession.transferFile`로 아이폰에 보내는 구조를 시뮬레이터 페어에서 검증하며 알게 된 것.

## 핵심 정리

### 시뮬레이터는 `transferFile`을 배달하지 않는다 ★

**Apple 문서가 명시한 한계**: 시뮬레이터는 `transferFile(_:metadata:)`를 지원하지 않으며 실기기 페어에서 테스트하라고 한다. 증상이 교묘해서 앱 버그로 오진하기 쉽다.

- 워치 쪽 `session(_:didFinish:error:)`는 **호출 9~17ms 만에 `error=nil`로 온다** — 성공처럼 보인다.
- 아이폰 `wcd` 데몬 로그엔 IDS 메시지가 **실제로 도착한다**(`com.apple.private.alloy.watchconnectivity received message … 28222 bytes` — 25KB 녹음 파일 크기와 일치). 그런데 앱의 `session(_:didReceive file:)`는 영원히 안 불리고 `Documents/Inbox/com.apple.watchconnectivity/`도 비어 있다.
- `WCSessionState`는 `reachable: YES, paired: YES, appInstalled: YES` — 페어링 상태로는 구분이 안 된다.
- 판정 순서: 워치 `didFinish` 즉시 성공 + 아이폰 `didReceive` 무호출이면 코드를 더 파지 말고 실기기로 간다.

### 시뮬레이터에서 되는 것

- **워치 시뮬레이터 마이크 녹음은 된다** — 맥 마이크가 입력으로 잡히고(`availableInputs=1`, `iOSSimulatorAudioDevice`), 8.6초 녹음에 AAC 16kHz mono 32kbps로 25KB가 나온다. `AVAudioSession.setCategory(.record)` + `AVAudioRecorder.record()` 그대로.
- 페어 만들기: `xcrun simctl pair <watch-udid> <phone-udid>` — 아이폰이 부팅된 채로도 된다. `xcrun simctl list pairs`로 확인.
- 워치 앱 설치는 iOS 앱 번들 안의 `WristNote.app/Watch/WristNoteWatch.app`을 워치 시뮬레이터에 `simctl install`. 마이크 권한은 `simctl privacy <watch> grant microphone <bundle id>`가 통한다.
- 워치 앱에 env 훅(`WRISTNOTE_AUTOTEST=1` → 2초 뒤 8초 녹음·정지·전송)을 두면 `simctl`에 탭이 없어도 흐름을 돌릴 수 있다. env는 `SIMCTL_CHILD_` 접두사로 `simctl launch`에 넘긴다.

### `WCSessionFile` URL은 델리게이트 반환 후 무효

`session(_:didReceive file:)` 안에서 **동기적으로** `FileManager.moveItem`으로 옮긴 뒤 비동기 처리로 넘긴다. 델리게이트는 백그라운드 스레드에서 불린다. 이 노트에선 검증 못 했지만(위 한계) 문서상 규칙이라 코드에 반영해 두었다.

### 아직 모르는 것 (실기기 M0)

- **화면 꺼짐·손목 내림 상태에서 30~60분 녹음이 유지되는가.** `WKBackgroundModes: [audio]` + 활성 `.record` 세션으로 구현했고, 시뮬레이터로는 판정 불가.
- 배터리 소모.

## 기록

### 2026-09-01 — WristNote v1 골격 시뮬레이터 페어 검증

- 맥락: [[프로젝트/개인/WristNote/README|WristNote]] 야간 자율 작업. 워치 녹음 → `transferFile` → 아이폰 수신 흐름을 Series 11(46mm) + iPhone 17 Pro 시뮬레이터 페어에서 돌렸다.
- 배운 것: 위 「핵심 정리」 전부. 워치 녹음·전송 호출까지는 되고 아이폰 배달만 시뮬레이터 한계.
- 근거: `(로컬 경로)` 커밋 `f8c4bd3` (`WatchRecorder.swift`·`WatchSession.swift`·`PhoneSession.swift`). 워치 로그 `subsystem == "com.hong.wristnote.watch"`: `recording started inputs=1` → `stop: … exists=true size=25120` → `didFinish … error=nil`. 아이폰 `wcd` 로그 21:14:28 `received message … 28222 bytes`.

## 참고 자료

- [WCSession transferFile Not Working in Simulator — Apple Developer Forums](https://developer.apple.com/forums/thread/128205) — 시뮬레이터 미지원, 실기기 테스트 권고 (2026-09-01 확인)
- [WCSession transferFile not doing anything — Apple Developer Forums](https://developer.apple.com/forums/thread/733155) — 같은 증상(워치 didFinish 성공, 아이폰 didReceive 무호출), 실기기에선 정상 (2026-09-01 확인)
