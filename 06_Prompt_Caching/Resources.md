# Resources: Prompt Caching

> Curated list of official guides, academic papers, tools, frameworks, and communities for LLM prompt caching.

---

## Official Provider Caching Guides

### Anthropic
- **Anthropic Prompt Caching Documentation** — https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
  Official guide covering `cache_control`, pricing, minimum token thresholds, and best practices.
- **Anthropic Prompt Caching Pricing** — https://www.anthropic.com/pricing
  Current pricing for cached vs. uncached tokens across all Claude models.
- **Anthropic Message API Reference** — https://docs.anthropic.com/en/api/messages
  API reference showing `cache_control` parameter in system and content blocks.

### OpenAI
- **OpenAI Prompt Caching Documentation** — https://platform.openai.com/docs/guides/prompt-caching
  Official guide for automatic prompt caching with GPT-4o and GPT-4o-mini.
- **OpenAI API Reference — Chat Completions** — https://platform.openai.com/docs/api-reference/chat
  Usage response includes `prompt_tokens_details.cached_tokens`.

### Google (Gemini)
- **Gemini Context Caching Guide** — https://ai.google.dev/gemini-api/docs/caching
  Official guide covering `CachedContent.create()`, TTL configuration, and pricing.
- **Gemini Context Caching Pricing** — https://ai.google.dev/pricing
  Storage costs and cached token discounts for Gemini 1.5 Pro and Flash.

---

## Academic Papers

### KV Cache & Inference Optimization

