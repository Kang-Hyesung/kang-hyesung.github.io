---
title: 10단계 - 실전 시나리오 판단 훈련
date: 2026-06-24 09:09 +0900
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

이 단계는 앞 단계의 개념을 실제 시험 문제처럼 적용하는 훈련이다.

AB-620 문제는 보통 다음 형태로 나온다.

```text
요구사항이 주어진다.
제약조건이 주어진다.
가장 적합한 Copilot Studio 구성 또는 통합 방식을 고른다.
```

따라서 최종 단계에서는 "단어 암기"보다 "상황 판단"이 중요하다.

## 1. 실전 판단 순서

문제를 보면 다음 순서로 판단한다.

1. 사용자가 누구인가?  
   내부 직원인가, 외부 고객인가?

2. 요구가 답변인가, 작업 실행인가?  
   문서 기반 답변이면 knowledge, 시스템 작업이면 tool.

3. 정해진 대화 절차가 있는가?  
   있으면 topic.

4. 외부 시스템 연결 방식은 무엇인가?  
   connector, custom connector, REST API, MCP, A2A 중 선택.

5. 다른 agent에게 위임해야 하는가?  
   Foundry, Fabric, existing agent, A2A를 검토.

6. 보안 요구가 있는가?  
   인증, credential, DLP, governance 확인.

7. 운영 요구가 있는가?  
   evaluation, Application Insights, ALM 확인.

## 2. 빠른 선택표

| 문제 단서 | 정답 후보 |
| --- | --- |
| 문서 기반 답변 | Knowledge source / Generative answers |
| 정해진 절차 | Topic |
| 외부 시스템 작업 | Tool |
| 여러 단계 업무 자동화 | Agent flow |
| 기존 SaaS 연결 | Connector |
| 조직 API 재사용 | Custom connector |
| OpenAPI endpoint | REST API tool |
| MCP server | MCP |
| GUI만 있는 legacy app | Computer use |
| 다른 agent에게 위임 | A2A / Connected agent |
| Foundry agent ID | Foundry agent |
| Fabric semantic model | Fabric data agent |
| 운영 telemetry | Application Insights |
| 반복 품질 테스트 | Test set / Evaluation |
| 환경별 설정값 | Environment variable |
| dev/test/prod 배포 | Solution / Pipeline |

## 3. 시나리오 1 - 내부 HR 정책 agent

## 문제

직원들이 Microsoft Teams에서 HR 정책을 질문한다.  
정책 문서는 SharePoint에 있고, 직원은 자신이 권한을 가진 문서에 대해서만 답변을 받아야 한다.

## 판단

- 내부 직원 → Authenticate with Microsoft
- Teams → Teams channel
- 문서 기반 답변 → Knowledge source / Generative answers
- SharePoint → SharePoint knowledge source
- 사용자별 권한 → End user credentials 또는 permission-aware access

## 정답 구성

- Authenticate with Microsoft
- Add SharePoint as knowledge source
- Use generative answers
- Ensure user permissions are respected
- Test with user profiles

## 4. 시나리오 2 - 휴가 신청 자동화

## 문제

직원이 휴가 시작일, 종료일, 사유를 입력하면 manager approval을 거쳐 Dataverse에 신청이 저장되어야 한다.

## 판단

- 여러 입력값 수집 → Topic
- 업무 자동화 → Agent flow
- manager approval → Human-in-the-loop
- 저장 → Dataverse connector/action
- 결과 반환 → output parameter

## 정답 구성

- Create topic to collect required values
- Create human-in-the-loop agent flow
- Add input/output parameters
- Save request to Dataverse
- Return approval status to agent

## 5. 시나리오 3 - 주문 상태 조회

## 문제

고객이 주문 번호를 입력하면 외부 주문 시스템의 REST API에서 배송 상태를 조회한다. API는 OpenAPI specification을 제공한다.

## 판단

- 시스템 조회 → Tool
- REST API + OpenAPI → REST API tool
- 주문 번호 필요 → input parameter
- 결과 표시 → response formatting 또는 adaptive card

