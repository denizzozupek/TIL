

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

**Token Embeddings:** They are the numeric representation space utilized to capture the meanings and patterns in language.