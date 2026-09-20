---
title: 09단계 - Evaluation, Monitoring, ALM
date: 2026-06-24 09:08 +0900
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

이 단계에서는 만든 agent를 테스트하고, 운영 중 모니터링하고, 환경 간에 배포하는 방법을 학습한다.

핵심 구분:

| 목적 | 개념 |
| --- | --- |
| 답변 품질 평가 | Evaluation, Test set |
| 운영 로그/오류 추적 | Application Insights |
| 환경 간 배포 관리 | ALM, Solution, Pipeline |

## 1. Test Chat

Test chat은 maker가 agent와 직접 대화하면서 동작을 확인하는 기능이다.

확인할 수 있는 것:

- topic이 시작되는지
- node가 예상대로 실행되는지
- tool이 호출되는지
- 응답이 맞는지
- activity map으로 흐름 확인

시험 단서:

- manually test conversation
- see which node fired
- track between topics
- activity map

정답:

- Use Test your agent panel

## 2. Evaluation

Evaluation은 agent 품질을 test set으로 반복 평가하는 기능이다.

Test chat이 수동 확인이라면, evaluation은 반복 가능한 품질 검증이다.

## 3. Test Case와 Test Set

| 개념 | 의미 |
| --- | --- |
| Test case | 하나의 질문 또는 대화 테스트 |
| Test set | 여러 test case 묶음 |
| Expected response | 기대 응답 |
| Test result | Pass, Fail, Error, Invalid 등 |

## 4. Test Set을 만드는 방식

가능한 방식:

- 수동 작성
- agent instructions/capabilities/knowledge에서 생성
- 과거 test chat에서 생성
- CSV import

시험 단서:

- create a test set
- import test cases
- generate from knowledge
- use past conversations

## 5. Evaluation Methods

중요 평가 방식 세 가지:

| 방법 | 의미 | 적합한 상황 |
| --- | --- | --- |
| Text match | 정확한 문구 일치 또는 포함 | 고정 문구, disclaimer |
| Similarity | 의미 유사성 비교 | 표현은 달라도 뜻이 같으면 됨 |
| Quality | 품질 평가 | relevance, groundedness, completeness |

## 5.1 Text Match

정확한 단어나 문구가 필요할 때 쓴다.

예:

- "Your ticket has been created"
- "Contact HR for final confirmation"
- 특정 번호 포함

## 5.2 Similarity

정답 표현이 여러 가지일 수 있을 때 쓴다.

예:

- 업무 시간 안내
- 정책 요약
- 제품 설명

## 5.3 Quality

생성형 답변 품질을 평가할 때 쓴다.

평가 관점:

- relevance
- groundedness
- completeness
- abstention

시험 암기:

> 고정 문구는 Text match, 의미는 Similarity, 생성형 답변 품질은 Quality.

## 6. Test Result 분석

Evaluation 결과에서 확인할 수 있는 것:

- pass rate
- 실패한 test case
- agent의 실제 응답
- expected response
- 사용한 topic, tool, knowledge
- activity map
- 이전 실행과 비교

시험 단서:

- review test results
- compare runs over time
- identify regression
- see resources used

## 7. User Profile과 Connection

Evaluation은 사용자 profile과 connection에 따라 결과가 달라질 수 있다.

중요한 이유:

- 사용자의 권한이 다름
- connector connection이 작동해야 함
- knowledge access가 사용자별로 다름

시험 단서:

- simulate user permissions
- user profile
- connections must be working

## 8. Application Insights

Application Insights는 Azure Monitor의 APM 도구로, agent telemetry를 수집한다.

수집 가능한 것:

- incoming/outgoing messages
- events
- triggered topics
- node execution events
- custom telemetry
- exceptions
- latency
- tool usage

시험 단서:

- monitor agents
- capture telemetry
- Application Insights
- latency
- exceptions
- custom events

정답:

- Connect agent to Application Insights

## 9. Evaluation vs Application Insights

| 구분 | Evaluation | Application Insights |
| --- | --- | --- |
| 목적 | 답변 품질 검증 | 운영 telemetry 수집 |
| 시점 | 테스트/개선 과정 | 운영 중 |
| 결과 | pass/fail, score | logs, traces, metrics |
| 시험 단서 | test set, evaluation method | telemetry, exceptions, latency |

시험 암기:

> 품질 평가는 Evaluation, 운영 관찰은 Application Insights.

## 10. ALM이란?

ALM은 Application Lifecycle Management다.

Agent를 개발, 테스트, 배포, 운영하는 전체 생명주기 관리다.

핵심 구성:

- Environment
- Solution
- Environment variable
- Connection reference
- Managed solution
- Pipeline

## 11. Environment Strategy

일반적으로 최소 세 환경을 둔다.

| 환경 | 역할 |
| --- | --- |
| Development | 개발 및 수정 |
| Test | 검증 |
| Production | 실제 사용자 운영 |

