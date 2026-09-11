---
type: study
area: AppStore
audience: ai
status: active
created: 2026-09-11
updated: 2026-09-11
projects:
  - "탭탭"
---

# 앱 이름 속 Apple 상표 (5.2.5)

탭탭 macOS 1.1.0 (6)이 **Guideline 5.2.5 - Legal - Intellectual Property**로 리젝됐다(2026-09-10, Submission `4d37df45-d9f5-44a9-9b54-4938e4a14c22`). 지적 문구는 한 줄이다:

> Terms for Mac in the app name that displays on the device.

## 핵심 — 심사는 스토어 등재명이 아니라 **기기에 표시되는 이름**을 본다 ★★

리젝 사유가 "metadata"라고 쓰여 있어서 App Store Connect의 앱 이름·부제·설명을 먼저 뒤지게 되는데, 실제로 걸린 것은 **설치된 `.app`의 이름**(Finder·Dock·메뉴바에 뜨는 것)이었다. 즉 **ASC 화면을 아무리 봐도 원인이 없다.** 확인은 아카이브 plist를 직접 읽어야 한다.

```bash
# macOS
/usr/libexec/PlistBuddy -c "Print :CFBundleDisplayName" "<Archive>/Products/Applications/<App>.app/Contents/Info.plist"
/usr/libexec/PlistBuddy -c "Print :CFBundleName"        "<Archive>/Products/Applications/<App>.app/Contents/Info.plist"
# iOS는 Contents/ 없이 <App>.app/Info.plist
```

**표시 이름의 우선순위는 `CFBundleDisplayName` → `CFBundleName` → 실행 파일명**이다. 그래서 `CFBundleDisplayName`이 **없으면 조용히 다음 것으로 내려가고**, `CFBundleName`의 기본값은 `$(PRODUCT_NAME)` = **타깃 이름**이다. 타깃 이름에 `Mac`·`iPhone`·`iPad`·`Watch`·`Apple`이 들어 있으면 그게 그대로 스토어 심사 대상이 된다.

금지되는 말은 Apple 제품·서비스명 전반이다 (`Mac`, `iPhone`, `iPad`, `Apple`, `Watch`, `AirPods`, `Safari`…). **`for Mac`·`Mac용`처럼 호환성을 알리는 표현조차 이름 안에서는 안 된다** — 그건 설명(description)에서 한다.

## 구조적 함정 — 플랫폼별로 갈려서 한쪽만 샌다 ★

탭탭에서 실제로 일어난 모양이다. Tuist 프로젝트에서 릴리스 설정은 `INFOPLIST_KEY_CFBundleDisplayName = "탭탭"`을 잘 넣고 있었는데:

| 번들 | 표시 이름 | 왜 |
|---|---|---|
| iOS 앱 | `탭탭` ✅ | `App/Project.swift`의 `infoPlist`에 `"CFBundleDisplayName": "$(INFOPLIST_KEY_CFBundleDisplayName)"` 한 줄이 있다 |
| 맥 Safari 확장 | `탭탭` ✅ | `MacSafariExtension/Info.plist`에 문자열로 직접 박혀 있다 |
| **macOS 앱** | **`TapTapMac`** ❌ | `TapTapMac/Project.swift`의 `.extendingDefault(with: [...])`에 그 매핑이 **없다** → 빌드 설정값이 버려지고 `CFBundleName = $(PRODUCT_NAME)`만 남는다 |

**빌드 설정에 이름을 넣어도 Info.plist에 매핑이 없으면 그냥 사라진다** — 에러도 경고도 없다. 같은 레포의 세 번들 중 둘은 한글로 나가고 하나만 영문 타깃명으로 나갔으니, **"우리 앱 이름은 탭탭이다"라고 믿고 있으면 영원히 안 보인다.**

### 덤으로 나온 것 — 릴리스 설정에 Dev 접미사가 박혀 있었다

`Target+Templates.swift`의 맥 분기가 `releaseSettings["INFOPLIST_KEY_CFBundleDisplayName"] = "\(name)Dev"` 였다(디버그에 들어갈 줄이 릴리스에 있었다). 지금은 매핑이 없어서 드러나지 않지만, **매핑만 추가하면 다음 빌드가 `TapTapMacDev`로 나간다** — `Mac`도 `Dev`도 그대로다. 두 곳을 같이 고쳐야 한다.

## 고칠 때 건드리면 안 되는 것

애플 메시지가 직접 경고한다: **`CFBundleIdentifier`는 바꾸지 않는다.** 번들 ID를 바꾸면 업데이트가 아니라 **새 앱**이 되어 기존 사용자가 업그레이드를 못 받는다. 표시 이름만 고치는 일이고, 타깃 이름(`TapTapMac`)도 소스 구조라 바꿀 필요가 없다 — **`CFBundleDisplayName`을 넣어 덮으면 끝난다.**

## 예방 — 제출 전 한 줄 점검

```bash
# 아카이브 안의 모든 번들 표시 이름을 한 번에 본다
find "<Archive>/Products" -name Info.plist | while read p; do
  printf '%s\t%s\t%s\n' "$p" \
    "$(/usr/libexec/PlistBuddy -c 'Print :CFBundleDisplayName' "$p" 2>/dev/null)" \
    "$(/usr/libexec/PlistBuddy -c 'Print :CFBundleName' "$p" 2>/dev/null)"
done
```

영문 타깃명이 하나라도 보이면 멈춘다. **TestFlight를 6번 올리는 동안 아무도 못 잡았다** — 내부 테스터는 앱 이름이 영문인 걸 당연하게 보기 때문이다.

## 기록

### 2026-09-11 — 탭탭 macOS 1.1.0 (6) 리젝 원인 규명
제출 전 점검 문서(출시 준비)가 예측한 1순위는 **2.3.7(앱 이름에 설명이 붙음 — `탭탭-글 읽기 부터 스크랩까지`)** 이었는데 **그건 지적되지 않았고**, 아무도 보지 않던 **번들 표시 이름**이 걸렸다. 근거: 제출된 아카이브(`(로컬 경로)`)에서 `CFBundleDisplayName` 부재·`CFBundleName = TapTapMac` 확인, 같은 아카이브의 확장(`탭탭`)과 iOS 아카이브(`탭탭`)와 대조.

**교훈: 리젝 예측은 스토어 등재값만 훑어서는 반쪽이다.** 기기에 설치되는 쪽(표시 이름·아이콘·번들)도 같은 목록에 올려야 한다.
