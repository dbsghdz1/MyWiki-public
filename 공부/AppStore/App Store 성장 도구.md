---
type: study
area: 도구·인프라
audience: ai
status: active
created: 2026-08-19
updated: 2026-08-19
projects:
  - "[[프로젝트/개인/Zappy/README|Zappy]]"
  - "[[프로젝트/개인/BarStack/README|BarStack]]"
---

# App Store 성장 도구

**Mac 전용 앱은 App Store의 성장 도구 대부분을 쓸 수 없다.** 광고(Apple Ads)·인앱 이벤트·커스텀 제품 페이지가 전부 iPhone/iPad 앱 전용이고, Mac 앱에 남는 것은 **피처링 노미네이션과 제품 페이지 문구**뿐이다.

## 핵심 정리

### Mac 전용 앱에서 쓸 수 없는 것

| 도구 | Mac 앱 | 근거 |
|---|---|---|
| **Apple Ads (구 Apple Search Ads)** | ❌ | **애초에 iOS/iPadOS 앱을 광고하는 채널이다** — 광고 지면 4종(Today 탭·Search 탭·검색 결과·제품 페이지)이 전부 iPhone/iPad의 App Store 화면이고, Mac App Store는 지면에 없다. 계정 요건 "To set up an Apple Ads account, you must have an app for iPhone or iPad currently on the App Store"는 그 사실의 **결과**다. 즉 계정을 만들어도 **Mac 앱은 캠페인 대상이 될 수 없다** |
| **인앱 이벤트 (In-App Events)** | ❌ | 이벤트 카드는 "iOS 15 and iPadOS 15 and later"에만 노출. ASC 노미네이션 도움말도 "In-App Events can only be attached to iPhone and iPad apps"로 못 박는다 |
| **커스텀 제품 페이지 (Custom Product Pages)** | ❌ | "up to 70 additional versions of your product page ... **for iPhone and iPad**" |

→ **"시즌 이벤트를 앱 이벤트로 알린다"는 iOS 앱의 플레이북이다.** Mac 앱에 그대로 옮길 수 없다.

> [!note] 근거 수준 (2026-08-19)
> "Mac 앱은 Apple Ads의 캠페인 대상이 될 수 없다"를 **한 문장으로 부정하는 Apple 공식 문서는 찾지 못했다.** 근거는 둘의 조합이다 — ① 계정 요건이 iPhone/iPad 앱을 명시(원문 확인) ② 광고 지면 설명이 전부 App Store의 iOS/iPadOS 화면 기준이고 Mac App Store 언급이 없다(딥링크는 "iOS 18 and later"). 실무 결론은 바뀌지 않지만, **"Apple이 명시적으로 금지했다"가 아니라 "지면이 iOS/iPadOS뿐이라 성립하지 않는다"가 정확한 진술**이다.

### Mac 전용 앱에도 되는 것

