---
type: study
area: AppStore
audience: ai
status: active
created: 2026-08-25
updated: 2026-09-03
projects:
  - "사업자 운영"
---

# 미국 세금 양식 W-8BEN

**App Store Connect의 웹 세금 양식은 IRS 원본 W-8BEN의 일부 필드를 노출하지 않는다.** 특히 Part II Line 10 "Special rates and conditions"의 Article 입력란이 없어서, 한미조세조약 조항을 지정해야 하는 경우 ASC 폼만으로는 정확히 작성할 수 없다. 해결책은 IRS PDF를 직접 작성해 Apple 세무팀에 업로드하는 별도 경로다.

## 핵심 정리

### ASC 웹폼의 한계

- ASC 세금 양식에는 **Line 10(Article·세율·소득 종류) 입력란이 나오지 않는다.** 조약이 로열티 종류별로 다른 세율을 두는 나라(한국 포함)는 이 칸이 필수인데도 그렇다.
- **제출한 세금 양식의 변경은 ASC 화면에 반영되지 않는다.** Apple 명시: 그 화면은 static이고 백엔드에서 바뀐 이름·주소·필드가 보이지 않는다. → 안 바뀐 것처럼 보인다고 재제출하면 안 된다.

### Apple 세무팀 제출 경로 (Finance Support, Apple Media Services)

1. 개발자 지원에 문의 → **8자리 Case ID**가 붙은 회신 메일이 온다.
2. 회신 메일에 W-9 / W-8 업로드 링크가 각각 들어 있다(도메인 apple.com 확인 필수 — 양식에 TIN·주소가 들어간다).
3. IRS.gov에서 양식 원본을 직접 받아 작성·서명한다. **개인 = W-8BEN, 법인 = W-8BEN-E.**
4. **파일명 = Case ID 8자리 그대로** (`21321618.pdf`). 다른 문자를 붙이지 않는다.
5. 링크에 드래그 → UPLOAD.
6. **같은 메일 스레드에 반드시 회신한다.** 회신이 없으면 Apple이 업로드 파일을 찾지 못해 반영되지 않는다 — 업로드만으로는 접수가 아니다.

### 양식에 쓸 이름·주소는 ASC 「비즈니스」 화면에서 읽는다

