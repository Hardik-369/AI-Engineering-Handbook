# Resources

A curated list of the best learning resources for AI engineering. Organized by type for quick reference.

---

## Books

| Title | Author(s) | Year | Why Read It |
|-------|-----------|------|-------------|
| [The Alignment Problem](https://bostrom.com/alignment) | Brian Christian | 2020 | Essential context on AI safety; explains the key challenges in getting AI systems to do what we actually want. |
| [Understanding Deep Learning](https://udlbook.com/) | Simon J.D. Prince | 2023 | Most accessible deep learning textbook; strong focus on transformers and modern architectures. |
| [Deep Learning](https://www.deeplearningbook.org/) | Goodfellow, Bengio, Courville | 2016 | The canonical deep learning textbook; heavier on math but comprehensive. |
| [Speech and Language Processing](https://web.stanford.edu/~jurafsky/slp3/) | Jurafsky & Martin | 2023 (draft) | Free online NLP textbook; covers everything from N-grams to LLMs. |
| [Introduction to Information Retrieval](https://nlp.stanford.edu/IR-book/) | Manning, Raghavan, Schütze | 2008 | Foundation for all retrieval-based systems (RAG). |
| [Designing Machine Learning Systems](https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/) | Chip Huyen | 2022 | Practical production ML engineering; MLOps, data management, monitoring. |
| [Building Machine Learning Pipelines](https://www.oreilly.com/library/view/building-machine-learning/9781492057844/) | Hapke & Nelson | 2021 | Hands-on guide to TFX, Kubeflow, and production pipelines. |
| [Natural Language Processing with Transformers](https://transformersbook.com/) | Tunstall, Werra, Wolf | 2022 | HuggingFace-focused; practical transformer usage, fine-tuning, deployment. |
| [System Design Interview](https://www.amazon.com/System-Design-Interview-Insiders-Guide/dp/1736049119) | Alex Xu | 2020 | System design patterns that apply directly to LLM architecture. |

---

## Courses

| Course | Instructor | Platform | Focus |
|--------|-----------|----------|-------|
| [Neural Networks: From Zero to Hero](https://karpathy.ai/zero-to-hero.html) | Andrej Karpathy | YouTube | Build neural networks from scratch; essential for understanding the fundamentals. |
| [CS224n: Natural Language Processing with Deep Learning](https://web.stanford.edu/class/cs224n/) | Stanford | YouTube / Online | Classic NLP course with recent LLM updates. |
| [DeepLearning.AI Short Courses](https://www.deeplearning.ai/short-courses/) | Andrew Ng + various | Coursera | Bite-sized courses on prompt engineering, RAG, LangChain, agents. |
| [Fast.ai Practical Deep Learning](https://course.fast.ai/) | Jeremy Howard | Fast.ai | Top-down approach: start with working code, then understand theory. |
| [Full Stack Deep Learning](https://fullstackdeeplearning.com/) | Various | Online | Production ML engineering; deployment, monitoring, infrastructure. |
| [Hugging Face NLP Course](https://huggingface.co/learn/nlp-course) | Hugging Face | Online | Free, hands-on; transformers, tokenizers, datasets, fine-tuning. |
| [CS330: Deep Multi-Task and Meta Learning](https://cs330.stanford.edu/) | Chelsea Finn | Stanford | Advanced: multi-task learning, transfer learning, foundation models. |
| [Berkeley CS188: Intro to AI](https://inst.eecs.berkeley.edu/~cs188/) | Various | edX / Online | Classic AI; search, planning, reinforcement learning — agent foundations. |
| [Reinforcement Learning (David Silver)](https://www.davidsilver.uk/teaching/) | David Silver | YouTube | RL fundamentals; essential for understanding RLHF. |

---

## Papers

Organized by topic. Full reading list with summaries is in the [papers directory](../papers/README.md).

| Topic | Key Papers |
|-------|-----------|
| **Foundations** | Attention Is All You Need, GPT-2/3/4, InstructGPT, LLaMA |
| **Prompting** | Chain-of-Thought, Tree of Thoughts, ReAct, Self-Refine, Reflexion |
| **Context** | Lost in the Middle, LongBench, Efficient Transformers survey |
| **RAG** | Retrieval-Augmented Generation, Dense Passage Retrieval, GraphRAG |
| **Agents** | WebGPT, Generative Agents, SWE-agent, Toolformer |
| **Alignment** | Constitutional AI, RLHF, DPO, The Superalignment Problem |
| **Efficiency** | LLMLingua, AutoCompressors, Streaming LLM, Speculative Decoding |
| **Evaluation** | MMLU, HumanEval, GSM8K, BIG-bench, HELM, AlpacaEval |

---

## Documentation

| Provider | Docs | Key References |
|----------|------|---------------|
| **OpenAI** | [platform.openai.com/docs](https://platform.openai.com/docs) | API reference, tokenizer, model pricing, prompt engineering guide |
| **Anthropic** | [docs.anthropic.com](https://docs.anthropic.com/) | Claude prompt engineering, tool use, context caching, safety |
| **Google AI** | [ai.google.dev](https://ai.google.dev/) | Gemini API, model config, safety settings, grounding |
| **Hugging Face** | [huggingface.co/docs](https://huggingface.co/docs) | Transformers, Datasets, Tokenizers, PEFT, TRL, Hub |
| **LangChain** | [python.langchain.com](https://python.langchain.com/) | Chains, agents, retrievers, LCEL, integrations |
| **LangChain (JS/TS)** | [js.langchain.com](https://js.langchain.com/) | JavaScript versions of the above |
| **LlamaIndex** | [docs.llamaindex.ai](https://docs.llamaindex.ai/) | Indexing, retrieval, query engines, data connectors |
| **Ollama** | [github.com/ollama/ollama](https://github.com/ollama/ollama) | Local model serving, Modelfile, API |
| **vLLM** | [docs.vllm.ai](https://docs.vllm.ai/) | High-throughput LLM serving, KV cache optimization |
| **MLflow** | [mlflow.org/docs](https://mlflow.org/docs/) | LLM tracking, evaluation, model registry |
| **Weights & Biases** | [docs.wandb.ai](https://docs.wandb.ai/) | Experiment tracking, dataset versioning, LLM monitoring |
| **Phoenix (Arize)** | [docs.arize.com/phoenix](https://docs.arize.com/phoenix) | LLM observability, tracing, evaluation |
| **Langfuse** | [langfuse.com/docs](https://langfuse.com/docs) | Open-source LLM observability, prompt management |

---

## Blogs

| Blog | Why Follow |
|------|-----------|
| [Anthropic Blog](https://www.anthropic.com/blog) | Safety research, interpretability, Claude updates, alignment |
| [OpenAI Blog](https://openai.com/blog) | Frontier model releases, capability research, policy |
| [Google AI Blog](https://ai.googleblog.com/) | Gemini, research papers, TensorFlow updates |
| [Microsoft Research Blog](https://www.microsoft.com/en-us/research/blog/) | AI research, Phi models, Copilot |
| [Chip Huyen's Blog](https://huyenchip.com/blog/) | Production ML, real-world AI engineering, MLOps |
| [Simon Willison's Blog](https://simonwillison.net/) | Practical LLM usage, prompt patterns, tool building |
| [Lil'Log (Lilian Weng)](https://lilianweng.github.io/) | In-depth paper summaries, agent architectures, RLHF |
| [Stability AI Blog](https://stability.ai/blog) | Open-source models, generative media |
| [Cohere Blog](https://cohere.com/blog) | RAG, embeddings, enterprise AI |
| [Hugging Face Blog](https://huggingface.co/blog) | Model releases, benchmarks, open-source tools |
| [LangChain Blog](https://blog.langchain.dev/) | Agent patterns, production lessons, user stories |
| [Meta AI Blog](https://ai.meta.com/blog/) | LLaMA releases, open-source research |
| [DeepMind Blog](https://deepmind.google/blog/) | Frontier AI research, safety, AGI |
| [Sebastian Raschka's Blog](https://sebastianraschka.com/blog/) | LLM training, fine-tuning, math explained |

---

## Communities

| Community | Platform | Topics |
|-----------|----------|--------|
| [r/LocalLLaMA](https://reddit.com/r/LocalLLaMA) | Reddit | Self-hosting, open models, quantization, inference |
| [r/MachineLearning](https://reddit.com/r/MachineLearning) | Reddit | General ML research and discussion |
| [AI Alignment Forum](https://www.alignmentforum.org/) | Forum | AI safety, alignment research, interpretability |
| [Hugging Face Discord](https://discord.gg/huggingface) | Discord | Model sharing, fine-tuning, Transformers library |
| [LangChain Discord](https://discord.gg/langchain) | Discord | LLM app development, agent patterns, help |
| [LlamaIndex Discord](https://discord.gg/llamaindex) | Discord | RAG, data indexing, query engines |
| [OpenAI Developer Forum](https://community.openai.com/) | Forum | API usage, troubleshooting, best practices |
| [Anthropic Discord (Claude)](https://discord.gg/claude) | Discord | Prompt engineering, tool use, Claude API |
| [Hacker News (AI threads)](https://news.ycombinator.com/) | Forum | Tech discussion, new papers, tools |
| [r/OpenAI](https://reddit.com/r/OpenAI) | Reddit | OpenAI news, ChatGPT discussions |
| [r/ClaudeAI](https://reddit.com/r/ClaudeAI) | Reddit | Claude tips, use cases, Anthropic news |

---

## Tools

### Development Frameworks

| Tool | What It Does |
|------|-------------|
| [LangChain](https://github.com/langchain-ai/langchain) | Orchestration framework for LLM applications; chains, agents, retrievers |
| [LangGraph](https://github.com/langchain-ai/langgraph) | Graph-based agent orchestration (LangChain team) |
| [LlamaIndex](https://github.com/run-llama/llama_index) | Data framework for LLM apps; indexing, retrieval, querying |
| [DSPy](https://github.com/stanfordnlp/dspy) | Programming—not prompting—framework; compiler optimizes prompts |
| [Haystack](https://github.com/deepset-ai/haystack) | Search and RAG framework; production pipelines |
| [AutoGen](https://github.com/microsoft/autogen) | Multi-agent conversation framework (Microsoft) |
| [CrewAI](https://github.com/joaomdmoura/crewai) | Multi-agent orchestration; role-based agents |
| [Semantic Kernel](https://github.com/microsoft/semantic-kernel) | AI orchestration SDK (Microsoft); multi-language |
| [Flowise](https://github.com/FlowiseAI/Flowise) | Low-code LLM app builder; drag-and-drop |
| [Embedchain](https://github.com/embedchain/embedchain) | Easy RAG; one-liner to create LLM apps from data sources |

### Model Serving

| Tool | What It Does |
|------|-------------|
| [vLLM](https://github.com/vllm-project/vllm) | High-throughput LLM serving; PagedAttention, continuous batching |
| [Ollama](https://github.com/ollama/ollama) | Local model runner; one-command setup, API |
| [TGI (Text Generation Inference)](https://github.com/huggingface/text-generation-inference) | HuggingFace's production server; optimized inference |
| [llama.cpp](https://github.com/ggerganov/llama.cpp) | C++ inference for local/edge; quantization, cross-platform |
| [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) | NVIDIA's optimized inference; maximum GPU utilization |
| [LocalAI](https://github.com/mudler/LocalAI) | OpenAI API-compatible local inference; self-hosted |
| [BentoML](https://github.com/bentoml/BentoML) | Model serving framework; deploy any ML model |

### Monitoring & Observability

| Tool | What It Does |
|------|-------------|
| [Phoenix (Arize)](https://github.com/Arize-AI/phoenix) | LLM observability; span traces, evaluations, embeddings |
| [Langfuse](https://github.com/langfuse/langfuse) | Open-source LLM observability; tracing, prompt management |
| [LangSmith](https://www.langchain.com/langsmith) | LangChain's observability platform; traces, evals, datasets |
| [Weights & Biases](https://wandb.ai/) | Experiment tracking; prompt versioning, model registry |
| [Arize AI](https://arize.com/) | ML monitoring; drift detection, model performance |

### Vector Stores

| Tool | Type | What It Does |
|------|------|-------------|
| [ChromaDB](https://github.com/chroma-core/chroma) | Open-source | Simple, embedded; great for prototyping |
| [Qdrant](https://github.com/qdrant/qdrant) | Open-source | High-performance; filtering, quantization |
| [Weaviate](https://github.com/weaviate/weaviate) | Open-source | Hybrid search; graph+vector, modules |
| [Pinecone](https://www.pinecone.io/) | Managed | Fully managed, scalable, serverless |
| [Milvus](https://github.com/milvus-io/milvus) | Open-source | Distributed, GPU-accelerated, large-scale |
| [pgvector](https://github.com/pgvector/pgvector) | Extension | PostgreSQL vector storage; easy if already using Postgres |
| [Elasticsearch](https://www.elastic.co/what-is/vector-search) | Managed/OSS | Full-text + vector hybrid; mature ecosystem |
| [LanceDB](https://github.com/lancedb/lancedb) | Open-source | Embedded, columnar, multi-modal |

### Prompt Engineering

| Tool | What It Does |
|------|-------------|
| [OpenAI Playground](https://platform.openai.com/playground) | Interactive prompt testing; system/user/assistant roles |
| [Anthropic Console](https://console.anthropic.com/) | Claude prompt testing; comparison view |
| [PromptLayer](https://github.com/MagnivOrg/prompt-layer) | Prompt logging, versioning, analysis |
| [PromptPerfect](https://promptperfect.jina.ai/) | Automated prompt optimization |
| [Portkey](https://github.com/Portkey-AI/gateway) | AI gateway; prompt management, guardrails, caching |

### Fine-Tuning

| Tool | What It Does |
|------|-------------|
| [Axolotl](https://github.com/OpenAccess-AI-Collective/axolotl) | Fine-tuning framework; LoRA, QLoRA, full fine-tuning |
| [Unsloth](https://github.com/unslothai/unsloth) | 2x faster fine-tuning; memory efficient (LoRA/QLoRA) |
| [HuggingFace TRL](https://github.com/huggingface/trl) | Transformer Reinforcement Learning; PPO, DPO, SFT |
| [HuggingFace PEFT](https://github.com/huggingface/peft) | Parameter-Efficient Fine-Tuning; LoRA, AdaLoRA, IA3 |
| [OpenPipe](https://openpipe.ai/) | Managed fine-tuning; auto-collect training data |

### Evaluation

| Tool | What It Does |
|------|-------------|
| [LangChain Evaluation](https://docs.langchain.com/docs/guides/evaluation/) | String evaluators, pairwise, criteria-based |
| [DeepEval](https://github.com/confident-ai/deepeval) | LLM evaluation framework; metrics, CI integration |
| [RAGAS](https://github.com/explodinggradients/ragas) | RAG evaluation; faithfulness, relevance, precision |
| [EleutherAI LM Eval Harness](https://github.com/EleutherAI/lm-evaluation-harness) | Standardized benchmark evaluation; 200+ tasks |
| [AlpacaEval](https://github.com/tatsu-lab/alpaca_eval) | Automated LLM evaluation; pairwise comparison |
| [OpenAI Evals](https://github.com/openai/evals) | Evaluation framework; custom test cases, model grading |
| [Promptfoo](https://github.com/promptfoo/promptfoo) | Prompt testing and eval; CLI, matrix testing |

### Data

| Tool | What It Does |
|------|-------------|
| [Unstructured](https://github.com/Unstructured-IO/unstructured) | Ingestion and preprocessing of unstructured documents |
| [LlamaParse](https://github.com/run-llama/llama_parse) | Parse PDFs, docs, slides into structured markdown |
| [Docling](https://github.com/DS4SD/docling) | Document understanding; PDF, Word, PPT → structured |
| [pdfplumber](https://github.com/jsvine/pdfplumber) | Extract text/tables from PDFs; precise |
| [LangChain Document Loaders](https://python.langchain.com/docs/modules/data_connection/document_loaders/) | 100+ integration loaders for various data sources |

### Security & Guardrails

| Tool | What It Does |
|------|-------------|
| [Guardrails AI](https://github.com/guardrails-ai/guardrails) | Guardrails framework; input/output validation |
| [NVIDIA NeMo Guardrails](https://github.com/NVIDIA/NeMo-Guardrails) | Enterprise guardrails; content moderation, safety |
| [Rebuff](https://github.com/protectai/rebuff) | Prompt injection detection; multi-layered |
| [LMQL](https://github.com/eth-sri/lmql) | Constrained LLM generation; type-safe, scriptable |

---

## Frameworks Comparison

| Framework | Strengths | Weaknesses | Best For |
|-----------|-----------|------------|----------|
| **LangChain** | Huge ecosystem, many integrations, active community | Can be over-engineered, breaking changes | General-purpose LLM apps, prototyping |
| **LangGraph** | Clean agent architecture, cycles, state management | Steeper learning curve, newer | Agent loops, multi-step workflows |
| **LlamaIndex** | Excellent RAG, data connectors, query engines | Less focused on agents | Document Q&A, knowledge retrieval |
| **DSPy** | Optimizes prompts programmatically, research-backed | Less mature, smaller ecosystem | Prompt optimization, structured pipelines |
| **Haystack** | Production-ready, strong search, modular | Less community support for agents | Search/RAG, enterprise pipelines |
| **AutoGen** | Multi-agent, Microsoft-backed, conversation patterns | Complex setup, new | Multi-agent systems, assistant patterns |
| **CrewAI** | Simple API, role-based agents, easy to start | Limited flexibility for complex use | Role-based multi-agent workflows |

---

## Contributing

To add a resource, see the [contribution guide](../.github/CONTRIBUTING.md).

Resources are organized into categories. Please ensure new additions are:
- High quality and actively maintained (or historically significant)
- Directly relevant to AI engineering
- Properly categorized
- Described with a brief explanation of why it's useful
