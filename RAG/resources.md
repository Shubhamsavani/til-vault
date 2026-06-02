# Hallucination Detection & Evaluation for RAG Systems
> *Notes on RAGAS and LYNX*

---

## What is Hallucination?

Hallucination occurs when an LLM generates information that **sounds plausible but is factually incorrect or unsupported by real data**.

**Real-world example:** Air Canada's chatbot invented a refund policy that didn't actually exist — and the airline was legally held to it.
→ [Read more](https://arstechnica.com/tech-policy/2024/02/air-canada-must-honor-refund-policy-invented-by-airlines-chatbot/)

> Hallucination is fundamentally one of the hardest problems to solve when working with LLMs.

---

## What is RAG?

**RAG (Retrieval-Augmented Generation)** is a technique that reduces hallucinations by grounding the LLM in real, retrieved facts.

### How it works:
1. Retrieve relevant documents/chunks based on the query
2. Provide them as context to the LLM
3. Ask the LLM to answer using **only** the provided information

While RAG significantly reduces hallucinations, they can **still occur** even in well-built RAG pipelines.

---

## What to Evaluate in a RAG System?

### 1. Faithfulness
> *Is the LLM response grounded in the provided context?*

The most critical metric for hallucination detection. Checks whether the model's answer is supported by the retrieved documents.

### 2. Context Relevance
> *How much of the retrieved content is actually relevant to answering the question?*

Evaluates the **retrieval pipeline** — are we fetching useful content, or noisy irrelevant chunks?

### 3. Answer Relevance
> *Does the answer properly address the user's question?*

Checks whether the generated answer directly responds to what the user actually asked.

---

## RAGAS (2023)

**RAGAS** is a major evaluation framework introduced in a 2023 paper that defined the three metrics above.

Since the original paper, several metrics have been renamed and many more have been added.

🔗 Official website: [ragas.io](https://www.ragas.io/)

---

## How Are These Metrics Measured?

### Approach 1 — Sentence Embedding Models
Use **cosine similarity** to compare:
- Answer ↔ Context
- Generated Questions ↔ Original Question

### Approach 2 — LLM-as-a-Judge
Use a powerful LLM (e.g., GPT-4) to evaluate:
- Whether the answer is grounded in context
- Whether the response is relevant
- Whether retrieved context supports the claims

The LLM also provides **reasoning** for its judgment.

---

## LYNX — Specialized Hallucination Evaluator

Instead of a general-purpose LLM, **LYNX** is purpose-built for evaluation.

| Property | Details |
|---|---|
| Type | Open-source hallucination evaluation LLM |
| Base Model | Fine-tuned LLaMA 3 |
| Purpose | Specifically designed for evaluation tasks |
| Performance | Outperforms GPT-4 on hallucination evaluation benchmarks |

🔗 Paper: [arxiv.org/abs/2407.08488](https://arxiv.org/abs/2407.08488)

---

## How RAGAS Measures Each Metric

### Faithfulness — 2-Step Pipeline

**Step 1:** An LLM breaks the answer into smaller factual statements.

*Example:*
> "Paris is the capital of France and has a population of 2 million."

Broken into:
- Paris is the capital of France
- Paris has a population of 2 million

**Step 2:** Another LLM verifies each statement against the retrieved context to determine if it's supported.

---

### Answer Relevance — Question Generation

1. An LLM generates possible questions that the answer could respond to
2. Embeddings of these generated questions are compared with the **original user question**
3. The similarity score determines answer relevance

---

### Context Relevance — Sentence Ratio

```
Context Relevance Score =
    (# of relevant extracted sentences)
    ─────────────────────────────────────
    (Total # of retrieved sentences)
```

*Simple interpretation: How much of what was retrieved was actually useful?*

---

## What To Do If Scores Are Low?

| Low Score | Possible Fix |
|---|---|
| **Faithfulness** | Try different LLMs; current models have explainability limits |
| **Context Relevance** | Try different embedding models; improve chunking strategy; refine retrieval pipeline |
| **Answer Relevance** | Improve prompt design; ensure the question is clearly stated |

---

## Key Takeaway

Hallucinations are a **by-product of LLMs** — they cannot be completely eliminated.

However, they **can be managed and significantly reduced** using:
- ✅ RAG pipelines
- ✅ Better retrieval systems
- ✅ Evaluation frameworks like **RAGAS**
- ✅ Specialized evaluators like **LYNX**

---

## Resources

| Resource | Link |
|---|---|
| Source Video | [YouTube](https://www.youtube.com/watch?v=xsDNArrmyuo) |
| RAGAS | [ragas.io](https://www.ragas.io/) |
| LYNX Paper | [arxiv.org/abs/2407.08488](https://arxiv.org/abs/2407.08488) |
| Air Canada Case | [Ars Technica](https://arstechnica.com/tech-policy/2024/02/air-canada-must-honor-refund-policy-invented-by-airlines-chatbot/) |