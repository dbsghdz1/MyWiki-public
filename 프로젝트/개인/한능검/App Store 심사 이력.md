---
type: project
status: active
created: 2026-08-28
updated: 2026-08-28
---

# App Store 심사 이력 — 한능검 정복

| | |
|---|---|
| 앱 이름 | **한능검 정복** (홍이 ASC에서 생성) |
| App ID | `6806144570` |
| Bundle ID | `com.hong.hangeom` |
| 팀 | `WN2B884S76` |

## 1.0 (build 2) — 2026-08-30 아이폰 전용 전환

**아이패드 스크린샷 요구를 피하려고 iPad 지원을 뺐다** (홍 결정 2026-08-30). `TARGETED_DEVICE_FAMILY = "1,2"` → `1` (Debug·Release 2곳), 빌드 넘버 1→2. IPA의 `UIDeviceFamily [1]`·`CFBundleVersion 2`를 업로드 전에 확인했다. 아이폰 전용이어도 아이패드에선 호환 모드로 실행된다. build 2 업로드 → VALID → 버전 1.0에 연결 완료. 남은 것은 build 1 때와 동일(아래 셋).

## 1.0 (build 1) — 2026-08-28 준비 중

### 진행 상태

| 단계 | 상태 |
|---|---|
| 아카이브·IPA | ✅ `fastlane/build/App.ipa` (1.3MB) |
| 바이너리 업로드 | ✅ build 1 · **VALID** → **build 2로 교체 (2026-08-30)** |
| 버전 연결 | ✅ `asc attach-build` |
| 메타데이터(ko) | ✅ 이름·부제·설명·키워드·심사메모 |
| 스크린샷 | ✅ 5장 (1320×2868) — **중복 제거 후 5장 확정** |
| **지원/개인정보 URL** | ⏳ 홍이 Notion으로 작성 예정 |
| **가격 ₩4,400** | ⏳ ASC 웹 |
| **App Privacy** | ⏳ ASC 웹 (전부 "수집 안 함") |
| 심사 제출 | ⏳ 위 셋 완료 후 `fastlane ios submit` |

### 이름이 바뀌었다

제안은 `한국사 정복 - 한능검 기출`이었는데 홍이 ASC에서 **`한능검 정복`**으로 생성했다. 메타데이터를 거기 맞추고, 빠진 "한국사" 검색어는 **부제(`한국사 심화 기출 1,102문항`)와 키워드**로 채웠다.

### 밟은 지뢰 넷 (전부 스킬 플레이북에 반영)

1. **`build_app`이 `invalid byte sequence in UTF-8`로 죽었다** — 진짜 원인은 **Xcode 26부터 export method가 `app-store` → `app-store-connect`로 바뀐 것**. gym이 옛 값을 보내 실패했고, 그 에러를 처리하던 코드가 **한글 경로 때문에 UTF-8 예외**로 죽어 원인이 가려졌다. `export_options: { method: "app-store-connect" }`로 해결.
2. **설명의 괘선(`─`)을 ASC가 거부** — *"Description can't contain the following character(s): ─"*. ASCII 하이픈으로 교체. (`■`는 통과)
3. **스크린샷이 이중 업로드됐다** — 로그에 `Successfully uploaded all screenshots`가 2번 찍혔고 실제로 **10장**이 올라갔다. 플레이북이 경고한 그대로라 **제출 전 API로 장수를 검증**해 5장으로 정리했다.
4. **`submit_for_review: false`면 deliver가 빌드를 연결하지 않는다** — `build_number`를 줘도 붙지 않는다. `asc attach-build`로 직접 연결.

### 도구 개선

`asc`에 명령 넷을 추가했다 — `builds` · `attach-build` · `screenshots` · `dedupe-screenshots`. 3번 지뢰(이중 업로드)를 API로 잡으려면 목록·삭제가 필요한데 기존 도구엔 없었다.
