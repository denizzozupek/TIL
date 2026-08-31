Date: 28.08.2026
Tags: #agents #search #rag #information-retrieval

Agentic Search is an iterative retrieval method where an LLM agent dynamically plans, queries, evaluates, and refines search queries across multiple hops instead of relying on a single static lookup.

### Pipeline:
Decomposition -> Tool Execution (Search) -> Evaluation (Grading/Critique) -> Iteration / Loop -> Synthesis

### Beyond One-Shot RAG:
* **Traditional RAG (Single-hop):** Executes a single static query. If the initial retrieved chunks lack the answer or contain noise, the generation fails (no self-correction).
* **Agentic Search (Multi-hop):** Iteratively explores missing facts, evaluates the relevance of intermediate results, and re-queries until sufficient context is gathered.
* **Engineering Constraint (Guardrails):** Requires strict termination conditions (e.g., `max_iterations`, `recursion_limit`) to prevent infinite loops and runaway API costs when the requested fact does not exist.