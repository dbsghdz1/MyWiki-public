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

### 워치 백그라운드 오디오 키는 `UIBackgroundModes`다 (2026-09-02)

watchOS 앱 Info.plist에 `WKBackgroundModes: [audio]`를 넣으면 빌드·실기기 실행은 되지만 **ASC 업로드에서 거절된다**: `Invalid Info.plist value. The value for the key 'WKBackgroundModes' in bundle WristNote.app/Watch/WristNoteWatch.app is invalid. (90362)`. `WKBackgroundModes`는 `workout-processing`·`self-care`·`mindfulness` 같은 **세션 종류** 전용이고, 백그라운드 오디오(재생·녹음)는 iOS와 같은 **`UIBackgroundModes: [audio]`** 로 선언한다. 로컬에선 아무 경고가 없어 업로드 때 처음 드러난다.

### `WCSessionFile` URL은 델리게이트 반환 후 무효

`session(_:didReceive file:)` 안에서 **동기적으로** `FileManager.moveItem`으로 옮긴 뒤 비동기 처리로 넘긴다. 델리게이트는 백그라운드 스레드에서 불린다. 이 노트에선 검증 못 했지만(위 한계) 문서상 규칙이라 코드에 반영해 두었다.

### 실기기 워치 녹음 — 시뮬레이터에선 절대 안 드러나는 세 가지 ★ (2026-09-02)

Apple Watch(watchOS 26.2) 실기기에서 같은 코드가 세 단계로 실패했다. 셋 다 로그 없이 조용히 실패해서 진단 로그(`rec.currentTime`·파일 크기 tick)를 넣어야 갈렸다.

1. **`stop()` 직후 `transferFile`은 잘린 파일을 보낸다.** `AVAudioRecorder.stop()`이 돌아와도 파일은 마무리 전(헤더·버퍼 flush 안 됨). 그 시점에 WCSession이 복사해 가면 아이폰엔 24,588바이트·`afinfo` 0.06초짜리가 도착한다 — 15초든 269초든 크기가 똑같은 것이 신호. **전송은 `audioRecorderDidFinishRecording(_:successfully:)` 델리게이트 뒤로 미룬다.** 시뮬레이터는 우연히 비슷한 크기가 나와 정상처럼 보였다.
2. **실기기 `AVAudioRecorder`의 AAC 인코더는 패킷을 전혀 쓰지 않는다.** 16kHz/32kbps도, 세션 샘플레이트(48kHz)+`AVEncoderAudioQualityKey`도 녹음 중 `fileSize=28`(헤더)에 고정되고 마무리 때 61,452바이트 쓰레기만 남는다(`audio=0.02s`). `rec.currentTime`은 5.0→10.0으로 전진하므로 입력 문제가 아니다. 시뮬레이터는 맥 인코더라 된다. **해법은 소프트웨어 코덱** — 코드엔 사다리를 두었다: IMA4 → µ-law → PCM16 16k → PCM16 세션레이트, 2초→5초 사이 파일이 안 자라면 다음 포맷으로 자동 재시작. IMA4는 `AVAudioRecorder` 생성에서 `NSOSStatusErrorDomain 1718449215`('fmt?')로 즉시 실패(시뮬레이터·실기기 동일), **µ-law 16kHz 모노(16KB/s, 30분 ≈ 29MB)가 실기기에서 최종 동작.** 용량이 아쉬우면 `AVAudioEngine` + 소프트웨어 AAC 변환이 다음 후보.
3. **watchOS는 `record()`가 마이크 권한 프롬프트를 띄우지 않는다.** 권한 미결정 상태에서도 `record()`는 `true`를 돌려주고 `currentTime`이 흐르지만 파일은 4,096바이트(헤더 한 페이지)에 고정된다(µ-law로 바꾼 뒤에도 그랬다). 시뮬레이터에선 `simctl privacy … grant microphone`으로 미리 줬기 때문에 안 보였다. **녹음 전에 `AVAudioApplication.requestRecordPermission()`을 명시 호출**하면 워치에 프롬프트가 뜨고, 그 뒤부터 파일이 초당 16KB씩 자란다.

판정 순서(다음에 워치 녹음이 "되는데 비어 있다"면): ① 권한 `AVAudioApplication.shared.recordPermission` ② 녹음 중 파일 크기 성장 ③ `didFinishRecording` 뒤 `AVAudioFile.length`로 실제 길이 — 이 셋을 5초 tick으로 로그에 찍는 것이 가장 빠르다.

### 실기기에서 확인된 것

- `transferFile` → 아이폰 `didReceive`는 실기기에서 몇 초 안에 배달된다(시뮬레이터 미지원과 대비). WCSession 활성화는 뷰 `onAppear`가 아니라 **앱 `init`에서** — iOS가 배달을 위해 앱을 백그라운드로 깨울 때 뷰가 안 뜨기 때문.
- 아직 미검증: 화면 꺼짐·손목 내림 30~60분 지속, 배터리.

## 기록

### 2026-09-02 — WristNote 실기기 M0: 잘린 파일 → AAC 무출력 → 마이크 권한, 세 겹

- 맥락: [[프로젝트/개인/WristNote/README|WristNote]] 홍의 실기기(Apple Watch watchOS 26.2 + iPhone iOS 26.5) 테스트. 전송은 됐는데 전사가 "빈 텍스트"로 실패해 `devicectl device copy from --domain-type appDataContainer`로 아이폰 컨테이너의 `meetings.json`·녹음 파일을 꺼내 분석.
- 배운 것: 위 「실기기 워치 녹음 — 세 가지」 전부. 순서대로 벗겨졌고 각 단계는 앞 단계를 고치기 전엔 보이지 않았다.
- 근거: `(로컬 경로)` 커밋 `9bcdaff`(전송을 didFinish 뒤로) → `fc903de`(AAC 48kHz 시도, 실패) → `44cd002`(코덱 사다리, µ-law) → `61aae30`(권한 요청). 홍이 붙여준 워치 콘솔: `finished … size=24588 audio=0.06s wall=12.61s`, `tick wall=10 … fileSize=28`, `tick … fileSize=4096` → 권한 요청 후 정상.

### 2026-09-01 — WristNote v1 골격 시뮬레이터 페어 검증

- 맥락: [[프로젝트/개인/WristNote/README|WristNote]] 야간 자율 작업. 워치 녹음 → `transferFile` → 아이폰 수신 흐름을 Series 11(46mm) + iPhone 17 Pro 시뮬레이터 페어에서 돌렸다.
- 배운 것: 위 「핵심 정리」 전부. 워치 녹음·전송 호출까지는 되고 아이폰 배달만 시뮬레이터 한계.
- 근거: `(로컬 경로)` 커밋 `f8c4bd3` (`WatchRecorder.swift`·`WatchSession.swift`·`PhoneSession.swift`). 워치 로그 `subsystem == "com.hong.wristnote.watch"`: `recording started inputs=1` → `stop: … exists=true size=25120` → `didFinish … error=nil`. 아이폰 `wcd` 로그 21:14:28 `received message … 28222 bytes`.

## 참고 자료

- [WCSession transferFile Not Working in Simulator — Apple Developer Forums](https://developer.apple.com/forums/thread/128205) — 시뮬레이터 미지원, 실기기 테스트 권고 (2026-09-01 확인)
- [WCSession transferFile not doing anything — Apple Developer Forums](https://developer.apple.com/forums/thread/733155) — 같은 증상(워치 didFinish 성공, 아이폰 didReceive 무호출), 실기기에선 정상 (2026-09-01 확인)
