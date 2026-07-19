# Resources: Tools, Libraries & Services

A curated reference organized by project type.

---

## General LLM Development

| Resource | URL | Use Case |
|----------|-----|----------|
| OpenAI API Docs | https://platform.openai.com/docs | Chat, embeddings, STT, vision |
| Anthropic API Docs | https://docs.anthropic.com | Claude models, tool use, long context |
| Google AI Studio | https://makersuite.google.com | Gemini models, free tier available |
| LiteLLM | https://github.com/BerriAI/litellm | Unified API for 100+ providers |
| OpenRouter | https://openrouter.ai | Access multiple models via single API |
| Portkey | https://portkey.ai | LLM observability, gateway, caching |

---

## RAG & Retrieval

| Resource | URL | Use Case |
|----------|-----|----------|
| LangChain | https://python.langchain.com | RAG pipelines, document loaders, chains |
| LlamaIndex | https://www.llamaindex.ai | Data framework for RAG |
| ChromaDB | https://www.trychroma.com | Local vector store, zero-setup |
| Qdrant | https://qdrant.tech | Production vector store, self-hosted or cloud |
| Pinecone | https://www.pinecone.io | Managed vector database |
| pgvector | https://github.com/pgvector/pgvector | PostgreSQL vector extension |
| Weaviate | https://weaviate.io | Vector + hybrid search |
| Milvus | https://milvus.io | Distributed vector database |
| RAGAS | https://docs.ragas.io | RAG evaluation framework |
| Unstructured | https://unstructured.io | Document parsing (PDF, HTML, DOCX) |

---

## Agent Frameworks

| Resource | URL | Use Case |
|----------|-----|----------|
| OpenAI Assistants API | https://platform.openai.com/docs/assistants | Managed agent with tools, code interpreter |
| Anthropic Tool Use | https://docs.anthropic.com/en/docs/build-with-claude/tool-use | Function calling with Claude |
| LangGraph | https://langchain-ai.github.io/langgraph | Stateful agent orchestration |
| AutoGen | https://github.com/microsoft/autogen | Multi-agent conversation framework |
| CrewAI | https://www.crewai.com | Role-based multi-agent orchestration |
| Semantic Kernel | https://learn.microsoft.com/semantic-kernel | Microsoft's AI orchestration |

---

## Memory & State

| Resource | URL | Use Case |
|----------|-----|----------|
| MemGPT/Letta | https://github.com/letta-ai/letta | OS-level memory for LLM agents |
| Zep | https://getzep.com | Long-term memory for AI assistants |
| LangMem | https://github.com/langchain-ai/langmem | Memory abstraction for LangChain |

---

## Speech & Audio

| Resource | URL | Use Case |
|----------|-----|----------|
| OpenAI Whisper | https://platform.openai.com/docs/guides/speech-to-text | Speech-to-text API |
| Whisper.cpp | https://github.com/ggerganov/whisper.cpp | Local STT, CPU-optimized |
| Deepgram | https://deepgram.com | Real-time speech recognition |
| ElevenLabs | https://elevenlabs.io | Text-to-speech, voice cloning |
| pyannote-audio | https://speaker-diarization.readthedocs.io | Speaker diarization |

---

## Knowledge Graphs

| Resource | URL | Use Case |
|----------|-----|----------|
| Neo4j | https://neo4j.com | Graph database (self-hosted + cloud) |
| NetworkX | https://networkx.org | Python graph analysis library |
| GraphRAG (Microsoft) | https://github.com/microsoft/graphrag | Microsoft's GraphRAG implementation |
| Diffbot | https://www.diffbot.com | Automated entity extraction from web pages |

---

## Search & Data

| Resource | URL | Use Case |
|----------|-----|----------|
| Tavily | https://tavily.com | AI-optimized search API |
| SerpAPI | https://serpapi.com | Google Search results API |
| Brave Search API | https://brave.com/search/api/ | Privacy-focused search API |
| Exa (Metaphor) | https://exa.ai | Semantic search for the web |
| Bing Search API | https://www.microsoft.com/en-us/bing/apis/bing-web-search-api | Microsoft search API |

---

## Financial Data

| Resource | URL | Use Case |
|----------|-----|----------|
| yfinance | https://github.com/ranaroussi/yfinance | Yahoo Finance stock data |
| Alpha Vantage | https://www.alphavantage.co | Free stock & forex API |
| Polygon.io | https://polygon.io | Real-time market data |
| Alpaca | https://alpaca.markets | Trading API (paper + live) |
| FRED API | https://fred.stlouisfed.org/docs/api/fred/ | Economic data from Fed |
| OpenBB | https://openbb.co | Open-source investment research |

---

## GitHub & DevOps

| Resource | URL | Use Case |
|----------|-----|----------|
| PyGithub | https://github.com/PyGithub/PyGithub | GitHub API v3 Python library |
| GitHub CLI | https://cli.github.com | GitHub from terminal |
| Octokit | https://github.com/octokit | GitHub API (JS/Python) |

