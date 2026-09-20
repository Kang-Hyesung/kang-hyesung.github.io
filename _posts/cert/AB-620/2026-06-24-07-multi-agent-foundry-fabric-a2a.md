---
title: 07단계 - Multi-Agent, Foundry, Fabric, A2A
date: 2026-06-24 09:06 +0900
author: hyesung
published: false
categories: CERT AB-620
tags:
  - AB-620
  - Copilot Studio
  - Microsoft Certification
  - AI Agent
---

## 학습 목표

이 단계에서는 여러 agent가 협업하는 구조를 학습한다.

핵심 질문은 다음이다.

> 이 요구사항은 API 호출인가, tool 사용인가, 아니면 다른 전문 agent에게 task를 위임해야 하는가?

## 1. Multi-agent solution이란?

Multi-agent solution은 하나의 agent가 모든 일을 처리하지 않고, 여러 agent가 역할을 나누어 협업하는 구조다.

예:

```text
Main HR Agent
→ Payroll Agent
→ Benefits Agent
→ Policy Search Agent
→ Approval Agent
```

장점:

- 역할 분리
- 전문성 강화
- 재사용성 증가
- 복잡도 감소
- 확장 쉬움

## 2. 언제 Multi-agent를 쓰는가?

다음 상황에서 적합하다.

- 업무 영역이 여러 개로 나뉨
- 각 영역에 전문 agent가 있음
- 외부 agent가 이미 존재함
- 메인 agent가 전체 요청을 받아 적절한 agent에게 위임해야 함
- 복잡한 agentic workflow가 필요함

시험 단서:

- multi-agent collaboration
- delegate to a specialized agent
- connect to another agent
- domain-specific agent
- modular design

정답 후보:

- Add/connect another agent
- Design multi-agent solution

## 3. Connected Agent 유형

| 유형 | 의미 |
| --- | --- |
| Existing Copilot Studio agent | 이미 만든 Copilot Studio agent 연결 |
| Microsoft Foundry agent | Foundry에서 만든 agent 연결 |
| Fabric data agent | Microsoft Fabric 데이터 질의 agent 연결 |
| A2A agent | Agent2Agent protocol을 지원하는 외부 agent 연결 |

## 4. Existing Copilot Studio Agent

이미 Copilot Studio에서 만든 agent를 다른 agent에 연결하는 방식이다.

적합한 경우:

- 조직 내에 이미 전문 agent가 있음
- 같은 tenant/환경에서 재사용
- 메인 agent가 특정 domain 업무를 위임

시험 단서:

- existing Copilot Studio agent
- reuse another agent
- connect an existing agent

정답:

- Integrate existing Copilot Studio agent

## 5. Microsoft Foundry Agent

Microsoft Foundry agent는 Microsoft Foundry/Azure AI Foundry에서 만든 agent다.

Copilot Studio agent는 Foundry agent를 연결해 특정 작업을 위임할 수 있다.

## 5.1 언제 Foundry agent를 쓰는가?

다음 상황에서 적합하다.

- Foundry에서 이미 agent를 만들었음
- 고급 AI workflow나 모델 구성이 Foundry에 있음
- Copilot Studio agent가 Foundry agent를 호출해야 함
- Foundry project endpoint와 agent ID가 요구됨

시험 단서:

- Microsoft Foundry agent
- Foundry project endpoint
- Agent ID
- external agent
- agent created in Foundry

정답:

- Connect Microsoft Foundry agent

## 5.2 Foundry agent 연결 시 중요한 것

- Foundry project endpoint
- Agent ID
- connection
- name과 description
- main agent가 언제 호출해야 하는지 설명
- 테스트와 보안 검토

## 6. Foundry Model Catalog

Foundry model catalog는 prompt tool에서 특정 모델을 선택할 때 등장한다.

주의:

- Foundry agent 연결과 Foundry model catalog 사용은 다르다.

| 구분 | 의미 |
| --- | --- |
| Foundry agent | Foundry에서 만든 agent를 연결 |
| Foundry model catalog | Prompt tool에서 사용할 모델 선택 |

시험 단서:

- "connect a Foundry agent" → Foundry agent
- "custom prompt should use a model from Foundry" → Foundry model catalog

## 7. Fabric Data Agent

Fabric data agent는 Microsoft Fabric 데이터에 자연어로 질의하는 agent다.

대상 데이터:

- OneLake
- Lakehouse
- Warehouse
- Power BI semantic model
- KQL database
- mirrored database

## 7.1 언제 Fabric data agent를 쓰는가?

다음 상황에서 적합하다.

- 사용자가 Fabric 데이터에 대해 질문
- KPI, 매출, 재고, 분석 지표 질의
- semantic model 기반 답변
- 데이터 분석 agent를 Copilot Studio agent와 연결

시험 단서:

- Fabric data agent
- OneLake
- semantic model
- warehouse
- lakehouse
- natural-language insights

정답:

- Connect Microsoft Fabric data agent

## 8. Fabric Data Agent vs Azure AI Search

