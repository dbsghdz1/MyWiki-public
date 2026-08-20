---
type: project
status: paused
aliases:
  - math-sprint
  - toss-2048
  - 암산 스프린트
created: 2026-07-29
updated: 2026-08-19
slack_channel: math-sprint
repos:
  - "github.com/dbsghdz1/math-sprint"
related_wiki:
  - "[[_wiki/React TypeScript 제품 개발]]"
---

# math-sprint — 토스 미니앱 암산 스프린트

Apps in Toss 환경에서 실행되는 60초 암산 게임이다. 현재 설명은 2026-07-29에 실제 코드, README, production build를 확인한 결과를 기준으로 한다.

## 현재 확인된 구현

- 60초 동안 객관식 암산 문제를 푸는 게임
- 정답 점수와 연속 정답 콤보 보너스
- 오답 시 콤보 초기화와 2초 시간 페널티
- 5문제마다 난이도 상승
- 점수, 정답 수, 정확도, 최대 콤보 결과 화면
- localStorage 최고 기록 저장
- Apps in Toss 햅틱 피드백
- ready → playing → result 상태 흐름

## 현재 기술 상태

- Apps in Toss Web Framework 2.10.8
- Granite 개발 환경
- React 19, TypeScript 5.7, Vite 6
- `npm run lint` 통과
- `npm run build`와 `.ait` 생성 통과
- 비공개 GitHub 저장소 연결 완료

> [!note] 2026-08-18 **보류 결정** (홍) — **잠시 중지된 프로젝트다 (중단·폐기가 아니다)**
> 07-30 이후 활동 없음. DayTune·BarStack 1.2처럼 지원 마감(2026-11-14) 이후 재개 검토. 정규 슬롯 없음.
> **중지 시점의 상태는 온전하다** — `npm run build`·`.ait` 생성 통과, 저장소 연결 완료. 재개 시 이 지점에서 이어간다.
> **재개 조건**: 2026-11-14 경과 후 슬롯 재배분 시 검토.

## 코드 저장소

- 로컬: `(로컬 경로)`
- GitHub: `dbsghdz1/math-sprint` (private)
- 기본 브랜치: `main`

## 이 작업공간에서 관리할 것

- 문제 생성 규칙과 난이도 곡선
- 점수·콤보·페널티 밸런스
- 토스 미니앱 심사와 배포 준비
- 사용자 플레이 테스트와 지표
- 기능 구현, 오류, 테스트와 릴리즈 기록

## 다음에 확인할 것

- 실제 출시 목표와 마감일
- 앱인토스 콘솔 등록 및 배포 상태
- 사용자 타깃과 반복 플레이를 만드는 핵심 지표
- 추가 문제 유형, 공유, 랭킹 기능의 필요 여부

## 위키로 보낼 것

여러 웹 프로젝트에서 재사용할 수 있는 React 상태 설계, 게임 루프, 브라우저 저장소, Apps in Toss 통합 지식은 [[_wiki/React TypeScript 제품 개발|React·TypeScript로 제품 만들기]] 또는 별도 공용 허브에 종합한다.
