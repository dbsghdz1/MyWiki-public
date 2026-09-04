---
type: project
status: active
created: 2026-09-02
updated: 2026-09-05
related_wiki: []
---

# WristNote App Store 심사 이력

ASC appId `6807479115`, 번들 `com.hong.wristnote` (+ `.watchkitapp`). 파이프라인은 `fastlane/Fastfile`(archive · upload · release · submit · resubmit) + appstore-release 스킬의 `asc` 도구.

## 타임라인

### 1.1.1 (build 3) — 2026-09-05 03:15 제출 → **스크린샷 정리 대기 중** (홍 실행 필요)

- 내용: 워치 녹음 타이머 표시 수정(`Text(timerInterval:)`, 커밋 `7e557ec`) + 컴플리케이션 URL 스킴 등록(`CFBundleURLTypes: wristnote`, 커밋 `5d43e5b`). 워치 스크린샷 2장 재촬영 — 타이머 표기가 `00:00`/`00:04`에서 `0:00`/`0:07`로 바뀌어 기존 스크린샷이 실제 화면과 달라졌다.
- `fastlane ios release` 성공(03:15:31 `Successfully submitted the app for review!`), build 3 VALID·연결됨, WAITING_FOR_REVIEW.
- **deliver 이중 업로드가 세 번째로 재발** — 로케일당 6 → 12장. 로그에 `Successfully uploaded all screenshots`가 2번(03:12:10) + 그 직전 `Failed to upload all screenshots... Tries remaining: 4`(03:11:55). **재시도가 이중 업로드의 방아쇠로 보인다** — 실패한 시도가 실제로는 일부 올린 뒤 재시도가 전체를 다시 올리는 것으로 추정(미검증).
- 정리는 **홍이 직접 실행해야 한다** — 자동 모드 분류기가 `asc cancel-review`(외부 상태 변경)를 차단했다. `scripts/fix-screenshots-resubmit.sh` 한 방으로 cancel → dedupe → `resubmit` lane까지 돈다.

### 1.1.0 (build 2) — 2026-09-04 제출 → **2026-09-05 승인·출시 (READY_FOR_SALE)**

- 주제 그래프·칩·옵시디언 마크다운 내보내기·노션 전송·워치 컴플리케이션. 스크린샷을 6장으로 개편(목록·**주제 그래프**·상세(칩)·설정 + 워치 2장), 릴리즈 노트 "회의가 쌓이면 지도가 됩니다".
- **기본 언어를 ko → en-US로 변경**(`asc set-primary`) — 홍 지시 "그 외 언어권 사용자는 영어로". 한국 사용자는 ko 로케일 그대로.
- deliver 스크린샷 이중 업로드가 **또** 재발(전 로케일·전 세트 2배) → cancel-review → dedupe(12장 삭제) → `resubmit` lane. 이 지뢰는 이제 첫 제출 표준 절차로 취급할 것.
- 제출 전 맥 쪽 사고: 터미널(cmux)의 데스크탑 TCC 권한이 세션 중 풀려 `(로컬 경로)` 전체 EPERM — 시스템 설정 → 파일 및 폴더에서 cmux·Ghostty 재허용으로 복구.

### 1.0 (build 1) — 2026-09-03 승인·출시 (READY_FOR_SALE)

제출(09-02 02:26)부터 약 하루 만에 통과, automatic_release로 즉시 출시. 리젝 0회.

### 1.0 (build 1) — 2026-09-02 02:26 제출, WAITING_FOR_REVIEW

- 홍이 ASC 웹에서 앱 레코드 생성(이름 "WristNote" 통과) + 지원·개인정보 URL(Notion, 둘 다 Privacy Policy 페이지) 입력. 나머지는 fastlane.
- 메타데이터: ko(이름 WristNote) · en-US(이름 **"WristNote: Watch Meeting Notes"** — 영어권에서 "WristNote"를 타 앱이 선점해 `Cannot add localization due to app name`, `asc add-locale`로 차별화 이름 선생성). 카테고리 생산성/비즈니스, 연령 등급 전부 NONE, 수출 규정 암호화 없음.
- 스크린샷: iPhone 6.9″ 3장(목록·상세·설정, `WRISTNOTE_DEMO` 데모 데이터) + Apple Watch 2장(대기·녹음 중), ko·en-US 동일.
- 밟은 지뢰 (전부 appstore-release 플레이북에 반영):
  1. `WKBackgroundModes: audio` → altool 90362 거절 → `UIBackgroundModes`로 수정 후 재업로드
  2. Fastfile `platform` 블록 안 `def submission_info` → `method_missing`. 최상위로 이동
  3. en-US 로케일 이름 선점 → `asc add-locale` 선생성
  4. deliver 스크린샷 이중 업로드(전 로케일·전 세트 5→10장) → `cancel-review` → `dedupe-screenshots` → `resubmit` lane, 3분
- 심사 노트에 워치+아이폰 테스트 절차와 "Apple Intelligence 미지원 기기는 전사만 제공" 명시.
- 자동 출시(automatic_release) 설정.

## 다음 버전에서 할 것

- **deliver 이중 업로드의 진짜 원인 규명** — 세 번 연속 재발했고 `overwrite_screenshots: true`로도 안 막힌다. 1.1.1 로그에서 처음으로 **실패→재시도**가 선행한 것이 잡혔다. 다음 제출 때 `--verbose`로 재시도 전후 업로드 수를 세어 확인할 것. 확인되면 `deliver_all`에서 스크린샷을 빼고 별도 lane으로 분리하는 쪽이 낫다.
- 실기기 장시간(30분+) 백그라운드 녹음 실측 — 1.0은 이 검증 없이 제출됐다.
- 워치 녹음 포맷: 현재 실기기에서 µ-law 16kHz(16KB/s)로 동작. AAC는 `AVAudioRecorder`에서 무출력. `AVAudioEngine` + 소프트웨어 AAC로 용량 1/4 줄이기 검토.
- 콜드 스타트 반응 지연(권한 요청 + 오디오 세션 활성화가 첫 탭에 겹침).
