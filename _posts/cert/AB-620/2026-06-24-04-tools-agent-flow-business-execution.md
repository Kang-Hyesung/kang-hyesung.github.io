---
title: 04단계 - Tools, Agent Flow, 업무 실행
date: 2026-06-24 09:03 +0900
author: hyesung
categories: CERT AB-620
tags:
  - AB-620
  - Copilot Studio
  - Microsoft Certification
  - AI Agent
---

## 학습 목표

이 단계에서는 agent가 단순히 답변하는 수준을 넘어 실제 업무를 수행하는 방법을 학습한다.

핵심 구분은 다음이다.

> 지식으로 답하면 Knowledge, 시스템에 일을 시키면 Tool.

## 1. Tool이란?

Tool은 agent가 외부 시스템에서 데이터를 가져오거나 실제 작업을 수행하게 하는 기능이다.

예:

- 주문 상태 조회
- 티켓 생성
- 고객 정보 업데이트
- 이메일 전송
- API 호출
- agent flow 실행
- connector action 실행
- MCP tool 실행

## 2. Tool을 쓰는 상황

다음 표현이 나오면 tool을 떠올린다.

- retrieve data
- perform a task
- create a record
- update a record
- call an API
- send an email
- submit a request
- check current status

## 3. Tool의 주요 유형

| Tool 유형 | 의미 | 대표 시나리오 |
| --- | --- | --- |
| Prompt | AI 모델에게 특정 작업 지시 | 요약, 분류, 추출 |
| Agent flow | 여러 단계 업무 자동화 | 승인, 알림, 시스템 업데이트 |
| Connector | 표준 서비스 연결 | SharePoint, ServiceNow, Outlook |
| Custom connector | 조직 API를 connector로 사용 | 내부 업무 API |
| REST API | OpenAPI 기반 API 호출 | 특정 endpoint 호출 |
| MCP | MCP server의 tools/resources 사용 | 외부 tool 생태계 연결 |
| Computer use | GUI 화면 조작 | API 없는 legacy app |

## 4. Tool 설명이 중요한 이유

Generative orchestration은 tool의 이름, 설명, 입력/출력 설명을 보고 어떤 tool을 사용할지 판단한다.

나쁜 설명:

```text
Gets info.
```

좋은 설명:

```text
Use this tool when the user asks for the current shipping status of an order by order number.
```

시험 단서:

- agent invokes the wrong tool
- tool selection is inaccurate
- tools have overlapping descriptions
- improve orchestration

정답 후보:

- Update tool description
- Provide clear input/output descriptions
- Make metadata specific

## 5. Tool 입력과 출력

Tool은 입력값과 출력값이 명확해야 한다.

예:

```text
Tool: GetOrderStatus
Input:
- orderId
- customerEmail

Output:
- orderStatus
- expectedDeliveryDate
- trackingNumber
```

시험에서 입력/출력은 다음과 연결된다.

- 사용자가 제공해야 할 값
- agent가 되물어야 할 값
- flow/API에 전달할 parameter
- tool 실행 후 대화에서 사용할 결과

## 6. Agent Flow란?

Agent flow는 Copilot Studio에서 만드는 업무 자동화 흐름이다.

특징:

- 여러 action을 순서대로 실행
- 조건과 반복 사용 가능
- connector와 action 조합 가능
- agent에 tool로 추가 가능
- deterministic한 업무 처리에 적합

쉽게 말하면:

> Agent가 호출할 수 있는 low-code 업무 자동화 프로세스.

## 7. Agent Flow를 쓰는 상황

다음 요구가 나오면 agent flow를 고려한다.

- 여러 단계의 업무 처리
- 승인 흐름
- connector 여러 개 조합
- 조건별 분기
- 오류 처리
- 반복 작업 자동화

예:

```text
사용자가 휴가 신청
→ employee ID 확인
→ 잔여 연차 조회
→ manager approval 요청
→ Dataverse에 신청 저장
→ Teams로 결과 알림
```

## 8. Agent Flow vs Topic

| 구분 | Agent Flow | Topic |
| --- | --- | --- |
| 중심 | 업무 자동화 | 대화 흐름 |
| 역할 | 시스템 작업 수행 | 사용자와 대화 진행 |
| 예시 | 승인 요청, 레코드 생성 | 날짜 입력 받기, 분기 |
| 시험 단서 | automate process, actions, connectors | guide conversation, trigger phrases |

시험 암기:

> 사용자와 대화하는 흐름은 Topic, 시스템에서 일하는 흐름은 Agent flow.

## 9. Agent Flow 입력/출력

Agent flow를 agent가 호출하려면 입력/출력이 필요하다.

예:

```text
Input:
- employeeId
- startDate
- endDate
- reason

Output:
- requestId
- approvalStatus
- approverName
```

