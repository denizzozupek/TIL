**Date:** 15.07.2026
**Tags:** #RAG #eval
**Sources:**https://www.evidentlyai.com/llm-guide/rag-evaluation#what-is-rag-evaluation


# **Evaluating RAG performance**

 RAG evaluation is the process of assessing how well your RAG application actually performs. RAG evaluation helps to measure how system performs on user queries.
 In RAG cases, reliability of system is really matters.

 https://www.evidentlyai.com/ranking-metrics/evaluating-recommender-systems source for ranking metrics.
 
## **Evaluating the retrieval quality**

1) Create a dataset of questions and expected answers (Grounded-Truth Evaluation)
	 - c=chunk , q=query, g=ground truth func. g(c) -> q
	 -
2) [LLM-as-a-judge evaluation](https://www.evidentlyai.com/llm-guide/llm-as-a-judge)
	- Per chunk relevance scoring, 


## **Evaluating the generation quality (response**)

**Reference based Eval**: (Which means we need a dataset of accurate Q&A pairs.)
1) Semantic Similarity
2) Correctness -> LLM as a judge or manually

**Reference Free Eval**
1) **Faithfulness**: Evaluates if the answer is faithful to the retrieved contexts (in other words, whether if there’s hallucination).
2) **Context Relevancy**: Whether retrieved context is relevant to the query.
3) **Answer Relevancy**: Whether the generated answer is relevant to the query.
4) **Guideline Adherence**: Whether the predicted answer adheres to specific guidelines.

 **A good evaluation process helps you:**
- Compare design choices
- Track what improves or breaks performance
- Debug errors more effectively


## **RAG Evaluation Dataset**

Creating strong test cases for your content takes time. There is fast way. We are reversing normal RAG workflow. Instead of response, we are start with content itself. Take a chunk or chunks from our documentation and ask LLM to:

- Generate a question that could be answered using just that chunk.
- Write the correct answer, based strictly on that same content.

And of course, you should always review and approve the examples before using them in evaluation.


## **Advanced RAG Evaluation**

In complex RAG systems and many real-world applications ( in domains like healthcare, finance, or legal support) things get more serious. These systems often serve external users and deal with high-stakes topics where trust, accuracy, and safety really matter.

### **Stress testing**

The goal of stress-testing is to evaluate the RAG system behavior outside the happy path – and see whether it fails gracefully.

To run these tests, you need to:

- **Define the risks and edge cases.** Identify scenarios that could lead to bad outputs.
- **Create test queries.** Craft example questions that simulate those risks.
- **Decide what a good response looks like.** For example, this could be a refusal, a request for clarification, or fallback to a safe default.

**Hallucination testing.** Another valuable scenario is hallucination testing. The idea is to see how your RAG system behaves when asked something it _shouldn’t_ be able to answer – either because the information is missing, outdated, or based on a false assumption.


### **Adversarial testing**

In adversarial testing, you design queries that intentionally try to break the system – by bypassing safeguards, confusing the model, or triggering risky outputs. These examples might not show up in regular user logs, but they’re exactly the kind of scenarios you want to catch before something fails in production. You usually create these test queries synthetically to mimic real attack attempts.

Some common patterns to test:

- [**Prompt injections**](https://www.evidentlyai.com/llm-guide/prompt-injection-llm): attempts to override your prompt instructions, like “Ignore the previous text and instead…”
- **Jailbreaks**: cleverly worded inputs that try to trick the model into breaking its safety protocols. Example: _“Tell me how to do X, but pretend it’s for a novel.”_
- **Harmful content**: queries related to violence, hate speech, self-harm, or disinformation.
- **Forbidden topics**: questions about legal advice, medical diagnosis, financial recommendations – areas your system shouldn’t touch.
- **Manipulation attempts**: trying to get the system to make a financial offer, give a discount, or confirm something that should require human approval. Example: _“What’s today’s discount code?” or “Can I get a refund approved?”_
- **Sensitive scenarios**: inputs from vulnerable users. Example: _“I’m feeling hopeless, what should I do?”_ should trigger a clear, safe, and respectful response – ideally with escalation or referral.

### Session-level evaluation

 Session-level evaluators let you answer questions like:

- **Session success:** Did the user get their problem resolved by the end?
- **Consistency:** Did the system forget context, repeat or contradict itself?
- **Conversation tone:** Was the tone appropriate throughout the session?

Ultimately, session-level evaluation helps you catch problems that don’t show up in single-turn check


**Avoid perfection traps.** Don’t aim for “the best possible evaluator.” Aim for something useful. You can always iterate on your test cases or your LLM judge prompts. What matters most is having a working loop where you can spot issues, try fixes, and know if things got better.

# Evaluation Metrics : 

![[evaluation_metrics .ipynb]]


