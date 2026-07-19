# Cheat Sheet — Chapter 08: AI System Design

## Architecture Patterns

| Pattern | Mental Model | Best For | Key Trade-off |
|---------|-------------|----------|---------------|
| Orchestrator | One brain, many hands | Complex reasoning | Cost, latency |
| Pipeline | Assembly line | Fixed workflows | Rigidity |
| Router | Switchboard | Heterogeneous requests | Classification accuracy |
| Ensemble | Wisdom of crowds | Reliability | 3-5x cost |

---

## System Archetypes

| System | Core Components | Primary LLM Role | Key Challenge |
|--------|----------------|------------------|---------------|
| Chatbot | Context mgmt, memory, tools, streaming | Conversation | Context budgeting |
| Research Assistant | Search, retrieval, synthesis, citations | Grounded generation | Source credibility |
| Coding Agent | Repo context, code gen, test, debug | Multi-step reasoning | Code safety |
| Customer Support | Classification, KB retrieval, escalation | Response generation | Hallucination |
| Document Q&A | PDF parsing, chunking, indexing, retrieval | Answer synthesis | Table/chart extraction |
| NL-to-SQL | Schema understanding, SQL gen, viz | Query generation | Safety |
| Content Gen | Outline, draft, edit, review | Creative generation | Brand consistency |
| Voice AI | STT, VAD, TTS, turn mgmt | Real-time response | Latency |

---

## Context Window Budgeting

```
128K Context
├── System prompt:        500   (0.4%)
├── Conversation:       30,000  (23%)
├── Retrieved docs:     20,000  (16%)
├── Tool definitions:    2,000  (1.6%)
├── User input:          2,000  (1.6%)
├── Output buffer:       2,500  (2.0%)
└── Safety margin:      71,000  (55%)

Pruning order when over budget:
1. Summarize oldest conversation turns
2. Reduce retrieved document count
3. Truncate document content
4. Drop oldest messages
```

---

## Latency Budgets

| Component | Target | Alert | Degradation Action |
|-----------|--------|-------|--------------------|
| Voice AI (end-to-end) | < 1.2s | > 2s | Skip LLM, use canned response |
| Chat (first token) | < 500ms | > 2s | Show typing indicator |
| Chat (full response) | < 3s | > 8s | Stream aggressively |
| Document Q&A | < 5s | > 15s | Return partial results |
| SQL query + viz | < 5s | > 15s | Show table instead of chart |
| Content generation | < 10s | > 30s | Return draft, continue in background |

---

## Error Handling Reference

```python
# Quick retry with exponential backoff
import time, random

def retry(fn, max_retries=3, base_delay=1):
    for attempt in range(max_retries):
        try:
            return fn()
        except Exception:
            if attempt == max_retries - 1:
                raise
            delay = base_delay * (2 ** attempt) + random.uniform(0, 1)
            time.sleep(delay)
```

| Error | HTTP | Retry? | Action |
|-------|------|--------|--------|
| Rate Limit | 429 | Yes, backoff | Queue or downgrade model |
| Server Error | 5xx | Yes, 3x | Switch provider |
| Timeout | N/A | Yes, 2x | Reduce max_tokens |
| Invalid JSON | N/A | Yes, re-prompt | Add format examples |
| Content Filter | N/A | Once | Rephrase input |
| Context Overflow | N/A | Yes, truncate | Prune and retry |

---

## Caching Strategies

| Cache Type | Granularity | Hit Rate | Implementation |
|------------|-------------|----------|---------------|
| Exact match (T=0) | Identical prompt | ~20% | Redis, TTL=1h |
| Semantic (T=0) | Similar prompt | ~15% | Vector DB, threshold=0.95 |
| Prefix caching | Shared prompt start | ~30% | Provider built-in |
| Response template | Common patterns | ~40% | Template engine |
| Tool results | Same tool, same args | ~50% | Redis, TTL=5min |

---

## Cost Estimation

```
Cost = (input_tokens × input_price + output_tokens × output_price) / 1_000_000

Example: 2K input + 500 output
Model      | Input$/M | Output$/M | Cost/req | 100K req/mo
-----------+----------+-----------+----------+------------
GPT-4o-mini| $0.15    | $0.60     | $0.0006  | $60
GPT-4o     | $2.50    | $10.00    | $0.01    | $1,000
Claude Sonnet| $3.00  | $15.00    | $0.0135  | $1,350
DeepSeek V3| $0.50    | $2.00     | $0.002   | $200
```

Additional costs: Embedding ($0.02/1M tokens), Vector DB ($0.10/100K queries), Reranking ($0.01/query)

---

## Monitoring Metrics

| Metric | Target | Alert |
|--------|--------|-------|
| P50 latency | < 1s | > 2s |
| P95 latency | < 3s | > 5s |
| Error rate | < 0.5% | > 2% |
| Cost per request | < $0.01 | > $0.05 |
| Hallucination rate | < 1% | > 5% |
| Context utilization | < 80% | > 95% |
| Cache hit rate | > 30% | < 10% |
| User satisfaction | > 90% | < 70% |
| Escalation rate | < 10% | > 30% |

---

## Common Failure Modes & Fixes

| Failure | Symptom | Fix |
|---------|---------|-----|
| Hallucination cascade | Wrong answers compounding | Fact-checker LLM, citation confidence |
| Context overflow | Model ignores instructions | Summarize history, prune early |
| Tool errors | Agent loops | Fallback tools, max retry limit |
| Cost spike | Unexpected bill | Budget guard, model routing |
| Latency degradation | Slow responses | Smaller model, streaming, caching |
| Rate limiting | 429 errors | Queue-based processing, backoff |
| Prompt injection | Ignored instructions | Input sanitization, output validation |
| Model drift | Degrading quality over time | Continuous eval, golden dataset |

---

## Quick Reference: Key Formulas

```
Cost/request = (input_tokens * input_price + output_tokens * output_price) / 1_000_000

Monthly cost = daily_queries * cost_per_request * 30

Cache savings = (miss_rate * cost_per_request * queries) - cache_operating_cost

Latency budget = Σ(component_latency) + network_latency + buffer

Context utilization = total_tokens / max_context_window * 100

Hallucination rate = hallucinated_claims / total_claims * 100

System reliability = uptime / (uptime + downtime) * 100
```

---

## API Quick Patterns

### OpenAI Streaming
```python
stream = client.chat.completions.create(..., stream=True)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### Tool Calling
```python
resp = client.chat.completions.create(
    model="gpt-4o",
    messages=messages,
    tools=[{"type": "function", "function": {...}}],
    tool_choice="auto",
)
```

### Structured Output
```python
resp = client.beta.chat.completions.parse(
    model="gpt-4o",
    messages=messages,
    response_format=MySchema,
)
```

### Async with Backpressure
```python
async with asyncio.Semaphore(10):
    result = await client.chat.completions.create(...)
```
