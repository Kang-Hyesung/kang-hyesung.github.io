---
title: 03단계 - Topic, Knowledge, Generative Answers
date: 2026-06-24 09:02 +0900
author: hyesung
categories: CERT AB-620
tags:
  - AB-620
  - Copilot Studio
  - Microsoft Certification
  - AI Agent
---

## 학습 목표

이 단계에서는 agent가 사용자 질문에 응답하는 두 가지 큰 방식을 구분한다.

1. **Topic**: 정해진 대화 흐름으로 처리
2. **Knowledge / Generative answers**: 지식 원본을 검색해 답변

시험에서 가장 자주 나오는 기본 구분은 이것이다.

> 절차는 Topic, 지식 답변은 Generative answers.

## 1. Topic이란?

Topic은 사용자의 특정 의도에 대해 agent가 따라야 하는 대화 흐름이다.

예:

- 휴가 신청
- 주문 상태 조회
- 비밀번호 초기화
- 환불 요청
- 티켓 생성

Topic은 사용자의 입력을 받고, 조건에 따라 분기하고, tool을 호출하고, 응답을 구성할 수 있다.

## 2. Topic을 쓰는 상황

다음 요구가 나오면 topic을 생각한다.

- 사용자의 입력을 단계적으로 받아야 함
- 정해진 절차를 따라야 함
- 조건에 따라 분기해야 함
- 특정 tool을 명시적으로 호출해야 함
- 승인 또는 확인 단계가 필요함
- 대화 흐름을 maker가 통제해야 함

시험 단서:

- guide the user through a process
- ask a series of questions
- branch based on user input
- trigger a specific flow
- collect required information

## 3. Topic의 구성 요소

| 구성 요소 | 의미 |
| --- | --- |
| Trigger phrases | 사용자의 어떤 표현이 topic을 시작하게 할지 정의 |
| Nodes | 메시지, 질문, 조건, tool 호출 등 대화 단계 |
| Variables | 입력값, 응답값, 중간값 저장 |
| Conditions | 값에 따른 분기 |
| Tool node | topic 안에서 tool 실행 |
| Generative answers node | 특정 지식 기반 답변 생성 |
| Adaptive card | 카드 UI로 응답 또는 입력 수집 |

## 4. Trigger phrases

Trigger phrases는 classic orchestration에서 특히 중요하다.

예:

- "휴가 신청하고 싶어요"
- "연차를 등록해줘"
- "휴가 요청"

시험에서는 trigger phrase가 "정해진 topic을 시작하는 조건"으로 등장한다.

단, generative orchestration에서는 trigger phrase만이 아니라 topic 설명, tool 설명, agent instructions 등도 함께 사용된다.

## 5. Variables

Variable은 대화 중 필요한 값을 저장한다.

예:

- 사용자 이름
- 주문 번호
- 시작일
- 종료일
- API 응답값
- 선택한 옵션

## 5.1 변수 사용 예시

휴가 신청 topic:

```text
사용자 입력: 2026-07-01부터 2026-07-03까지 휴가 신청
저장할 값:
- startDate = 2026-07-01
- endDate = 2026-07-03
- leaveType = annual leave
```

이 값은 이후 agent flow나 API tool에 전달된다.

## 6. Adaptive Cards

Adaptive Card는 구조화된 카드 UI다.

사용 예:

- 승인/거절 버튼
- 날짜 선택
- 목록 선택
- 주문 정보 카드
- 입력 폼

시험 단서:

- structured response
- card
- button
- form
- collect input
- show approval options

정답 후보:

- Configure adaptive cards

## 7. Knowledge Source란?

Knowledge source는 agent가 답변할 때 참고하는 정보 원본이다.

예:

- SharePoint 문서
- OneDrive 파일
- Dataverse 데이터
- 업로드 파일
- 공개 웹사이트
- Azure AI Search index
- custom knowledge source

Knowledge source는 주로 "정보를 찾아 답변"하는 데 사용된다.

## 8. Generative Answers란?

Generative answers는 knowledge source에서 관련 내용을 검색하고, 그 결과를 바탕으로 자연어 답변을 생성한다.

간단히 말하면:

```text
사용자 질문
→ 관련 문서 검색
→ 검색 결과 요약
→ 자연어 답변 생성
→ 가능하면 citation 제공
```

## 9. 언제 Knowledge / Generative Answers를 쓰는가?

다음 상황에서 적합하다.

- 문서 기반 Q&A
- 정책 문서 답변
- 제품 설명서 검색
- FAQ를 하나하나 topic으로 만들기 어려움
- 근거 기반 답변 필요
- citation 필요
- hallucination을 줄여야 함

시험 단서:

- answer questions from documents
- use SharePoint as a knowledge source
- provide citations
- ground responses
- summarize information from files
- reduce manually authored FAQ topics

## 10. Topic vs Generative Answers

