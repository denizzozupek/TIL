**Date:** 11.08.2026
**Tags:** #RAG #LangChain #LLM
# Table of Contents
- [What is LangChain?](#what-is-langchain)
- [1. Data Connection & Preprocessing](#1-data-connection--preprocessing)
  - [1.1 Document Loaders](#11-document-loaders)
  - [1.2 Text Splitters (Chunking Strategies)](#12-text-splitters-chunking-strategies)
- [2. Vector Stores & Embeddings](#2-vector-stores--embeddings)
  - [2.1 Embeddings Models](#21-embeddings-models)
  - [2.2 Vector Stores](#22-vector-stores)
- [3. Advanced Retrieval Strategies](#3-advanced-retrieval-strategies)
  - [3.1 Contextual Compression](#31-contextual-compression)
  - [3.2 MultiQueryRetriever](#32-multiqueryretriever)
  - [3.3 EnsembleRetriever (Hybrid Search)](#33-ensembleretriever-hybrid-search)
- [4. Output Parsers & Structured Outputs](#4-output-parsers--structured-outputs)
  - [4.1 PydanticOutputParser](#41-pydanticoutputparser)
- [5. Interoperability with LCEL](#5-interoperability-with-lcel)

---

## What is LangChain?

LangChain is an open-source framework designed to simplify the creation of applications using large language models (LLMs). While **LCEL (LangChain Expression Language)** serves as the execution engine for composing components via mathematical function composition, LangChain provides the surrounding ecosystem of abstractions required to build production-grade Retrieval-Augmented Generation (RAG) pipelines.

Architecturally, a complete RAG system consists of four primary layers:
$$\text{Pipeline} = \text{Ingestion} \longrightarrow \text{Indexing} \longrightarrow \text{Retrieval} \longrightarrow \text{LCEL Generation}$$

---

## 1. Data Connection & Preprocessing

Before passing data into an LCEL chain, unstructured external data must be ingested, parsed, and chunked into optimal sizes.

### 1.1 Document Loaders
Document Loaders standardize external sources (PDFs, Markdown, SQL tables, Web Pages) into LangChain `Document` objects.

Mathematically, a `Document` $D$ is a tuple consisting of text content $c$ and metadata mapping $M$:
$$D = (c, M) \quad \text{where } c \in \text{String}, \; M \in \{\text{key}: \text{value}\}$$

```python
from langchain_community.document_loaders import PyPDFLoader, DirectoryLoader

# Load all PDF files from a directory
loader = DirectoryLoader(
    path="./data",
    glob="*.pdf",
    loader_cls=PyPDFLoader
)
documents = loader.load()
````

### 1.2 Text Splitters (Chunking Strategies)

LLMs have finite context windows, and retrieval precision degrades with oversized text chunks. Text Splitters break large `Document` objects into smaller, overlapping chunks while preserving semantic boundaries.

  

#### RecursiveCharacterTextSplitter

Recursively splits text using a ordered list of separators `["\n\n", "\n", " ", ""]` until chunks reach the specified target size.

  



```Python
from langchain_text_splitters import RecursiveCharacterTextSplitter

text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,       # Target character length per chunk
    chunk_overlap=200,     # Overlap to preserve contextual continuity across boundaries
    length_function=len
)

chunks = text_splitter.split_documents(documents)
```

## 2. Vector Stores & Embeddings

### 2.1 Embeddings Models

Embedding models map discrete text chunks into a high-dimensional continuous vector space $\mathbb{R}^d$, where semantic similarity corresponds to geometric proximity.

  

Mathematically, an embedding function $E$ maps a string $s$ to a vector $\vec{v}$:

  

$$E: \text{String} \to \mathbb{R}^d$$


```Python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
```

### 2.2 Vector Stores

Vector stores index and store embedding vectors alongside their raw text and metadata, providing efficient vector similarity search (e.g., Cosine Similarity, Euclidean Distance).

  

Given a query vector $\vec{q}$ and stored document vectors $\vec{v}_i$, Cosine Similarity is computed as:

  

$$\text{Sim}(\vec{q}, \vec{v}_i) = \frac{\vec{q} \cdot \vec{v}_i}{\Vert{}\vec{q}\Vert{} \Vert{}\vec{v}_i\Vert{}}$$


```Python
from langchain_community.vectorstores import Chroma

# Index chunks into a persistent vector store
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# Convert vector store to an LCEL-compatible Retriever Runnable
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4} # Retrieve top 4 most similar chunks
)
```

## 3. Advanced Retrieval Strategies

Basic similarity search often retrieves redundant or noisy chunks. Advanced retrievers sit between the vector store and the LCEL prompt to optimize the retrieved context $C$.

  
### 3.1 Contextual Compression

Compresses retrieved documents using an LLM or reranker model to drop irrelevant sentences before injecting them into the prompt, reducing token overhead and noise.

  

```Python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import LLMChainExtractor
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
compressor = LLMChainExtractor.from_llm(llm)

compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=retriever
)
```

### 3.2 MultiQueryRetriever

Automates query expansion by using an LLM to generate multiple paraphrased variations of a user's input query. It executes retrieval across all query variations and takes the union of retrieved unique documents.

  

Given user query $q$, the LLM generates $Q = \{q_1, q_2, \dots, q_n\}$. The final document set $D_{\text{final}}$ is:

  

$$D_{\text{final}} = \bigcup_{i=1}^n \text{Retrieve}(q_i)$$

```Python

from langchain.retrievers.multi_query import MultiQueryRetriever

multi_query_retriever = MultiQueryRetriever.from_llm(
    retriever=vectorstore.as_retriever(),
    llm=llm
)
```

### 3.3 EnsembleRetriever (Hybrid Search)

Combines dense retrieval (Vector Embeddings for semantic matching) with sparse retrieval (BM25 for exact keyword matching) using **Reciprocal Rank Fusion (RRF)** to re-rank chunks.

Mathematically, the RRF score for a document $d \in D$ given rank positions $r_m(d)$ across retrieval methods $M$ is:

$$ text{RRF\_Score}(d \in D) = \sum_{m \in M} \frac{1}{k + r_m(d)}$$


```Python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever

# Sparse Keyword Retriever (BM25)
bm25_retriever = BM25Retriever.from_documents(chunks)
bm25_retriever.k = 4

# Dense Semantic Retriever (Chroma)
chroma_retriever = vectorstore.as_retriever(search_kwargs={"k": 4})

# Hybrid Retriever combining both techniques
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, chroma_retriever],
    weights=[0.3, 0.7] # 30% Keyword matching, 70% Semantic matching
)
```

## 4. Output Parsers & Structured Outputs

Output Parsers transform raw, unstructured string outputs $m \in \text{AIMessage}$ from LLMs into structured Python data types, JSON objects, or Pydantic models.

### 4.1 PydanticOutputParser

Forces the LLM to output valid JSON matching a predefined Pydantic schema, automatically parsing and validating the types at runtime.

  
```Python
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

# Define target data structure
class AnswerSchema(BaseModel):
    answer: str = Field(description="Direct concise answer to the query")
    confidence_score: float = Field(description="Confidence score between 0.0 and 1.0")
    sources: list[str] = Field(description="List of sources used from context")

parser = PydanticOutputParser(pydantic_object=AnswerSchema)

# Inject formatting instructions into the prompt
prompt = ChatPromptTemplate.from_template(
    "Answer the user query based on context.\n{format_instructions}\nContext: {context}\nQuestion: {question}"
).partial(format_instructions=parser.get_format_instructions())
```

## 5. Interoperability with LCEL

All data loaders, splitters, embeddings, vector stores, retrievers, and output parsers seamlessly integrate into LCEL pipelines as standard `Runnable` components.

```Python
from langchain_core.runnables import RunnablePassthrough

# End-to-End Advanced RAG Chain utilizing Hybrid Search and Structured Outputs
rag_chain = (
    RunnablePassthrough.assign(
        context=ensemble_retriever | (lambda docs: "\n\n".join(d.page_content for d in docs))
    )
    | prompt
    | llm
    | parser
)

# Returns a validated Pydantic object (AnswerSchema)
structured_response = rag_chain.invoke({"question": "What is LCEL composition?"})
```
For more: [[LangChain LCEL(LC Expression Language)]]