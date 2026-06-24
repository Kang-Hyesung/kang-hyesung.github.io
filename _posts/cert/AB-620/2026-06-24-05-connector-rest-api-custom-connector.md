---
title: 05단계 - Connector, REST API, Custom Connector
date: 2026-06-24 09:04 +0900
author: hyesung
categories: CERT AB-620
tags:
  - AB-620
  - Copilot Studio
  - Microsoft Certification
  - AI Agent
---

## 학습 목표

이 단계에서는 AB-620에서 가장 자주 나올 수 있는 통합 방식 세 가지를 구분한다.

1. Connector
2. Custom connector
3. REST API tool

핵심 질문은 다음이다.

> 외부 시스템과 연결할 때 이미 표준 connector가 있는가, 조직 API를 재사용해야 하는가, 아니면 REST endpoint를 직접 호출하면 되는가?

## 1. Connector란?

Connector는 Microsoft 또는 외부 서비스와 연결하기 위한 Power Platform의 표준 연결 방식이다.

예:

- SharePoint
- Outlook
- Teams
- Dataverse
- ServiceNow
- Salesforce
- SAP
- SQL Server
- Azure DevOps

Connector는 인증, 연결, action, governance를 표준화한다.

## 2. Connector를 쓰는 상황

다음 상황이면 connector를 우선 고려한다.

- 이미 해당 서비스의 connector가 있다.
- 표준 action으로 요구사항을 충족할 수 있다.
- Power Platform DLP 정책의 통제를 받아야 한다.
- 사용자 또는 maker credential을 사용해야 한다.

시험 단서:

- existing connector
- Power Platform connector
- Microsoft 365 service
- ServiceNow, SAP, Salesforce
- use connector as a tool

정답 후보:

- Add connector as a tool
- Use existing Power Platform connector

## 3. Custom Connector란?

Custom connector는 조직의 API를 Power Platform connector로 정의한 것이다.

쉽게 말하면:

> 내부 API를 Power Platform과 Copilot Studio에서 재사용 가능한 표준 connector로 포장한 것.

## 4. Custom Connector를 쓰는 상황

다음 상황에서 적합하다.

- 조직 내부 API를 여러 agent/app/flow에서 재사용
- API 인증과 action 정의를 표준화
- maker가 API 세부 구현을 몰라도 사용 가능
- Power Platform 정책으로 통제 필요
- 기존 custom connector가 이미 있음

시험 단서:

- proprietary API
- internal line-of-business API
- reusable API
- existing custom connector
- share with organization

정답 후보:

- Create a custom connector
- Add existing custom connector as a tool
- Share connector/connection permissions

## 5. REST API Tool이란?

REST API tool은 OpenAPI specification을 기반으로 REST API endpoint를 Copilot Studio agent tool로 추가하는 방식이다.

## 6. REST API Tool을 쓰는 상황

다음 상황에서 적합하다.

- API endpoint를 agent가 직접 호출해야 함
- OpenAPI specification이 있음
- 특정 method/endpoint를 선택해 tool로 노출해야 함
- custom connector까지 만들 필요는 없음

시험 단서:

- REST API
- OpenAPI specification
- endpoint
- method
- upload API specification
- provide authentication details

정답 후보:

- Add REST API tool
- Provide OpenAPI specification
- Configure authentication and descriptions

## 7. 세 가지 통합 방식 비교

| 구분 | Connector | Custom connector | REST API tool |
| --- | --- | --- | --- |
| 대상 | 이미 제공되는 서비스 | 조직/전용 API | 특정 REST API |
| 재사용성 | 높음 | 매우 높음 | 중간 |
| 설정 난이도 | 낮음 | 중간~높음 | 중간 |
| 거버넌스 | Power Platform 정책 | Power Platform 정책 | tool/API 설정 중심 |
| 시험 단서 | existing connector | internal API, reusable | OpenAPI, endpoint |

## 8. 선택 우선순위

시험에서 외부 시스템 연결 문제가 나오면 다음 순서로 생각한다.

1. 이미 표준 connector가 있는가?
2. 조직 API를 여러 곳에서 재사용해야 하는가?
3. 특정 REST endpoint를 직접 호출하면 되는가?
4. 외부 시스템이 MCP server로 제공되는가?
5. 상대가 API가 아니라 agent인가?

이 단계에서는 1-3번을 다룬다.

## 9. Authentication 고려

외부 연결은 항상 인증을 고려해야 한다.

## 9.1 End user credentials

사용자의 권한으로 실행한다.

적합한 경우:

- 사용자별 데이터 접근
- 각 사용자가 볼 수 있는 record가 다름
- 감사 로그에서 사용자 단위 추적 필요