시험 원칙:

- 운영 환경에서 직접 수정하지 않는다.
- 개발 환경에서 수정 후 test/prod로 승격한다.
- 환경 접근 권한을 제한한다.

## 12. Solution

Solution은 Power Platform 구성 요소를 묶어 환경 간 이동하는 컨테이너다.

포함 가능:

- agent
- topic
- agent flow
- environment variables
- connection references
- custom connector
- Dataverse components

시험 단서:

- create a solution
- add existing agents to a solution
- export/import solution
- add required objects

## 13. Managed vs Unmanaged Solution

| 구분 | Unmanaged | Managed |
| --- | --- | --- |
| 용도 | 개발 | 테스트/운영 배포 |
| 수정 | 직접 수정 가능 | 통제된 배포 |
| 시험 기준 | dev | prod |

시험 암기:

> 개발은 unmanaged, 운영 배포는 managed.

## 14. Environment Variables

Environment variable은 환경별로 달라지는 값을 분리한다.

예:

- dev API URL
- prod API URL
- SharePoint site
- service endpoint
- secret reference

시험 단서:

- different value per environment
- avoid hardcoding
- dev/test/prod configuration

정답:

- Use environment variables

## 15. Connection References

Connection reference는 solution이 사용하는 connector connection을 환경별로 연결하게 해준다.

시험에서는 environment variable과 함께 ALM 구성 요소로 나올 수 있다.

예:

- dev 환경의 ServiceNow connection
- prod 환경의 ServiceNow connection

## 16. Pipelines

Power Platform Pipelines는 solution 배포를 자동화한다.

용도:

- dev → test → prod 배포
- 배포 단계 정의
- 품질 gate
- 수동 실수 감소
- governance 강화

시험 단서:

- automate deployment
- Power Platform Pipelines
- promote solution
- deployment stages

## 17. ALM에서 주의할 설정

일부 항목은 solution으로 이동 후 별도 확인이 필요할 수 있다.

예:

- Application Insights settings
- manual authentication settings
- channel security settings
- deployed channels
- sharing settings

시험에서 "post-deployment steps"가 나오면 이 개념을 떠올린다.

## 18. 시험 문제 풀이 패턴

## 패턴 1: 반복 품질 검증

요구:

- 변경 후 답변 품질을 비교
- 같은 질문 세트로 반복 테스트

정답:

- Test set
- Evaluation
- Compare test results

## 패턴 2: 운영 오류 분석

요구:

- 어떤 topic/tool에서 exception 발생하는지 확인
- latency 추적

정답:

- Application Insights

## 패턴 3: 환경별 endpoint

요구:

- dev와 prod API URL이 다름
- agent 수정 없이 값만 바꾸고 싶음

정답:

- Environment variable

## 패턴 4: 환경 간 배포

요구:

- dev/test/prod 배포 자동화

정답:

- Solution
- Power Platform Pipelines

## 패턴 5: 운영 환경 직접 수정 방지

요구:

- 프로덕션 변경 통제

정답:

- Managed solution
- ALM process

## 09단계 복습 질문

**Q1. Test chat과 Evaluation의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Test chat은 maker가 agent와 직접 대화하며 topic, node, tool 호출을 수동으로 확인하는 기능이다. Evaluation은 test set을 사용해 답변 품질을 반복 가능하게 검증하고 변경 전후 결과를 비교하는 기능이다.
</div>
</details>

**Q2. Text match, Similarity, Quality는 각각 언제 쓰는가?**

<details>
<summary>정답 확인</summary>
<div>
Text match는 정확한 문구나 특정 단어가 필요한 경우에 사용한다. Similarity는 표현은 달라도 의미가 같으면 되는 경우에 사용한다. Quality는 relevance, groundedness, completeness처럼 생성형 답변 품질을 평가할 때 사용한다.
</div>
</details>

**Q3. Application Insights는 어떤 정보를 수집하는가?**

<details>
<summary>정답 확인</summary>
<div>
Incoming/outgoing messages, events, triggered topics, node execution events, custom telemetry, exceptions, latency, tool usage 같은 운영 telemetry를 수집한다.
</div>
</details>

**Q4. Solution은 ALM에서 어떤 역할을 하는가?**

<details>
<summary>정답 확인</summary>
<div>
Solution은 agent, topic, agent flow, environment variables, connection references, custom connector 같은 Power Platform 구성 요소를 묶어 환경 간 이동과 배포를 관리하는 컨테이너다.
</div>
</details>

**Q5. Environment variable이 필요한 대표 상황은 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
개발 환경과 운영 환경의 API URL, SharePoint site, service endpoint, secret reference처럼 환경별로 값이 달라지는 설정을 하드코딩하지 않고 분리해야 할 때 필요하다.
</div>
</details>
