---
date: 2026-06-18
tags: [n8n, langchain, agent, tool-calling, ollama, gemma]
source: raw/light study/add_n8n.json
---

# n8n으로 만든 LangChain Agent + Tool 워크플로

## 구조

n8n의 `@n8n/n8n-nodes-langchain` 노드들을 이용해 채팅 기반 AI 에이전트를 구성했다. 노드 연결은 다음과 같다.

```
When chat message received (chatTrigger)
        │ main
        ▼
     AI Agent ──ai_languageModel── Ollama Chat Model (gemma4:e4b)
        │
        └──ai_tool── Code Tool (덧셈)
```

- **Chat Trigger**: 사용자가 채팅으로 입력하면 워크플로가 시작된다.
- **AI Agent**: 실제 추론을 담당하는 LangChain 에이전트 노드. system message로 도구 호출 포맷을 지정해뒀다.
- **Ollama Chat Model**: 로컬 Ollama에서 구동 중인 `gemma4:e4b` 모델을 LLM으로 연결.
- **Code Tool**: 에이전트가 호출할 수 있는 커스텀 도구로, JS 코드로 숫자 배열의 합을 계산.

## Agent의 system message로 도구 호출 형식 강제하기

```
덧셈을 하기 위해서는 출력을 다음과 같이 만들어
ex) { "numbers": [1, 2, 3] }
```

모델이 자유 형식으로 답하는 대신, 도구(Code Tool)가 기대하는 입력 스키마(`{"numbers": [...]}`)에 맞춰 출력하도록 system message에서 명시적으로 예시를 줬다. 이건 [[2026-06-18-tool-calling-loop]]에서 다룬 "모델이 도구 호출 JSON을 생성하고 런타임이 파싱해서 실행한다"는 루프를 n8n이라는 노코드 툴 위에서 그대로 구현한 것이다.

## Code Tool 정의

```js
return query.numbers.reduce((a, b) => a + b, 0);
```

- `specifyInputSchema: true`로 입력 스키마를 직접 지정 (`{"numbers": [1,2,3,4]}`).
- `description`에 "언제 이 도구를 써야 하는지"를 적어두면, 에이전트가 사용자의 발화("더해줘", "합계 알려줘" 등)와 도구를 매칭할 때 이 설명을 활용한다. 도구 description의 품질이 곧 에이전트의 도구 선택 정확도에 직결된다.

## 관찰: 로컬 gemma3:e4b의 덧셈 정확도

로컬에서 Ollama로 띄운 `gemma3:e4b` 모델은 (도구 호출 없이 모델 자체 추론만으로도) 단순 덧셈을 비교적 잘 틀리지 않는 편이었다. 다만 이 워크플로처럼 Tool Calling 구조를 갖춰두면, 모델의 산술 정확도에 의존하지 않고 실제 계산은 결정적인 코드(Code Tool)가 수행하므로 더 안정적이다 — 모델은 "숫자를 추출해서 도구에 넘기는 역할"만 하면 된다.

## 배운 점

- n8n의 LangChain 노드는 일반 LangChain 코드의 `AgentExecutor` + `Tool` 구성을 그대로 노드 그래프로 옮긴 것에 가깝다. `ai_languageModel`, `ai_tool` 같은 연결 타입이 LangChain의 "어떤 컴포넌트가 에이전트에 주입되는가"를 표현한다.
- 산술처럼 모델이 가끔 틀릴 수 있는 연산은, 모델에게 직접 계산을 시키기보다 Tool로 위임하는 패턴이 더 안전하다. 모델은 자연어 → 구조화된 입력 변환만 책임진다.
