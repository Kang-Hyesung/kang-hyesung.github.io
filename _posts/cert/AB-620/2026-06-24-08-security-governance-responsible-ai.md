---
title: 08단계 - 보안, Governance, Responsible AI
date: 2026-06-24 09:07 +0900
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

이 단계에서는 enterprise agent를 안전하게 운영하기 위한 보안과 거버넌스 개념을 학습한다.

AB-620에서 보안 문제는 보통 다음 질문으로 나온다.

> 누가 어떤 데이터에 접근할 수 있고, 어떤 기능을 사용할 수 있으며, 조직 정책을 어떻게 강제할 것인가?

## 1. Governance란?

Governance는 agent를 조직 기준에 맞게 안전하게 만들고 운영하기 위한 정책과 통제 체계다.

포함 요소:

- 인증
- 사용자 권한
- DLP policy
- connector 정책
- knowledge source 제한
- channel publish 제한
- audit logging
- monitoring
- responsible AI
- environment strategy

## 2. 보안 통제 수준

| 수준 | 의미 | 예 |
| --- | --- | --- |
| Tenant | 조직 전체 통제 | generative AI publish 제한, 전역 DLP |
| Environment | 환경별 통제 | dev/test/prod 분리, connector 제한 |
| Agent | 개별 agent 설정 | 인증, channel, generative orchestration |

시험 암기:

> 조직 전체는 tenant, 환경별 정책은 environment, 개별 agent 동작은 agent level.

## 3. Authentication

Agent 인증은 사용자가 agent에 접근할 때의 인증이다.

| 옵션 | 사용 시점 |
| --- | --- |
| No authentication | 공개 정보만 제공 |
| Authenticate with Microsoft | 내부 직원, Teams, Microsoft 365 |
| Authenticate manually | 다른 채널에서도 인증 필요 |

## 4. No Authentication의 위험

No authentication은 누구나 링크로 접근할 수 있다.

적합한 경우:

- 공개 FAQ
- 공개 제품 설명
- 민감 정보 없음

부적합한 경우:

- 내부 문서 접근
- 개인 데이터 조회
- 업무 시스템 작업
- 사용자별 권한 필요

시험 단서:

- anyone with the link
- unauthenticated users
- public data only

## 5. Tool Credential 보안

Tool 실행 시 credential도 중요하다.

| 방식 | 의미 | 보안 관점 |
| --- | --- | --- |
| End user credentials | 사용자 권한으로 실행 | 사용자별 접근 제어 |
| Maker-provided credentials | 제작자/공유 연결로 실행 | 공통 접근, 권한 확대 주의 |

시험 판단:

- 사용자별 권한이 중요하면 End user credentials
- 공통 리소스 접근이면 Maker-provided credentials 가능

## 6. DLP Policy

DLP는 Data Loss Prevention이다.

Power Platform DLP 정책으로 agent 기능 사용을 통제할 수 있다.

통제 예:

- 인증 없는 agent 차단
- 특정 connector 차단
- HTTP request 차단
- 특정 knowledge source 차단
- 특정 channel publish 차단
- event trigger 차단
- Application Insights 사용 제한

시험 단서:

- admin must block
- prevent makers from using
- restrict connector
- require authentication
- block public websites
- block HTTP requests

정답:

- Configure data policies / DLP policies

## 7. Connector Governance

Connector는 외부 데이터와 시스템에 접근하므로 통제가 필요하다.

고려할 점:

- 어떤 connector를 허용할지
- 어떤 environment에서 사용할 수 있는지
- maker가 어떤 connection을 만들 수 있는지
- 사용자 credential인지 maker credential인지
- 고권한 connector 사용 여부

시험 단서:

- govern connector usage
- advanced connector policies
- DLP policy
- block connector action

## 8. Knowledge Source Governance

Knowledge source도 보안 대상이다.

검토할 점:

- 공개 웹사이트를 허용할지
- SharePoint 문서 권한이 반영되는지
- 민감도 레이블이 있는지
- official source로 지정할 수 있는지
- 사용자가 볼 수 없는 문서를 답변에 사용하지 않는지

시험 단서:

- restrict knowledge sources
- block public websites
- sensitivity labels
- authorized content

## 9. Channel Governance

Agent를 어디에 publish할지도 통제해야 한다.

예:

- Teams만 허용
- 공개 웹사이트 publish 차단
- 특정 환경에서만 publish 허용
- web channel security 적용

시험 단서:

- block publishing to specific channels
- web channel security
- Teams + Microsoft 365

## 10. Audit Logs

Audit log는 누가 무엇을 했는지 추적하는 데 필요하다.

사용 목적:

