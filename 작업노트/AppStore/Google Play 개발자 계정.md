---
type: study
area: AppStore
audience: ai
status: active
created: 2026-08-26
updated: 2026-09-01
projects:
  - "[[프로젝트/개인/한능검/README|한능검]]"
---

# Google Play 개발자 계정

**Android 첫 출시의 병목은 코드가 아니라 계정 유형이다.** 개인 계정으로 만들면 신규 앱마다 **테스터 12명 × 14일 연속 비공개 테스트**를 통과해야 출시할 수 있고, 조직(사업자) 계정이면 면제된다. 그런데 조직 계정에는 **D-U-N-S 번호가 필수이고 발급에 최대 30일**이 걸린다 — **코드를 짜기 전에 신청해둬야 하는 유일한 항목**이다.

## 핵심 정리

| | 개인 계정 | 조직(사업자) 계정 |
|---|---|---|
| 12명 × 14일 비공개 테스트 | **필수** (2023-11-13 이후 생성 계정) | **면제** |
| D-U-N-S 번호 | 불필요 | **필수** |
| 준비 기간 | 즉시 | **최대 30일** (D-U-N-S 발급) |
| 개인사업자로 가능한가 | — | **가능** (법인 아니어도 됨) |

- **12명 × 14일은 숫자 채우기가 아니다.** 테스터가 "최근 14일 이상 지속적으로 참여하겠다고 선택한 상태"여야 하며, 설치 후 방치가 아니라 연속 참여로 판정된다. 실제로는 사람을 모으는 것 자체가 병목이 된다.
- **D-U-N-S는 무료다.** 국내 발급기관은 **NICE D&B**이고, **90일 이내 발급된 영문 사업자등록증명**이 필요하다(홈택스에서 별도 발급). 개인·법인 사업자 모두 발급 가능하다.
- **순서가 중요하다** — D-U-N-S 신청 → 조직 계정 등록 → 앱 개발. 반대로 하면 앱을 다 만들고 30일을 기다리거나, 테스터 12명을 구하러 다녀야 한다.

