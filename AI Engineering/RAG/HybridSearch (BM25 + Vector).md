**Date:** 24.08.2026
**Tags:** #RAG #eval #hybridsearch

---

**Hybrid Search** combines keyword-based lexical search with dense semantic retrieval:

$$\text{Hybrid Search} = \text{Keyword Search (Sparse / BM25)} + \text{Semantic Search (Dense / Vector)}$$

#### 2. BM25 (Best Matching 25)

**BM25** is a probabilistic ranking algorithm used in Information Retrieval (IR) to estimate the relevance of a document ($D$) to a given search query ($q$).

The algorithm scores a document by evaluating **three core components**:

- **Term Frequency (TF) Saturation:** Measures how frequently a query term appears in the document. Unlike linear TF, BM25 applies asymptotic saturation controlled by $k_1$. Initial term occurrences significantly boost the score, while additional occurrences yield diminishing marginal gains, preventing keyword-stuffed documents from dominating.
    
- **Document Length Normalization:** Adjusts scores based on document length ($\vert{}D\vert{}$) relative to the average document length ($\text{avgdl}$). A term match in a short document is stronger evidence of relevance than a coincidental match in a 100-page document. The penalty strength is controlled by $b$.
    
- **Inverse Document Frequency (IDF):** Evaluates the rarity and discriminative power of a term across the **entire corpus** ($N$ documents). Rare terms (e.g., specific error codes or technical IDs) receive high weights, whereas ubiquitous terms receive low weights.
    

#### 3. Mathematical Formulation

For a query $q$ composed of terms $q_1, q_2, \dots, q_n$, the BM25 score of a document $D$ is defined as:

$$\text{BM25}(D, q) = \sum_{i=1}^{n} \text{IDF}(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot \left(1 - b + b \cdot \frac{\vert{}D\vert{}}{\text{avgdl}}\right)}$$

Where the **IDF component** is calculated as:

$$\text{IDF}(q_i) = \ln\left( \frac{N - n(q_i) + 0.5}{n(q_i) + 0.5} + 1 \right)$$

#### 4. Parameter and Variable Definitions

- **$q$**: The input search query consisting of tokens $\{q_1, q_2, \dots, q_n\}$.
- **$D$**: The candidate document being evaluated.
- **$f(q_i, D)$**: Raw term frequency of $q_i$ in document $D$.
- **$\vert{}D\vert{}$**: Length of document $D$ (total token / word count).
- **$\text{avgdl}$**: Average document length across the entire corpus:
- $$\text{avgdl} = \frac{1}{N} \sum_{k=1}^{N} \vert{}D_k\vert{}$$
- **$N$**: Total number of documents in the collection.
- **$n(q_i)$**: Number of documents containing the query term $q_i$.
- **$k_1$ (TF Saturation Parameter)**: Controls how quickly the term frequency reaches saturation (Default: $1.2 \le k_1 \le 2.0$). Higher values allow higher term frequency to have more impact before plateauing.
- **$b$ (Length Normalization Parameter)**: Controls document length penalization ($0 \le b \le 1$, Default: $0.75$).
    - $b = 1$: Full length normalization.
    - $b = 0$: No length normalization (document length is ignored).


### 5. Reciprocal Rank Fusion (RRF)

RRF is a simple method to combine search results from different retrieval systems (like BM25 and Vector Search).

#### The Problem: Why not just add scores?

- BM25 gives scores from $0$ to $+\infty$ (unbounded).
- Vector Search (Cosine Similarity) gives scores from $-1$ to $1$ or $0$ to $1$.
- You cannot add these raw scores directly because their scales are completely different
#### The Solution

Instead of looking at the **raw scores**, RRF only looks at the **rank position** (1st place, 2nd place, 3rd place, etc.) of each document. Documents that appear near the top in multiple lists get the highest combined score.

### 6. Mathematical Formulation and Parameters
The RRF score of a document $d$ across multiple ranked lists is defined as:

$$\text{RRF\_Score}(d) = \sum_{m \in M} \frac{1}{k + r_m(d)}$$

#### Parameter and Variable Definitions:

- $M$: The set of retrieval systems used (e.g., $M = \{\text{Dense}, \text{BM25}\}$).
- $m$: A single retrieval system in $M$
- $d$: The candidate document being evaluated.
- $r_m(d)$: The rank position (1-based index: $1, 2, 3, \dots$) of document $d$ in the result list of system $m$. If a document is not in the list, its score from that system is $0$.
- $k$: A constant smoothing parameter (standard default: $k = 60$). It prevents top-ranked documents from dominating the final score completely and balances the impact of lower ranks.

#### Quick Example:
Assume $k = 60$:
- If Document A is **Rank 1** in Dense Search and **Rank 3** in BM25:

$$\text{Score}(A) = \frac{1}{60 + 1} + \frac{1}{60 + 3} = \frac{1}{61} + \frac{1}{63} \approx 0.01639 + 0.01587 = 0.03226$$
    
- If Document B is **Rank 2** in Dense Search but **not found** in BM25:
$$\text{Score}(B) = \frac{1}{60 + 2} + 0 = \frac{1}{62} \approx 0.01613$$
Result: Document A wins because it was found by **both** search engines.