시험 단서:

- add input parameters
- add output parameters
- respond to the agent
- pass values to the flow
- return values to the conversation

## 10. Human-in-the-loop

Human-in-the-loop은 업무 흐름 중 사람이 승인하거나 검토하는 단계다.

필요한 경우:

- 결제 승인
- 환불 승인
- 민감 정보 변경
- 정책 예외 승인
- AI 판단 결과 검토

시험 단서:

- requires human approval
- manager must review
- human must confirm before proceeding
- manual review step

정답 후보:

- Create a human-in-the-loop agent flow
- Add approval step

## 11. Error Handling

업무 실행에는 실패 가능성이 있다.

예:

- API timeout
- connector 인증 실패
- 필수 입력 누락
- 권한 없음
- 외부 시스템 오류
- 잘못된 데이터 형식

시험에서 error handling은 다음과 연결된다.

- 실패 시 사용자에게 이해 가능한 메시지 제공
- 재시도 또는 대체 흐름 구성
- 관리자에게 알림
- 로그 기록
- flow run 모니터링

## 12. Tool 실행 전 사용자 확인

다음 작업은 실행 전 확인이 필요할 수 있다.

- 데이터 삭제
- 주문 취소
- 고객 정보 변경
- 결제 실행
- 외부 시스템에 공식 요청 제출

시험 단서:

- ask the end user before running
- avoid accidental updates
- user must confirm

정답 후보:

- Configure the tool to ask the user before running

## 13. Prompt Tool

Prompt tool은 특정 AI 작업을 수행하는 tool이다.

예:

- 텍스트 요약
- 감정 분석
- 카테고리 분류
- JSON 추출
- 고객 문의 응답 초안 작성

Prompt tool은 다음과 다르다.

| 구분 | Prompt Tool | Generative Answers |
| --- | --- | --- |
| 목적 | 특정 AI 작업 수행 | 지식 검색 기반 답변 |
| 입력 | 지정된 텍스트/데이터 | 사용자 질문 |
| 출력 | 정해진 형식 가능 | 자연어 답변 |

## 14. Computer Use

Computer use는 GUI 기반 작업 자동화다.

적합한 경우:

- API 없음
- connector 없음
- desktop-only app
- 웹 화면 조작 필요
- legacy system

시험 단서:

- graphical user interface
- buttons, menus, text fields
- no API available
- legacy application

정답 후보:

- Configure computer use

## 15. 시험 문제 풀이 패턴

## 패턴 1: 주문 조회

요구:

- 사용자가 주문 번호 입력
- 현재 배송 상태 조회
- 외부 API 사용

정답:

- Topic으로 주문 번호 수집
- REST API tool 또는 connector 호출
- output을 사용자에게 표시

## 패턴 2: 휴가 신청 자동화

요구:

- 여러 단계 업무
- manager approval
- Dataverse 저장
- Teams 알림

정답:

- Agent flow
- Human-in-the-loop
- Input/output parameters
- Error handling

## 패턴 3: API 없는 레거시 앱

요구:

- 데스크톱 앱 화면에서만 처리 가능

정답:

- Computer use

## 패턴 4: 답변을 JSON으로 변환

요구:

- 고객 문의를 카테고리/우선순위/요약으로 변환

정답:

- Prompt tool

## 4단계 복습 질문

**Q1. Tool과 Knowledge의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Knowledge는 문서나 데이터 원본을 검색해 답변하는 데 사용한다. Tool은 외부 시스템 조회, 레코드 생성, 업데이트, 이메일 전송, API 호출처럼 실제 업무를 실행할 때 사용한다.
</div>
</details>

**Q2. Agent flow와 Topic의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Topic은 사용자와의 대화 흐름을 제어한다. Agent flow는 connector action, 승인, 조건 분기, 시스템 업데이트 등 여러 단계의 업무 자동화를 수행한다.
</div>
</details>

**Q3. Human-in-the-loop은 어떤 상황에서 필요한가?**

<details>
<summary>정답 확인</summary>
<div>
결제 승인, 환불 승인, 민감 정보 변경, 정책 예외 승인처럼 사람이 검토하거나 승인해야 하는 업무에서 필요하다.
</div>
</details>

**Q4. Tool description이 중요한 이유는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Generative orchestration은 tool 이름, 설명, 입력/출력 설명을 보고 어떤 tool을 사용할지 판단한다. 설명이 모호하면 agent가 잘못된 tool을 선택하거나 입력값을 잘못 채울 수 있다.
</div>
</details>

**Q5. Computer use를 쓰는 대표 조건은 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
API나 connector가 없고, legacy 웹/데스크톱 애플리케이션의 GUI 화면을 조작해야 할 때 사용한다.
</div>
</details>