- **피처링 노미네이션 (Featuring Nominations)** — ASC의 앱 → 사이드바 `Featuring` → `Nominations`. 유형은 **New Content / App Enhancements / App Launch** 셋이고, 플랫폼 선택에 **macOS가 포함된다**("Most nominations work across all platforms (iOS, iPadOS, macOS, tvOS, visionOS, watchOS)"). 필수 입력은 이름·유형·상세 설명·게시 희망 시점·플랫폼 최소 1개. **리드타임 최소 3주**, 넓은 노출을 노리면 더 일찍. CSV 일괄 업로드도 되는데 CSV는 **초안 없이 즉시 제출**되므로 주의. Account Holder/Admin/App Manager/Marketing 역할 필요.
- **프로모션 텍스트(170자)** — 새 빌드·심사 없이 언제든 교체된다. Mac 앱에서 "시즌 이벤트 배너" 역할을 대신할 수 있는 유일한 상시 문구 슬롯.
- **새 소식(What's New)**, 스크린샷·설명·키워드, 오퍼 코드, 앱 내 이벤트가 아닌 **버전 출시 자체**.

### 그래서 Mac 앱의 유료 광고 판단

- Apple Ads가 막혀 있으므로 남는 유료 채널은 **Meta 등 일반 광고뿐인데, 여기서도 "앱 설치" 캠페인 목표를 못 쓴다** — 앱 설치 최적화는 iOS/Android 스토어 대상이라, Mac 앱은 랜딩/스토어 링크로 보내는 **트래픽·전환 캠페인**밖에 안 된다. 설치 이벤트를 광고 플랫폼에 돌려줄 방법이 없으니 최적화도 리타게팅도 성립하지 않는다.
- 즉 Mac 전용 앱에서 "전환율이 낮을 것 같아서 유료 광고를 안 한다"가 아니라, **효율적인 유료 채널이 애초에 존재하지 않는다**가 더 정확한 진술이다. 예산 판단이 아니라 구조 판단이다.
- 외부 1인 개발 사례에서도 무료 채널(블로그·지식인)만으로 매출이 났다는 보고가 있다 — [[_wiki/1인 개발 앱 수익화|1인 개발 앱 수익화]]. **다만 그쪽은 선택이고 여기는 구조적 강제라는 점이 다르다.**

## 기록

### 2026-08-19 (2) — "계정이 막힌 것"이 아니라 "광고 대상이 아닌 것"
- 맥락: 홍보 자동화 Phase 4를 "iOS 앱을 하나 내면 되살아난다"로 정리했다가 사용자 지적으로 정정했다 — "Apple Search Ads는 iOS/iPad에서만 가능하고 Zappy는 macOS라서 불가능한 것뿐".
- 배운 것: **제약을 어느 층위에서 진술하느냐가 다음 행동을 바꾼다.** "계정 개설 요건이 막혔다"로 적으면 해결책이 *계정 만들기*(= iOS 앱 아무거나 출시)로 보이지만, 실제 제약은 **광고 지면이 iOS/iPadOS뿐이라 macOS 앱은 캠페인 대상이 될 수 없다**는 것이다. 이 층위로 적으면 DayTune을 출시해도 Zappy·BarStack은 여전히 광고할 수 없다는 게 바로 보인다. **요건은 원인이 아니라 결과였다.**
- 일반화: 플랫폼 제약을 기록할 때는 **관문(gate)이 아니라 그 관문이 존재하는 이유**를 적는다. 관문만 적으면 "관문을 통과하면 된다"는 잘못된 해결책이 따라온다.
- 근거: 광고 지면 4종의 플랫폼 서술(2026-08-19 확인, 아래 참고 자료) + 계정 요건 원문. 단 위 `근거 수준` 노트대로 Apple이 명시적으로 부정한 문장은 없다.

### 2026-08-19 — 홍보 피드백("Apple Search·Meta 정도, 아니면 앱 이벤트·피처링만") 검증
- 맥락: [[프로젝트/개인/Zappy/README|Zappy]] 1.6 런치 실행 중 받은 외부 홍보 피드백 — "유료는 Apple Search, Meta 정도가 맞다. 전환율이 높을 것 같지 않으면 유료 마케팅을 안 하는 것도 방법(앱 이벤트·피처링만 진행)". 결론(유료 스킵)은 타당했지만 **거명된 수단 3개 중 2개가 Mac 앱엔 존재하지 않는 옵션**이었다.
- 배운 것: 위 표 그대로. Apple Ads는 계정 개설 요건에서, 인앱 이벤트·커스텀 제품 페이지는 노출 플랫폼에서 Mac이 빠진다. 남는 실행 수단은 **피처링 노미네이션 + 프로모션 텍스트 + 버전 출시**.
- 적용: Zappy는 유료 광고 미집행 확정, 시즌 테마 캘린더를 **피처링 노미네이션 캘린더**로 바꿔 붙였다(각 시즌 출시 최소 3주 전 New Content 노미네이션). [[프로젝트/개인/Zappy/Zappy 마케팅 플랜|마케팅 플랜]] · [[프로젝트/개인/Zappy/Zappy 로드맵과 유료화 계획|로드맵]].
- 근거: 아래 참고 자료 4건 전부 2026-08-19 WebFetch로 본문 확인.

## 참고 자료

- [Apple Ads — Solve setup and access issues](https://ads.apple.com/app-store/help/get-started/0052-solve-setup-and-access-issues) — 계정 개설 요건 원문("an app for iPhone or iPad") (2026-08-19 확인)
- [Apple Ads — Ad Placement Options](https://ads.apple.com/app-store/help/ad-placements/0081-ad-placement-options) — 광고 지면 4종과 가용 지역·딥링크("iOS 18 and later"). **Mac App Store는 지면 목록에 없다** (2026-08-19 확인)
- [Ads on the App Store — Apple Ads](https://ads.apple.com/app-store/) — 지면 4종 소개, 설명·스크린샷이 전부 iPhone 기준 (2026-08-19 확인)
- [In-App Events — Apple Developer](https://developer.apple.com/app-store/in-app-events/) — 노출 플랫폼·기간(최대 31일)·동시 게시 10개 등 규격 (2026-08-19 확인)
- [Overview of In-App Events — App Store Connect Help](https://developer.apple.com/help/app-store-connect/offer-in-app-events/overview-of-in-app-events/) — "iOS 15 and iPadOS 15 and later" 명시 (2026-08-19 확인)
- [Nominate your app for featuring — App Store Connect Help](https://developer.apple.com/help/app-store-connect/manage-featuring-nominations/nominate-your-app-for-featuring/) — 노미네이션 절차·유형·플랫폼(macOS 포함)·리드타임 3주 (2026-08-19 확인)
- [Custom Product Pages — Apple Developer](https://developer.apple.com/app-store/custom-product-pages/) — "for iPhone and iPad" (2026-08-19 확인)
