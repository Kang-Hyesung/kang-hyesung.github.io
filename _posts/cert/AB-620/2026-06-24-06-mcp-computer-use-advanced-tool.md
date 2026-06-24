---
title: 06단계 - MCP, Computer Use, 고급 Tool
date: 2026-06-24 09:05 +0900
author: hyesung
categories: CERT AB-620
tags:
  - AB-620
  - Copilot Studio
  - Microsoft Certification
  - AI Agent
---

## 학습 목표

이 단계에서는 기본 connector/API를 넘어서는 고급 tool 통합을 학습한다.

핵심 개념:

- MCP
- Computer use
- Prompt tool
- Tool orchestration
- Tool 실행 제어

## 1. MCP란?

MCP는 Model Context Protocol의 약자다.

MCP는 외부 시스템이 tools와 resources를 표준 방식으로 노출하고, agent가 이를 사용할 수 있게 하는 프로토콜이다.

쉽게 말하면:

> 외부 tool server와 agent를 연결하는 표준 방식.

## 2. MCP를 쓰는 상황

다음 상황에서 MCP를 고려한다.

- 외부 MCP server가 이미 있음
- 외부 서비스가 tools/resources를 MCP로 제공
- 여러 agent가 같은 tool server를 사용
- 단순 REST API보다 agent-tool 생태계가 중요

시험 단서:

- Model Context Protocol
- MCP server
- tools and resources
- connect to external tools
- standard protocol

정답 후보:

- Add/configure MCP tool
- Connect to MCP server
- Review MCP tools and resources

## 3. MCP와 REST API의 차이

| 구분 | MCP | REST API |
| --- | --- | --- |
| 연결 대상 | MCP server | HTTP endpoint |
| 목적 | tools/resources 표준 제공 | API method 호출 |
| 시험 단서 | MCP server, resources | OpenAPI, endpoint, method |
| 느낌 | agent tool ecosystem | service integration |

시험 암기:

> MCP는 tool server, REST API는 endpoint.

## 4. MCP와 A2A의 차이

MCP와 A2A는 이름이 비슷하게 어렵지만 목적이 다르다.

| 구분 | MCP | A2A |
| --- | --- | --- |
| 의미 | Model Context Protocol | Agent2Agent Protocol |
| 연결 대상 | Tool/resource server | 다른 agent |
| 목적 | 외부 tool 사용 | agent에게 task 위임 |
| 시험 단서 | MCP tools/resources | delegate to external agent |

시험 암기:

> 도구 서버는 MCP, 다른 agent는 A2A.

## 5. Computer Use란?

Computer use는 agent가 웹사이트나 데스크톱 앱 화면을 보고, 버튼 클릭이나 텍스트 입력 같은 GUI 작업을 수행하는 기능이다.

적합한 경우:

- API가 없음
- connector가 없음
- legacy app
- desktop-only workflow
- 사람이 화면으로만 처리하던 업무

시험 단서:

- graphical user interface
- legacy application
- no API
- desktop app
- click buttons
- enter text

정답 후보:

- Configure computer use
- Monitor computer use

## 6. Computer Use 판단 기준

Computer use는 강력하지만 첫 번째 선택지는 아니다.

우선순위:

1. Connector가 있으면 connector 사용
2. API가 있으면 REST API 또는 custom connector 사용
3. MCP server가 있으면 MCP 사용
4. 아무 통합 API가 없고 GUI만 있으면 Computer use 고려

시험에서 "legacy", "no API", "GUI only"가 나오면 computer use 가능성이 높다.

## 7. Computer Use의 위험과 운영

Computer use는 화면을 조작하므로 다음을 고려해야 한다.

- 화면 변화에 취약
- 실행 환경 필요
- 오류 발생 가능성
- 민감 작업 전 사용자 확인 필요
- 모니터링 필요

시험 단서:

- monitor computer use
- ensure reliability
- human oversight
- confirmation before action

## 8. Prompt Tool 심화

Prompt tool은 특정 AI 작업을 수행하는 재사용 가능한 tool이다.

적합한 작업:

- 요약
- 분류
- 감정 분석
- 엔터티 추출
- JSON 생성
- 이메일 초안 작성
- 정책 문구 변환

## 9. Prompt Tool과 Foundry Model

Prompt tool은 Azure AI Foundry model catalog의 모델을 사용할 수 있다.

시험 단서:

- use Foundry model catalog
- bring your own model
- custom prompt uses a specific model
- connect model from Azure AI Foundry

정답 후보:

- Create Prompt tool
- Connect/select model from Azure AI Foundry

