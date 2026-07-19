# Cheat Sheet: Context Engineering

## Context Window Limits by Model

| Model | Limit | Effective Range | Tokenizer |
|-------|-------|----------------|-----------|
| GPT-4o / GPT-4 Turbo | 128K | <64K | cl100k_base / o200k_base |
| GPT-3.5 Turbo | 16K | <8K | cl100k_base |
| Claude 3 Haiku | 200K | <100K | Anthropic |
| Claude 3.5 Sonnet | 200K | <100K | Anthropic |
| Claude 3 Opus | 200K | <100K | Anthropic |
| Gemini 1.5 Pro | 2M | <500K | SentencePiece |
| Gemini 1.5 Flash | 1M | <250K | SentencePiece |
| Llama 3 / 3.1 70B | 8K / 128K | <32K | tiktoken (cl100k) |
| Llama 3.1 405B | 128K | <32K | tiktoken (cl100k) |
| Mistral Large | 128K | <32K | SentencePiece |
| Qwen 2.5 72B | 128K | <32K | tiktoken |
| DeepSeek-V2 | 128K | <32K | tokenizers |

## Chunking Parameters

| Strategy | Method | Best For | Tradeoff |
|----------|--------|----------|----------|
| Fixed-size | Split by character/token count | Simple documents, testing | Splits sentences |
| Recursive | Try separators in order (\n\n, \n, ., ...) | Markdown, code, structured text | Slightly slower |
| Semantic (embedding) | Split where cosine sim drops | Coherent paragraphs | Requires model, slow |
| Semantic (model) | LLM identifies section boundaries | High-quality chunks | Expensive, slow |

### Recommended Chunk Sizes

| Type | Size | Overlap |
|------|------|---------|
| FAQ / Short QA | 128-256 tokens | 10% |
| General RAG | 512 tokens | 10% |
| Document analysis | 1024 tokens | 5% |
| Code | 200-400 lines | 2-3 lines |
| Scientific papers | 512-1024 tokens | 10% |

### Overlap Formula

```
overlap_tokens = chunk_size × overlap_pct
start_position = i × (chunk_size - overlap_tokens)
```

## Retrieval Methods Comparison

| Method | Speed | Semantic Understanding | Exact Match | Storage | Best For |
|--------|-------|----------------------|-------------|---------|----------|
| BM25 | ⚡⚡⚡ | ❌ | ✅✅✅ | None | Keyword search, low-resource |
| Dense (embeddings) | ⚡⚡ | ✅✅✅ | ❌ | Vectors + model | Semantic search |
| Hybrid (BM25 + Dense) | ⚡ | ✅✅ | ✅✅ | Vectors + index | Production RAG |
| Graph-based (see Ch. 5) | ⚡ | ✅✅✅ | ✅ | Graph + vectors | Structured knowledge |

### Retrieval Pipeline

```
Query → BM25/Dense (Top 50-100) → Cross-encoder Reranker (Top 5-10) → Context
```

### Reranking Models

| Model | Size | Speed | Quality |
|-------|------|-------|---------|
| cross-encoder/ms-marco-MiniLM-L-2-v2 | 22M | ⚡⚡⚡ | ⚡⚡ |
| cross-encoder/ms-marco-MiniLM-L-6-v2 | 80M | ⚡⚡ | ⚡⚡⚡ |
| cross-encoder/ms-marco-electra-base | 110M | ⚡ | ⚡⚡⚡⚡ |
| Cohere Rerank v3 | API | ⚡ | ⚡⚡⚡⚡⚡ |

## Memory Types

| Type | Scope | Storage | Retrieval | Eviction |
|------|-------|---------|-----------|----------|
| Working | Current turn | Raw tokens | Immediate | Per turn |
| Episodic | Past sessions | Summaries, key points | BM25/dense over summaries | FIFO or importance |
| Semantic | Persistent knowledge | Embeddings, graphs | Similarity search | Never (append only) |
| Procedural | How to do things | Tool definitions, code | Exact match | Never |

### Memory Budget Allocation

```
Conversation Turn Budget:
  Recent 3-5 turns:  30% (verbatim)
  Summarized history: 20% (compressed)
  Retrieved memories: 20% (query-dependent)
  System/tools:       20% (fixed)
  Scratchpad:         10% (agent reasoning)
```

## Lost in the Middle: Key Facts

- **Published:** Liu et al., 2023
- **Finding:** Accuracy on information in the middle of context is 20-50% worse than at edges
- **Effect persists across:** Model size, model family, number of documents
- **Gets worse with:** More documents, longer contexts, dense packing

### Mitigations

| Strategy | Improvement | Cost |
|----------|------------|------|
| Position critical info at start/end | High | Free |
| XML/JSON structured format | Medium | Free |
| Explicit document labeling | Low | Free |
| "Search carefully" instruction | Low | <10 tokens |
| Rerank to prioritize top docs | High | Reranker compute |

## Token Counting Reference

### Tokens per Word (approximate)

| Language | Tokens per Word |
|----------|----------------|
| English | 1.3 - 1.5 |
| Code | 1.1 - 1.3 |
| Special characters | 1.0 - 2.0 |
| Chinese | 1.5 - 3.0 |

### Quick Estimation

```
# characters ÷ 4 ≈ tokens
# words × 1.3 ≈ tokens
```

### Python: Token Counting

```python
import tiktoken
enc = tiktoken.encoding_for_model("gpt-4o")
tokens = enc.encode("Your text here")
count = len(tokens)
```

## Context Budget Template

```
MODEL_LIMIT:        128,000  (100%)
Output reserve:     -4,000   (3.1%)
                     ─────
Input available:    124,000  (96.9%)

  System prompt:       500   (0.4%)
  Tool defs:         1,000   (0.8%)
  Retrieved docs:    5,000   (3.9%)
  Memory summary:      500   (0.4%)
  History:         117,000  (91.3%)
                     ─────
  Total used:      124,000  (96.9%)
  Remaining:             0   (0.0%)
```

## Sliding Window Policies

| Policy | Behavior | Use Case |
|--------|----------|----------|
| FIFO | Evict oldest first | Simple chat |
| Prioritized | Keep system/tools, evict rest | Production |
| Summary-buffer | Keep last N verbatim, summarize rest | Long sessions |
| Importance-scored | Evict lowest score | Complex reasoning |
| Recency-weighted | Weight by inverse age | Conversation |

## Compression Techniques

| Technique | Ratio | Quality Loss | Speed |
|-----------|-------|-------------|-------|
| Extractive (top-k sentences) | 2-3x | Low | ⚡⚡⚡ |
| Abstractive summarization | 5-10x | Medium | ⚡ |
| LLMLingua | 2-5x | Low-Medium | ⚡⚡ |
| Selective context | 1.5-3x | Low | ⚡⚡ |
| AutoCompressor | 4-8x | Medium | ⚡ |

## Common Pitfalls Checklist

- [ ] Are you filling the entire context window? (Don't. Leave room.)
- [ ] Is your most important document in the middle? (Move it.)
- [ ] Are you retrieving more documents than fit? (Rerank harder.)
- [ ] Is your memory growing unbounded? (Summarize and evict.)
- [ ] Are you using the same strategy for every query? (Adapt.)
- [ ] Are you measuring context utilization? (You should.)
- [ ] Is your system prompt efficient? (Every token counts.)
- [ ] Are you reserving output tokens? (Models need room to answer.)