- Apple이 대조하는 값은 **ASC → 비즈니스(Business) 상단의 Legal Entity 이름·주소**다. 사업자등록증명·D&B·Google Play에 넣은 영문 표기(사업자 영문 정보)와 **다를 수 있다** — 2026-09-03 실측: ASC 개인 계정의 Legal Entity는 `Yunhong Kim`(성·이름 순서가 사업자 문서의 `KIM YUNHONG`과 다름)이고 주소는 사업자 주소(서울)가 아니라 **가입 당시 주소(광주)** 였다. 사업자 문서 기준으로 양식을 쓰면 반려된다.
- **개인 계정의 Legal Entity 주소 변경 경로** (2026-09-03 확인, [Updating your account information](https://developer.apple.com/help/account/membership/updating-your-account-information/)): `developer.apple.com/account` → Membership details 카드 → **"Update your information"** → Membership Information Update 폼. 선택지 Name / Address / Both, 주소는 **Roman character + local character** 두 칸, Additional information(선택). **서류 업로드 칸·처리 기간 안내는 폼에 없다** — Apple 문서는 "must be verified and approved… asked to provide documentation"이라 하므로 제출 후 메일로 증빙을 요구할 수 있다. `account.apple.com`에서 Apple 계정 이름을 바꿔도 개인 멤버십의 법인명·판매자명은 바뀌지 않는다(Apple 명시).
- 읽기 전용 확인은 `aside "Open App Store Connect → Business … READ-ONLY"`로 충분하다(로그인 세션 재사용). 세금 양식 섹션에서 현재 양식·제출일·상태(활성화됨)도 같이 보인다.
- IRS `fw8ben.pdf`(Rev. 10-2021)는 AcroForm이라 PyMuPDF로 채울 수 있다. 위젯 이름 매핑: `f_1` Line 1 이름 · `f_2` 국적 · `f_3`/`f_4`/`f_5` Line 3 주소·시/우편번호·국가 · `f_6`~`f_8` Line 4 · `f_9` Line 5 · `f_10` 6a FTIN · `c1_01` 6b · `f_11` Line 7 · `f_12` Line 8 DOB · `f_13` Line 9 · `f_14` Article · `f_15` 세율 · `f_16` 소득 종류 · `f_17`+`f_18` 설명 · `c1_02` 서명 권한 체크 · `f_20` 서명(Signature 위젯) · `Date` · `f_21` 서명자 이름. `w.field_value = …; w.update()` 후 `doc.save()`.

### 반려되는 조건

- **서명 필드 미서명** — Apple이 명시적으로 접수 불가.
- **양식의 주소가 개발자 프로그램 법인 주소와 불일치** — 반영 안 됨.
- **주소 변경 목적의 양식 제출** — 반려. 이름·주소는 **ASC 계정에서 먼저 바꾸고** 그 다음에 세금 양식을 낸다. 이 경로로 바꿀 수 있는 건 **양식 선택·상태·Tax ID뿐**이다.

### W-8BEN 작성 요점 (IRS 설명서 기준)

| 줄 | 내용 |
|---|---|
| 1 | 영문 이름 — 여권/ASC와 동일 |
| 2 | Country of citizenship |
| 3 | 상시 거주지 — **PO Box·"c/o" 불가** |
| 5 | U.S. TIN(SSN/ITIN) — 없으면 공란. 조약 청구엔 6a로 충분 |
| 6a | FTIN(거주국 납세번호). 채웠으면 **6b는 체크하지 않는다** |
| 7 | Reference number — Team ID를 넣어두면 Apple 쪽 매칭이 쉽다 |
| 8 | 생년월일 **MM-DD-YYYY** |
| 9 | 조약 거주국 |
| 10 | Article·paragraph / 세율 % / 소득 종류 / 요건 충족 설명 |

- **Line 10이 필수인 경우**: 조약이 로열티 종류별로 다른 세율을 두는 경우, 유학생·연구자, 고정사업장 귀속되지 않는 사업소득, 송금 기준 청구. 일반 이자·배당은 공란 가능.
- **서명**: IRS는 *"이름을 타이핑하는 것만으로는 서명이 아니다"*라고 명시한다. 전자서명은 승인 의사와 타임스탬프/문구가 있어야 한다 → **인쇄·손서명·스캔이 가장 안전**. 날짜도 MM-DD-YYYY.

### 한미조세조약 로열티 조항

- **Article 14 (Royalties).** **14(1)**이 로열티 일반을 **15%**로 제한하고, **14(2)**가 저작권(literary, dramatic, musical, or artistic work) 및 영화 필름 로열티를 **10%**로 더 낮춘다. 14(3)은 미국 내 고정사업장에 귀속되면 적용 제외, 14(4)(a)가 로열티 정의(copyrights of literary, artistic, or scientific works …). → Line 10에는 `14(2)`를 쓴다 (2026-09-03 irs.gov/pub/irs-trty/korea.pdf 원문 확인, 이전 기록의 "14(1)이 10%"는 오기).
- 소프트웨어/앱은 통상 저작권 로열티로 보아 10%를 주장하지만, **조약 원문이 소프트웨어를 명시하지 않아 다툼 여지가 있다** — paragraph 번호와 10% 적용 여부는 세무사 확인 대상.
- 별개로, **Apple이 App Store 수익을 미국 원천 로열티로 취급하는지 자체가 먼저 확인할 사항이다.** 정산 내역에 미국 원천징수가 잡힌 적이 없다면 Line 10이 실제로 적용되지 않을 수 있다.

## 기록

### 2026-08-25 — ASC 폼에 Article 칸이 없어 W-8BEN을 IRS 원본으로 재제출

- 맥락: 사업자 운영 — ASC에 W-8BEN을 제출한 뒤 "Special rates and conditions"를 검토하려 했으나 **Article 입력란이 ASC 폼에 표시되지 않아** 조약 정보를 채우지 못했다. Apple 개발자 지원에 재편집 요청(2026-07-30) → Finance Support 회신(Case ID 부여).
- 배운 것:
  - Apple의 답은 "ASC 폼을 다시 열어주겠다"가 아니라 **"IRS 원본 PDF를 작성해 별도 링크로 올려라"**였다. ASC 웹폼은 세금 양식의 완전한 대체재가 아니다.
  - **업로드 + 메일 회신**이 한 쌍이다. 업로드만 하면 접수되지 않는다.
  - **파일명이 Case ID여야 한다** — 사람이 수동으로 매칭하는 프로세스라는 뜻.
  - 세금 양식 갱신은 **ASC 화면에 절대 반영되지 않는다.** 확인 수단은 Apple 회신뿐이다.
  - 이름·주소는 이 경로로 못 바꾼다. **ASC 계정 → 세금 양식** 순서가 강제된다.
- 근거: Apple Media Services Finance Support 회신 메일(Case ID 8자리), [IRS Form W-8BEN Instructions](https://www.irs.gov/instructions/iw8ben), [US-Korea Income Tax Convention](https://www.irs.gov/pub/irs-trty/korea.pdf)

### 2026-09-03 — ASC Legal Entity 실측 후 W-8BEN 초안 작성

- 맥락: 사업자 운영 — Case ID 21321618 재제출을 진행. 체크리스트 1번(Legal Entity 확인)을 aside로 실행했더니 **이름 표기와 주소가 사업자 영문 정보와 달랐다.** 위키 체크리스트가 가정한 서울 주소로 냈으면 "address does not match"로 반려됐을 것.
- 배운 것:
  - Apple 세금 양식의 정답지는 **ASC 비즈니스 화면**이지 사업자등록증명이 아니다. 개인 계정은 가입 당시 개인 주소가 Legal Entity 주소다.
  - ASC 세금 양식 목록에 이미 `U.S. Form W-8BEN` 2026-07-30 **활성화됨** 상태 — 이번 제출은 Line 10 조약 정보를 추가하는 **갱신**이다.
  - 조약 항 번호는 **14(2)** (10% 저작권). 이전 기록의 14(1)은 오기라 정정했다.
  - PyMuPDF로 `fw8ben.pdf` 필드를 채워 `(로컬 경로)`·`-gwangju.pdf` 두 벌 생성. 6a FTIN·8 DOB·서명은 개인정보라 파일에 넣지 않고 인쇄 후 손기입.
  - 홍 확인: 사업장도 거주지도 서울 → **순서는 ① Membership 주소 변경 요청 → ② 승인·ASC 비즈니스 화면에 서울 반영 확인 → ③ 서울 주소 W-8BEN 업로드 + 회신.** 광주 주소로 먼저 내면 Line 3(상시 거주지)이 사실과 달라진다.
  - 주소 변경 폼 제출은 Claude Code 권한 분류기가 차단했다(외부 계정 변경). 읽기 전용 조회까지만 자동화 가능.
- 근거: aside 세션 `2026-09-03_BU74Tabj8PygkuiK` 스크린샷, irs.gov/pub/irs-trty/korea.pdf Article 14 원문

## 참고 자료

- [Instructions for Form W-8BEN](https://www.irs.gov/instructions/iw8ben) — Line 6a/6b·9·10과 Part III 서명 요건의 1차 출처 (2026-08-25 확인)
- [Form W-8BEN (PDF)](https://www.irs.gov/pub/irs-pdf/fw8ben.pdf) — 작성할 원본 양식 (2026-08-25 확인)
- [UNITED STATES - REPUBLIC OF KOREA INCOME TAX CONVENTION](https://www.irs.gov/pub/irs-trty/korea.pdf) — Article 14 원문 (2026-08-25 확인)
- [Publication 515](https://www.irs.gov/publications/p515) — 비거주자 원천징수 일반. 조약 요약표(Table 1~3)는 별도 페이지로 분리됨 (2026-08-25 확인)