---

## PDF Processing

| Resource | URL | Use Case |
|----------|-----|----------|
| PyPDF2 | https://pypdf2.readthedocs.io | PDF text extraction |
| PyMuPDF (fitz) | https://pymupdf.readthedocs.io | Fast PDF rendering & extraction |
| pdfplumber | https://github.com/jsvine/pdfplumber | Table extraction from PDFs |
| Camelot | https://camelot-py.readthedocs.io | Table extraction for complex PDFs |
| Azure Document Intelligence | https://azure.microsoft.com/products/ai-services/ai-document-intelligence | OCR, form recognition |
| Marker | https://github.com/VikParuchuri/marker | PDF to Markdown conversion |

---

## Code Execution & Sandboxing

| Resource | URL | Use Case |
|----------|-----|----------|
| Docker | https://docs.docker.com | Containerized code execution |
| Pyodide | https://pyodide.org | Python in browser WASM |
| Piston API | https://piston.readthedocs.io | Remote code execution API |
| E2B | https://e2b.dev | Cloud sandbox for AI agents |

---

## Evaluation & Observability

| Resource | URL | Use Case |
|----------|-----|----------|
| LangSmith | https://smith.langchain.com | Trace, evaluate, monitor LLM apps |
| Weights & Biases (Prompts) | https://wandb.ai/site/prompts | Prompt versioning & evaluation |
| MLflow | https://mlflow.org | Open-source ML lifecycle |
| Cleanlab | https://cleanlab.ai | LLM output quality monitoring |
| Arize | https://arize.com | ML observability platform |
| Helicone | https://helicone.ai | LLM cost & latency monitoring |

---

## Prompt Engineering

| Resource | URL | Use Case |
|----------|-----|----------|
| Prompt Engineering Guide | https://www.promptingguide.ai | Comprehensive guide & techniques |
| OpenAI Prompt Examples | https://platform.openai.com/examples | Official example prompts |
| Anthropic Prompt Library | https://docs.anthropic.com/en/prompt-library | Claude-optimized prompt templates |
| DSPy | https://dspy.ai | Programmatic prompt optimization |

---

## Model Access & Inference

| Resource | URL | Use Case |
|----------|-----|----------|
| Hugging Face | https://huggingface.co | Open-source models, datasets, spaces |
| Together AI | https://together.ai | Open-source model inference |
| Groq | https://groq.com | Ultra-fast inference |
| Fireworks | https://fireworks.ai | Fast model inference |
| Replicate | https://replicate.com | Model API marketplace |

---

## Deployment & Infrastructure

| Resource | URL | Use Case |
|----------|-----|----------|
| Modal | https://modal.com | Serverless GPU compute |
| Beam | https://beam.cloud | Serverless AI infrastructure |
| Railway | https://railway.app | Simple app deployment |
| Fly.io | https://fly.io | Edge deployment |
| Cloudflare Workers AI | https://developers.cloudflare.com/workers-ai | Edge LLM inference |

---

## Books & Papers

| Title | Author | Relevance |
|-------|--------|-----------|
| Building LLM Applications | Valentina Alto | End-to-end app development |
| LLM Engineer's Handbook | Paul Iusztin | Production LLM systems |
| AI Engineering | Chip Huyen | Full-stack AI engineering |
| Attention Is All You Need | Vaswani et al. | Original transformer paper |
| Retrieval-Augmented Generation... | Lewis et al. | RAG introduced |
| ReAct: Synergizing Reasoning... | Yao et al. | Agent reasoning pattern |
| GraphRAG: Unifying LLMs and KGs | Edge et al. | Graph-based RAG |
| Search-o1: Agentic Search-Enhanced Reasoning | Sun et al. | Advanced agent search |

---

## Project-Specific Quick Links

| Project | Key Resource |
|---------|-------------|
| ChatGPT Clone | https://platform.openai.com/docs/api-reference/chat |
| GraphRAG | https://github.com/microsoft/graphrag |
| Memory Agent | https://github.com/letta-ai/letta |
| Research Agent | https://docs.tavily.com |
| AI Coding Agent | https://docs.anthropic.com/en/docs/agents |
| PDF Chat | https://pymupdf.readthedocs.io |
| Meeting Assistant | https://platform.openai.com/docs/guides/speech-to-text |
| Personal AI | https://python.langchain.com/docs/modules/memory |
| Knowledge Base | https://docs.trychroma.com |
| Support Agent | https://python.langchain.com/docs/tutorials/agents |
| SQL Agent | https://platform.openai.com/docs/guides/function-calling |
| GitHub Agent | https://docs.github.com/en/rest |
| Writing Assistant | https://platform.openai.com/docs/guides/structured-outputs |
| AI Tutor | https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering |
| Financial Assistant | https://pypi.org/project/yfinance |