## 정답 구성

- Add REST API tool
- Upload OpenAPI specification
- Configure authentication
- Define tool and parameter descriptions
- Use topic to collect order number if needed

## 6. 시나리오 4 - 조직 내부 API 재사용

## 문제

회사 내부 고객 관리 API를 여러 Copilot Studio agent와 Power Apps에서 재사용해야 한다. Maker가 API endpoint를 직접 다루지 않게 하고 싶다.

## 판단

- 조직 API → custom connector
- 여러 app/agent 재사용 → custom connector
- 표준화된 인증/action → custom connector

## 정답 구성

- Create custom connector
- Define actions and authentication
- Share connector with the organization
- Add custom connector as a tool

## 7. 시나리오 5 - MCP server 통합

## 문제

외부 개발팀이 MCP server를 제공한다. 이 서버는 여러 검색 tool과 resource를 노출하며, Copilot Studio agent가 이를 사용해야 한다.

## 판단

- MCP server → MCP
- tools/resources → MCP
- API endpoint 단순 호출이 아님 → REST API 아님

## 정답 구성

- Connect to MCP server
- Review available MCP tools/resources
- Configure descriptions
- Test tool invocation

## 8. 시나리오 6 - API 없는 레거시 앱

## 문제

오래된 데스크톱 애플리케이션에서만 고객 계정 상태를 변경할 수 있다. API나 connector는 없다.

## 판단

- API 없음 → REST API 아님
- connector 없음 → connector 아님
- 데스크톱 GUI → Computer use

## 정답 구성

- Configure computer use
- Provide monitored execution environment
- Add confirmation before sensitive updates
- Monitor runs

## 9. 시나리오 7 - Fabric 매출 분석

## 문제

사용자가 "지난 분기 지역별 매출 추이를 알려줘"라고 질문하면 Fabric semantic model 기반으로 답변해야 한다.

## 판단

- Fabric semantic model → Fabric data agent
- 데이터 분석 질의 → Fabric data agent
- 문서 검색이 아님 → Azure AI Search 우선 아님

## 정답 구성

- Publish Fabric data agent
- Connect Fabric data agent to Copilot Studio agent
- Provide clear description
- Ensure permissions and tenant alignment

## 10. 시나리오 8 - Foundry 전문 agent 호출

## 문제

Azure AI Foundry에서 만든 전문 agent가 이미 있다. Copilot Studio agent는 특정 분석 요청을 이 Foundry agent에게 위임해야 한다.

## 판단

- Foundry agent 존재 → Connect Microsoft Foundry agent
- Agent ID, endpoint 필요
- 다른 agent에게 위임 → connected agent

## 정답 구성

- Create connection to Microsoft Foundry
- Provide Foundry project endpoint
- Enter Agent ID
- Write clear description for invocation
- Test connected agent

## 11. 시나리오 9 - 외부 A2A agent

## 문제

외부 공급사가 Agent2Agent protocol을 지원하는 agent를 제공한다. Copilot Studio agent는 사용자 요청 일부를 해당 agent에게 위임해야 한다.

## 판단

- Agent2Agent protocol → A2A
- 외부 agent에게 task 위임 → A2A
- 단순 API 호출 아님 → REST API 아님
- MCP server 아님 → MCP 아님

## 정답 구성

- Connect agent over A2A protocol
- Configure connection
- Test delegation
- Review data handling and observability

## 12. 시나리오 10 - 답변 품질 회귀 방지

## 문제

Agent prompt와 knowledge source를 수정한 후, 기존 질문들에 대한 답변 품질이 떨어지지 않았는지 반복 검증해야 한다.

## 판단

- 반복 검증 → Test set
- 수정 전후 비교 → Evaluation results comparison
- 답변 품질 → Quality 또는 Similarity

## 정답 구성

- Create test set
- Choose evaluation methods
- Run evaluation before/after changes
- Review pass rate and failed cases

## 13. 시나리오 11 - 운영 오류 분석

