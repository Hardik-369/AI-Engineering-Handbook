# Resources: Context Engineering

## Core Papers

### Lost in the Middle

Liu, N. F., Lin, K., Hewitt, J., Paranjape, A., Bevilacqua, M., Petroni, F., & Liang, P. (2023). **Lost in the Middle: How Language Models Use Long Contexts.**

- The foundational paper on context position effects.
- Finds that models perform best on information at the start and end of context, with a significant drop in the middle.
- Tests across multiple model families and sizes.
- *Link:* https://arxiv.org/abs/2307.03172

### Context Windows & Attention

Press, O., Smith, N. A., & Lewis, M. (2022). **Train Short, Test Long: Attention with Linear Biases Enables Input Length Extrapolation.**

- Introduces ALiBi positional encoding, which enables better extrapolation to longer contexts.
- *Link:* https://arxiv.org/abs/2108.12409

Su, J., Lu, Y., Pan, S., Murtadha, A., Wen, B., & Liu, Y. (2024). **RoFormer: Enhanced Transformer with Rotary Position Embedding.**

- RoPE is used in Llama, Mistral, and many modern models.
- Understanding RoPE helps explain why some models handle long contexts better than others.
- *Link:* https://arxiv.org/abs/2104.09864

### Chunking & Retrieval

Lewis, P., Perez, E., Piktus, A., Petroni, F., Karpukhin, V., Goyal, N., ... & Kiela, D. (2020). **Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks.**

- The original RAG paper — introduced the retrieve-then-generate paradigm.
- *Link:* https://arxiv.org/abs/2005.11401

Karpukhin, V., Oğuz, B., Min, S., Lewis, P., Wu, L., Edunov, S., ... & Yih, W. T. (2020). **Dense Passage Retrieval for Open-Domain Question Answering.**

- DPR: The standard approach for dense retrieval in QA systems.
- *Link:* https://arxiv.org/abs/2004.04906

Robertson, S., & Zaragoza, H. (2009). **The Probabilistic Relevance Framework: BM25 and Beyond.**

- The definitive reference for BM25 and its variants.
- *Link:* https://dl.acm.org/doi/10.1561/1500000019

### Memory Systems

Graves, A., Wayne, G., & Danihelka, I. (2014). **Neural Turing Machines.**

- Early work on differentiable memory for neural networks.
- *Link:* https://arxiv.org/abs/1410.5401

Weston, J., Chopra, S., & Bordes, A. (2014). **Memory Networks.**

- Introduced the idea of explicit memory for QA.
- *Link:* https://arxiv.org/abs/1410.3916

### Context Compression

Chevalier, A., Wettig, A., Ajith, A., & Chen, D. (2023). **Adapting Language Models to Compress Long Contexts.**

- AutoCompressors: Training models to produce summary vectors for long texts.
- *Link:* https://arxiv.org/abs/2305.14739

Jiang, H., Wu, Q., Lin, C., Yang, Y., & Qiu, L. (2023). **LLMLingua: Compressing Prompts for Accelerated Inference of Large Language Models.**

- A learned approach to prompt compression using a small LM to predict token importance.
- *Link:* https://arxiv.org/abs/2310.05736

### Long-Context Evaluation

Bai, Y., Lv, X., Zhang, J., Lyu, H., Tang, J., Huang, Z., ... & Yuan, L. (2024). **LongBench: A Bilingual, Multitask Benchmark for Long Context Understanding.**

- Comprehensive benchmark for evaluating long-context capabilities.
- *Link:* https://arxiv.org/abs/2308.14508

## Libraries & Tools

### Retrieval

