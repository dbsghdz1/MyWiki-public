---
type: project
title: "Zappy"
summary: "배터리 상태를 캐릭터로 보여주는 macOS 메뉴바 앱의 작업 진입점"
status: active
aliases:
  - Zappy
  - CuteBattery
  - 귀여운 배터리
created: 2026-07-23
updated: 2026-10-01
slack_channel: 22-개인-zappy
repos:
  - "github.com/dbsghdz1/Zappy"
related_wiki: []
---

# Zappy — 배터리가 표정이 되는 메뉴바 앱

## 현재 카드
- **단계**: 운영
- **현재**: **1.17.0 (build 27) 제출, WAITING_FOR_REVIEW**(10-01 16:12 KST, `asc state` 5로케일 × 5장 확인) — 펫 2D + 배터리 러너웨이(아이콘 옆 남은 시각·충전기 W·리포트 무료) + 너구리 삭제(18종) + 스토어 이름 5로케일 변경. main `6b34beb`. 1.15.0 READY_FOR_SALE — [개발 기록 10-01](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-10-01.md) · [심사 이력](App%20Store%20%EC%8B%AC%EC%82%AC%20%EC%9D%B4%EB%A0%A5.md) · [ASO 설계](Zappy%20%EB%A7%88%EC%BC%80%ED%8C%85%20%ED%94%8C%EB%9E%9C.md)
- **다음 판정**: 1.17 출시 후 14일 다운로드 ≥ 직전 14일 × 1.5 → 2단계(캘린더 「오늘 버틸까」·zh-Hans) 착수, 미만이면 유틸 추가 중단·캐릭터/시즌 복귀. 전환율은 하한선만 — [로드맵 1.17 결정 8](Zappy%20%EB%A1%9C%EB%93%9C%EB%A7%B5%EA%B3%BC%20%EC%9C%A0%EB%A3%8C%ED%99%94%20%EA%B3%84%ED%9A%8D.md)
- **지금 할 일**: ASC Featuring → Nominations에 `App Enhancements` 1건(문구는 마케팅 플랜 ASO §9) · 승인되면 `asc state`로 확인하고 14일 측정 시작(직전 14일 다운로드를 ASC 앱 분석에서 먼저 적어 둘 것)
- **하지 않을 일**: 14일 판정 전 캘린더·zh-Hans 착수, 기존 18종 소급 수정, 유료 광고

macOS 메뉴바의 배터리를 살아있는 캐릭터로 바꿔주는 앱이다. 핵심 문법은 **"잔량 = 표정과 형태"** — 달이 이지러지고, 눈사람이 녹고, 불꽃이 사그라들어서 숫자를 읽지 않아도 배터리 상태가 한눈에 보인다. 리포지터리·내부 코드명은 `CuteBattery`, 사용자에게 보이는 제품명은 Zappy다.

## 핵심 결정 (2026-07-29 기준)

