---
type: study
area: Flutter
audience: ai
status: active
created: 2026-09-03
updated: 2026-09-03
projects:
  - "보험찾개냥"
---

# 머티리얼 날짜 피커와 ko 로케일

`showDatePicker`의 키보드 입력 모드는 **한국어 로케일에서 안내 문구와 파서가 어긋나 있다** — 안내대로 치면 무엇을 쳐도 거부된다.

## 핵심 정리

- `MaterialLocalizationKo.dateHelpText`는 **`yyyy.mm.dd`**라고 안내한다. 그런데 실제 파서는 `GlobalMaterialLocalizations.parseCompactDate` → `DateFormat.yMd('ko').parseStrict`라서 **`1995. 3. 2.`(점 뒤 공백 + 끝점)만** 받는다. `parseStrict`가 던진 `FormatException`은 `null`로 삼켜지고 화면에는 `invalidDateFormatLabel`(**"형식이 잘못되었습니다."**)만 뜬다.
- **통과하는 형태가 화면 어디에도 안 적혀 있다** — 사용자가 알아낼 방법이 없다. 그래서 증상은 "날짜 입력이 잘 안 된다"로 보고된다.
- **플랫폼 문제가 아니다.** 로케일 데이터 문제라 iOS·Android가 같다. iPhone에서만 겪었다는 보고는 거기서 시도했기 때문이지 iOS 전용 결함이 아니다.
- 대응 둘:
  1. **직접 입력은 우리 `TextField`로 받는다** — 숫자만 받는 `TextInputFormatter`로 `YYYY.MM.DD` 점을 끼우고 파싱도 우리가 한다.
  2. **달력을 열 때 `initialEntryMode: DatePickerEntryMode.calendarOnly`를 준다** — 다이얼로그의 연필 아이콘(깨진 입력 모드로 들어가는 유일한 문)을 없앤다.
- 직접 파싱할 때 **`DateTime(2026, 2, 31)`이 3월 3일로 조용히 굴러간다.** 만든 뒤 `year·month·day`가 입력값과 같은지 되봐야 없는 날짜를 거른다.

## 기록

### 2026-09-03 — ko 로케일 프로브로 실측

- 맥락: 소프트웨어마에스트로 SSH-455 「날짜 선택 컴포넌트 수정」 — 티켓 본문이 *"iPhone에서 날짜 입력이 잘 안됨"* 한 줄이라 증상부터 재현했다
- 실측: `MaterialApp(locale: Locale('ko'), localizationsDelegates: GlobalMaterialLocalizations.delegates)` 안에서 `MaterialLocalizations.of(context)`를 꺼내 호출

  | 친 것 | `parseCompactDate` |
  | --- | --- |
  | `1995.03.02` (**`dateHelpText` 그대로**) | `null` |
  | `1995.3.2` · `19950302` · `1995-03-02` | `null` |
  | `1995. 3. 2.` | `1995-03-02` ✅ |
  | `1995. 03. 02.` | `1995-03-02` ✅ |

  `formatCompactDate(DateTime(1995,3,2))`는 `1995. 3. 2.` — **포맷터와 파서는 서로 맞고, 안내 문구만 어긋나 있다.**
- 근거: 버려지는 프로브 테스트(`Client/test/zz_probe_test.dart`, 커밋 안 함)로 `flutter test` 1회. SDK 원본은 `flutter/packages/flutter_localizations/lib/src/l10n/generated_material_localizations.dart`(`MaterialLocalizationKo`)와 `.../src/material_localizations.dart:183`(`parseCompactDate`)
- 이 프로젝트의 대응: `AppField.date`를 탭 전용에서 입력형으로 바꾸고(달력은 칸 안 오른쪽 아이콘 버튼), `parseDateLabel`·`DateInputFormatter`를 우리 쪽에 둔다 — spec `docs/spec/SSH-455/spec.md`, Draft PR #115, 브랜치 `feat/SSH-455`
