

# Part I. Understanding Language Models

**Language AI** = Subfield of AI that focuses on developing technologies capable of understanding, processing, and generating human language.

**Encoder** = Embedding Models
**Decoder** = LLM Models (like GPT-4o-mini)

Encoder has can be used to look at entire sequence one in go. Compared to the encoder, decoder masks future positions. So it only attend earlier positions (Which causes the Halucinations).

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

**Token Embeddings:** They are the numeric representation space utilized to capture the meanings and patterns in language. (N x D)

**Contextual Embeddings:** Token's (or word's) coordinates shift based on the surrounding words in that specific sentence. 

**Text Embeddings:** A single vector represent the semantic meaning of entire sentence paragraph or document. (1 x D)


# Chapter 3. Looking Inside Large Language Models

**Self Attention Mechanism:** It compares every words to all other words in the same sentence at the same time. This helps the model figure out true meaning of a word based on its context. To do this, the transformer turns every word into three new parts:

1. **Query (Q):** What the word is "looking for" or asking about.
2. **Key (K):** What the word "contains" or offers to others.
3. **Value (V):** The actual meaning or information of the word.

 **Multi-Head Self-Attention:** Multi-Head Attention extends the self attention mechanism by using multiple attention heads in parallel. This mean transformer repeat self attention mechanism multiple times in parallel. Every head lets the model look at different things at the same time. [More Source For This Topic](https://www.geeksforgeeks.org/nlp/self-attention-in-nlp/)

**KV Cache (Key-Value Cache):** KV cache is optimization in LLMs used to generate faster. It stores previously computed attentions for text. But when material gets longer, it RAM usage run high. 

**Greedy Decoding:** At each step, the model guesses all possible next words, gives each a mathematical probability, and picks the single most likely word

**Why we chunking materials?** 

Because of **"Parallel Token Processing"**, and **Multi-Head Self-Attention** (explanation in above) we multiply all words by each other. Hardware needs more RAM when text, material, document when it gets longer, because RAM usage increasing $N^2$ geometrically. This is the answer of why we can't give 10000 page document into the model at once.