- maker 활동 추적
- agent 변경 기록
- 보안 조사
- 규정 준수
- 사용자 활동 분석

시험 단서:

- auditability
- traceability
- compliance
- Microsoft Purview
- Microsoft Sentinel

## 11. Responsible AI

Responsible AI는 agent가 안전하고 신뢰할 수 있게 동작하도록 하는 원칙과 기능이다.

핵심 요소:

- grounded responses
- hallucination 감소
- content moderation
- 민감 정보 보호
- human oversight
- transparency
- monitoring

## 12. Responsible AI 시나리오

## 시나리오 1: 부정확한 답변 방지

요구:

- 회사 정책과 다르게 답변하면 안 됨

정답 후보:

- trusted knowledge source
- generative answers with grounding
- official source
- evaluation

## 시나리오 2: 민감한 결정 자동화 방지

요구:

- 환불 승인 또는 징계 결정에 사람 검토 필요

정답 후보:

- human-in-the-loop
- approval step

## 시나리오 3: 유해 콘텐츠 처리

요구:

- 부적절한 응답이나 필터링 오류 분석

정답 후보:

- content moderation
- Application Insights
- conversation transcripts

## 13. Agent Identity

Copilot Studio agent는 agent identity를 통해 어떤 권한과 connector 사용 범위를 가지는지 관리될 수 있다.

시험에서 중요한 점:

- agent가 어떤 connector 작업을 할 수 있는지 가시성 제공
- Entra와 Power Platform 정책으로 통제
- Conditional Access와 연결 가능
- DLP는 runtime에서 connector 사용을 제어

시험 단서:

- agent identity
- API permissions
- Entra
- Conditional Access
- connector permissions

## 14. Security Scan과 Warning

Agent publish 전 보안 경고나 security scan이 등장할 수 있다.

시험에서 의미:

- maker가 위험한 설정을 인지
- 인증 없음, 공개 channel, 민감 connector 등 점검
- publish 전 보안 상태 확인

## 15. 시험 문제 풀이 패턴

## 패턴 1: 인증 없는 agent 금지

요구:

- 조직 내 모든 agent는 인증 필요
- maker가 No authentication 선택 못 하게 함

정답:

- DLP/data policy로 unauthenticated usage 차단

## 패턴 2: 특정 connector 사용 차단

요구:

- maker가 Dropbox connector를 쓰면 안 됨

정답:

- DLP policy에서 connector 차단

## 패턴 3: 사용자별 권한 반영

요구:

- 각 직원은 자신이 볼 수 있는 문서만 검색 가능

정답:

- Authenticate with Microsoft
- End user credentials 또는 permission-aware access

## 패턴 4: 민감 작업 승인

요구:

- agent가 자동으로 환불 승인하면 안 됨

정답:

- Human-in-the-loop

## 패턴 5: 운영 중 필터 오류 분석

요구:

- Responsible AI filter exception 확인

정답:

- Application Insights telemetry
- conversation transcript

## 08단계 복습 질문

**Q1. DLP policy로 통제할 수 있는 항목은 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
DLP policy는 인증 없는 agent, 특정 connector, HTTP request, 특정 knowledge source, 특정 channel publish, event trigger, Application Insights 사용 제한 등을 통제할 수 있다.
</div>
</details>

**Q2. No authentication이 위험한 경우는 언제인가?**

<details>
<summary>정답 확인</summary>
<div>
내부 문서, 개인 데이터, 업무 시스템 작업, 사용자별 권한이 필요한 데이터에 접근할 때 위험하다. No authentication은 링크를 가진 누구나 접근할 수 있으므로 공개 FAQ처럼 민감 정보가 없는 경우에만 적합하다.
</div>
</details>

**Q3. End user credentials가 필요한 대표 상황은 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
각 사용자가 자신에게 허용된 SharePoint 문서, CRM record, 업무 데이터만 볼 수 있어야 하는 사용자별 접근 제어 시나리오에서 필요하다.
</div>
</details>

**Q4. Responsible AI에서 human-in-the-loop은 왜 중요한가?**

<details>
<summary>정답 확인</summary>
<div>
환불 승인, 징계 결정, 민감 정보 변경처럼 AI가 자동으로 처리하면 위험한 결정을 사람이 검토하게 해 안전성, 책임성, 감사 가능성을 높이기 때문이다.
</div>
</details>

**Q5. Tenant, environment, agent 수준 통제의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Tenant 수준은 조직 전체 정책을 통제한다. Environment 수준은 dev/test/prod 같은 환경별 정책과 connector 제한을 통제한다. Agent 수준은 개별 agent의 인증, channel, orchestration 같은 동작 설정을 통제한다.
</div>
</details>
