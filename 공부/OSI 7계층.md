---
type: study
area: CS
status: active
created: 2026-08-15
updated: 2026-08-17
projects: []
---

# OSI 7계층

네트워크 통신을 7개 계층으로 나눈 참조 모델. 실제 인터넷은 TCP/IP 4~5계층으로 돌아가지만, "이 문제가 어느 계층 문제인가"를 말하는 공용 언어로 쓰인다. [[공부/네트워크|네트워크]]의 출발점.

## 핵심 정리

(아직 없음 — 학습 후 각 계층의 역할·대표 프로토콜·PDU 이름·TCP/IP 모델과의 대응을 한 표로 정리한다)

## 학습 계획

**2026-08-17 조정**: 별도 주차를 두지 않고 [[공부/네트워크|네트워크]] 강의 **44강(TCP/IP 4계층 — 개념·캡슐화·PDU·OSI 7계층)** 에서 함께 다룬다 (W1-2, 08-19 수). 완료하면 체크하고 `## 핵심 정리`에 대응표를 남긴다.

- [ ] W1-2 (2026-08-19): 44강 수강 후 7계층 표 직접 그리기 (계층 / 역할 / 프로토콜 예 / PDU) + OSI 7계층 ↔ TCP/IP 4계층 대응
- [ ] 보강(선택): RFC 1122 §1.1.3 읽고 "왜 실무는 OSI가 아니라 TCP/IP인가"에 답할 수 있으면 완료. Imperva 글은 복습용
- [ ] 정리: `## 핵심 정리` 작성 + 소마 백엔드(영수증 API·HTTP 통신)에서 어느 계층을 다루고 있는지 한 줄 연결

## 기록

(없음)

## 참고 자료

- [What is the OSI Model? — Imperva](https://www.imperva.com/learn/application-security/osi-model/) — 7계층을 예시와 함께 짧게 설명, 첫 진입용 (2026-08-15 확인)
- [RFC 1122 — Requirements for Internet Hosts](https://www.rfc-editor.org/rfc/rfc1122) — 실제 인터넷 계층 모델의 원전. §1.1.3만 읽어도 충분 (2026-08-15 확인)
- [Computer Networking: A Top-Down Approach (Kurose & Ross) 강의 자료](https://gaia.cs.umass.edu/kurose_ross/) — 네트워크 표준 교과서의 무료 강의·인터랙티브 자료. Ch.1.5 "Protocol Layers" (2026-08-15 확인)
- [MDN — An overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) — 응용 계층 대표 프로토콜을 계층 관점에서 보는 예 (2026-08-15 확인)