## 문제

운영 중 agent가 특정 topic에서 자주 실패한다. 어떤 node가 실행되었고 어떤 exception이 발생했는지 확인해야 한다.

## 판단

- 운영 telemetry → Application Insights
- node execution events → Application Insights
- exception/latency → Application Insights

## 정답 구성

- Connect agent to Application Insights
- Enable logging/node execution events
- Query telemetry
- Review exceptions and conversation flow

## 14. 시나리오 12 - 환경별 API URL

## 문제

개발 환경에서는 dev API를, 운영 환경에서는 prod API를 호출해야 한다. Agent나 topic을 환경마다 수정하고 싶지 않다.

## 판단

- 환경별 값 → Environment variable
- hardcoding 방지 → Environment variable
- ALM → Solution에 포함

## 정답 구성

- Create environment variable
- Use it for API endpoint
- Add environment variable to solution
- Set different values per environment

## 15. 시나리오 13 - 배포 자동화

## 문제

Agent를 development 환경에서 만든 뒤 test와 production으로 일관되게 배포하고 싶다.

## 판단

- 환경 간 이동 → Solution
- 자동 배포 → Power Platform Pipelines
- 운영 배포 → Managed solution

## 정답 구성

- Add agent to solution
- Add required objects
- Use Power Platform Pipelines
- Deploy managed solution downstream

## 16. 시나리오 14 - 인증 없는 agent 차단

## 문제

보안팀은 maker가 인증 없는 agent를 publish하지 못하게 해야 한다.

## 판단

- 조직 정책 강제 → DLP/data policy
- No authentication 차단 → data policy

## 정답 구성

- Configure Power Platform data policy
- Block unauthenticated usage

## 17. 시나리오 15 - 민감 작업 전 사용자 확인

## 문제

Agent가 고객 계정을 비활성화할 수 있지만, 실행 전 사용자가 반드시 확인해야 한다.

## 판단

- 민감한 변경 작업 → Tool confirmation
- 실행 전 확인 → Ask end user before running

## 정답 구성

- Configure tool to ask the end user before running
- Consider human-in-the-loop for stronger control

## 18. 마지막 실전 판단 공식

시험장에서 헷갈리면 다음 순서로 좁힌다.

```text
1. 답변인가 작업인가?
   - 답변: Knowledge / Generative answers
   - 작업: Tool / Agent flow

2. 작업이면 어떤 연결인가?
   - 기존 서비스: Connector
   - 조직 API 재사용: Custom connector
   - OpenAPI endpoint: REST API tool
   - MCP server: MCP
   - GUI only: Computer use

3. 다른 agent인가?
   - Foundry agent: Foundry
   - Fabric data: Fabric data agent
   - A2A 지원 외부 agent: A2A

4. 운영/배포 문제인가?
   - 품질 평가: Evaluation
   - 로그/오류: Application Insights
   - 환경별 값: Environment variable
   - 배포: Solution / Pipeline
```

## 19. 최종 복습 체크리스트

- [ ] Topic과 Knowledge를 구분할 수 있다.
- [ ] Knowledge와 Tool을 구분할 수 있다.
- [ ] Agent flow와 Topic을 구분할 수 있다.
- [ ] Connector, Custom connector, REST API tool을 구분할 수 있다.
- [ ] MCP, Computer use, A2A를 구분할 수 있다.
- [ ] Foundry agent와 Fabric data agent를 구분할 수 있다.
- [ ] Authentication과 tool credential을 구분할 수 있다.
- [ ] DLP policy가 필요한 상황을 찾을 수 있다.
- [ ] Evaluation과 Application Insights를 구분할 수 있다.
- [ ] Solution, Environment variable, Pipeline을 연결해서 설명할 수 있다.

## 20. 시험 직전 암기 문장

> AB-620 문제는 대부분 "사용자 요청을 처리하기 위해 topic, knowledge, tool, agent 중 무엇을 쓰고, 그 통합을 어떻게 보안/평가/배포할 것인가"로 환원된다.

