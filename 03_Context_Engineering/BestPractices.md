# Best Practices: Production Context Management

## 1. Token Budget Allocation

### The 50/30/20 Rule (for RAG Systems)

| Category | Allocation | Notes |
|----------|-----------|-------|
| System + Instructions | 20% | Fixed, rarely changes |
| Retrieved Context | 30% | Most important for factual accuracy |
| Conversation History | 50% | Includes memory summaries |

**Adjust based on use case:**
- **Chatbots:** Increase history to 70%, decrease retrieval to 20%.
- **QA systems:** Increase retrieval to 50%, decrease history to 30%.
- **Coding assistants:** Increase system/tools to 30%, balance retrieval and history.

### Always Reserve for Output

Never fill the entire context window with input. Reserve 5-10% of the budget for the model's response. Models that run out of output budget produce truncated responses.

```python
MAX_INPUT_TOKENS = int(MODEL_LIMIT * 0.90)  # 10% reserved for output
```

## 2. Context Prioritization

### Information Value Hierarchy

Rank context elements by value:

1. **System prompt** (highest value) — Always include, never evict.
2. **Current user query** — Always include.
3. **Retrieved documents answering the query** — Include if budget allows.
4. **Recent conversation history** — Last 3-5 turns verbatim.
5. **Memory summaries** — Compressed past.
6. **Older conversation history** — Lowest value, first to evict.

### The Prime Slots

The first 10% and last 10% of context are the most valuable positions. Place your most critical information there.

```
[CRITICAL] ... [moderate] ... [moderate] ... [CRITICAL]
   Start                                          End
   (Best)                                     (Second Best)
```

## 3. Memory Refresh Strategies

### When to Summarize

- **Turn-based:** After every N turns (N=5-10 depending on complexity).
- **Token-based:** When history reaches 60% of budget, trigger summarization.
- **Topic-based:** When the conversation shifts topic, summarize and archive the previous topic.

### How to Structure Summaries

```
## Prior Conversation Summary
- User is building an AI handbook
- They prefer Python examples
- They work at a tech company (details unclear)

## Key Technical Decisions
- Using GPT-4o for main model
- Using all-MiniLM-L6-v2 for embeddings
- Chunk size: 512 tokens with 10% overlap
```

### Refresh Cadence

```
Turn 1-5: Full verbatim history
Turn 6:   Summarize turns 1-5, keep turns 6-10 verbatim
Turn 11:  Update summary (turns 1-10), keep turns 11-15 verbatim
...
```

## 4. Chunk Size Selection Guide

### Decision Matrix

| Use Case | Chunk Size | Overlap | Strategy |
|----------|-----------|---------|----------|
| FAQ / Short answers | 128-256 tokens | 10% | Semantic |
| Document QA | 512 tokens | 10% | Recursive |
| Code retrieval | 200-400 tokens | 15% | Line-based |
| Scientific papers | 512-1024 tokens | 10% | Section-based |
| Long-form analysis | 1024+ tokens | 5% | Semantic |

### Chunk Size Heuristic

```
Chunk Size = 2 × Expected Answer Length × (Number of Supporting Facts)
```

If answers are typically 50 tokens and need 3 supporting facts: chunk = 2 × 50 × 3 = 300 tokens.

## 5. Retrieval Best Practices

### The 3-Stage Pipeline

```
Stage 1: Broad retrieval (BM25 or dense) → Top 50-100
Stage 2: Reranking (cross-encoder) → Top 5-10
Stage 3: Context packing → Final selection
```

### Reranking Thresholds

- **Precision-critical applications:** Use a high threshold (score > 0.8).
- **Recall-critical applications:** Accept lower scores (score > 0.3).
- **Default:** Keep top 5-10 regardless of score.

### Retrieval Guardrails

1. **Max documents per query** — Never exceed 15 documents in context.
2. **Min relevance threshold** — Skip docs below score threshold.
3. **Deduplication** — Remove near-duplicate chunks (cosine similarity > 0.95).
4. **Source diversity** — Prefer documents from different sources over multiple chunks from the same source.