| 항목 | 현재 상태 |
|---|---|
| 앱 이름 | Zappy (번들 `com.hong.zappy`, 내부 코드명 CuteBattery) |
| 한 줄 문제 정의 | 숫자로만 보여 무시하기 쉬운 배터리 상태를, 캐릭터의 표정·형태 변화로 직관적으로 전달 |
| 대상 사용자 | 맥 셋업 꾸미기를 즐기는 일반 사용자, 전 연령 |
| 현재 개발 단계 | **1.17.0 (build 27) 제출(2026-10-01, WAITING_FOR_REVIEW) — 데스크톱 펫 2D 복귀 + 배터리 러너웨이(남은 시각·충전기 W·리포트 무료) + 너구리 삭제 + ASO** — [개발 기록 10-01](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-10-01.md). 이전: **1.15.0 (build 26) 출시(2026-09-22) — 할로윈 호박 테마(Zappy+) + 로그인 자동 실행 opt-in**, 1.14.0은 09-19 출시 확인 — [개발 기록 09-22](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-09-22.md). 이전: **1.14.0 (build 24) 제출(09-04) — 기존 테마 4종 다듬기: 사과 연속 갉아먹기 + 충전 고치·완충 나비, 날씨 구름 가림(얼굴은 해에만) + 번개 번쩍임, 선인장 팔·벌·봉오리·물방울, 해파리 추진 리듬(눈만)** — [개발 기록 09-04](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-09-04.md). 1.13.0은 READY_FOR_SALE(09-04 확인). 이전: **1.13.0 (build 23) 제출(09-01) — 리텐션 P0+P1: 체험형 온보딩 2장(실제 잔량 캐릭터·무료 테마 선택·슬라이더) + 완료는 시작 버튼에서만 + 알림 권한은 켠 순간에 + 메뉴 재정리·위젯 힌트·리포트 진행·NEW 배지 + 풍선 펌프 8프레임 + 스토어 스크린샷 5로케일 코드 생성(`docs/store-screenshots`)** — [개발 기록 09-01](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-09-01.md). 1.12.0은 READY_FOR_SALE(09-01 확인). 이전: 1.12.0 — 새 테마 풍선(Zappy+ 10번째 모션, 잔량=부푼 크기·충전=발펌프) + AirPods 표시 제거(macOS 26 실기기 반증: IORegistry에 노드 자체가 없음) + 설명문 낡은 광고 2건 정정 + 랜딩 나무 누락 보충. 커밋 `8a159fe` → 같은 날 제출, WAITING_FOR_REVIEW** — [개발 기록 08-28](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-28.md) · [개발 기록 08-26](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-26.md) · [개발 기록 08-25](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-25.md). 1.7 = Zappy+ 강화 **배터리 리포트 + 위젯 라지/테마 선택** — **데스크톱 펫은 캐릭터 재설계가 필요해 출시에서 제외**(코드는 남기고 `DesktopPet.featureEnabled` 플래그로 차단) — [개발 기록 08-20](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-20.md) · [개발 기록 08-19](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-08-19.md) · [마케팅 플랜](Zappy%20%EB%A7%88%EC%BC%80%ED%8C%85%20%ED%94%8C%EB%9E%9C.md) · [심사 이력](App%20Store%20%EC%8B%AC%EC%82%AC%20%EC%9D%B4%EB%A0%A5.md) |
| 기술 스택 | AppKit(NSStatusItem) + IOKit 전원 API + UserNotifications + ServiceManagement, App Sandbox, 권한은 알림 1개(+ Zappy+ 주변기기 배터리를 쓸 때만 블루투스), 네트워크 없음, macOS 13+ |
| 핵심 기능 | 테마 18종(1.17: 모찌·고양이·슬라임·유령·달·눈사람·픽셀 무료 7종 + 불꽃·로봇·역도·날씨·사과·소다·선인장·해파리·나무·풍선·호박 Zappy+ 11종 — 너구리는 1.17에서 삭제, 로봇·역도·사과 상시 모션 + 날씨 비/번개 + 소다 충전 기포 + 풍선 충전 펌프 모션) · 모노크롬/컬러 · 충전 휴식 씬(1.2.0: 눈·물·콘센트·라면, 충전 중 모션 정지) · 저전력 알림 기준 선택(10~30%) · 80% 충전 알림 · 자동 실행 · IOKit 이벤트 갱신 · Zappy+ 일회성 IAP(₩3,300, StoreKit 2) |
| 배포 방식 | Mac App Store 전용. BM은 [로드맵과 유료화 계획](Zappy%20%EB%A1%9C%EB%93%9C%EB%A7%B5%EA%B3%BC%20%EC%9C%A0%EB%A3%8C%ED%99%94%20%EA%B3%84%ED%9A%8D.md) 참고 |

## 프로젝트를 규정한 결정 4가지

