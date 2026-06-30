---
title: "🗺️ 지식 지도"
weight: 1
bookToc: false
# === 개념 그래프 (이 front matter가 지도의 단일 진실의 원천) ===
# 필드:
#   id      : 개념 식별자(영문 kebab). needs에서 이 값으로 참조한다.
#   label   : 지도에 표시될 이름(생략 시 id).
#   status  : stub(아직 얕음) | learning(배우는 중) | solid(탄탄함)
#   needs   : 선행 개념 id 목록(이 개념의 화살표가 들어오는 출발점들).
#   days    : 이 개념을 다룬 날짜 노트(AI가 본문으로 점프할 때 사용).
concepts:
  # --- 트랜스포머 트랙 ---
  - id: backprop
    label: "역전파 / autograd"
    status: solid
    days: [2026-06-15]
  - id: pytorch
    label: "PyTorch 기초"
    status: learning
    days: [2026-06-16]
  - id: nlp-basics
    label: "딥러닝 NLP 기초 (RNN·전이학습)"
    status: learning
    needs: [pytorch]
    days: [2026-06-16]
  - id: tokenization
    label: "토큰화 / 임베딩"
    status: learning
    needs: [nlp-basics]
    days: [2026-06-17]
  - id: attention
    label: "어텐션 (QKV)"
    status: solid
    needs: [tokenization, backprop]
    days: [2026-06-17, 2026-06-21]
  - id: multi-head-attention
    label: "멀티 헤드 어텐션"
    status: solid
    needs: [attention]
    days: [2026-06-30]
  - id: layer-norm
    label: "층 정규화"
    status: learning
    needs: [nlp-basics]
    days: [2026-06-30]
  - id: feed-forward
    label: "피드 포워드 층"
    status: learning
    needs: [nlp-basics]
    days: [2026-06-30]
  - id: transformer
    label: "트랜스포머 (인코더·디코더)"
    status: learning
    needs: [multi-head-attention, layer-norm, feed-forward]
    days: [2026-06-21, 2026-06-30]
  - id: pretrained-models
    label: "사전학습 모델 (BERT·GPT·T5)"
    status: stub
    needs: [transformer]
    days: [2026-06-30]

  # --- LLM 애플리케이션 / 에이전트 트랙 ---
  - id: langchain
    label: "LangChain (프롬프트·체인)"
    status: learning
    needs: [tokenization]
    days: [2026-06-17, 2026-06-18, 2026-06-22]
  - id: structured-output
    label: "출력 파서 / 구조화 출력"
    status: learning
    needs: [langchain]
    days: [2026-06-18]
  - id: tool-calling
    label: "도구 호출 / 함수 호출"
    status: learning
    needs: [langchain]
    days: [2026-06-18]
  - id: prompt-engineering
    label: "프롬프트 엔지니어링"
    status: learning
    needs: [langchain]
    days: [2026-06-24]
  - id: embedding-store
    label: "임베딩 / 벡터 스토어"
    status: learning
    needs: [tokenization]
    days: [2026-06-22]
  - id: rag
    label: "RAG"
    status: learning
    needs: [embedding-store]
    days: [2026-06-22]
  - id: knowledge-graph
    label: "지식 그래프 / Neo4j"
    status: learning
    days: [2026-06-22, 2026-06-23]
  - id: graph-rag
    label: "Graph RAG"
    status: stub
    needs: [rag, knowledge-graph]
    days: [2026-06-22, 2026-06-23]
  - id: alignment
    label: "정렬 (SFT·RLHF·스케일링)"
    status: stub
    needs: [pretrained-models]
    days: [2026-06-22]
  - id: agent
    label: "에이전트"
    status: learning
    needs: [tool-calling]
    days: [2026-06-19, 2026-06-22, 2026-06-23]
  - id: agent-eval
    label: "에이전트 평가 (eval·grader)"
    status: learning
    needs: [agent]
    days: [2026-06-19, 2026-06-22]
  - id: agent-workflows
    label: "에이전트 워크플로 (오케스트레이터-워커 등)"
    status: learning
    needs: [agent]
    days: [2026-06-23]
  - id: mcp
    label: "MCP"
    status: stub
    needs: [tool-calling]
    days: [2026-06-23]
  - id: agent-harness
    label: "장기 실행 에이전트 하네스"
    status: learning
    needs: [agent]
    days: [2026-06-25]
  - id: multi-agent
    label: "멀티 에이전트 시스템"
    status: learning
    needs: [agent-workflows, agent-eval]
    days: [2026-06-29]
---

아래 지도는 이 페이지 front matter의 `concepts`만으로 자동 생성됩니다. 손으로 그리지 않습니다 — front matter를 고치면 지도가 바뀝니다.

{{< concept-map >}}

<p>
색은 학습 상태입니다 —
<span style="background:#f1f1f1;border:1px solid #c4c4c4;color:#8a8a8a;padding:1px 6px;border-radius:4px;">stub 아직 얕음</span>
<span style="background:#fff3cd;border:1px solid #e0a800;color:#664d03;padding:1px 6px;border-radius:4px;">learning 배우는 중</span>
<span style="background:#d1e7dd;border:1px solid #198754;color:#0f5132;padding:1px 6px;border-radius:4px;">solid 탄탄함</span>
. 화살표는 <strong>선행 → 후행</strong> 의존입니다.
</p>

화살표가 들어오지도 나가지도 않는 외딴 노드, 또는 `stub`으로 오래 남은 노드가 **채울 빈틈**입니다.
