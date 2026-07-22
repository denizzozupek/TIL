

# Part I. Understanding Language Models

# Chapter 1. An Introduction to Large Language Models

**Language AI** = Subfield of AI that focuses on developing technologies capable of understanding, processing, and generating human language.

**Encoder** = Embedding Models
**Decoder** = LLM Models (like GPT-4o-mini)

Encoder has can be used to look at entire sequence one in go. Compared to the encoder, decoder masks future positions. So it only attend earlier positions.

**Encoder-Only Models:** Encoder-only models are designed to read and understand text.  They use "bi-directional" attention. This means the model looks at words before and after a specific word to understand its true meaning.

**Decoder-Only Models** (Generative Models): Decoder-only models are designed to **generate new text**. They use "unidirectional" (one-way) attention. The model can only look at the words that came before it. It guesses the next logical word.

**Context Window:** AI model's "working memory".  Maximum amount of text - measured in tokens - a model can read, process, and reference in a single prompt.

**Autoregressive Trap:**  Autoregressive trap is a major flaw in Large Language Models (LLMs) where a chatbot commits to an answer too early and then hallucinates or bends facts just to justify its initial mistake.

**The 2-Step LLM Training Lifecycle:** 

Step 1 is **Pretraining (Language Modeling)**; like grammar context, next token prediction. Extremely high compute and resources. Base Model. 
Step 2 is **Fine-Tuning (Post-Training)** is adapt the base model to exhibit desired behavior or excel at a narrow task. Highly specific, high-quality dataset. Low compute compare to step 1. 


# Chapter 2. Tokens and Embeddings

**Token:** The basic unit of data that LLM read and generate. A token can be a whole word (mathematics), a word root (writing), a suffix (-cı, -lar), a single letter (x), or a punctuation mark (!). Every token corresponds a different integer. 

**Tokenizer:** It takes the string entered by the user, splits into units and convert each unit into unique integer from dict resulting in a sequence of integers (Token IDs).

**Tokenization (Token Optimization Problem):** The process which decide the rules for splitting string into tokens. Most efficient one is Subword Tokenization.

**Byte Pair Encoding (BPE):** The industry-standard subword tokenization algorithm used by modern models like GPT and Llama. BPE iteratively merges the most frequently occurring character pairs in the training data to create a highly optimized vocabulary of subwords, preventing the out-of-vocabulary (OOV) problem efficiently.

**Token Embeddings:** They are the numeric representation space utilized to capture the meanings and patterns in language. (N x D)

**Contextual Embeddings:** Token's (or word's) coordinates shift based on the surrounding words in that specific sentence. 

**Text Embeddings:** A single vector represent the semantic meaning of entire sentence paragraph or document. (1 x D)


# Chapter 3. Looking Inside Large Language Models

**Feedforward Neural Network (FNN):** While the Attention mechanism acts as the "detective" to determine which words look at each other to build context, the FNN acts as the model's memory and generalization center. After attention is calculated, the FNN uses the massive amount of knowledge the model memorized during pre-training to predict the next logical step or pattern. 

_Transformer = Self-Attention (Understanding context) + Feedforward (Applying memorized knowledge/generalization)._

**Attention Layer:** The attention layer lets words (tokens) in a sentence look at each other to gain meaning.

**Self Attention Mechanism:** It compares every words to all other words in the same sentence at the same time. This helps the model figure out true meaning of a word based on its context. To do this, the transformer turns every word into three new parts:

1. **Query (Q):** What the word is "looking for" or asking about.
2. **Key (K):** What the word "contains" or offers to others.
3. **Value (V):** The actual meaning or information of the word.

 **Multi-Head Self-Attention:** Multi-Head Attention extends the self attention mechanism by using multiple attention heads in parallel. This mean transformer repeat self attention mechanism multiple times in parallel. Every head lets the model look at different things at the same time. [More Source For This Topic](https://www.geeksforgeeks.org/nlp/self-attention-in-nlp/)

Performance and Efficiency Comparison

| **Attention Variant**             | **Primary Benefit**                    | **How It Works**                                                                                                          | **Hardware Efficiency**                                                                            |
| --------------------------------- | -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **Multi-Head Attention (MHA)**    | High accuracy & reasoning              | Every token looks at every other token; unique Queries (Q), Keys (K), and Values (V) per attention head.                  | **Poor** (Slow speed, extremely high memory usage for long texts).                                 |
| **Local / Windowed Attention**    | Reduces math operations                | Tokens only look at nearby tokens within a fixed "window" instead of scanning the whole document.                         | **Good** (Cuts computational complexity from $O(N^2)$ to $O(N)$).                                  |
| **Multi-Query Attention (MQA)**   | Drastically cuts memory                | All attention heads share the exact same Key (K) and Value (V) vectors; only Queries (Q) remain unique.                   | **Great** (Massively reduces data loading bottlenecks during text generation, but drops accuracy). |
| **Grouped-Query Attention (GQA)** | Best of both worlds (Speed + Accuracy) | Instead of sharing 1 K/V across _all_ heads, it groups them (e.g., 8 Q heads share 1 K/V). Standard in Llama 3 & Mistral. | **Excellent** (Almost as fast as MQA, but retains the high accuracy of MHA).                       |
| **Ring Attention**                | Massive Context Windows (1M+ tokens)   | Splits the attention computation into blocks and passes them in a "ring" across multiple GPUs.                            | **Exceptional** (Allows processing entire books by overcoming single-GPU memory limits).           |
| **FlashAttention (V2/V3)**        | Raw hardware optimization              | Re-engineers the math steps to fit strictly into fast GPU memory (SRAM), minimizing slow read/write cycles.               | **State-of-the-Art** (2x to 4x speedups with zero loss in model accuracy).                         |

**KV Cache (Key-Value Cache):** KV cache is optimization in LLMs used to generate faster. It stores previously computed attentions for text. But when material gets longer, it RAM usage run high. 

**Greedy Decoding:** At each step, the model guesses all possible next words, gives each a mathematical probability, and picks the single most likely word.

**Generation Control Parameters (Beyond Greedy Decoding):** In production, we rarely use pure greedy decoding. We use parameters to control the model's output determinism and creativity:

- **Temperature:** Adjusts the probability distribution of the next token. A low temperature (e.g., 0.1) makes the model more deterministic and robotic (closer to greedy decoding). A high temperature (e.g., 0.8) flattens the distribution, making the model choose less likely words, increasing creativity but also the risk of hallucination.
    
- **Top-K Sampling:** The model only considers the _K_ most likely next tokens, ignoring the long tail of low-probability words.
    
- **Top-P (Nucleus) Sampling:** The model considers a dynamic pool of tokens whose cumulative probability reaches a defined threshold _P_ (e.g., 0.9).

**Why we chunking materials?** 

Because of **"Parallel Token Processing"**, and **Multi-Head Self-Attention** (explanation in above) we multiply all words by each other. Hardware needs more RAM when text, material, document when it gets longer, because RAM usage increasing $N^2$ geometrically. This is the answer of why we can't give 10000 page document into the model at once.

# Part II. Using Pretrained Language Models

# Chapter 4. Text Classification
