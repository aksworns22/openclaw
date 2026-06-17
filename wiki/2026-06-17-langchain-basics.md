---
date: 2026-06-17
tags: [llm, langchain, ollama, prompt-template, chat-model]
source: raw/한시간으로 끝내는 LangChain 기본기/2026-06-17-실습.ipynb
---

# LangChain 기본기 실습

## 로컬 LLM 연결하기

`langchain-ollama`를 통해 로컬에서 실행 중인 Ollama 모델을 LangChain의 채팅 모델 인터페이스로 감쌀 수 있다.

```python
from langchain_ollama import ChatOllama

chat = ChatOllama(model='gemma4:e4b')
chat.invoke("프랑스의 수도가 어디야?")
```

`invoke()`는 단순 문자열 질문뿐 아니라, 아래에서 만드는 프롬프트나 메시지 리스트도 그대로 받아들인다.

## PromptTemplate: 텍스트 프롬프트 템플릿화

`{변수}` 형태의 자리표시자를 채워서 프롬프트를 동적으로 생성한다.

```python
from langchain_core.prompts import PromptTemplate

prompt_template = PromptTemplate(
    template="What is the capital of {country}?",
    input_variables=["country"],
)

prompt = prompt_template.invoke({'country': 'Japan'})
```

만들어진 `prompt`는 `chat.invoke(prompt)`로 바로 모델에 전달할 수 있다 — 템플릿 채우기와 모델 호출이 분리되어 있다.

## 메시지 타입으로 대화 맥락 구성하기

LangChain은 역할별로 메시지 클래스를 구분한다.

- `SystemMessage`: 모델의 역할/태도를 지시
- `HumanMessage`: 사용자의 발화
- `AIMessage`: 모델의 이전 응답 (대화 기록을 이어줄 때 사용)

```python
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage

message_list = [
    SystemMessage(content='you are a helpful assistant! 💪'),
    HumanMessage(content="What is the capital of South Korea?"),
    AIMessage(content="The capital of South Korea is Seoul. 😎"),
    HumanMessage(content="What is the capital of Germany?"),
]

chat.invoke(message_list)
```

이전 턴의 `AIMessage`를 리스트에 포함시켜서 모델에게 대화 맥락(이전에 한 톤·스타일 포함)을 전달할 수 있다.

## ChatPromptTemplate: 메시지 템플릿화

`PromptTemplate`이 단일 문자열을 다루는 반면, `ChatPromptTemplate`은 역할(`system`/`human`)이 포함된 메시지 시퀀스를 템플릿으로 만든다.

```python
from langchain_core.prompts import ChatPromptTemplate

chat_prompt_template = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant!"),
    ("human", "What is the capital of {country}?"),
])

chat_prompt = chat_prompt_template.invoke({'country': 'America'})
chat.invoke(chat_prompt).content
```

즉 `PromptTemplate` → 문자열 한 개를 채우는 용도, `ChatPromptTemplate` → system/human 등 여러 역할의 메시지를 한 번에 채우는 용도로 구분해서 쓴다.

## 관련 노트
- [[2026-06-16-llm-overview]]
- [[2026-06-17-transformer-architecture]]
