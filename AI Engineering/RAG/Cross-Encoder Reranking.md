**Date:** 24.08.2026
**Tags:** #RAG #eval #rearanking

---

A **Cross-Encoder** is a Transformer-based neural network model used to perform pairwise scoring between a search query ($q$) and a candidate document ($D$).

Unlike Bi-Encoders that process texts separately, a Cross-Encoder combine both texts into a **single input sequence** and feeds them simultaneously into the model:

$$\text{Input} = \text{[CLS]} \circ q \circ \text{[SEP]} \circ D \circ \text{[SEP]}$$

- **Mechanism:** It runs **Full Cross-Attention** across all Transformer layers. Every single token in the query attends to every single token in the document from layer 1 to the final layer.
    
- **Output:** A classification or regression head on top of the `[CLS]` token outputs a single scalar score $s \in [0, 1]$ (or relativity score) representing the direct semantic relevance of $D$ with respect to $q$.
    

$$\text{Score}(q, D) = \sigma\left(\mathbf{W} \cdot \mathbf{h}_{\text{[CLS]}} + b\right)$$

_(where $\mathbf{h}_{\text{[CLS]}}$ is the final hidden state of the `[CLS]` token, $\mathbf{W}$ and $b$ are learnable projection weights, and $\sigma$ is typically the sigmoid function)._

#### 2. Bi-Encoder vs. Cross-Encoder: The Engineering Trade-off

|**Architectural Dimension**|**Bi-Encoder (Embedding Retrieval)**|**Cross-Encoder (Reranker)**|
|---|---|---|
|**Input Processing**|Encodes $q$ and $D$ **independently** into fixed-size dense vectors $\mathbf{u}, \mathbf{v} \in \mathbb{R}^d$.|Encodes $(q, D)$ **jointly** as a unified sequence with token-level cross-attention.|
|**Pre-computation**|**Yes:** Document embeddings are computed offline once and stored in a vector index.|**No:** Cannot pre-compute. Must execute full model forward pass at runtime for each $(q, D)$ pair.|
|**Scoring Mechanism**|Cosine Similarity / Dot Product ($\mathbf{u} \cdot \mathbf{v}$).|Deep interaction layers $\rightarrow$ Single relevance logit.|
|**Time Complexity**|$O(N \cdot d)$ via Flat scan or $O(\log N)$ via ANN graphs (HNSW).|$O(K \cdot L^2 \cdot \text{Layers})$ where $K$ is candidate count and $L$ is sequence length.|
|**Relevance Accuracy**|Moderate (Information is compressed into a single vector, losing fine-grained interactions).|**State-of-the-Art (SOTA)** (Captures subtle token-level nuances and syntactic relationships).|

#### 3. Why Two-Stage Retrieval is Mandatory

Because of the high computational latency of Cross-Encoders ($O(K \cdot L^2)$), running them across an entire corpus of $N = 100,000$ documents in real-time is computationally impossible.

- **Stage 1 (High Recall / Broad Retrieval):** Use Hybrid Search (BM25 + Bi-Encoder) to filter $N = 100,000$ documents down to the top $K = 50$ candidates in milliseconds ($O(\log N)$).
    
- **Stage 2 (High Precision / Reranking):** Use a Cross-Encoder only on the $K = 50$ candidates to perfectly re-sort them and select the final top $M = 4$ context chunks for the LLM ($O(K)$).



pot 1
real madrid
inter

pot 2 
porto
dortmund 

pot 3
napoli 
villareal

pot 4
aek
como