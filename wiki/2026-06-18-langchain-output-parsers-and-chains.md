---
date: 2026-06-18
tags: [llm, langchain, ollama, output-parser, structured-output, runnable, chain]
source: raw/한시간으로 끝내는 LangChain 기본기/2026-06-18 실습.ipynb
---

# LangChain 출력 파싱과 체인 연결

[[2026-06-17-langchain-basics]]에서 다룬 `ChatOllama`와 `PromptTemplate`을 기반으로, 모델 출력을 원하는 형태로 받아내고 여러 단계를 체인으로 엮는 방법을 실습했다.

## StrOutputParser: AIMessage에서 문자열만 꺼내기

`llm.invoke()`의 결과는 `content` 외에 메타데이터를 함께 담은 `AIMessage` 객체다. `StrOutputParser`는 여기서 `content`만 뽑아 순수 문자열로 변환한다.

```python
from langchain_core.output_parsers import StrOutputParser

result = llm.invoke(prompt)          # AIMessage(content='Tokyo', ...)
output_parser = StrOutputParser()
parsed_result = output_parser.invoke(result)   # 'Tokyo'
```

## JsonOutputParser: JSON 응답을 dict로

프롬프트에서 "JSON 형식으로만 답하라"고 지시하면, 모델이 JSON 문자열을 생성한다. `JsonOutputParser`는 이를 파싱해 Python dict로 변환한다.

```python
from langchain_core.output_parsers import JsonOutputParser

country_detail_prompt = PromptTemplate(
    template="""Give following information about {country}:
    - Capital
    - Population
    - Language
    - Currency

    return it in JSON format, and return the JSON dictionary only
    """,
    input_variables=["country"],
)

json_output_parser = JsonOutputParser()
json_output_parser.invoke(llm.invoke(country_detail_prompt.invoke({'country': 'Japan'})))
# {'Capital': 'Tokyo', 'Population': 'Approx. 125 million', ...}
```

다만 이 방식은 모델이 형식 지시를 따르는 데 의존하므로, 필드 이름이나 자료형이 매번 달라질 수 있다.

## with_structured_output: Pydantic 모델로 강제하기

원하는 출력 스키마를 Pydantic `BaseModel`로 정의하고 `llm.with_structured_output(스키마)`로 감싸면, 모델이 그 스키마에 맞는 객체를 반환하도록 강제할 수 있다. 필드명과 타입이 고정되므로 `JsonOutputParser`보다 안정적이다.

```python
from pydantic import BaseModel, Field

class CountryDetail(BaseModel):
    capital: str = Field(description="The capital of the country")
    population: int = Field(description="The population of the country")
    language: str = Field(description="The language of the country")
    currency: str = Field(description="The currency of the country")

structured_llm = llm.with_structured_output(CountryDetail)
structured_llm.invoke(country_detail_prompt.invoke({'country': 'Japan'}))
# CountryDetail(capital='Tokyo', population=125990378, language='Japanese', currency='Japanese Yen')
```

`population`이 정수형(`int`)으로 정확히 채워지는 점에서, 자유 형식 JSON 파싱보다 타입 안정성이 높다는 걸 확인할 수 있다.

## `|` 연산자로 체인 만들기 (LCEL)

LangChain은 `프롬프트 | 모델 | 파서`처럼 `|`로 컴포넌트를 이어 붙여 하나의 실행 가능한 체인(Runnable)을 만든다. 각 단계의 출력이 다음 단계의 입력으로 그대로 전달된다.

```python
country_chain = country_prompt | llm | output_parser
country_chain.invoke({'information': '에펠탑으로 유명한 나라'})
# '프랑스'
```

## 체인을 다시 체인으로 합성하기

체인 자체도 Runnable이므로, 체인과 체인을 다시 `|`로 연결할 수 있다.

```python
capital_chain = country_detail_prompt | llm | output_parser
final_chain = country_chain | capital_chain

final_chain.invoke({'information': '콜로세움으로 유명한 나라'})
```

여기서 주의할 점: `country_chain`의 출력(국가 이름 문자열)이 `capital_chain`의 입력으로 그대로 들어가는데, `capital_chain`이 기대하는 입력은 `{'country': ...}` 형태의 dict가 아니라 `country_detail_prompt`의 `input_variables`에 맞는 값이어야 한다. 실습에서는 문자열이 바로 들어가도 `PromptTemplate`이 단일 변수를 자동으로 채워주는 동작에 의존하고 있어, 두 체인의 입출력 타입이 자연스럽게 맞아떨어진 경우다.

## RunnablePassthrough: 원본 입력을 그대로 다음 단계로

체인 중간에서 원본 입력값을 가공하지 않고 그대로 다음 단계의 특정 키에 꽂아 넣고 싶을 때 `RunnablePassthrough`를 쓴다.

```python
from langchain_core.runnables import RunnablePassthrough

real_final_chain = (
    {'information': RunnablePassthrough()}
    | {'country': country_chain}
    | capital_chain
)

real_final_chain.invoke({'information': '김치의 나라'})
```

이 패턴은 "입력을 그대로 보존하면서, 동시에 그 입력을 가공한 결과도 함께 다음 단계에 넘기고 싶을 때" 자주 쓰인다. 여기서는 `{'information': ...}` 형태의 입력을 받아 `RunnablePassthrough`로 그대로 통과시키고, 그 값을 `country_chain`에 흘려보내 `{'country': ...}` 형태로 변환한 뒤 `capital_chain`에 전달하는 흐름이다.

## 관련 노트
- [[2026-06-17-langchain-basics]]
- [[2026-06-16-llm-overview]]