> [!danger] 개인사업자는 이 번호를 Apple에 쓸 수 없다 (2026-08-26 정정)
> 처음엔 *"한 번 받아두면 애플·구글 양쪽에 쓴다"*라고 적었는데 **틀렸다.** Apple 공식 문서는 개인사업자를 Organization에서 명시적으로 배제한다.
> > *"If your legal status is a sole proprietorship/single person business, **enroll as an individual**."*
> Organization으로 받는 형태는 **Corporation · Limited Partnership · LLC**뿐이고, **sole proprietorship · DBA · 상호명 · 지점은 불가**다. 그리고 Individual 등록에는 D-U-N-S가 **애초에 필요 없다.**
> **즉 개인사업자에게 D-U-N-S는 Google Play 전용 투자다.** Apple 쪽에서 회수되지 않는다.
> → [Apple — D-U-N-S 번호](https://developer.apple.com/help/account/membership/D-U-N-S) (2026-08-26 확인)

## 사업자가 있어도 D-U-N-S를 안 받는 이유

받을 수 있는데도 안 받는 사람이 많다. 이유가 다섯 개고, **대부분 합리적이다.**

1. **개인 계정으로도 출시가 된다.** D-U-N-S 없이 앱 등록·출시가 가능하고, 12명 × 14일만 한 번 통과하면 된다. *"D-U-N-S가 없으면 구글 플레이 등록 자체가 불가능"*은 널리 퍼진 오해다.
2. **조직 계정은 웹사이트를 요구한다.** 계정 생성 필수 정보에 **조직 웹사이트**가 들어 있고, 실무상 **도메인이 연결된 홈페이지**만 인정된다 — 블로그·SNS 채널은 안 된다. 앱만 만드는 1인 개발자에게는 이게 D-U-N-S보다 큰 벽이다.
3. **최대 30일이 걸린다.** 출시를 앞두고 알게 되면 못 기다린다.
4. **이름이 세 군데에서 정확히 일치해야 한다.** ① 개발자 계정의 사업자명 ② 사업자등록증의 사업자명 ③ D-U-N-S에 등록된 사업자명. 영문 표기가 조금만 달라도 검증에서 막히고, 고치려면 D&B 쪽 수정을 또 기다린다.
5. **Apple에서는 회수가 안 된다** (개인사업자 한정). 위 경고 참고 — 애플은 개인사업자를 Individual로만 받고 거기엔 D-U-N-S가 필요 없다. "양쪽에 쓴다"는 기대가 깨지면 동기의 절반이 사라진다.

> [!tip] 그래서 언제 받을 가치가 있나
> **① 도메인 있는 웹사이트가 이미 있거나 생길 예정이고 ② 앱을 한 번이 아니라 계속 낼 계획이면** 받는 게 낫다. 12명 × 14일은 **신규 앱마다** 반복되는 비용인데, D-U-N-S는 한 번이다.
> 반대로 앱 하나만 내고 말 거라면 개인 계정으로 12명을 모으는 쪽이 빠르다.

> [!tip] iOS만 해본 사람이 놓치는 지점
> App Store는 개인이든 조직이든 심사 절차가 같아서 **"계정 유형이 출시 가능 여부를 가른다"는 감각이 없다.** Google Play는 다르다. 계정을 만드는 시점의 선택이 이후 모든 앱의 출시 조건을 결정한다. **개인→조직 전환은 공식 절차가 있다**(아래 2026-09-01 정정). 조직→개인은 불가.

## 기록

### 2026-08-26 — 웹 + 양대 스토어 출시를 검토하다 D-U-N-S 30일을 발견했다 (같은 날 일부 정정)

- 맥락: [[프로젝트/개인/한능검/README|한능검]] 학습 앱을 웹으로 만들고 iOS·Android 양쪽에 내는 구조를 검토하면서, Android 출시 요건을 처음 확인했다. 홍은 iOS·macOS 출시 경험만 있고 **Google Play는 처음**이다.
- 배운 것:
  - **개인 계정이면 신규 앱마다 12명 × 14일 비공개 테스트가 필수다.** 2023-11-13 이후 생성된 개인 계정에 적용되며 2026년 기준 그대로다. 이건 심사가 아니라 **출시 자격 요건**이라 잘 만든다고 면제되지 않는다.
  - **조직 계정은 면제되고, 개인사업자로도 조직 계정을 만들 수 있다.** 법인일 필요가 없다.
  - **대신 조직 계정에는 D-U-N-S 번호가 필수이고 발급에 최대 30일이 걸린다.** 이게 실질적인 리드타임이다 — 무료지만 시간이 든다.
  - **결론: 계정 준비가 개발보다 먼저다.** 스토어 출시 의사가 조금이라도 있으면 D-U-N-S를 먼저 신청해두고 코드를 짠다. 병렬로 굴리면 비용이 0이고, 순차로 하면 한 달을 잃는다.
- 근거: [Play Console — 새로운 개인 개발자 계정의 앱 테스트 요구사항](https://support.google.com/googleplay/android-developer/answer/14151465?hl=ko) · [Play Console — 개발자 계정 유형 선택](https://support.google.com/googleplay/android-developer/answer/13634885?hl=ko) (둘 다 2026-08-26 확인)

### 2026-09-01 — DUNS 없이 조직 계정은 못 만들지만, 개인으로 결제하고 나중에 조직으로 전환할 수 있다 (08-26 기록 정정)

- 맥락: HSW — 홍이 DUNS를 08-27 신청(미발급)한 상태에서 **9월 4일까지 Play Console 등록비를 결제해야** 하는 상황. "조직 계정을 DUNS 없이 먼저 결제만 할 수 있나?"가 질문이었다.
- 배운 것:
  - **조직 계정은 DUNS 없이 생성 자체가 안 된다.** 공식 원문: *"You will not be able to create a developer account for an organization without one."* 결제 프로필을 조직으로 만드는 단계에서 DUNS 입력이 필수다.
  - **하지만 개인 계정으로 만든 뒤 조직으로 전환하는 공식 절차가 있다** — 08-26의 *"나중에 바꾸는 것은 간단하지 않다"*는 틀렸다. 절차: 계정 소유자가 ① **조직 웹사이트를 먼저 등록·검증**(이게 안 되면 전환 옵션이 안 열린다: *"Before changing your account type, you need to provide and verify your official organization website."*) → ② 새 **조직용 결제 프로필** 생성(DUNS 입력, 법적 이름·주소는 D&B 프로필과 일치) → ③ 검증 후 Play Console **About you → Link your payments profile to your developer account**. 전환 후 새 앱 제출은 **72시간 대기** 권고.
  - **역방향(조직→개인)은 불가** — *"we do not support changing an account from organization to individual."* 그래서 마감이 있으면 개인으로 먼저 결제하는 쪽이 안전하다.
  - **기존 결제 프로필에서는 국가·계정 유형·DUNS를 수정할 수 없다.** 새 프로필을 만들어 링크하는 방식이다.
  - **전환 시 쓸 Google 계정은 처음부터 사업 전용으로.** 전환은 "account owner"만 할 수 있고, 소유자 변경은 별도 절차다.
  - 미확인: 전환 후 **기존 개인 계정 시절의 12명×14일 요건이 풀리는지**는 문서에 명시가 없다. 첫 앱 만들기 전에 전환을 끝내는 게 안전.
- 근거: [Update developer identity details managed by a Google payments profile](https://support.google.com/googleplay/android-developer/answer/16260648?hl=en) · [Keeping your developer account information up to date](https://support.google.com/googleplay/android-developer/answer/13634888?hl=en) · [Required information to create a Play Console developer account](https://support.google.com/googleplay/android-developer/answer/13628312?hl=en) — 모두 2026-09-01 확인

### 2026-09-02 — 발급됐다는 DUNS가 다른 회사의 레코드였다 (미해결)

- 맥락: HSW — 08-27 NICE D&B에 신청한 DUNS가 09-02 "D&B 애플 고객 지원" 메일로 **963252083 발급 완료**로 통보됐다(사건번호 34798515). 그런데 통보서에 적힌 회사 정보가 홍의 사업자등록증과 **구 단위로** 달랐다.
- 대조:

  | | 통보된 DUNS 레코드 | 홍의 사업자등록증 |
  |---|---|---|
  | 회사명 | HSW 주식회사 | (개인사업자 상호) |
  | 법적 구조 | **주식회사** | **개인사업자** |
  | 주소 | 서울 **마포구** ○○로○길 (도로명이 깨진 채로 옴) | 서울 **관악구** ○○로○길 |

- 배운 것:
  - **D&B는 신청 내용으로 새 레코드를 만드는 대신 기존 레코드에 매칭할 수 있다.** 구·도로명·번지·법적 구조가 전부 다르면 표기 오류가 아니라 **다른 엔티티**다. 발급 통보를 받으면 번호를 어디에도 입력하기 전에 **회사명·법적 구조·주소 3개를 사업자등록증과 대조**한다.
  - **통보서의 한글 주소는 역번역이라 도로명이 깨져서 온다.** 적혀 온 도로명이 실존하지 않길래 한 글자씩 되짚으니 다른 구의 실존 도로명이었다 — 한글→영문 로마자→한글로 돌면서 깨진 것이다. 즉 그 레코드는 **실재하는 남의 주소**를 가리킨다. 주소가 이상하면 먼저 역번역을 되돌려보고, 그래도 내 주소가 아니면 그때 다른 레코드로 판정한다.
  - **개인사업자인데 법적 구조가 Corporation으로 잡히면 Play 조직 계정에서 막힌다.** 위 "이름이 세 군데에서 정확히 일치해야 한다" 항목의 실사례다 — 이름뿐 아니라 **법적 구조도 일치 대상**이다.
  - **D&B 통보 메일에 회신하면 반송된다.** 발신이 `noreply@dnb.com`이라 답장이 `554 5.2.2 mailbox full`로 튕겼다 — 사유가 "메일함 꽉 참"인 게 오히려 증거다. 아무도 안 비우는 수신 전용 주소라 꽉 찰 때까지 방치된 것이고, 재시도해도 읽히지 않는다. 같은 패턴으로 **`applecs@dnb.com`도 2019-01-10 폐쇄됐는데 아직 D&B 메일 본문에 실려 나온다** — D&B 메일에 적힌 주소는 접수구가 아니다.
  - **정식 접수구는 웹 폼과 국내 발급기관 둘이다.** ① [D&B Apple 지원 포털](https://support.dnb.com/?CUST=APPLEDEV) ② NICE D&B(국내 발급기관, 한국어). 메일 제목의 `[ ref:...:ref ]`는 Salesforce Email-to-Case 토큰이라, 폼에 넣을 때 **사건번호와 이 토큰을 제목에 유지**하면 같은 케이스에 붙는다.
- 대응: 사건번호 34798515로 회신해 ① 이 레코드는 내 사업자가 아님 ② 사업자등록증대로(개인사업자·소재지) 정정 또는 신규 발급을 요청. **번호 963252083은 어디에도 입력하지 않는다.** 결과는 회신 오면 이어서 기록.
- 여파 없음: iOS는 Individual 등록이라 DUNS가 필요 없고, Play는 개인 계정으로 9/4 결제를 이미 마쳤다 — 조직 전환만 밀린다.

## 참고 자료

- [Play Console — 개발자 계정 유형 선택](https://support.google.com/googleplay/android-developer/answer/13634885?hl=ko) — 2026-08-26 확인
- [Play Console — 새로운 개인 개발자 계정의 앱 테스트 요구사항](https://support.google.com/googleplay/android-developer/answer/14151465?hl=ko) — 2026-08-26 확인
- [Play Console — 개발자 계정 생성에 필요한 정보](https://support.google.com/googleplay/android-developer/answer/13628312?hl=ko) — 2026-08-26 확인
- [Apple — D-U-N-S 번호](https://developer.apple.com/help/account/membership/D-U-N-S) — 개인사업자의 Organization 등록 불가 근거, 2026-08-26 확인
- [NICE D&B — 던스번호 발급](https://global.nicednb.com/servOtsInfo.do) — 국내 유일 발급기관, 2026-08-26 확인
- [Play Console — Update developer identity details managed by a Google payments profile](https://support.google.com/googleplay/android-developer/answer/16260648?hl=en) — 개인→조직 전환 절차(웹사이트 검증 → 조직 결제 프로필 → 링크), 조직→개인 불가, 2026-09-01 확인
- [Play Console — Keeping your developer account information up to date](https://support.google.com/googleplay/android-developer/answer/13634888?hl=en) — 결제 프로필의 국가·계정 유형·DUNS는 수정 불가, 새 프로필 생성 후 링크, 2026-09-01 확인
