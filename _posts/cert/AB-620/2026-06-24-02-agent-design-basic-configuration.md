---
title: 02단계 - Agent 설계와 기본 구성
date: 2026-06-24 09:01 +0900
author: hyesung
categories: CERT AB-620
tags:
  - AB-620
  - Copilot Studio
  - Microsoft Certification
  - AI Agent
---

## 학습 목표

이 단계에서는 Copilot Studio agent를 만들 때 가장 먼저 결정해야 하는 설계 요소를 학습한다.

핵심은 다음 질문에 답하는 것이다.

> 이 agent는 누구를 위해, 어떤 데이터를 사용하고, 어떤 채널에서, 어떤 권한으로 동작해야 하는가?

## 1. Agent란 무엇인가?

Copilot Studio agent는 사용자의 자연어 요청을 받아 다음 일을 수행하는 AI 애플리케이션이다.

- 질문에 답변
- 문서나 데이터 검색
- 외부 시스템 조회
- 업무 처리
- 다른 agent 호출
- 사용자와 여러 단계 대화

Agent는 단순 chatbot이 아니다.  
AB-620에서는 agent를 **엔터프라이즈 시스템과 연결되는 업무 수행 주체**로 본다.

## 2. Agent의 주요 구성 요소

| 구성 요소 | 역할 |
| --- | --- |
| Instructions | agent의 역할, 제약, 응답 방식 정의 |
| Knowledge sources | 답변에 사용할 정보 원본 |
| Topics | 특정 업무에 대한 제어된 대화 흐름 |
| Tools | 외부 시스템 조회 또는 작업 실행 |
| Agents | 다른 agent와의 협업 |
| Channels | Teams, Microsoft 365 Copilot, 웹 등 배포 위치 |
| Security | 인증, 권한, DLP, governance |
| Evaluation | 답변 품질 평가 |
| Monitoring | 운영 로그, 오류, 성능 추적 |

## 3. Internal agent와 External agent

시험에서 가장 먼저 구분해야 하는 설계 기준이다.

## 3.1 Internal agent

조직 내부 직원이 사용하는 agent다.

예시:

- HR 정책 안내 agent
- IT 헬프데스크 agent
- 영업 데이터 조회 agent
- 사내 문서 검색 agent

중요한 설계 포인트:

- Microsoft Entra ID 인증
- 사용자별 권한 적용
- SharePoint, Teams, Microsoft 365 데이터 접근
- DLP 정책 준수
- 감사 로그 및 보안 정책

시험 단서:

- employees
- internal users
- Microsoft Teams
- Microsoft 365 Copilot
- users should only access authorized data

정답 후보:

- Authenticate with Microsoft
- End user credentials
- Power Platform DLP policies
- Microsoft 365 / Teams channel

## 3.2 External agent

고객, 파트너, 익명 사용자 등 조직 외부 사용자가 쓰는 agent다.

예시:

- 공개 고객지원 agent
- 제품 FAQ agent
- 파트너 포털 agent
- 웹사이트 상담 agent

중요한 설계 포인트:

- 공개 정보와 내부 정보 분리
- 인증 필요 여부 결정
- 웹 채널 보안
- 민감 데이터 노출 방지
- 악용 방지와 responsible AI

시험 단서:

- external customers
- public website
- anonymous users
- anyone with the link
- protect internal data

정답 후보:

- Configure web channel security
- Avoid No authentication when sensitive data is used
- Restrict knowledge sources
- Apply DLP policies

## 4. Identity Strategy

Identity strategy는 agent와 사용자가 어떤 인증/권한으로 시스템에 접근할지 정하는 것이다.

## 4.1 Agent 인증 옵션

| 옵션 | 설명 | 적합한 상황 |
| --- | --- | --- |
| No authentication | 로그인 없이 사용 | 공개 FAQ, 민감 정보 없음 |
| Authenticate with Microsoft | Microsoft Entra ID 인증 | 내부 직원, Teams, Microsoft 365 |
| Authenticate manually | 수동 인증 구성 | 외부 채널에서도 인증 필요 |

## 4.2 No authentication

장점:

- 사용자가 쉽게 접근 가능
- 공개 정보 제공에 적합

위험:

- 링크를 가진 누구나 접근 가능
- 내부 데이터, 사용자별 데이터, 업무 시스템 접근에는 부적합

시험 판단:

민감 정보가 있으면 No authentication은 피해야 한다.

## 4.3 Authenticate with Microsoft

Microsoft Entra ID 기반 인증이다.

적합한 경우:

- 조직 내부 사용자
- Teams 사용
- Microsoft 365 Copilot 사용
- SharePoint, Outlook, Teams 같은 Microsoft 365 서비스 접근

시험 판단:

"직원이 Teams에서 사용한다"는 요구가 나오면 우선 이 옵션을 떠올린다.

## 4.4 Authenticate manually

수동 인증 구성이 필요한 경우다.

적합한 경우:

- Teams 외 채널에서 인증이 필요
- 특정 인증 공급자를 사용해야 함
- Microsoft 자동 인증만으로 요구사항을 충족하지 못함

## 5. Tool Credential Strategy

Agent 자체 인증과 별개로, tool을 실행할 때 어떤 credential을 사용할지도 중요하다.

| 방식 | 의미 | 사용 시점 |
| --- | --- | --- |
| End user credentials | 사용자의 권한으로 실행 | 사용자별 권한을 지켜야 할 때 |
| Maker-provided credentials | 제작자의 연결로 실행 | 공통 리소스 접근, 사용자별 로그인 불필요 |

## 5.1 End user credentials

사용자의 권한으로 tool을 실행한다.

예:

- 사용자가 접근 가능한 SharePoint 문서만 조회
- 사용자가 볼 수 있는 CRM record만 조회
- 사용자 권한에 따라 결과가 달라짐