## 10. Prompt Tool vs Generative Answers

| 구분 | Prompt Tool | Generative Answers |
| --- | --- | --- |
| 목적 | 특정 AI 작업 | 지식 원본 검색 기반 답변 |
| 예시 | 문의 분류, JSON 추출 | 정책 문서 기반 답변 |
| 입력 | 지정된 텍스트, 변수 | 사용자 질문 |
| 출력 | 정해진 형식 가능 | 자연어 답변 |

시험 암기:

> 분류/추출/변환은 Prompt tool, 문서 기반 Q&A는 Generative answers.

## 11. Tool 자동 선택

Generative orchestration이 켜져 있으면 agent는 tool을 동적으로 선택할 수 있다.

선택에 영향을 주는 요소:

- tool name
- tool description
- input parameter description
- output description
- agent instructions
- conversation context

## 12. Tool을 명시적으로 호출하는 경우

반대로 topic 안에서 특정 tool을 명시적으로 호출할 수도 있다.

적합한 경우:

- 정해진 절차에서 반드시 특정 tool을 실행
- tool 실행 전 사용자 확인
- 조건에 따라 다른 tool 실행
- 엄격한 대화 흐름 필요

시험 구분:

- 자동 선택 → Generative orchestration
- 정해진 흐름에서 호출 → Topic 안의 tool node

## 13. Ask before running

Tool 실행 전 사용자 확인이 필요한 경우가 있다.

예:

- 주문 취소
- 결제
- 고객 정보 수정
- 티켓 제출
- 이메일 발송

시험 단서:

- ask user before running
- confirmation required
- avoid accidental change

정답:

- Configure tool to ask the end user before running

## 14. Tool Authentication

Tool 실행 시 인증 방식은 중요하다.

| 방식 | 사용 시점 |
| --- | --- |
| End user credentials | 사용자별 권한 필요 |
| Maker-provided credentials | 공통 연결로 실행 |

MCP, connector, prompt, API 등 tool 유형에 따라 인증 설정 방식은 다르지만, 시험의 판단 기준은 같다.

> 사용자별 접근 제어가 필요하면 end user credentials를 우선 고려한다.

## 15. 시험 문제 풀이 패턴

## 패턴 1: 외부 tool server

요구:

- 외부 서버가 MCP로 tools/resources 제공
- agent가 해당 tool 사용

정답:

- MCP server 연결

## 패턴 2: GUI만 있는 시스템

요구:

- API 없음
- 데스크톱 앱 화면에서만 처리 가능

정답:

- Computer use

## 패턴 3: 문의 분류

요구:

- 고객 문의를 category, urgency, summary JSON으로 변환

정답:

- Prompt tool

## 패턴 4: 잘못된 tool 선택

요구:

- agent가 비슷한 tool 중 잘못 선택

정답:

- tool description과 parameter description 개선

## 패턴 5: 실행 전 승인

요구:

- 사용자가 확인하기 전에는 주문 취소 금지

정답:

- Ask end user before running

## 06단계 복습 질문

**Q1. MCP와 REST API의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
MCP는 외부 MCP server가 제공하는 tools와 resources를 agent가 사용하게 하는 표준 프로토콜이다. REST API는 HTTP endpoint와 method를 직접 호출하는 통합 방식이다.
</div>
</details>

**Q2. MCP와 A2A의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
MCP는 tool/resource server와 연결한다. A2A는 다른 agent와 연결해 task를 위임한다. 즉 도구 서버는 MCP, 다른 agent는 A2A다.
</div>
</details>

**Q3. Computer use를 선택해야 하는 대표 조건은 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
API, connector, MCP server 같은 통합 수단이 없고 웹사이트나 데스크톱 앱의 GUI 화면만 조작할 수 있는 legacy workflow에서 선택한다.
</div>
</details>

**Q4. Prompt tool은 어떤 작업에 적합한가?**

<details>
<summary>정답 확인</summary>
<div>
요약, 분류, 감정 분석, 엔터티 추출, JSON 생성, 이메일 초안 작성처럼 특정 AI 작업을 재사용 가능한 tool로 만들 때 적합하다.
</div>
</details>

**Q5. Tool 자동 선택 정확도를 높이려면 무엇을 개선해야 하는가?**

<details>
<summary>정답 확인</summary>
<div>
Tool name, tool description, input parameter description, output description, agent instructions를 구체적으로 작성해야 한다. 비슷한 tool이 있다면 언제 어떤 tool을 써야 하는지 차이를 명확히 설명해야 한다.
</div>
</details>