1. **모노크롬 기본 + 컬러는 토글.** 기본은 시스템 아이콘과 어울리는 흑백 템플릿(`isTemplate`, RunCat 포지션), 컬러 캐릭터는 꾸미기 모드로 분리했다. RunCat의 시그니처(달리는 동물 애니메이션 속도 = 시스템 지표)는 의도적으로 피하고, "표정·형태" 문법을 우리 정체성으로 삼았다.
2. **잔량은 연속적으로 표현한다.** 20% 임계값에서만 바뀌는 이진 표현을 버리고, 배터리형은 아래에서 차오르는 물, 달은 위상(50%=반달), 눈사람은 녹는 정도, 불꽃은 크기로 어떤 잔량에서든 상태가 읽히게 했다.
3. **텍스트는 담백하게, 귀여움은 캐릭터로만.** 메뉴 문구·알림에서 "냠냠 충전 중" 류의 멘트를 전부 제거했다("배터리 44% · 충전 중"). 캐릭터가 귀여움을 전담하고 텍스트는 시스템 톤을 유지한다.

4. **캐릭터는 전부 코드로 그린다 — 이미지 에셋 0개.** (2026-08-19 소급 기록: 그동안 결과만 적혀 있고 결정 근거가 없었다.) 선택이 아니라 위 1·2번의 **귀결**이다. ① 모노크롬 템플릿은 알파 채널만 쓰므로 "채움 위의 얼굴"을 knockout·splitFace로 **그리는 순간에** 뚫어야 하고, ② 잔량이 연속값이라 에셋으로 하면 `14테마 × 101단계 × 모노/컬러 × 4페이즈 × 씬` ≈ 수만 장이 된다. 대신 `NSImage(size:flipped:drawingHandler:)`가 그릴 때마다 핸들러를 재호출해 **해상도 독립**을 얻었고, 이 이득을 iconScale 보정(07-26)·위젯 6배(1.6)·알림 첨부 8배(1.6)·**데스크톱 펫 7배(1.7)** 에서 새 에셋 0개로 회수했다 — 1.7 펫이 반나절에 나온 이유. 대가는 **그림을 눈으로 볼 수 없다는 것**이라 렌더 스크립트·`-renderShots`가 절차로 존재하고, Figma 시안은 손으로 좌표를 옮겨 포팅해야 한다. 실측: 드로잉 코드 2,977줄·`NSBezierPath` 378회, 번들 `Assets.car`에는 `AppIcon`만, 스토어 표기 0.8MB.

상세 경위와 기술 노트는 [개발 기록 1차(07-23~25)](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-07-23.md)와 [개발 기록 2차(07-26~29)](Zappy%20%EA%B0%9C%EB%B0%9C%20%EA%B8%B0%EB%A1%9D%202026-07-26.md) 참고.

## 관련 리소스

- 코드: `(로컬 경로)` — GitHub `dbsghdz1/Zappy` (private)
  - **레포 `CLAUDE.md`(2026-08-15)**: Claude 세션은 이 위키(README → 심사 이력 → 최신 개발 기록)를 먼저 읽고 시작하며, push 시 위키를 함께 갱신한다 — 코드 규칙·개발자 플래그·배포 절차 요약 포함
  - **제출·아카이브는 `Zappy.xcodeproj`로** (앱 타깃, 동기화 폴더로 `Sources/CuteBattery` 참조). `Package.swift`(SPM)는 개발 편의용 — 이걸로 Archive하면 Generic Archive가 되어 업로드 불가.
  - `package-app.sh` — 로컬 테스트용 무서명 `Zappy.app` 조립 (스토어 업로드 불가)
  - `Assets/icon-1024.png` — App Store Connect용 아이콘 마스터
