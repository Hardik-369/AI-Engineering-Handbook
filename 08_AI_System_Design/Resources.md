# Resources — Chapter 08: AI System Design

---

## Foundational Papers

### System Architecture

| Paper | Year | Why It Matters |
|-------|------|----------------|
| [Designing Machine Learning Systems](https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/) (Chip Huyen) | 2022 | Comprehensive guide to ML system design. Covers data engineering, feature stores, model deployment, monitoring. |
| [The ML Test Score: A Rubric for ML Production Readiness](https://research.google/pubs/pub46555/) | 2017 | Google's framework for evaluating ML systems' production readiness. Defines 28 tests across 4 categories. |
| [Hidden Technical Debt in Machine Learning Systems](https://papers.nips.cc/paper_files/paper/2015/hash/86df7dcfd896fcaf2674f757a2463eba-Abstract.html) | 2015 | Classic paper on the challenges of maintaining ML systems at scale. Introduces the CACE principle (Changing Anything Changes Everything). |
| [Rules of Machine Learning: Best Practices for ML Engineering](https://developers.google.com/machine-learning/guides/rules-of-ml) | 2018 | Google's 43 rules for building ML systems. Practical and battle-tested. |

### LLM-Specific Architecture

| Paper | Why It Matters |
|-------|----------------|
| [Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks](https://arxiv.org/abs/2005.11401) | The original RAG paper. Introduced the pattern of retrieval + generation. |
| [REACT: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) | Foundation for tool-using agents. Reason → Act → Observe loop. |
| [Toolformer: Language Models Can Teach Themselves to Use Tools](https://arxiv.org/abs/2302.04761) | How LLMs learn to use APIs and tools autonomously. |
| [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903) | Step-by-step reasoning pattern used in most agent systems. |
| [Tree of Thoughts: Deliberate Problem Solving with Large Language Models](https://arxiv.org/abs/2305.10601) | Multi-path reasoning with search and backtracking. |
| [Self-Consistency Improves Chain of Thought Reasoning in Language Models](https://arxiv.org/abs/2203.11171) | Ensembling multiple reasoning paths for better accuracy. |
| [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073) | Safety guardrails for AI systems. |
| [GraphRAG: Unlocking LLM Discovery on Narrative Private Data](https://www.microsoft.com/en-us/research/project/graphrag/) | Microsoft's graph-based RAG for complex query answering. |

### Evaluation & Monitoring

| Paper | Why It Matters |
|-------|----------------|
| [Evaluating Large Language Models Trained on Code](https://arxiv.org/abs/2107.03374) | HumanEval benchmark. Standard for code generation evaluation. |
| [Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena](https://arxiv.org/abs/2306.05685) | Using LLMs to evaluate LLMs. The LMSYS approach to quality assessment. |
| [Holistic Evaluation of Language Models (HELM)](https://arxiv.org/abs/2211.09110) | Stanford's comprehensive evaluation framework across 40+ scenarios. |

---

## Books

| Title | Author | Focus |
|-------|--------|-------|
| [Designing Machine Learning Systems](https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/) | Chip Huyen | ML system design, data engineering, production best practices |
| [Building Machine Learning Pipelines](https://www.oreilly.com/library/view/building-machine-learning/9781492053187/) | Hannes Hapke & Catherine Nelson | TFX and Kubeflow pipelines |
| [Machine Learning Engineering](https://www.mllib.com/) | Andriy Burkov | Production ML systems |
| [The AI Engineering Handbook](https://github.com/anthropics/ai-engineering-handbook) | Anthropic | Applied AI engineering patterns (this book!) |
| [System Design Interview](https://www.amazon.com/System-Design-Interview-Insiders-Guide/dp/1736049119) | Alex Xu | General system design (apply patterns to AI systems) |
| [Designing Data-Intensive Applications](https://www.oreilly.com/library/view/designing-data-intensive-applications/9781491903063/) | Martin Kleppmann | Distributed systems fundamentals (pre-requisite for AI systems) |

---

## Libraries & Frameworks

### Orchestration & Agent Frameworks

| Library | Purpose | GitHub Stars |
|---------|---------|-------------|
| [LangChain](https://github.com/langchain-ai/langchain) | Agent framework with chains, tools, memory | 95K+ |
| [LangGraph](https://github.com/langchain-ai/langgraph) | Graph-based agent orchestration | 7K+ |
| [CrewAI](https://github.com/crewAIInc/crewAI) | Multi-agent orchestration | 25K+ |
| [AutoGen](https://github.com/microsoft/autogen) | Microsoft's multi-agent framework | 35K+ |
| [Semantic Kernel](https://github.com/microsoft/semantic-kernel) | Microsoft's AI orchestration SDK | 22K+ |
| [Dify](https://github.com/langgenius/dify) | Open-source LLM app development platform | 55K+ |
| [Flowise](https://github.com/FlowiseAI/Flowise) | Low-code LLM app builder | 32K+ |
| [Vercel AI SDK](https://github.com/vercel/ai) | TypeScript toolkit for AI apps | 12K+ |

### RAG & Retrieval

| Library | Purpose |
|---------|---------|
| [LlamaIndex](https://github.com/run-llama/llama_index) | Data framework for RAG (30K+ stars) |
| [Haystack](https://github.com/deepset-ai/haystack) | NLP pipeline framework |
| [Chroma](https://github.com/chroma-core/chroma) | Open-source embedding database |
| [Weaviate](https://github.com/weaviate/weaviate) | Vector database with hybrid search |
| [Qdrant](https://github.com/qdrant/qdrant) | High-performance vector database |
| [Pinecone](https://www.pinecone.io/) | Managed vector database |
| [Milvus](https://github.com/milvus-io/milvus) | Distributed vector database |
| [Jina](https://github.com/jina-ai/jina) | Neural search framework |

### Document Processing

| Library | Purpose |
|---------|---------|
| [Unstructured](https://github.com/Unstructured-IO/unstructured) | Document parsing for LLMs (text, PDFs, HTML) |
| [PyMuPDF (fitz)](https://github.com/pymupdf/PyMuPDF) | Fast PDF text extraction |
| [pdfplumber](https://github.com/jsvine/pdfplumber) | PDF table extraction |
| [marker](https://github.com/VikParuchuri/marker) | PDF to markdown conversion |
| [docling](https://github.com/DS4SD/docling) | Document understanding by IBM |
| [Tesseract OCR](https://github.com/tesseract-ocr/tesseract) | Open-source OCR engine |
| [LayoutLM](https://huggingface.co/docs/transformers/model_doc/layoutlm) | Document layout understanding |

### Voice & Audio

| Library | Purpose |
|---------|---------|
| [Whisper](https://github.com/openai/whisper) | OpenAI's speech-to-text |
| [faster-whisper](https://github.com/SYSTRAN/faster-whisper) | Optimized Whisper with CTranslate2 |
| [Deepgram](https://deepgram.com/) | Real-time speech-to-text API |
| [ElevenLabs](https://elevenlabs.io/) | Text-to-speech API |
| [Cartesia](https://cartesia.ai/) | Low-latency TTS (Sonic) |
| [Silero VAD](https://github.com/snakers4/silero-vad) | Voice Activity Detection |
| [WebRTC VAD](https://github.com/wiseman/py-webrtcvad) | Lightweight VAD |
| [LiveKit](https://github.com/livekit/livekit) | Real-time audio infrastructure |

### Monitoring & Observability

| Tool | Purpose |
|------|---------|
| [LangSmith](https://smith.langchain.com/) | LLM observability and evaluation |
| [LangFuse](https://github.com/langfuse/langfuse) | Open-source LLM observability |
| [Weights & Biases](https://wandb.ai/) | ML experiment tracking (now supports LLMs) |
| [Helicone](https://www.helicone.ai/) | LLM observability platform |
| [Arize AI](https://arize.com/) | ML monitoring and observability |
| [WhyLabs](https://whylabs.ai/) | AI observability platform |
| [MLflow](https://mlflow.org/) | ML lifecycle management |
| [Grafana](https://grafana.com/) | Metrics visualization (use with Prometheus) |
| [Datadog](https://www.datadoghq.com/) | Full-stack monitoring |

### Caching & Performance

| Library | Purpose |
|---------|---------|
| [Redis](https://redis.io/) | In-memory caching (exact match cache) |
| [Semantic Cache](https://github.com/zilliztech/semantic-cache) | Embedding-based caching for LLMs |
| [GPTCache](https://github.com/zilliztech/gpucache) | Semantic caching for LLM queries |
| [vLLM](https://github.com/vllm-project/vllm) | High-throughput LLM serving |
| [TGI](https://github.com/huggingface/text-generation-inference) | HuggingFace's LLM server |
| [llama.cpp](https://github.com/ggerganov/llama.cpp) | Local LLM inference |

---

## Tools & Platforms

| Tool | Use Case |
|------|----------|
| [OpenAI API](https://platform.openai.com/) | GPT-4o, GPT-4o-mini, o3, embeddings |
| [Anthropic API](https://docs.anthropic.com/) | Claude 3.5, Claude 4 |
| [Together AI](https://www.together.ai/) | Multiple open-source models API |
| [Groq](https://groq.com/) | Ultra-fast LLM inference |
| [Replicate](https://replicate.com/) | Cloud API for open-source models |
| [Hugging Face](https://huggingface.co/) | Model hub, datasets, Spaces |
| [Modal](https://modal.com/) | Serverless GPU infrastructure |
| [Render](https://render.com/) | Web service hosting |
| [Fly.io](https://fly.io/) | Edge deployment for low latency |
| [Supabase](https://supabase.com/) | Open-source backend (DB + vector) |

---

## Courses & Tutorials

### System Design

| Resource | Creator | Focus |
|----------|---------|-------|
| [CS 329S: ML System Design](https://stanford-cs329s.github.io/) | Chip Huyen (Stanford) | Comprehensive ML system design |
| [Made With ML](https://madewithml.com/) | Goku Mohandas | Full-stack ML system design |
| [Full Stack Deep Learning](https://fullstackdeeplearning.com/) | UC Berkeley | Production ML course |
| [Stanford CS224N](https://web.stanford.edu/class/cs224n/) | Stanford | NLP with Deep Learning |
| [LLM Engineering](https://github.com/anthropics/llm-engineering) | Anthropic | Applied LLM engineering patterns |
| [Building LLM Apps](https://www.deeplearning.ai/short-courses/building-systems-with-chatgpt/) | DeepLearning.AI | Practical LLM system building |

### Architecture Patterns

| Resource | Description |
|----------|-------------|
| [LangGraph Tutorial](https://langchain-ai.github.io/langgraph/tutorials/) | Building graph-based AI systems |
| [CrewAI Examples](https://docs.crewai.com/examples/) | Multi-agent system patterns |
| [Haystack Tutorials](https://haystack.deepset.ai/tutorials) | NLP pipeline patterns |
| [LlamaIndex Guides](https://docs.llamaindex.ai/en/stable/) | RAG architecture patterns |
| [OpenAI Cookbook](https://cookbook.openai.com/) | Production patterns from OpenAI |
| [Vercel AI SDK Examples](https://sdk.vercel.ai/docs/guides) | Frontend + LLM integration |

---

## Blog Posts & Articles

| Title | Author | Why Read |
|-------|--------|----------|
| [Emerging Architectures for LLM Applications](https://a16z.com/2023/06/20/emerging-architectures-for-llm-applications/) | a16z | Foundational overview of LLM app architecture patterns |
| [The Building Blocks of LLM Applications](https://www.latent.space/p/building-blocks) | Latent Space | Practical component breakdown |
| [A Survey of Techniques for Maximizing LLM Performance](https://www.youtube.com/watch?v=ahnGLM-RC1Y) | OpenAI | System-level optimization techniques |
| [LLM Application Architecture](https://www.oreilly.com/radar/applying-large-language-models-in-the-real-world/) | O'Reilly | Real-world application patterns |
| [Building a Production RAG System](https://blog.llamaindex.ai/building-a-production-rag-system-2024) | LlamaIndex | Production RAG patterns |
| [The 10 Components of LLM Apps](https://medium.com/data-science-at-microsoft/the-10-components-of-llm-apps-8dbda8e2bed5) | Microsoft | Component breakdown |
| [Good System Design for AI Agents](https://www.anthropic.com/engineering/good-caching) | Anthropic | Agent design patterns |
| [Building LLM Applications for Production](https://huyenchip.com/2024/01/16/building-llm-apps.html) | Chip Huyen | Production considerations |

---

## Communities

| Platform | Where to Go |
|----------|-------------|
| [r/LocalLLaMA](https://reddit.com/r/LocalLLaMA) | Open-source LLM deployment |
| [r/MachineLearning](https://reddit.com/r/MachineLearning) | General ML discussion |
| [LangChain Discord](https://discord.gg/langchain) | LLM application development |
| [LlamaIndex Discord](https://discord.gg/llamaindex) | RAG and data frameworks |
| [Hugging Face Discord](https://discord.gg/huggingface) | Open-source models and tools |
| [AI Engineer Newsletter](https://aiengineer.substack.com/) | Curated AI engineering content |
| [The Batch (Andrew Ng)](https://www.deeplearning.ai/the-batch/) | Weekly AI news and tutorials |

---

## Reference Implementations

| Project | Description | GitHub |
|---------|-------------|--------|
| [ChatGPT Clone](https://github.com/vercel/ai-chatbot) | Full-stack chatbot with Next.js | Vercel |
| [OpenAI Demo](https://github.com/openai/openai-quickstart-python) | Official OpenAI quickstart | OpenAI |
| [LangChain Templates](https://github.com/langchain-ai/langchain/tree/master/templates) | Reference architectures | LangChain |
| [LlamaIndex Starter](https://github.com/run-llama/llama_index_starter) | RAG starter template | LlamaIndex |
| [Haystack Starter](https://github.com/deepset-ai/haystack-starter) | NLP pipeline starter | deepset |
| [RAG Evaluation](https://github.com/rungalileo/ragas) | RAG evaluation framework | Ragas |

---

## Chapter Cross-References

| Chapter | Connection |
|---------|------------|
| [Chapter 01: LLM Foundations] | Understanding models, tokenization, inference |
| [Chapter 02: Prompt Engineering] | Prompt patterns used across all system types |
| [Chapter 03: Context Engineering] | RAG, chunking, memory — core to most systems |
| [Chapter 04: Loop Engineering] | Agent loops, reflection, self-correction |
| [Chapter 05: Graph Engineering] | GraphRAG, agent graphs, multi-step reasoning |
| [Chapter 06: Prompt Caching] | Caching strategies for production systems |
| [Chapter 07: Agent Engineering] | Tool use, planning, multi-agent patterns |
| [Chapter 09: Production AI] | Monitoring, evaluation, deployment, operations |
| [Chapter 10: Projects] | End-to-end implementations of each system type |