| 구분 | Topic | Generative answers |
| --- | --- | --- |
| 목적 | 제어된 대화 흐름 | 지식 기반 답변 |
| 입력 | 필요한 값을 단계적으로 수집 | 자연어 질문 |
| 출력 | 정해진 응답, tool 실행 결과 | 검색 기반 자연어 답변 |
| 강점 | 절차, 분기, 검증 | 넓은 문서 검색 |
| 예시 | 휴가 신청 처리 | 휴가 정책 설명 |

## 11. RAG

RAG는 Retrieval-Augmented Generation이다.

의미:

> 모델이 자체 지식만으로 답변하지 않고, 외부 지식 원본에서 관련 내용을 검색한 뒤 그 내용을 근거로 답변하는 방식.

AB-620에서 RAG는 다음과 연결된다.

- Knowledge sources
- Generative answers
- Azure AI Search
- Grounded responses
- Citations

시험 단서:

- grounded answer
- enterprise data
- search index
- reduce hallucination
- retrieval

## 12. Azure AI Search와 Knowledge

Azure AI Search는 대규모 문서나 데이터를 검색 인덱스로 관리할 때 중요하다.

일반 knowledge source보다 더 강하게 "검색/RAG/엔터프라이즈 데이터" 느낌이 난다.

사용 상황:

- 문서가 많음
- 검색 품질이 중요함
- semantic search 필요
- enterprise search index 사용
- Foundry와 함께 RAG 구성

## 13. Official Source

Official source는 신뢰할 수 있는 지식 원본을 표시하는 개념이다.

적합한 경우:

- 회사 공식 정책
- 검증된 제품 문서
- 규정 준수 문서
- 승인된 지식 원본

주의:

- 신뢰도가 낮거나 자주 바뀌는 외부 정보에 무분별하게 적용하지 않는다.

## 14. Custom Knowledge Source

Custom knowledge source는 기본 제공 knowledge source가 아니라 자체 검색 API나 enterprise search 시스템을 연결하는 방식이다.

시험에서 등장할 수 있는 단서:

- custom search endpoint
- existing enterprise search
- own search API
- query rewriting
- format search results for generative answers

정답 후보:

- Configure custom knowledge source
- Use OnKnowledgeRequested trigger

## 15. 시험 문제 풀이 패턴

## 패턴 1: 회사 정책 질문

요구:

- 직원이 휴가 정책을 질문
- SharePoint에 정책 문서 있음
- 문서 기반 답변 필요

정답:

- SharePoint knowledge source
- Generative answers
- Citation

## 패턴 2: 휴가 신청 처리

요구:

- 시작일, 종료일, 사유 입력 필요
- manager approval 필요
- 시스템에 신청 생성

정답:

- Topic으로 입력 수집
- Agent flow 또는 tool 호출
- Human-in-the-loop

## 패턴 3: 답변 UI 개선

요구:

- 주문 정보와 버튼을 보기 좋게 표시
- 사용자가 선택할 수 있어야 함

정답:

- Adaptive Card

## 패턴 4: 많은 문서를 검색

요구:

- 수천 개 기술 문서 검색
- 관련성 높은 답변 필요
- RAG 구성

정답:

- Azure AI Search
- Generative answers

## 3단계 복습 질문

**Q1. Topic과 Generative answers의 차이는 무엇인가?**

<details>
<summary>정답 확인</summary>
<div>
Topic은 사용자의 입력을 단계적으로 받고 조건에 따라 분기하는 제어된 대화 흐름이다. Generative answers는 knowledge source를 검색해 자연어 답변을 생성하는 지식 기반 응답 방식이다.
</div>
</details>

**Q2. Knowledge source는 어떤 상황에서 쓰는가?**

<details>
<summary>정답 확인</summary>
<div>
문서 기반 Q&A, 정책 문서 답변, 제품 설명서 검색, SharePoint 기반 답변, citation이나 grounded answer가 필요한 상황에서 사용한다.
</div>
</details>

**Q3. RAG는 무엇이며 AB-620에서 어떤 기능과 연결되는가?**

<details>
<summary>정답 확인</summary>
<div>
RAG는 Retrieval-Augmented Generation의 약자로, 외부 지식 원본에서 관련 내용을 검색한 뒤 그 내용을 근거로 답변하는 방식이다. AB-620에서는 Knowledge sources, Generative answers, Azure AI Search, grounded responses, citations와 연결된다.
</div>
</details>

**Q4. Adaptive Card는 어떤 문제에서 정답이 될 가능성이 높은가?**

<details>
<summary>정답 확인</summary>
<div>
버튼, 입력 폼, 날짜 선택, 승인/거절 선택지, 주문 정보 카드처럼 구조화된 UI로 정보를 보여주거나 사용자 입력을 받아야 하는 문제에서 정답이 될 가능성이 높다.
</div>
</details>

**Q5. Azure AI Search는 일반 knowledge source와 무엇이 다른가?**

<details>
<summary>정답 확인</summary>
<div>
Azure AI Search는 대규모 문서나 enterprise search index를 기반으로 검색 품질과 관련성을 높이는 데 적합하다. 수천 개 문서, semantic search, RAG 구성이 강조되면 Azure AI Search를 우선 고려한다.
</div>
</details>