- 디자인: Figma "개인" 파일 (key `lRDTmzs1SuurwdY2jgSv5J`) — 컨셉 시트 A(파스텔)·B(네온)·C(코지카페)·D(믹스)·E(밤하늘) 26종 시안(미출시 테마 재고), v2 모노크롬 시트, **App Store 스크린샷 5장(2880×1800, y=6618)**
- 심사 회신 자료: 화면 녹화 체크리스트 + 영문 답변 7항목 — [App Store 심사 이력](App%20Store%20%EC%8B%AC%EC%82%AC%20%EC%9D%B4%EB%A0%A5.md)에 수록
- 3D 마케팅 에셋 (Blender, 2026-08-09): 눈사람 히어로 스틸 + 방전→충전(함박눈·반짝) 8.3초 클립 — `(로컬 경로)`, 제작 경위는 [3D 눈사람 작업기록](Zappy%203D%20%EB%88%88%EC%82%AC%EB%9E%8C%20Blender%202026-08-09.md)
- 홍보 자동화 운영: [Zappy 자동 홍보 운영](Zappy%20%EC%9E%90%EB%8F%99%20%ED%99%8D%EB%B3%B4%20%EC%9A%B4%EC%98%81.md) — Oracle Hermes가 소재를 준비해 Slack `#zappy`로 전달하고, X 게시물은 사용자가 직접 올리는 흐름. 전체 설계는 홍보 자동화

## 캐릭터 디자인 규칙과 소급 기각 (2026-09-11)

