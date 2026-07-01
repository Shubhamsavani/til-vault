# Phase 1 — What is Phoenix & How It Works

> **Repo (Code):** [Arize-Phoenix-Learning](https://github.com/Shubhamsavani/Arize-Phoenix-Learning)  
> **Repo (Docs):** [til-vault/RAG/Phoenix](https://github.com/Shubhamsavani/til-vault/RAG/Phoenix)

---

## What is Phoenix?

Phoenix is a UI and backend tool for collecting, storing, visualizing, evaluating, and analyzing traces from LLM applications. It is built on top of OpenTelemetry and OpenInference.

**Applications it can be used for:**
- Tracing LLM calls (single and chained)
- Visualizing RAG pipelines
- Evaluating agent workflows
- Debugging multi-step LLM chains

---

## Key Dependencies

| Tool | Role |
|---|---|
| **OpenTelemetry** | The tracing standard |
| **OpenInference** | LLM-specific semantic layer on top of OpenTelemetry |
| **Phoenix** | Collector, storage, UI, and evaluation layer |

---

## Starting Phoenix

```bash
phoenix serve
```

---

## Trace Creation — Basic Script

```python
import phoenix as px
from langchain_ollama import ChatOllama
from phoenix.otel import register
from openinference.instrumentation.langchain import LangChainInstrumentor

# Connect to Phoenix
tracer_provider = register()

# Instrument LangChain
LangChainInstrumentor().instrument(tracer_provider=tracer_provider)

# LLM
llm = ChatOllama(model="fake-model")

# Invoke
response = llm.invoke("What is observability?")
print(response.content)
```

---

## Mental Model — What Happens on Every `invoke()`

```
invoke()
    ↓ Instrumentation
    ↓ Span Created
    ↓ Trace Updated
    ↓ OTLP Export
    ↓ Phoenix Collector
    ↓ Phoenix Storage
    ↓ Phoenix UI
```

Once this mental model is clear, the next step is a **multi-span trace** (Prompt → LLM → Output Parser) to visually see parent-child span relationships — that is where Phoenix becomes genuinely useful.

---

## Core Concepts Summary

| Concept | Definition |
|---|---|
| **Trace** | A complete request journey |
| **Span** | One operation inside the journey |
| **Instrumentation** | Code that creates spans automatically |
| **OpenTelemetry** | The tracing standard |
| **OpenInference** | LLM-specific semantic layer on top of OpenTelemetry |
| **Phoenix** | UI and backend that collects, stores, visualizes, evaluates, and analyzes traces |

---

## Evolution of Logging — Timeline

### Era 1 — Custom Logging (Pre-2019)

```python
logger.info("Starting retrieval")
logger.info("Calling LLM")
logger.info("Finished")
```

**Problems:** No hierarchy, no latency tracking, no distributed tracing.

---

### Era 2 — OpenTracing (~2016–2022)

Projects like **Jaeger** and **Zipkin** introduced structured spans:

```python
span = tracer.start_span(...)
```

**Problem:** Every vendor had slightly different implementations.

---

### Era 3 — OpenTelemetry (~2019–Present)

The industry standardized on a single tracing spec — **OpenTelemetry** — instead of Datadog, Jaeger, New Relic, Zipkin each doing it differently.

```python
from opentelemetry import trace

tracer = trace.get_tracer(__name__)

with tracer.start_as_current_span("retrieve_documents"):
    ...
```

This is the **foundation Phoenix uses today**.

---

### Era 4 — LLM Explosion (2023)

OpenTelemetry understood `Span`, `Trace`, `Attributes`, `Events` — but not `Prompt`, `Tool Call`, `Retriever`, `Agent`, `RAG`, `Messages`.

So **OpenInference** was introduced (by Phoenix, Arize, LangChain, OpenAI, and others) as an LLM-specific semantic layer on top of OpenTelemetry.

```
OpenInference
    ↓
OpenTelemetry
    ↓
OTLP
    ↓
Phoenix
```

---

## Production Tracing Pattern

In production, manual `with tracer.start_as_current_span(...)` blocks everywhere are noisy. Teams wrap them in a **decorator**:

```python
def traced_function(name):
    def decorator(func):
        def wrapper(*args, **kwargs):
            with tracer.start_as_current_span(name):
                return func(*args, **kwargs)
        return wrapper
    return decorator
```

Usage:

```python
@traced_function("agent.process_message")
def process_message():
    ...
```

---

## Chain Tracing

> **File:** `{{REPO_ROOT}}/2-chain-tracing/1_chain_trace_creation.py`

When a LangChain chain is invoked:

```python
chain = prompt | llm | parser
response = chain.invoke({"topic": "OpenTelemetry"})
```

The execution flows as: **Prompt creation → LLM invocation → Output parser**

Phoenix automatically converts this execution graph into a **trace tree** where each stage becomes a span with a parent-child relationship. This is the foundation for everything done later with RAG, agents, tools, and workflows.

---

## Manual Span Creation

> **File:** `{{REPO_ROOT}}/2-chain-tracing/2_manual_span_creation.py`

```python
def post_process(response: str):
    with tracer.start_as_current_span("Post Process"):
        ...
```

This creates manual spans visible in Phoenix UI.

---

## Span Attributes

Attributes are the **single most important thing to learn after spans** — they are how production teams filter, search, debug, and analyze traces.

> **Mental Model:**
> - `Span` = *What happened?*
> - `Attributes` = *Context about what happened.*

**Without attributes:**
```
Generate Response
```

**With attributes:**
```
Generate Response
  model=llama3.2:3b
  user_id=user_123
  prompt_length=450
  request_type=educational
  temperature=0.7
```

> **File:** `{{REPO_ROOT}}/2-chain-tracing/3_attributes.py`

### Setting Attributes — Two Ways

**1. With decorator:**

```python
register()
tracer = trace.get_tracer(__name__)

def traced_function(name):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            with tracer.start_as_current_span(name):
                return func(*args, **kwargs)
        return wrapper
    return decorator

@traced_function("load_user_context")
def load_user_context(user_id):
    span = trace.get_current_span()
    span.set_attribute("user.id", user_id)
    span.set_attribute("user.plan", "premium")
    span.set_attribute("user.region", "india")
    return {"user_id": user_id, "plan": "premium"}
```

**2. Without decorator:**

```python
def build_prompt(topic):
    with tracer.start_as_current_span("build_prompt") as span:
        prompt = f"Explain {topic} in simple terms."
        span.set_attribute("topic", topic)
        span.set_attribute("prompt.length", len(prompt))
        return prompt
```

---

## Span Events

Events are **timestamped occurrences inside a span**, as opposed to attributes which are properties of the span.

> **Mental Model:**
> - `Attribute` = A property of a span
> - `Event` = A timestamped occurrence inside a span

> **Files:**
> - With decorators: `{{REPO_ROOT}}/2-chain-tracing/4_events.py`
> - Without decorators: `{{REPO_ROOT}}/2-chain-tracing/4_events_without_decorators.py`

### Events with decorator:

```python
@traced_function("generate_response")
def generate_response(prompt):
    span = trace.get_current_span()
    span.set_attribute("llm.model", "llama3.2:3b")

    span.add_event("Prompt Ready")

    llm = ChatOllama(model="llama3.2:3b")
    span.add_event("LLM Request Sent")

    response = llm.invoke(prompt)
    span.add_event("LLM Response Received")

    span.set_attribute("response.length", len(response.content))
    return response.content
```

### Events without decorator:

```python
def build_prompt(topic):
    with tracer.start_as_current_span("build_prompt") as span:
        span.add_event("Prompt Construction Started")
        prompt = f"Explain {topic}"
        span.set_attribute("topic", topic)
        span.add_event("Prompt Constructed")
        return prompt
```

### Events can also carry attributes:

```python
span.add_event("LLM Request Sent", {"model": "llama3.2:3b", "temperature": 0.7})
span.add_event("Document Retrieved", {"document_id": "doc_123", "score": 0.92})
```

---

## Errors

By default, error logging in spans is on. You can turn it off or record your own exceptions manually.

> **Files:**
> - `{{REPO_ROOT}}/2-chain-tracing/5_simple_error.py`
> - `{{REPO_ROOT}}/2-chain-tracing/6_nested_spans.py`
> - `{{REPO_ROOT}}/2-chain-tracing/7_nested_error.py`
> - `{{REPO_ROOT}}/2-chain-tracing/8_manual_error.py`

---

## Nested Spans

When a spanned function is called inside another spanned function, it creates a **hierarchy of spans**. This is how production codebases naturally build trace trees.

> **File:** `{{REPO_ROOT}}/2-chain-tracing/6_nested_spans.py`

```python
# Root Span
with tracer.start_as_current_span("answer_question"):

    prompt = build_prompt("OpenTelemetry")   # → span: build_prompt
    answer = generate_response(prompt)       # → span: generate_response
                                             #     → child: call_ollama
                                             #     → child: validate_response
    print(answer)
```

The `generate_response` function creates two child spans internally:

```python
@traced_function("generate_response")
def generate_response(prompt):

    # Child Span 1
    with tracer.start_as_current_span("call_ollama") as span:
        model_name = "llama3.2:3b"
        span.set_attribute("llm.model", model_name)
        llm = ChatOllama(model=model_name)
        response = llm.invoke(prompt)

    # Child Span 2
    with tracer.start_as_current_span("validate_response") as span:
        response_text = response.content
        span.set_attribute("response.length", len(response_text))
        is_valid = len(response_text) > 20
        span.set_attribute("validation.passed", is_valid)

        if not is_valid:
            raise ValueError("Response too short")

    return response_text
```

> In production codebases, using `@traced_function` over function calls naturally builds this hierarchy. The nested `with tracer.start_as_current_span(...)` blocks above are shown explicitly to understand what's happening under the hood.