## 6. System Prompt Design for Context

### Keep It Concise

Every token in your system prompt is a token taken from retrieval or history.

```
❌ Bad: "You are an AI assistant created by [company] to help users with [domain]. 
Your purpose is to provide accurate, helpful, and safe responses. You should 
be polite, professional, and concise. When you don't know something, say so..."

✅ Good: "You are a [domain] assistant. Be concise and accurate. Cite sources."
```

### Embed Constraints, Not Personality

Put behavioral constraints in the system prompt. Put personality in a separate, lower-priority section.

## 7. Monitoring & Observability

### Metrics to Track

| Metric | What It Measures | Alert Threshold |
|--------|-----------------|-----------------|
| Context utilization % | How full is the window | >95% |
| Retrieval precision@5 | How many retrieved docs are used | <0.6 |
| Memory hit rate | How often memory recall is useful | <0.3 |
| Token efficiency | Tokens per successful response | >5000/turn |
| Eviction rate per turn | How often we remove content | >10/turn |
| Compression ratio | Tokens in vs tokens out after compression | <2x |

### Debugging Context Issues

**Symptom: Model ignores a document**
→ Check document position (is it in the middle?)
→ Check document length (too long?)
→ Check formatting (structured properly?)

**Symptom: Model repeats information**
→ Check for duplicate content in context
→ Check for redundancy between memory and retrieved docs

**Symptom: Model seems to forget earlier conversation**
→ Check sliding window eviction
→ Check if memory summarization is losing key facts

## 8. Anti-Patterns to Avoid

### The Kitchen Sink

> "My context window is 200K tokens, so I'll put everything in."

**Why it fails:** Lost in the middle, attention dilution, increased cost and latency.

**Better:** Be selective. 5 well-chosen documents beat 20 noisy ones.

### Memory as a Dumpster

> "I'll store every conversation forever and retrieve everything."

**Why it fails:** Retrieval quality degrades as memory grows. Cost explodes.

**Better:** Summarize aggressively. Evict aggressively. Store only what matters.

### One-Size-Fits-All

> "I'll use the same context strategy for every query."

**Why it fails:** Different queries need different context compositions.

**Better:** Classify queries and adjust budget allocation dynamically.

### Ignoring the Middle

> "I'll just concatenate everything in order of retrieval."

**Why it fails:** Any document that ends up in the middle is effectively invisible.

**Better:** Place critical information at the start and end. Structure with clear labels.

## 9. Performance Optimization

### Caching

- **System prompts:** Cache the tokenized version. They never change per session.
- **Retrieved documents:** Cache embeddings. Don't re-embed on every query.
- **Memory summaries:** Cache until new information arrives.

### Parallel Operations

```
Query arrives
  ├── Embed query (parallel)
  ├── Retrieve BM25 (parallel)
  └── Load memory (parallel)
      ↓
  Wait for all → Rerank → Pack → Send to LLM
```

### Batching

When possible, batch queries to the cross-encoder reranker. Cross-encoders are most efficient when processing multiple (query, document) pairs in a single forward pass.

## 10. Testing & Validation

### Context Quality Tests

```
Test 1: Critical information at all positions
  → Place answer at position 1, 5, 10, 15, 20
  → Verify model answers correctly at all positions

Test 2: Context overload
  → Fill 90% of context with irrelevant information
  → Place critical information at different positions
  → Measure recall

Test 3: Long conversation
  → Run 50-turn conversation
  → Check if model remembers facts from turn 1

Test 4: Compression fidelity
  → Compress a document, then ask questions about compressed version
  → Measure information loss vs compression ratio
```

### Regression Test Suite

Build a set of 50-100 test cases that cover:
- Factual recall from single document
- Factual recall from multiple documents
- Following instructions at different positions
- Consistent personality across turns
- Error handling when context is insufficient
