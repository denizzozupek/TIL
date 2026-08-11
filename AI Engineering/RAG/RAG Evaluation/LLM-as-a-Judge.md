LLM-as-a-Judge uses LLMs to evaluate retrieval and generation based on custom criteria defined in an evaluation prompt.

To apply this method, take the text output from your RAG response **and** feed it back to an LLM with **an evaluation** prompt. LLM-as-a-Judge **is** not an **evaluation** metric. It's a general technique where you use **an** LLM to **approximate** human labeling. Yes, **on** both sides of the process we **are** using **LLMs**, but we **are separating the** tasks. Generating **a** response handles many variables, but **the** LLM **evaluator is** only focused **on** the **evaluation** prompt.

### Types of LLM Judges

- **Pairwise Comparison:** Give **the** LLM two responses and ask **it to** choose the better one.
    
- **Evaluation by criteria:** Instead of focusing on general preference, it can handle one criterion at a time (tone, clarity, etc.).
    
- **Reference-based evaluation:**
    
    - Evaluating correctness based on **a** reference answer.
        
    - Evaluating answer quality considering the question (**like** completeness and **relevance**).
        
    - Scoring context **relevance** in RAG.
        
    - Evaluating hallucinations in RAG.
        

### How to create an LLM Judge

1. Define the evaluation scenario.
    
2. Prepare the evaluation dataset.
    
3. Label this dataset.
    
4. Craft your evaluation prompt.
    
5. **Evaluate** and iterate.
    

### LLM Evaluation Prompts

Tips for **LLM** evaluation prompts:

- Use binary or low-precision scoring.
    
- Explain the meaning of each score.
    
- Simplify **evaluation** by splitting criteria.
    
- Add examples to the prompt.
    
- Encourage step-by-step reasoning.
    
- Set a low temperature.
    
- Use a more capable model.
    
- Get structured outputs.
    

### LLM observability in production

Trace the data, schedule **evaluations**, and build a dashboard.

### Pros and Cons

**Pros**

LLM judges **are** easy to update (**the evaluation** prompt is easy to update), flexible, **need no reference** (**the** LLM's own knowledge can be used as **a** reference - because of that, use more capable models), and **the evaluations are of** high quality.

**Cons**

- **No evaluation** method is perfect. **The same applies to the** LLM judge. **The** LLM prompt **heavily affects** the **evaluation;** if **the** prompt is not **good**, then **the evaluation will be poor as well**. Manage expectations: for now, there **is no evaluation** method **which gives 100%** correctness.
    
- Bias risk: Position, verbosity, and **self-enhancement** bias.
    
- Data privacy: We are using **third-party LLMs**, and we are sharing our **data** with **third parties**.
    
- Faster than humans but not **fast** enough **compared to rule-based** checks or smaller ML models.
    
- Expensive, not free.