my_rag_project/
│
├── data/                       # Local document storage
│   ├── raw/                    # Original PDFs, Markdown, or text files
│   └── processed/              # Cleaned text or cache files
│
├── src/                        # Main source code
│   ├── __init__.py
│   ├── config.py               # Environment variables, model names, paths
│   │
│   ├── ingestion/              # PIPELINE 1: Parsing and Vectorization
│   │   ├── __init__.py
│   │   ├── parser.py           # Document loaders (e.g., PyPDF, MarkItDown)
│   │   ├── chunker.py          # Text splitters (fixed size or semantic)
│   │   └── embedder.py         # Vector database connectors & embedding logic
│   │
│   ├── retrieval/              # PIPELINE 2: Fetching and Prompting
│   │   ├── __init__.py
│   │   ├── search.py           # Similarity search & hybrid search logic
│   │   ├── re_ranker.py        # Optional: Cross-encoder ranking optimizations
│   │   └── prompts.py          # System instructions and prompt templates
│   │
│   ├── generation/             # LLM Execution
│   │   ├── __init__.py
│   │   └── llm_client.py       # OpenAI, Anthropic, or Ollama wrappers
│   │
│   └── pipeline.py             # Main coordinator orchestrating the RAG flow
│
├── tests/                      # Unit and integration tests (e.g., via PyTest)
│   ├── test_chunker.py
│   └── test_retrieval.py
│
├── main.py                     # CLI or application entry point
├── app.py                      # Optional: Web UI entry point (Gradio, Streamlit)
├── .env                        # Private API keys and credentials
├── .gitignore                  # Prevents committing data/ and .env
├── requirements.txt            # Python package dependencies
└── README.md                   # Setup and usage instructions
