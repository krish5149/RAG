# Document RAG Assistant

A FastAPI application for asking questions about uploaded documents. It uses a LangGraph pipeline for document ingestion and question answering, with LangSmith and Ragas scripts for evaluating answer quality and retrieval.

> **Status:** This repository does not currently include a dependency manifest or deployment configuration. The application also has environment-specific package and storage paths. Review the deployment considerations below before running it outside its original environment.

## Features

- Upload documents through a web interface or the API.
- Split documents into overlapping chunks and store embeddings in Chroma.
- Retrieve context using vector search and BM25 hybrid search.
- Rerank retrieved passages with a cross-encoder.
- Grade retrieved context for relevance, rewrite the query once when needed, and generate a context-grounded answer.
- Evaluate the RAG pipeline with LangSmith and Ragas.

The included evaluation dataset contains hospital-policy and medication examples. This project is not a medical device or a substitute for professional medical advice.

## How it works

### Ingestion

The upload endpoint saves files to the data directory and invokes the ingestion graph:

1. Load documents.
2. Split them into chunks using a 500-character chunk size and 200-character overlap.
3. Embed the chunks with `sentence-transformers/all-MiniLM-L6-v2`.
4. Store them in a Chroma collection named `chunked_documents`.

The document loader currently implements PDF and DOCX loading. Although the upload UI and endpoint also advertise `.txt`, plain-text loading is not implemented in the loader yet.

### Question answering

The query graph:

1. Routes greetings separately from document questions.
2. Retrieves passages using vector search and, when available, BM25 search.
3. Reranks passages with `cross-encoder/ms-marco-MiniLM-L-6-v2`.
4. Grades the context for relevance.
5. Rewrites the query and retries retrieval once if the context is not relevant.
6. Generates an answer using the retrieved context, or returns a fallback when relevant context is not found.

The generation and grading models are configured through OpenRouter in `rag_project/schemas/llms.py`.

## Repository layout

```text
rag_project/
├── fastapi_rag.py              # FastAPI application and query endpoint
├── upload_fie.py               # Document upload endpoint
├── graphs/
│   ├── storing_graph.py         # Document ingestion graph
│   └── query_graph.py           # Retrieval and answer-generation graph
├── ragservices/
│   ├── loaders.py               # PDF and DOCX loaders
│   ├── chunker.py               # Document chunking
│   ├── embed_store.py           # Chroma vector-store integration
│   ├── hybridsearch.py          # Vector + BM25 retrieval
│   └── reranker.py              # Cross-encoder reranking
├── ragevaluation/
│   ├── evaluation_dataset.json  # Evaluation questions and reference answers
│   ├── langsmith_evaluation.py  # LangSmith evaluation
│   └── ragass_evaluation.py     # Ragas evaluation
├── schemas/                     # API and structured LLM schemas
└── static/                      # Browser chat interface
