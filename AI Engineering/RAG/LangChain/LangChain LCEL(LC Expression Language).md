
Date: 11.08.2026
Tags: #RAG #LangChain #LLM #LCEL
source: https://github.com/aurelio-labs/langchain-course/blob/main/chapters/07-lcel.ipynb

----
# Table of Contents
- [Traditional Chains vs LCEL](#traditional-chains-vs-lcel)
  - [How Does the Pipe Operator Work?](#how-does-the-pipe-operator-work)
- [1. LCEL RunnableLambda](#1-lcel-runnablelambda)
- [2. RunnableParallel and RunnablePassthrough](#2-runnableparallel-and-runnablepassthrough)
  - [2.1. RunnablePassthrough.assign()](#21-runnablepassthroughassign)
- [3. RunnableConfig & configurable_fields / configurable_alternatives](#3-runnableconfig--configurable_fields--configurable_alternatives)
- [4. RunnableBranch & Dynamic Routing](#4-runnablebranch--dynamic-routing)
- [5. RunnableWithFallbacks (Fault Tolerance & Error Handling)](#5-runnablewithfallbacks-fault-tolerance--error-handling)
- [6. Streaming & Event Handling (astream_events)](#6-streaming--event-handling-astream_events)
- [7. RunnableWithMessageHistory (Stateful Memory Integration)](#7-runnablewithmessagehistory-stateful-memory-integration)
- [8. RunnableRetry (Exponential Backoff Retries)](#8-runnableretry-exponential-backoff-retries)

---
## Traditional Chains vs LCEL

LangChain Expression Language (LCEL) is the recommended approach to building chains in LangChain.
LLMChain is a legacy RAG chain method. 

```python
lcel_chain = prompt | llm | output_parser

```

We invoke this chain:

```python
lcel_chain.invoke("retrieval augmented generation")

```

### How Does the Pipe Operator Work?

The pipe operator ( `|` ) works like function composition in mathematics.

$$\text{lcel\_chain} = \text{prompt} \mid \text{llm} \mid \text{output\_parser}$$

Mathematically, if we represent each component as a function:

* $P(x)$: `prompt` (takes user input dictionary $x$, returns formatted prompt)
* $L(p)$: `llm` (takes formatted prompt $p$, returns `BaseMessage`)
* $O(m)$: `output_parser` (takes `BaseMessage` $m$, returns `str`)

The entire chain represents the composite function $(O \circ L \circ P)(x)$:

$$\text{lcel\_chain}(x) = O\Big(L\big(P(x)\big)\Big)$$

---

## 1. LCEL RunnableLambda

`RunnableLambda` converts a custom Python function $f$ into an LCEL-compatible `Runnable`, enabling arbitrary data transformations within the function composition pipeline $(f \circ g)(x)$.

```python
from langchain_core.runnables import RunnableLambda

def format_text(text: str) -> str:
    return text.strip().upper()

clean_text_runnable = RunnableLambda(format_text)

chain = prompt | llm | StrOutputParser() | clean_text_runnable

```

---

## 2. RunnableParallel and RunnablePassthrough

* **`RunnableParallel`**: Executes multiple runnables concurrently on the same input $x$ and returns a dictionary of results $H(x) = \{ \text{"k}_1\text{"}: f(x), \text{"k}_2\text{"}: g(x) \}$.
* **`RunnablePassthrough`**: Acts as the mathematical identity function $I(x) = x$, passing the input data unchanged to the next component in the pipeline.

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough

# Explicitly defining a RunnableParallel:
map_chain = RunnableParallel({
    "context": retriever | format_docs,
    "question": RunnablePassthrough()
})

# Note: In LCEL, passing a dict {} automatically coerces it into a RunnableParallel:
# map_chain = {"context": retriever | format_docs, "question": RunnablePassthrough()}

chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
)

```

Here, `RunnablePassthrough()` takes the string value from the incoming `{"question": "What is virtue?"}` input and passes it directly into the `"question"` key.

---

### 2.1. RunnablePassthrough.assign()

Merges new key-value pairs into the existing input dictionary without overwriting or dropping prior state.

Mathematically, given an input mapping $V$ and a key-value assignment $k = f(V)$, it computes the set union:

$$\text{assign}(k = f)(V) = V \cup \{ k : f(V) \}$$

**Key Benefit:** Unlike standard `RunnableParallel` (which rebuilds a dict from scratch), `.assign()` preserves all incoming fields (e.g., `chat_history`, `user_id`, `question`) and seamlessly injects computed fields (e.g., `context`).

#### Standard RunnableParallel:

```python
# Input: {"question": "Erdem nedir?", "chat_history": [...]}

chain = (
    {
        "context": retriever | format_docs,
        "question": lambda x: x["question"] # You have to write manually to keep other data.
    }
    | prompt
    | llm
)

```

#### RunnablePassthrough.assign():

```python
# Input: {"question": "Erdem nedir?", "chat_history": [...]}

chain = (
    # Injects "context" while keeping "question" and "chat_history" intact
    RunnablePassthrough.assign(
        context=history_search_chain | retriever | format_docs
    )
    | qa_prompt # Receives question, chat_history, AND context automatically
    | llm
)

```

*Output structure:* `{"question": "Erdem nedir?", "chat_history": ..., "context": "..."}`

---

## 3. RunnableConfig & configurable_fields / configurable_alternatives

Provides runtime flexibility by allowing parameters (e.g., `temperature`, `max_tokens`) or entire sub-components (e.g., swapping OpenAI for Claude) to be dynamically altered during the `.invoke()` call without modifying the chain architecture.

Mathematically, it parametrizes a function $f(x)$ with a configuration space $\theta \in C$, turning the composition into:

$$\text{chain}(x; \theta) = \Big(O \circ L_{\theta} \circ P\Big)(x)$$

* $x$: Input data
* $\theta$: Runtime config
* $L_{\theta}$: LLM function which specialized with theta parameter.

```python
from langchain_openai import ChatOpenAI
from langchain_core.runnables import ConfigurableField

# Modifying parameters dynamically at runtime
llm = ChatOpenAI(temperature=0.0).configurable_fields(
    temperature=ConfigurableField(id="llm_temp")
)

chain = prompt | llm | StrOutputParser()

# Pass custom parameters inside the config parameter during invoke:
response = chain.invoke(
    {"input": "Brainstorm project ideas"},
    config={"configurable": {"llm_temp": 0.8}}
)

```

---

## 4. RunnableBranch & Dynamic Routing

Dynamically routes the incoming input $x$ to different execution paths based on conditions or intent classification, operating as a mathematical piecewise function.

Mathematically, given predicates $P_i(x)$ and target runnables $f_i(x)$:

$$\text{chain}(x) = \begin{cases}  f_1(x), & P_1(x) = \text{True} \\  f_2(x), & P_2(x) = \text{True} \\  f_{\text{default}}(x), & \text{otherwise}  \end{cases}$$

#### Classical RunnableBranch:

```python
from langchain_core.runnables import RunnableBranch

branch = RunnableBranch(
    (lambda x: "math" in x["topic"].lower(), math_chain),
    (lambda x: "code" in x["topic"].lower(), code_chain),
    general_chain # Default fallback
)

```

#### Modern Router Function (Recommended):

```python
from langchain_core.runnables import RunnableLambda

# Custom routing function based on user intent/topic classification
def route_by_topic(input_dict: dict):
    # Check the classified topic and return the designated Runnable execution path
    if input_dict["topic"] == "math":
        return math_chain
    return general_chain

# Complete execution pipeline: 
# 1. Classifies the input intent -> 2. Dynamically routes to the corresponding chain
chain = classifier_chain | RunnableLambda(route_by_topic)

```

**Use Case for RAG:** Dynamic routing allows bypassing the Vector DB (`retriever`) for conversational queries (e.g., *"Hello"*, *"What did we talk about?"*) to save latency/cost, while routing document-specific questions to the full RAG pipeline.

**Routing Decision Strategies:**

1. **Rule-Based (Deterministic):** Uses explicit Python rules/keywords. Fast & zero cost, but rigid.
2. **LLM Classifier (Intent-Based):** Uses a fast/light LLM call to classify user intent dynamically. Highly accurate, small latency trade-off.
3. **Semantic Routing (Vector-Based):** Uses embedding similarity against predefined intent categories without invoking a full LLM. Fast and adaptable.

---

## 5. RunnableWithFallbacks (Fault Tolerance & Error Handling)

Wraps a primary `Runnable` $f$ with an ordered list of fallback `Runnables` $(g_1, g_2, \dots)$ to handle runtime exceptions (e.g., API outages, rate limits, or context window limits) gracefully.

Mathematically, it behaves as a guarded fallback composition:

$$\text{chain}(x) = \begin{cases}  f(x), & \text{if } \neg \text{Error}(f(x)) \\  g_1(x), & \text{otherwise}  \end{cases}$$

**Key Benefit:** Ensures high availability and production resilience at the LCEL layer without cluttering application code with manual `try-except` blocks.

```python
from langchain_openai import ChatOpenAI
from langchain_community.chat_models import ChatOllama

# Primary cloud model with an offline/local fallback model
primary_model = ChatOpenAI(model="gpt-4o-mini", max_retries=1)
local_fallback = ChatOllama(model="llama3.1")

# Create a resilient model with fallbacks
resilient_llm = primary_model.with_fallbacks([local_fallback])

# Pipeline setup remains clean and declarative
chain = prompt | resilient_llm | StrOutputParser()

```

---

## 6. Streaming & Event Handling (astream_events)

An asynchronous generator protocol in LCEL that yields fine-grained, real-time event streams ($S = \{e_0, e_1, \dots, e_T\}$) during chain execution.

Mathematically, it projects the internal execution states of a composite pipeline $(O \circ L \circ P)(x)$ into a temporal sequence of event tuples:

$$e_t = \Big( \text{type}_t, \; \text{component}_t, \; \Delta x_t \Big)$$

**Key Benefit:** Enables real-time UI token streaming (`on_llm_stream`) while concurrently providing intermediate pipeline status updates (e.g., retrieving context, re-writing queries) to the user interface.

```python
import asyncio
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("Tell me a short story about {topic}")
llm = ChatOpenAI(model="gpt-4o-mini")
parser = StrOutputParser()

chain = prompt | llm | parser

async def main():
    # Stream real-time events from the chain pipeline
    async for event in chain.astream_events({"topic": "software engineering"}, version="v2"):
        kind = event["event"]
        
        # Triggered when the LLM emits a new token chunk
        if kind == "on_chat_model_stream":
            content = event["data"]["chunk"].content
            if content:
                print(content, end="", flush=True)
                
        # Triggered when a retriever begins searching
        elif kind == "on_retriever_start":
            print(f"\n[Status]: Searching vector database via {event['name']}...")

# Run async event stream loop
# asyncio.run(main())

```

---

## 7. RunnableWithMessageHistory (Stateful Memory Integration)

Wraps a stateless LCEL chain to automatically load and save conversational history based on a session identifier (`session_id`).

Mathematically, it transforms a stateless mapping $f(x)$ into a stateful recurrence relation given a state store $S$:

$$\text{chain}(x; \text{session\_id}) = f\Big( x \cup S[\text{session\_id}] \Big) \quad \text{and updates} \quad S[\text{session\_id}] \leftarrow S[\text{session\_id}] \cup \{ \text{HumanMessage}(x), \text{AIMessage}(y) \}$$

**Key Benefit:** Keeps chains completely stateless while delegating session persistence out-of-band, avoiding manual state management inside the pipeline logic.

```python
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_community.chat_message_histories import ChatMessageHistory

# 1. Define Stateless Base Chain
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    MessagesPlaceholder(variable_name="chat_history"), # History injects here
    ("human", "{input}")
])

base_chain = prompt | ChatOpenAI(model="gpt-4o-mini") | StrOutputParser()

# 2. In-Memory Session Store
store = {}

def get_session_history(session_id: str):
    if session_id not in store:
        store[session_id] = ChatMessageHistory()
    return store[session_id]

# 3. Wrap Base Chain into a Stateful Chain
conversational_chain = RunnableWithMessageHistory(
    base_chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history"
)

# 4. Invoke using a Session Identifier via config
response1 = conversational_chain.invoke(
    {"input": "Hi, my name is Erdem"},
    config={"configurable": {"session_id": "user_session_1"}}
)

# Second invocation automatically retains "Erdem"
response2 = conversational_chain.invoke(
    {"input": "What is my name?"},
    config={"configurable": {"session_id": "user_session_1"}}
)

```

---

## 8. RunnableRetry (Exponential Backoff Retries)

Automatically retries a failing `Runnable` $f(x)$ with exponential backoff delay and jitter before throwing an exception or triggering a fallback.

**Key Benefit:** Handles transient network failures and rate limits (e.g., HTTP 429/503) gracefully at the component level without making unnecessary cross-provider fallbacks.

```python
from langchain_openai import ChatOpenAI
from langchain_core.runnables import RunnableRetry
from langchain_core.output_parsers import StrOutputParser

llm = ChatOpenAI(model="gpt-4o-mini")

# Method 1: Using .with_retry() shorthand on a Runnable (Recommended)
retryable_llm = llm.with_retry(
    stop_after_attempt=4,         # Retry up to 4 times
    wait_exponential_jitter=True   # Enables exponential backoff + random jitter
)

# Method 2: Explicit RunnableRetry wrapping
explicit_retry = RunnableRetry(
    bound=llm,
    max_attempt_number=3
)

# Pipeline incorporating resilient retries
chain = prompt | retryable_llm | StrOutputParser()

```

**`RunnableRetry` vs. `RunnableWithFallbacks`:**

* **`RunnableRetry`** handles **transient/temporary errors** (e.g., rate limits, network timeouts) by re-executing the **same** component with exponential backoff.
* **`RunnableWithFallbacks`** handles **permanent/systemic failures** (e.g., total service outage, context length exceeded) by switching execution to an **alternative** component (e.g., fallback LLM provider).

**Best Practice Pattern:** Combine both by chaining retries inside a fallback wrapper:

$$\text{final\_llm} = \text{primary\_llm.with\_retry}(\dots).\text{with\_fallbacks}([\text{backup\_llm}])$$

