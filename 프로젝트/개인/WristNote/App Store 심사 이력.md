---
type: project
status: active
created: 2026-09-02
updated: 2026-09-04
related_wiki: []
---

# WristNote App Store 심사 이력

ASC appId `6807479115`, 번들 `com.hong.wristnote` (+ `.watchkitapp`). 파이프라인은 `fastlane/Fastfile`(archive · upload · release · submit · resubmit) + appstore-release 스킬의 `asc` 도구.

## 타임라인

### 1.1.0 (build 2) — 2026-09-04 00:37 제출 → 스크린샷 정리 후 재제출, WAITING_FOR_REVIEW

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

- 실기기 장시간(30분+) 백그라운드 녹음 실측 — 1.0은 이 검증 없이 제출됐다.
- 워치 녹음 포맷: 현재 실기기에서 µ-law 16kHz(16KB/s)로 동작. AAC는 `AVAudioRecorder`에서 무출력. `AVAudioEngine` + 소프트웨어 AAC로 용량 1/4 줄이기 검토.
- 콜드 스타트 반응 지연(권한 요청 + 오디오 세션 활성화가 첫 탭에 겹침).
