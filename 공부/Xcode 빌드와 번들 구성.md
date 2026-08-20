---
type: study
area: 도구·인프라
status: active
created: 2026-08-20
updated: 2026-08-20
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# Xcode 빌드와 번들 구성

무엇이 앱 번들에 들어가고 무엇이 빠지는가 — 그리고 **정말 빠졌는지 어떻게 증명하는가**.

## 핵심 정리

- Xcode 16의 **동기화 폴더**(`fileSystemSynchronizedGroups`)는 폴더 안의 파일을 전부 타깃에 넣는다. 개별 파일을 빼려면 `PBXFileSystemSynchronizedBuildFileExceptionSet`을 만들어 `membershipExceptions`에 파일명을 적고, 그 폴더 그룹의 `exceptions`에서 참조한다. 이름과 달리 **"예외 = 제외"** 다.
- 리소스를 번들에서 빼는 것과 코드를 죽이는 것은 별개다. 둘 다 해야 실제로 안 나간다.
- **릴리즈 빌드의 데드 코드 스트리핑을 검증 도구로 쓸 수 있다.** `static let flag = false`로 분기를 막으면 옵티마이저가 그 분기를 통째로 걷어내므로, 릴리즈 바이너리의 `strings`에서 그 분기에만 쓰이던 문자열이 사라진다. 살아 있어야 할 문자열 하나를 대조군으로 같이 확인하면 "검색이 실패한 것"과 "정말 없는 것"이 구분된다.

## 기록

### 2026-08-20 — 기능을 코드를 지우지 않고 출시에서 빼기

- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] 1.7.0에서 데스크톱 펫을 빼고 제출해야 했다. 코드는 남기고 싶었다 — 캐릭터 설계만 다시 잡으면 되돌릴 것이라서.
- 배운 것:
  - **게이트는 가장 안쪽에도 하나 둔다.** 메뉴 항목을 안 만드는 것만으로는 부족했다. 개발 빌드에서 켜 뒀던 `UserDefaults` 값이 남아 있으면 출시판에서 창이 떠버린다. `isOn` 게터 자체를 `flag && defaults`로 바꿔서 저장된 설정과 무관하게 꺼지게 했다.
  - **동기화 폴더에서 파일 하나 빼기**는 pbxproj를 손으로 고쳐야 한다. Xcode 16 형식이라 `PBXFileSystemSynchronizedBuildFileExceptionSet { membershipExceptions = ("zappy-snowman.usdz"); target = <Zappy> }`를 새로 만들고, `AppShell` 동기화 루트 그룹에 `exceptions = (그 세트)`를 달았다. 원래 이 절엔 "위젯 타깃에서 앱 전용 소스를 뺀다"는 용례만 있었는데, **타깃이 하나뿐인 폴더에 걸면 = 아무 데도 안 들어간다**로 쓸 수 있다.
  - **검증은 산출물에서 한다.** 번들에 usdz가 없는지 `find`, 펫 메뉴가 안 만들어지는지 릴리즈 바이너리 `strings`. 후자가 이번의 수확 — `Desktop Pet`·`Keep on Top`은 사라졌고 `Restore Purchases`는 남아 있어서, grep이 헛돈 게 아니라 코드가 정말 빠졌다는 뜻이 됐다. 디버그 빌드로는 이게 안 된다(문자열이 `Zappy.debug.dylib`에 있고 최적화도 안 된다).
- 근거: 커밋 `e666e7d` (`Sources/CuteBattery/DesktopPet.swift`의 `featureEnabled`, `Zappy.xcodeproj/project.pbxproj`, `package-app.sh`). 번들 988KB(usdz 400KB 미포함).
