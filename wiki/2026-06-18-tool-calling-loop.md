---
date: 2026-06-18
tags: [llm, tool-calling, function-calling, ollama, langchain, gemma]
source: raw/light study/function_calling.ipynb
---

# Tool Calling 루프의 정체

모델이 "글을 쓰는 것"에서 "행동을 요청하는 것"으로 넘어가는 순간을 정리했다.

## 핵심 제약: 모델은 코드를 실행할 수 없다

Gemma 4 같은 모델은 다음 토큰을 예측할 뿐이다. "날씨를 알려줘"라는 요청을 받아도 모델이 실제로 API를 호출하지 않는다. 대신 *"이 함수를 이렇게 호출해야 한다"*는 구조화된 텍스트를 생성할 뿐이고, 이 텍스트를 파싱해서 실제로 함수를 실행하는 책임은 항상 개발자 코드에 있다.

> Google 공식 문서: "A Gemma model cannot execute code on its own." ([ai.google.dev](https://ai.google.dev/gemma/docs/capabilities/text/function-calling-gemma4))

## 4단계 루프

| 단계          | 내용                                                  |
| ----------- | --------------------------------------------------- |
| 1. tool 선언  | 사용 가능한 함수의 이름·설명·파라미터 스키마를 모델에게 알려준다                |
| 2. 모델이 "요청" | 모델은 직접 실행하지 않고 `tool_calls`라는 구조화된 데이터를 응답에 담아 돌려준다 |
| 3. 직접 실행    | `tool_calls` 안의 함수 이름·인자를 꺼내 내 코드가 직접 호출한다          |
| 4. 결과 반환    | 실행 결과를 `role: "tool"` 메시지로 대화 기록에 추가하고 다시 모델을 호출한다  |

```python
from ollama import chat

def get_temperature(city: str) -> str:
    temperatures = {"Seoul": "26°C", "Tokyo": "24°C"}
    return temperatures.get(city, "데이터 없음")

messages = [{"role": "user", "content": "서울 기온이 어때?"}]
response = chat(model="gemma4:e4b", messages=messages, tools=[get_temperature])

if response.message.tool_calls:
    call = response.message.tool_calls[0]
    result = get_temperature(**call.function.arguments)

    messages.append(response.message)
    messages.append({"role": "tool", "tool_name": call.function.name, "content": str(result)})
    final = chat(model="gemma4:e4b", messages=messages, tools=[get_temperature])
    print(final.message.content)  # "현재 서울의 기온은 26°C입니다."
```

실제로 `gemma4:e4b`로 돌려보면 모델이 `thinking` 필드에 "도시 이름을 추출하고 어떤 tool을 쓸지 정한다"는 추론을 남기고, `tool_calls=[ToolCall(function=Function(name='get_temperature', arguments={'city': 'Seoul'}))]` 형태로 응답한다. 두 번째 호출에서는 `tool_calls=None`이 되고 `content`에 자연어 답이 채워진다.

## LangChain은 같은 루프를 이름만 바꿔서 감싼다

새 개념이 아니라 같은 4단계에 LangChain식 이름표가 붙는 것뿐이다.

| 단계         | raw ollama                                   | LangChain                                    |
| ---------- | -------------------------------------------- | -------------------------------------------- |
| 1. tool 선언 | `tools=[get_temperature]`                    | `@tool` 데코레이터 + `.bind_tools([...])`         |
| 2. 모델의 요청  | `response.message.tool_calls`                | `AIMessage.tool_calls`                       |
| 3. 직접 실행   | `get_temperature(**call.function.arguments)` | `get_temperature.invoke(tool_call['args'])`  |
| 4. 결과 반환   | `{"role": "tool", ...}`                      | `ToolMessage(tool_call_id=..., content=...)` |

```python
from langchain.messages import AIMessage, ToolMessage
from langchain.tools import tool
from langchain_ollama import ChatOllama

@tool
def get_temperature(city: str) -> str:
    """주어진 도시의 현재 기온을 반환한다.

    Args:
        city: 영어 도시 이름 (예: "Seoul", "Tokyo")
    """
    temperatures = {"Seoul": "26°C", "Tokyo": "24°C"}
    return temperatures.get(city, "데이터 없음")

llm = ChatOllama(model="gemma4:e4b", temperature=0).bind_tools([get_temperature])
result = llm.invoke("서울 기온이 어때?")

if isinstance(result, AIMessage) and result.tool_calls:
    tool_call = result.tool_calls[0]
    tool_result = get_temperature.invoke(tool_call['args'])

    messages = [result, ToolMessage(tool_call_id=tool_call['id'], content=str(tool_result))]
    final = llm.invoke(messages)
    print(final.content)
```

## 헷갈리기 쉬운 지점: `.` 접근 vs `[]` 접근

raw ollama의 `tool_calls`는 Pydantic 객체 리스트라서 `call.function.name`처럼 점(`.`)으로 접근한다. 반면 LangChain의 `AIMessage.tool_calls`는 **dict의 리스트**라서 `tool_call['name']`처럼 `[]`로 접근해야 한다. "객체는 `.`, 딕셔너리는 `[]`"라는 기본 규칙이 두 라이브러리 사이에서 그대로 적용되는 것뿐이다.

```python
# raw ollama
call.function.name        # 객체 속성 접근
call.function.arguments   # {'city': 'Seoul'}

# LangChain
tool_call['name']         # dict 접근
tool_call['args']         # {'city': 'Seoul'}
```

`**args`로 딕셔너리를 키워드 인자로 풀어 넘기는 것도 두 경우 모두 동일하게 쓰인다(`get_temperature(**args)` ≡ `get_temperature(city='Seoul')`).

## 실전 교훈

- 모델은 절대 코드를 직접 실행하지 않는다 — `tool_calls`는 항상 "요청"일 뿐, 실행은 항상 내 코드의 책임이다.
- 모델이 프롬프트의 언어를 인자 값에 그대로 반영할 수 있다(예: "서울" vs "Seoul"). tool 함수의 키·enum 값은 한 언어로 고정하고, docstring에 매핑 규칙을 명시해 두는 게 안전하다.