시험 단서:

- "users should only see data they are authorized to access"
- "run using the user's permissions"
- "per-user access"

정답:

- End user credentials

## 5.2 Maker-provided credentials

제작자가 제공한 credential로 tool을 실행한다.

예:

- 공용 날씨 API 호출
- 공통 서비스 계정으로 사내 시스템 조회
- 모든 사용자가 같은 연결을 사용

시험 단서:

- "users should not need their own account"
- "shared connection"
- "maker's credentials"

정답:

- Maker-provided credentials

## 6. Channel Strategy

Channel은 agent를 사용자에게 제공하는 위치다.

| Channel | 적합한 시나리오 |
| --- | --- |
| Microsoft Teams | 내부 직원 업무 지원 |
| Microsoft 365 Copilot | Microsoft 365 업무 흐름 안에서 agent 사용 |
| Website | 고객용 공개 agent |
| SharePoint | 내부 포털 |
| Power Apps | 업무 앱 내부 |
| Custom app | 자체 애플리케이션 통합 |

시험에서는 channel과 authentication이 함께 나온다.

예:

- Teams + 내부 직원 → Authenticate with Microsoft
- 공개 웹사이트 + 민감 정보 없음 → No authentication 가능
- 외부 웹사이트 + 개인 정보 조회 → Manual authentication 또는 별도 보안 필요

## 7. Security와 Governance 기본

Enterprise agent는 보안 없이 만들면 안 된다.

시험에서 자주 묻는 governance 요소:

- DLP policy
- connector 사용 제한
- HTTP request 제한
- knowledge source 제한
- channel publish 제한
- authentication 강제
- audit log
- sensitivity label

## 8. Responsible AI Strategy

Responsible AI는 agent가 안전하고 신뢰할 수 있게 동작하도록 하는 전략이다.

시험에서 보는 포인트:

- 근거 있는 답변 제공
- 부정확한 답변 줄이기
- 민감 정보 보호
- 유해 콘텐츠 완화
- 사람이 검토해야 하는 단계 설계
- 모니터링과 감사 가능성 확보

대표 정답:

- trusted knowledge sources 사용
- generative answers grounding
- human-in-the-loop
- content moderation
- Application Insights / audit logging

## 9. Reusable Agent Components

재사용 가능한 agent 구성 요소를 설계하는 것도 시험 범위다.

재사용 가능한 요소:

- Topic
- Agent flow
- Custom connector
- Prompt
- Tool
- Connected agent
- Solution component

시험 판단:

여러 agent에서 같은 API나 업무 흐름을 반복해서 써야 한다면 재사용 가능한 구성 요소로 분리하는 것이 좋다.

## 10. 시험 문제 풀이 패턴

## 패턴 1: 내부 직원용 agent

요구사항:

- 직원이 Teams에서 사용
- SharePoint 문서 접근
- 사용자별 권한 필요

정답 흐름:

- Authenticate with Microsoft
- Teams channel
- SharePoint knowledge source
- End user credentials
- DLP policy

## 패턴 2: 공개 FAQ agent

요구사항:

- 고객이 웹사이트에서 사용
- 공개 제품 문서만 답변
- 로그인 없이 사용

정답 흐름:

- Website channel
- No authentication 가능
- Public knowledge source
- 내부 데이터 연결 금지

## 패턴 3: 고객 개인정보 조회 agent

요구사항:

- 고객이 본인 주문을 조회
- 개인 데이터 포함
- 웹 채널 사용

정답 흐름:

- 인증 필요
- No authentication 부적합
- Web channel security
- API/tool 접근 권한 검토

## 2단계 복습 질문

**Q1. Internal agent와 External agent의 가장 큰 설계 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Internal agent는 조직 내부 사용자를 대상으로 하므로 Microsoft Entra ID 인증, 사용자별 권한, Microsoft 365 데이터 접근, DLP 정책이 중요하다. External agent는 고객이나 파트너 등 외부 사용자를 대상으로 하므로 공개 정보와 내부 정보 분리, 웹 채널 보안, 민감 데이터 노출 방지가 중요하다.
</div>
</details>

**Q2. Authenticate with Microsoft는 어떤 상황에서 적합한가?**

<details>
<summary>정답 확인</summary>
<div>
조직 내부 직원이 Teams, Microsoft 365 Copilot, SharePoint 같은 Microsoft 365 환경에서 agent를 사용할 때 적합하다. Microsoft Entra ID 기반 인증이 필요한 내부 업무 시나리오에서 우선 고려한다.
</div>
</details>

**Q3. End user credentials와 Maker-provided credentials의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
End user credentials는 tool을 실행할 때 최종 사용자의 권한을 사용한다. 사용자별 접근 권한을 지켜야 할 때 적합하다. Maker-provided credentials는 제작자나 공유 connection을 사용하므로 모든 사용자가 같은 권한으로 공통 리소스에 접근하는 시나리오에 적합하다.
</div>
</details>

**Q4. DLP policy는 어떤 것을 통제할 수 있는가?**

<details>
<summary>정답 확인</summary>
<div>
DLP policy는 connector 사용, HTTP request, knowledge source, channel publish, 인증 없는 agent 사용 같은 Power Platform 리소스와 데이터 연결 방식을 통제할 수 있다.
</div>
</details>

**Q5. Responsible AI strategy에서 human-in-the-loop은 언제 필요한가?**

<details>
<summary>정답 확인</summary>
<div>
환불 승인, 민감 정보 변경, 정책 예외 승인처럼 AI가 자동으로 결정하면 위험한 업무에서 필요하다. 사람이 검토하거나 승인하도록 넣어 책임성과 안전성을 확보한다.
</div>
</details>