시험 단서:

- user permissions
- only authorized data
- per-user access

## 9.2 Maker-provided credentials

제작자의 credential 또는 공유 connection으로 실행한다.

적합한 경우:

- 사용자별 계정이 없음
- 공통 서비스 계정 사용
- 모든 사용자가 같은 공유 리소스 접근

시험 단서:

- users should not sign in individually
- shared connection
- maker credentials

## 10. Tool 설명과 API 설명

REST API tool이나 connector action을 추가할 때 description은 매우 중요하다.

Generative orchestration은 설명을 보고 tool을 선택한다.

좋은 설명에는 다음이 포함되어야 한다.

- 어떤 업무를 하는 tool인지
- 언제 사용해야 하는지
- 어떤 입력값이 필요한지
- 출력값이 무엇인지
- 비슷한 tool과 어떻게 다른지

## 11. API Parameter 설명

Parameter 설명이 빈약하면 agent가 값을 잘못 채우거나 사용자에게 잘못 질문할 수 있다.

예:

나쁜 설명:

```text
id
```

좋은 설명:

```text
The unique order number provided to the customer after purchase. Use this when checking shipping or delivery status.
```

## 12. Connection과 권한

Connector를 사용하려면 connection이 필요하다.

시험에서 볼 포인트:

- connection이 올바르게 생성되어야 함
- connector가 조직에 공유되어야 할 수 있음
- custom connector는 view/share 권한이 필요할 수 있음
- DLP 정책이 connector 사용을 막을 수 있음

## 13. DLP 정책과 Connector

관리자는 Power Platform DLP 정책으로 connector 사용을 제한할 수 있다.

예:

- 특정 connector 차단
- HTTP request 차단
- public data source 차단
- 인증 없는 agent 차단
- 특정 channel publish 차단

시험 단서:

- admin must block
- prevent makers from using
- data loss prevention
- connector policy

정답 후보:

- Configure DLP policy

## 14. 시험 문제 풀이 패턴

## 패턴 1: ServiceNow 티켓 생성

요구:

- ServiceNow에 티켓 생성
- 이미 Power Platform connector 있음

정답:

- ServiceNow connector를 tool로 추가

## 패턴 2: 사내 주문 API 재사용

요구:

- 여러 agent와 Power Apps에서 같은 API 사용
- 인증과 action을 표준화

정답:

- Custom connector 생성

## 패턴 3: 특정 API endpoint 호출

요구:

- OpenAPI specification 있음
- 한 agent에서 배송 상태 API 호출

정답:

- REST API tool 추가

## 패턴 4: 사용자 권한별 CRM 조회

요구:

- 사용자별로 조회 가능한 고객 정보가 다름

정답:

- Connector/tool을 end user credentials로 실행

## 패턴 5: 공용 API 조회

요구:

- 모든 사용자가 같은 API key로 날씨 정보 조회
- 사용자별 로그인 불필요

정답:

- Maker-provided credentials

## 05단계 복습 질문

**Q1. Connector와 Custom connector의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Connector는 Microsoft나 외부 서비스에 대해 이미 제공되는 표준 연결 방식이다. Custom connector는 조직 내부 API나 전용 API를 Power Platform에서 재사용 가능한 connector로 정의한 것이다.
</div>
</details>

**Q2. Custom connector와 REST API tool의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Custom connector는 조직 API를 여러 agent, app, flow에서 재사용하도록 표준화하는 방식이다. REST API tool은 OpenAPI specification을 기반으로 특정 REST endpoint를 Copilot Studio agent tool로 직접 추가하는 방식이다.
</div>
</details>

**Q3. OpenAPI specification이라는 단어가 나오면 무엇을 떠올려야 하는가?**

<details>
<summary>정답 확인</summary>
<div>
REST API tool을 떠올려야 한다. OpenAPI specification을 업로드해 endpoint, method, parameter, authentication을 설정하고 agent tool로 노출한다.
</div>
</details>

**Q4. 사용자별 권한이 중요한 경우 어떤 credential 방식을 써야 하는가?**

<details>
<summary>정답 확인</summary>
<div>
End user credentials를 사용해야 한다. tool이 최종 사용자의 권한으로 실행되어 사용자가 접근 권한이 있는 데이터만 조회하거나 변경하도록 보장한다.
</div>
</details>

**Q5. DLP policy는 connector 사용과 어떤 관계가 있는가?**

<details>
<summary>정답 확인</summary>
<div>
DLP policy는 특정 connector나 HTTP request 사용을 허용하거나 차단해 데이터 유출을 방지한다. 어떤 connector를 어떤 환경에서 사용할 수 있는지 통제하는 governance 수단이다.
</div>
</details>
