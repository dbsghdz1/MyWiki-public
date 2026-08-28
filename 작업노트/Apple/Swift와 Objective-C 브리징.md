---
type: study
area: Apple
audience: ai
status: active
created: 2026-08-18
updated: 2026-08-18
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
---

# Swift와 Objective-C 브리징

`@objc`를 붙인 Swift 메서드의 ObjC 셀렉터 이름은 **Swift 이름 + 인자 레이블에서 기계적으로 생성**된다. AppKit/UIKit이 이름으로 직접 호출하는 콜백에서는 이 생성 규칙과 프레임워크가 기대하는 셀렉터가 어긋날 수 있고, 그때 실패는 **크래시도 경고도 없이 조용하다**.

## 핵심 정리

- **셀렉터 이름 생성 규칙**: `func mouseEntered(with e: NSEvent)`에 `@objc`만 붙이면 셀렉터는 `mouseEntered:`가 아니라 **`mouseEnteredWith:`** 가 된다. 첫 인자 레이블이 이름 뒤에 대문자로 붙는다. 마찬가지로 `foo(with:)` → `fooWith:`, `foo(at:)` → `fooAt:`.
- **오버라이드·프로토콜 요구사항이면 안전하다.** 상위 ObjC 선언의 셀렉터를 그대로 물려받기 때문이다. `class V: NSResponder { override func mouseEntered(with:) }`는 `mouseEntered:`가 된다. 위험한 것은 **NSObject 직계 클래스**를 프레임워크 콜백의 수신자(owner·target·delegate)로 넘길 때다 — 물려받을 선언이 없으니 생성 규칙이 그대로 적용된다.
- **`#selector(...)`로 연결하는 target/action은 이 함정에 안 걸린다.** 컴파일러가 Swift 이름 → 셀렉터 변환을 해주고 존재 여부까지 검증한다. 위험한 것은 **프레임워크가 이름을 하드코딩해 보내는 경로**뿐이다: `NSTrackingArea(owner:)`, 비공식 프로토콜(informal protocol) 콜백, `perform(_:)`, KVC/KVO, target-action을 문자열로 만드는 코드.
- **해결은 셀렉터 명시**: `@objc(mouseEntered:) func mouseEntered(with e: NSEvent)`. 또는 수신자를 해당 메서드를 가진 클래스(NSResponder 등)의 서브클래스로 만들어 `override`로 선언한다.
- **왜 조용히 실패하나**: AppKit은 보내기 전에 `respondsToSelector:`로 거른다. 응답하지 않으면 그냥 아무 일도 일어나지 않는다 — 로그도 크래시도 없다. "코드는 다 맞는데 기능만 안 되는" 증상이면 셀렉터 이름을 먼저 의심한다.
- **검증하는 두 가지 방법** (추측하지 말고 확인할 것):
  - 런타임: `obj.responds(to: NSSelectorFromString("mouseEntered:"))` — 20줄짜리 스크립트로 선언 변형별 true/false를 바로 찍어볼 수 있다.
  - 빌드 산출물: `strings -a <바이너리> | grep -E "^mouseEntered"` — 셀렉터 문자열이 그대로 들어 있어, **이미 배포된 빌드**가 어느 이름으로 나갔는지도 사후 확인된다.
- `@objc` 추론은 Swift 4(SE-0160)부터 크게 줄었다 — NSObject 서브클래스라고 자동으로 `@objc`가 붙지 않는다. 그래서 프레임워크 콜백은 대부분 명시적으로 `@objc`를 쓰게 되는데, **`@objc`를 붙였다는 사실이 "이름까지 맞다"를 보장하지 않는다**는 게 핵심이다.

## 기록

### 2026-08-18 — 호버 하트가 한 번도 동작하지 않은 채 출시된 이유
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] 1.6.0에 넣은 "메뉴바 아이콘에 마우스를 올리면 하트가 뜬다" 기능이 실기에서 전혀 동작하지 않았다. 스토어 릴리즈 노트에 5개 언어로 광고된 상태였다.
- 배운 것:
  - `AppDelegate`(NSObject 직계)에 `@objc func mouseEntered(with:)`를 두고 `NSTrackingArea(owner: self)`로 연결했는데, 셀렉터가 `mouseEnteredWith:`로 나가 AppKit이 보내는 `mouseEntered:`와 어긋났다. tracking area 등록도, 하트 그리기도 전부 정상이었고 **호출만 없었다**.
  - 코드 리뷰로는 잘 안 잡힌다 — `@objc`가 붙어 있고 메서드 이름도 눈으로 보기엔 맞기 때문이다. 위 두 가지 검증(responds(to:) / strings)이 훨씬 빠르다.
- 근거: 재현 스크립트로 `Current(NSObject)` = `mouseEntered:` false / `mouseEnteredWith:` true, `@objc(mouseEntered:)` 명시본과 `NSResponder` 상속본은 반대로 나오는 것을 확인. 출시된 1.6.0 바이너리에서 `strings | grep`으로 `mouseEnteredWith:`만 존재함을 확인. 수정 커밋 `f7e0889`(`@objc(mouseEntered:)` / `@objc(mouseExited:)`), 1.6.1 핫픽스로 제출. 작업 기록 [[프로젝트/개인/Zappy/Zappy 개발 기록 2026-08-18|Zappy 개발 기록 2026-08-18]].

## 참고 자료

- [SE-0022 — Referencing the Objective-C selector of a method](https://raw.githubusercontent.com/swiftlang/swift-evolution/main/proposals/0022-objc-selectors.md) — `#selector`가 Swift 이름을 셀렉터로 변환하는 방식과 `@objc(name)`으로 임의의 ObjC 이름을 지정하는 근거 (2026-08-18 확인)
- [SE-0160 — Limiting @objc inference](https://raw.githubusercontent.com/swiftlang/swift-evolution/main/proposals/0160-objc-inference.md) — Swift 4에서 `@objc` 자동 추론이 줄어든 배경. Swift API 가이드라인대로 지으면 ObjC 관례에 안 맞는 이름이 나오므로 명시적 `@objc(...)`가 필요하다고 명시 (2026-08-18 확인)
- [SE-0021 — Naming Functions with Argument Labels](https://raw.githubusercontent.com/swiftlang/swift-evolution/main/proposals/0021-generalized-naming.md) — 인자 레이블까지 포함한 전체 이름으로 메서드를 가리키는 문법(`foo(_:with:)`). 셀렉터 표기의 토대 (2026-08-18 확인)