| Tool | Type | Description |
|------|------|-------------|
| [LangChain](https://github.com/langchain-ai/langchain) | Framework | Document loaders, text splitters, retrievers, memory |
| [LlamaIndex](https://github.com/run-llama/llama_index) | Framework | Indexing, retrieval, and context management |
| [Haystack](https://github.com/deepset-ai/haystack) | Framework | Pipelines for search and RAG |
| [Chroma](https://github.com/chroma-core/chroma) | Vector DB | Open-source embedding database |
| [Weaviate](https://github.com/weaviate/weaviate) | Vector DB | Vector search with hybrid capabilities |
| [Qdrant](https://github.com/qdrant/qdrant) | Vector DB | High-performance vector similarity search |
| [Pinecone](https://www.pinecone.io) | Vector DB | Managed vector database |
| [Milvus](https://github.com/milvus-io/milvus) | Vector DB | Scalable vector database |
| [Elasticsearch](https://github.com/elastic/elasticsearch) | Search | BM25 + vector search (since 8.0) |

### Embedding Models

| Model | Dimensions | Use Case |
|-------|-----------|----------|
| `text-embedding-3-small` (OpenAI) | 512-1536 | Production, high quality |
| `text-embedding-3-large` (OpenAI) | 256-3072 | Highest quality |
| `all-MiniLM-L6-v2` (sentence-transformers) | 384 | Fast, lightweight |
| `BAAI/bge-large-en-v1.5` (BGE) | 1024 | High quality, open |
| `intfloat/e5-mistral-7b-instruct` | 4096 | Very high quality, large |
| `Cohere Embed v3` | 1024 | Enterprise, multilingual |

### Reranking

| Tool | Description |
|------|-------------|
| [CrossEncoder](https://www.sbert.net/examples/applications/cross-encoder/README.html) (sentence-transformers) | Local cross-encoder rerankers |
| [Cohere Rerank](https://cohere.com/rerank) | API-based reranking |
| [RankLLM](https://github.com/castorini/rank_llm) | LLM-based listwise reranking |

### Chunking

| Tool | Description |
|------|-------------|
| [LangChain Text Splitters](https://python.langchain.com/docs/modules/data_connection/document_transformers/) | RecursiveCharacterTextSplitter, etc. |
| [LlamaIndex Node Parsers](https://docs.llamaindex.ai/en/stable/module_guides/loading/node_parsers/) | Semantic chunking, sentence splitting |
| [Unstructured.io](https://github.com/Unstructured-IO/unstructured) | Chunking for PDFs, docs, images |
| [tiktoken](https://github.com/openai/tiktoken) | Token counting for OpenAI models |

### Memory

| Tool | Description |
|------|-------------|
| [LangChain Memory](https://python.langchain.com/docs/modules/memory/) | ConversationBufferMemory, SummaryMemory, etc. |
| [Mem0](https://github.com/mem0ai/mem0) | Memory layer for LLM applications |
| [Zep](https://github.com/getzep/zep) | Long-term memory for AI assistants |

## Official Documentation

- [OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)
- [Anthropic Prompt Engineering Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/)
- [Google Gemini Context Caching](https://ai.google.dev/gemini-api/docs/caching)
- [LlamaIndex Context Management](https://docs.llamaindex.ai/en/stable/understanding/putting_it_all_together/q_and_a/)
- [LangChain Context Management Patterns](https://python.langchain.com/docs/use_cases/chatbots/memory/)

## Related Chapters in This Handbook

- **Chapter 01: Tokenization** — Understanding tokenizer behavior and token budgets.
- **Chapter 02: Prompt Engineering** — System prompt design and few-shot context usage.
- **Chapter 05: GraphRAG** — Graph-based retrieval for structured knowledge in context.

## Blog Posts & Articles

- [Building RAG-Based LLM Applications for Production (Anyscale)](https://www.anyscale.com/blog/building-rag-based-llm-applications-for-production)
- [Context Windows Are Not What You Think (Lilian Weng)](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/)
- [A Survey of Techniques for Maximizing LLM Context](https://blog.langchain.dev/survey-techniques-maximizing-llm-context/)
- [The RAG Triad: Evaluation for Retrieval-Augmented Generation](https://truera.com/ai-quality-education/generative-ai-rags/the-rag-triad/)