| 구분 | Fabric Data Agent | Azure AI Search |
| --- | --- | --- |
| 중심 | Fabric 데이터 분석 질의 | 검색 인덱스 기반 RAG |
| 데이터 | semantic model, lakehouse, warehouse | 문서, 인덱싱된 콘텐츠 |
| 문제 단서 | KPI, Power BI, OneLake | search index, semantic search |
| 정답 느낌 | 데이터 agent 연결 | knowledge/RAG 구성 |

시험 암기:

> Fabric 데이터 분석은 Fabric data agent, 검색 기반 지식 답변은 Azure AI Search.

## 9. A2A란?

A2A는 Agent2Agent protocol이다.

다른 agent와 통신하고 task를 위임하기 위한 표준 프로토콜이다.

쉽게 말하면:

> API를 호출하는 것이 아니라, 다른 agent에게 일을 맡기는 방식.

## 10. 언제 A2A를 쓰는가?

다음 상황에서 적합하다.

- 외부 agent가 A2A protocol을 지원
- Copilot Studio agent가 외부 agent에게 task 위임
- 외부 agent가 자체 reasoning 또는 workflow 보유
- multi-turn agent interaction 필요
- 풍부한 context metadata 필요

시험 단서:

- Agent2Agent protocol
- A2A
- delegate task to external agent
- agent-to-agent collaboration
- rich metadata
- multi-turn interaction

정답:

- Connect an agent over A2A protocol

## 11. A2A vs REST API

| 구분 | A2A | REST API |
| --- | --- | --- |
| 상대 | 다른 agent | API service |
| 목적 | task 위임 | endpoint 호출 |
| 문맥 | 대화 이력과 metadata 가능 | 요청/응답 중심 |
| 시험 단서 | delegate to agent | call API endpoint |

## 12. A2A vs MCP

| 구분 | A2A | MCP |
| --- | --- | --- |
| 상대 | agent | tool/resource server |
| 목적 | agent 협업 | tool 사용 |
| 시험 단서 | Agent2Agent | Model Context Protocol |

시험 암기:

> 다른 agent는 A2A, 외부 tool server는 MCP.

## 13. Multi-agent 보안 고려

다른 agent를 연결할 때는 다음을 검토해야 한다.

- 데이터 흐름
- 데이터 공유 범위
- 권한 경계
- 신뢰성
- 품질
- 관찰성
- identity
- traceability
- human oversight

시험 단서:

- external agent responsibility
- data handling
- permissions and boundaries
- observability
- traceability

## 14. Description이 중요한 이유

다른 agent도 tool처럼 description이 중요하다.

메인 agent는 description을 보고 언제 해당 agent를 호출할지 판단한다.

좋은 description 예:

```text
Use this agent when the user asks analytical questions about sales, revenue, inventory, or KPI data stored in Microsoft Fabric semantic models.
```

나쁜 description:

```text
Data agent.
```

## 15. 시험 문제 풀이 패턴

## 패턴 1: Foundry agent 재사용

요구:

- Azure AI Foundry에서 만든 agent가 있음
- Copilot Studio agent가 호출해야 함

정답:

- Connect Microsoft Foundry agent
- Provide endpoint and Agent ID

## 패턴 2: Fabric 매출 질의

요구:

- Power BI semantic model 기반 매출 질문
- 자연어 분석 답변

정답:

- Fabric data agent 연결

## 패턴 3: 외부 agent가 A2A 지원

요구:

- 외부 agent가 Agent2Agent protocol 지원
- task 위임 필요

정답:

- A2A connection

## 패턴 4: 외부 tool server

요구:

- 외부 MCP server가 tools/resources 제공

정답:

- MCP tool

## 패턴 5: 같은 조직의 기존 agent

요구:

- 이미 만든 Copilot Studio agent 재사용

정답:

- Integrate existing Copilot Studio agent

## 07단계 복습 질문

**Q1. Foundry agent와 Foundry model catalog의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Foundry agent는 Azure AI Foundry에서 만든 agent를 Copilot Studio agent에 연결해 task를 위임하는 대상이다. Foundry model catalog는 prompt tool에서 사용할 특정 모델을 선택할 때 등장한다.
</div>
</details>

**Q2. Fabric data agent는 어떤 데이터 시나리오에 적합한가?**

<details>
<summary>정답 확인</summary>
<div>
OneLake, Lakehouse, Warehouse, Power BI semantic model, KQL database 같은 Microsoft Fabric 데이터에 대해 자연어로 KPI, 매출, 재고, 분석 지표를 질의하는 시나리오에 적합하다.
</div>
</details>

**Q3. A2A와 REST API의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
A2A는 다른 agent에게 task를 위임하는 agent-to-agent 협업 방식이다. REST API는 agent가 특정 HTTP endpoint를 호출해 요청/응답을 처리하는 방식이다.
</div>
</details>

**Q4. A2A와 MCP의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
A2A는 다른 agent와 연결해 협업한다. MCP는 외부 tool/resource server와 연결해 tools와 resources를 사용한다.
</div>
</details>

**Q5. Connected agent의 description이 중요한 이유는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Main agent는 connected agent의 description을 보고 언제 해당 agent에게 task를 위임할지 판단한다. 설명이 모호하면 잘못된 agent를 호출하거나 위임하지 못할 수 있다.
</div>
</details>