홍이 [신한카드 캐릭터 리뉴얼](https://www.jungle.co.kr/magazine/203856)을 레퍼런스로 주면서 **«항상 선과 원을 조합해서 그리는 것 같다»**고 짚었다. 사실이었다 — `Themes.swift`는 **원·사각 234개 대 곡선 39개**다. 거기서 뽑은 규칙은 레포 `docs/sketches/README.md`(커밋 `02d7c48`)에 있다: **실루엣 한 획 + 외곽선 + 눈은 작게·낮게·좁게.**

> [!WARNING]
> **기존 18종 소급은 기각됐다. 다시 제안하지 말 것.**
>
> 이 규칙으로 전 테마를 고치자고 제안했다가 홍이 눈사람 수정본을 보고 **«훨씬 훨씬 별로»**로 기각했고, **기각이 옳았다.**
> - 제안의 근거였던 «흰 몸 + 연한 외곽선이라 밝은 배경에서 사라진다»가 **거짓이었다.** 회색 배경 한 장만 보고 한 말이고, 배경 5종에 실제 크기로 올려 보니 어디서도 안 사라진다. 연한 하늘색 외곽선은 약한 게 아니라 **«눈/얼음» 재료 표현**이었다.
> - 그 위에 세운 18종 판정표도 함께 폐기했다.
> - **눈사람은 공 두 개인 것이 정체성**인데 한 덩어리로 합치니 펭귄이 됐다. 단추·당근 홈·목도리 자락도 «22pt에서 안 보인다»며 뺐는데 그게 밀도를 만들고 있었다.
>
> 규칙은 **새로 그리는 캐릭터**에만 쓴다. 모찌·고양이·눈사람은 각자 문법이 서 있다.

## 배운 것

- [코드보다 먼저 정하는 것](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/%EA%B0%9C%EB%B0%9C%EB%B0%A9%EB%B2%95/%EC%BD%94%EB%93%9C%EB%B3%B4%EB%8B%A4%20%EB%A8%BC%EC%A0%80%20%EC%A0%95%ED%95%98%EB%8A%94%20%EA%B2%83.md) — PRD는 「무엇을·왜」만 적는 문서. 현재 카드와 펫 브랜치의 checks README가 이미 그 역할을 하고 있었다 (09-22)
- [도메인과 메일 MX](../../../%ED%95%99%EC%8A%B5/%EA%B3%B5%EB%B6%80/CS/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC/%EB%8F%84%EB%A9%94%EC%9D%B8%EA%B3%BC%20%EB%A9%94%EC%9D%BC%20MX.md) — 앱 디렉토리 제출에 도메인 이메일이 필요해 알아본 것: 도메인(이름표)과 메일 호스팅(우편함)은 따로이고 MX 레코드로 잇는다. 도메인 가격은 첫해가 아니라 연장가로 비교한다 (09-15)
- [Vercel 배포](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/%EB%8F%84%EA%B5%AC/Vercel%20%EB%B0%B0%ED%8F%AC.md) — 랜딩을 GitHub 연동으로 바꾸면서 Root Directory(`landing`)를 안 잡아 08-15~19 랜딩·구매 웹훅이 통째로 404였다. Git 연동 배포는 사람이 배포하던 폴더가 아니라 프로젝트 설정이 기준
- [App Store Server Notifications](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/AppStore/App%20Store%20Server%20Notifications.md) — 구매 알림이 안 오면 내 로그보다 `notifications/history`로 Apple 전송 결과를 먼저 본다. 실패해도 72h 안에 서버를 고치면 재시도로 도착한다
- [SceneKit과 3D 에셋 파이프라인](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/SceneKit%EA%B3%BC%203D%20%EC%97%90%EC%85%8B%20%ED%8C%8C%EC%9D%B4%ED%94%84%EB%9D%BC%EC%9D%B8.md) — Blender 눈사람을 SceneKit 펫으로 띄우며 얻은 것. **재export가 캐릭터를 바꾼다**는 것을 두 번 사고 내고 배웠고, 녹기는 코드로 짠 배율보다 **원본에 이미 조각돼 있던 셰이프 키**가 압도적으로 나았다
- [App Store 성장 도구](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/AppStore/App%20Store%20%EC%84%B1%EC%9E%A5%20%EB%8F%84%EA%B5%AC.md) — Mac 전용 앱은 Apple Ads 계정 개설조차 안 되고 인앱 이벤트·커스텀 제품 페이지도 iOS 전용. 유료 광고 미집행은 예산 판단이 아니라 **구조 판단**이며, 시즌 테마는 피처링 노미네이션으로 태운다
- [macOS 메뉴바와 샌드박스](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/macOS%20%EB%A9%94%EB%89%B4%EB%B0%94%EC%99%80%20%EC%83%8C%EB%93%9C%EB%B0%95%EC%8A%A4.md) — App Sandbox는 AX만 막고 **IORegistry는 안 막는다**. 배터리 사이클 수·설계 용량·온도는 `AppleSmartBattery`에서 entitlement 없이 읽힌다 — 1.7 배터리 리포트가 성립한 근거
- [WidgetKit과 AppIntents](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/WidgetKit%EA%B3%BC%20AppIntents.md) — AppIntents 문구는 컴파일 타임 리터럴만 받는다. 이 앱의 런타임 `L10n.pick(...)`을 못 써서 위젯 인텐트만 문자열 카탈로그로 분리했다
- [Xcode 빌드와 번들 구성](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/Xcode%20%EB%B9%8C%EB%93%9C%EC%99%80%20%EB%B2%88%EB%93%A4%20%EA%B5%AC%EC%84%B1.md) — 기능을 코드를 지우지 않고 출시에서 빼는 법. 게이트는 **가장 안쪽(`isOn` 게터)에도** 둬야 저장된 설정이 안 샌다. 동기화 폴더에서 에셋 하나 빼기는 pbxproj 예외 세트, 확인은 릴리즈 바이너리 `strings`(대조군 필수)
- [StoreKit 2 권리 확인](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/StoreKit%202%20%EA%B6%8C%EB%A6%AC%20%ED%99%95%EC%9D%B8.md) — 위젯이 컬러→모노로 튀던 원인은 `currentEntitlements` 빈 결과를 "미구매"로 내린 것. 권리 갱신은 비대칭(올리기 쉽고 내리기 어렵게) (08-21)
- [Swift와 Objective-C 브리징](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/Swift%EC%99%80%20Objective-C%20%EB%B8%8C%EB%A6%AC%EC%A7%95.md) — `@objc`를 붙여도 셀렉터 이름은 Swift 이름+인자 레이블에서 생성된다(`mouseEntered(with:)` → `mouseEnteredWith:`). AppKit이 이름으로 보내는 콜백은 어긋나면 조용히 호출되지 않는다 — 1.6.0 호버 하트가 이것 때문에 한 번도 동작하지 않았다
- [블루투스 기기 배터리 읽기](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/%EB%B8%94%EB%A3%A8%ED%88%AC%EC%8A%A4%20%EA%B8%B0%EA%B8%B0%20%EB%B0%B0%ED%84%B0%EB%A6%AC%20%EC%9D%BD%EA%B8%B0.md) — 1.9.0 주변기기 배터리가 사용자 맥에서 아무것도 못 잡은 건 버그가 아니라 **탐지 경로가 절반만 커버**된 것. 클래식 BT HID는 IORegistry `BatteryPercent`지만 **BLE는 그 키가 아예 없고** GATT 0x180F를 CoreBluetooth로 읽어야 한다. 권한 팝업 시점은 `CBCentralManager` 생성 시점으로 통제 (1.9.1, `969d623`). **AirPods는 macOS 26에서 길이 없다** — 08-26에 IORegistry 좌·우·케이스 키 경로로 구현했으나(`1de8b75`) 08-28 실기기 검증에서 반증: macOS 26.5.1은 에어팟을 IORegistry에 아예 안 올린다(잔량은 bluetoothd만 안다). Watch도 어느 경로에도 없다
- [AppKit 오프스크린 렌더와 argument domain](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/AppKit%20%EC%98%A4%ED%94%84%EC%8A%A4%ED%81%AC%EB%A6%B0%20%EB%A0%8C%EB%8D%94%EC%99%80%20argument%20domain.md) — 창 없는 뷰는 시스템 외관을 물려받아 다크 Mac에선 라벨이 흰색으로 그려진다(`view.appearance` 고정). `-AppleLanguages`·`-onboarded NO`·`-newThemes` 같은 argument domain으로 실제 설정을 안 건드리고 상태를 강제한다 — 한글 배열은 따옴표 필수 (1.13 스크린샷 생성기). **테스트 앱을 `-theme`로 띄우면 메뉴에서 테마를 골라도 안 바뀐다**(인자가 저장 설정을 덮음). `lockFocus`는 Retina에서 2x로 구워지니 1x가 필요하면 `NSBitmapImageRep`에 직접 (1.14)
- [macOS 템플릿 아이콘 그리기](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/Apple/macOS%20%ED%85%9C%ED%94%8C%EB%A6%BF%20%EC%95%84%EC%9D%B4%EC%BD%98%20%EA%B7%B8%EB%A6%AC%EA%B8%B0.md) — 템플릿 아이콘은 **알파만 남고 색은 버려진다**. '위에 얹힌 것'을 표현할 방법은 knockout 틈뿐이고, 컬러의 하이라이트·결은 대개 실루엣을 조각내므로 빼야 한다 (1.10.0 나무의 가지 위 눈)
- [유료 앱 판매 알림](../../../%EC%9E%91%EC%97%85%EB%85%B8%ED%8A%B8/AppStore/%EC%9C%A0%EB%A3%8C%20%EC%95%B1%20%ED%8C%90%EB%A7%A4%20%EC%95%8C%EB%A6%BC.md) — App Store Server Notifications는 IAP 전용이라 Zappy+는 웹훅으로 즉시 오지만 유료 다운로드는 안 온다. 한능검 판매 요약 크론(`landing/api/hangeom-sales.js`)이 같은 `zappy-landing` Vercel 프로젝트에 동거한다

## 다른 프로젝트와의 경계

- [BarStack](../BarStack/README.md)과는 별개 프로젝트다. 둘 다 메뉴바 유틸리티지만 BarStack은 **아이콘 정리(숨김)**, Zappy는 **배터리 캐릭터 표시**로 문제 영역이 다르다. 다만 App Store 심사 경험(2.1 정보 요청 등)은 서로 참고한다.
- Swift/macOS 프로젝트이므로 React·TypeScript 학습 위키와 연결하지 않는다.