| Paper | Year | Authors | Key Contribution | Link |
|-------|------|---------|-----------------|------|
| "Efficient Memory Management for Large Language Model Serving with PagedAttention" (vLLM) | 2023 | Kwon et al. | Paged KV cache for memory efficiency | [arXiv](https://arxiv.org/abs/2309.06180) |
| "LightLLM: A Fast and Efficient LLM Serving System" | 2023 | Yao et al. | KV cache optimization for serving | [GitHub](https://github.com/ModelTC/lightllm) |
| "FlexGen: High-Throughput Generative Inference of Large Language Models with a Single GPU" | 2023 | Sheng et al. | KV cache offloading strategies | [arXiv](https://arxiv.org/abs/2303.06865) |
| "LLM in a Flash: Efficient Large Language Model Inference with Limited Memory" | 2023 | Alizadeh et al. | Flash memory for KV cache | [arXiv](https://arxiv.org/abs/2312.11514) |
| "Medusa: Simple LLM Inference Acceleration Framework with Multiple Decoding Heads" | 2023 | Cai et al. | Speculative decoding with KV cache | [arXiv](https://arxiv.org/abs/2401.10774) |

### Caching Systems & Semantic Caching

| Paper | Year | Authors | Key Contribution | Link |
|-------|------|---------|-----------------|------|
| "GPTCache: Semantic Cache for LLM Applications" | 2023 | Fu et al. | Open-source semantic caching framework | [GitHub](https://github.com/zilliztech/GPTCache) |
| "Scaling Semantic Caching for LLMs" | 2024 | Chavan et al. | Distributed semantic cache architecture | Preprint |
| "Cache-Augmented Generation (CAG)" | 2024 | Chan et al. | Caching for RAG pipelines | Preprint |

### Attention & KV Cache Theory

| Paper | Year | Authors | Key Contribution | Link |
|-------|------|---------|-----------------|------|
| "Attention Is All You Need" | 2017 | Vaswani et al. | Foundation of KV cache in transformer attention | [arXiv](https://arxiv.org/abs/1706.03762) |
| "Efficient Transformers: A Survey" | 2020 | Tay et al. | Survey of efficient attention mechanisms | [arXiv](https://arxiv.org/abs/2009.06732) |
| "Training Compute-Optimal Large Language Models" | 2022 | Hoffmann et al. | Chinchilla scaling laws relevant to inference cost | [arXiv](https://arxiv.org/abs/2203.15556) |
| "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness" | 2022 | Dao et al. | IO-aware attention enabling longer KV cache | [arXiv](https://arxiv.org/abs/2205.14135) |

---

## Tools & Frameworks

### Caching Libraries

| Tool | Description | Language | Link |
|------|-------------|----------|------|
| **RedisVL** | Redis Vector Library — semantic search on Redis | Python | https://github.com/RedisVentures/redisvl |
| **GPTCache** | Semantic caching for LLM queries | Python | https://github.com/zilliztech/GPTCache |
| **Semantic Cache** | Lightweight semantic cache with sentence-transformers | Python | https://github.com/zilliztech/GPTCache |
| **CacheGPT** | LLM response caching with TTL and invalidation | Python | https://github.com/krrishdholakia/cache-gpt |
| **LiteLLM** | Proxy with built-in caching support | Python | https://github.com/BerriAI/litellm |

### Vector Databases

| Solution | Type | Semantic Cache Support | Link |
|----------|------|----------------------|------|
| **Redis** (with RedisVL) | In-memory + vector | Native (HNSW) | https://redis.io/ |
| **Pinecone** | Managed vector DB | Native | https://www.pinecone.io/ |
| **Weaviate** | Self-hosted vector DB | Native | https://weaviate.io/ |
| **Qdrant** | Self-hosted vector DB | Native | https://qdrant.tech/ |
| **Chroma** | Lightweight vector DB | Native | https://www.trychroma.com/ |
| **pgvector** | PostgreSQL extension | Open-source | https://github.com/pgvector/pgvector |
| **Milvus** | Distributed vector DB | Native | https://milvus.io/ |

### Key-Value Stores

| Solution | Type | Use Case | Link |
|----------|------|----------|------|
| **Redis** | In-memory KV + vector | Primary cache backend | https://redis.io/ |
| **Memcached** | In-memory KV | Simple L1 cache | https://memcached.org/ |
| **Dragonfly** | Redis-compatible, higher throughput | High-performance cache | https://www.dragonflydb.io/ |
| **KeyDB** | Redis fork, multi-threaded | High concurrency | https://docs.keydb.dev/ |

### Inference Optimization (KV Cache)

| Framework | Description | Link |
|-----------|-------------|------|
| **vLLM** | High-throughput LLM serving with PagedAttention | https://github.com/vllm-project/vllm |
| **TGI** (Text Generation Inference) | Hugging Face's inference server | https://github.com/huggingface/text-generation-inference |
| **TensorRT-LLM** | NVIDIA's optimized LLM serving | https://github.com/NVIDIA/TensorRT-LLM |
| **llama.cpp** | CPU-optimized LLM inference | https://github.com/ggerganov/llama.cpp |
| **Ollama** | Local LLM runner with built-in KV cache | https://ollama.ai/ |

---

## Infrastructure & Monitoring

| Tool | Use Case | Link |
|------|----------|------|
| **Prometheus** | Cache metrics collection | https://prometheus.io/ |
| **Grafana** | Cache dashboard visualization | https://grafana.com/ |
| **Datadog** | APM + cache monitoring (paid) | https://www.datadoghq.com/ |
| **RedisInsight** | Redis GUI and monitoring | https://redis.com/redis-enterprise/redis-insight/ |
| **Loki** | Log aggregation for cache logs | https://grafana.com/oss/loki/ |

---

## Tutorials & Blog Posts

- **"Prompt Caching in Production"** (Anthropic Blog) — https://www.anthropic.com/news/prompt-caching
- **"Reducing LLM Costs with Prompt Caching"** (OpenAI Blog) — https://openai.com/index/api-prompt-caching/
- **"Semantic Caching for LLMs with Redis"** (Redis Blog) — https://redis.io/blog/semantic-caching/
- **"Building a Semantic Cache for LLMs with GPTCache"** (Zilliz Blog) — https://zilliz.com/blog/gptcache-semantic-cache
- **"A Practical Guide to LLM Caching"** (Weights & Biases) — https://wandb.ai/guides/llm-caching
- **"KV Cache Explained"** (Hugging Face Blog) — https://huggingface.co/blog/kv-cache
- **"How KV Cache Works in Transformers"** (Jay Alammar) — https://jalammar.github.io/illustrated-gpt2/
- **"Cost Optimization for LLM Applications"** (LiteLLM) — https://docs.litellm.ai/docs/proxy/caching

---

## Community & Discussion

| Community | Focus | Link |
|-----------|-------|------|
| Redis Discord | RedisVL, caching patterns | https://discord.gg/redis |
| LangChain Discord | LLM caching discussion | https://discord.gg/langchain |
| r/LocalLLaMA | KV cache optimization for local models | https://reddit.com/r/LocalLLaMA |
| r/MachineLearning | Research papers on caching | https://reddit.com/r/MachineLearning |
| Anthropic Discord | Prompt caching Q&A | https://discord.gg/anthropic |

---

## Books & Courses

- **"Designing Data-Intensive Applications"** by Martin Kleppmann — Caching fundamentals, distributed cache patterns (Chapters 4-6).
- **"Redis in Action"** by Josiah L. Carlson — Redis caching patterns, TTL management, eviction policies.
- **"Database Internals"** by Alex Petrov — B-trees, LSM trees, and cache-aware data structures.
- **"AI Engineering"** by Chip Huyen — LLM application design including caching strategies.
- **Stanford CS224N: NLP with Deep Learning** — Transformer architecture and KV cache concepts (Lecture 7-8).
- **MIT 6.824: Distributed Systems** — Distributed caching, consistency models.
