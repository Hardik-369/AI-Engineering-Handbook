# Cheat Sheet: Prompt Caching Quick Reference

> One-page-per-topic quick reference for daily use.

---

## Caching Strategies Overview

| Strategy | Granularity | Match Method | Latency Hit | Storage | Best For |
|----------|-------------|-------------|-------------|---------|----------|
| KV Cache | Token-level | Exact prefix | <10ms | GPU memory | System prompts, context docs |
| Exact Cache | Full prompt | Exact string | <5ms | RAM/Redis | Identical queries, FAQs |
| Semantic Cache | Full prompt | Embedding similarity | 10-50ms | Vector DB | Similar intent, variations |
| Embedding Cache | Embedding vector | Vector identity | <5ms | RAM/Redis | RAG, classification |
| Response Cache | Full response | Key match | <5ms | RAM/Redis | Deterministic outputs |

---

## Provider KV Caching

| Provider | Activation | Min Tokens | TTL | Discount | Code Change |
|----------|-----------|------------|-----|----------|-------------|
| **Anthropic** | `cache_control: {"type": "ephemeral"}` | 1,024 (Sonnet/Haiku), 2,048 (Opus) | 5 min sliding | 50% read, +25% write | Required |
| **OpenAI** | Automatic (prefix detection) | 1,024 | 5-10 min | 50% on cached | None |
| **Gemini** | `CachedContent.create()` | 32,768 | Configurable (min to days) | ~50% | Required |

### Anthropic Example
```python
response = anthropic_client.messages.create(
    model="claude-3-5-sonnet-20241022",
    system=[{"type": "text", "text": sys_prompt,
             "cache_control": {"type": "ephemeral"}}],
    messages=[{"role": "user", "content": [
        {"type": "text", "text": document,
         "cache_control": {"type": "ephemeral"}},
        {"type": "text", "text": query}
    ]}]
)
```

### OpenAI Example
```python
response = openai_client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": long_system_prompt},  # cached prefix
        {"role": "user", "content": user_query}
    ]
)
cached = response.usage.prompt_tokens_details.cached_tokens
```

### Gemini Example
```python
cache = genai.caching.CachedContent.create(
    model='models/gemini-1.5-pro',
    system_instruction=long_prompt,
    ttl=datetime.timedelta(minutes=30)
)
model = genai.GenerativeModel.from_cached_content(cached_content=cache)
response = model.generate_content(user_query)
```

---

## Cache Key Construction

```python
# Deterministic cache key
import hashlib, json

def cache_key(model, messages, temperature=0, max_tokens=1024, **kwargs):
    canonical = {"model": model, "messages": messages,
                 "temperature": temperature, "max_tokens": max_tokens}
    canonical.update({k: v for k, v in kwargs.items() if v is not None})
    serialized = json.dumps(canonical, sort_keys=True)
    return hashlib.sha256(serialized.encode()).hexdigest()
```

---

## Input Normalization

```python
import re

def normalize(text: str) -> str:
    text = text.strip()
    text = re.sub(r'\s+', ' ', text)
    text = text.lower()
    text = text.rstrip('.,!?;:')
    return text
```

---

## Semantic Cache Threshold Guide

| Threshold | Behavior | Precision | Recall | Use Case |
|-----------|----------|-----------|--------|----------|
| 0.99+ | Near-exact | Very High | Very Low | Legal, medical, contracts |
| 0.95-0.99 | High precision | High | Low | Factual QA, code gen |
| 0.85-0.95 | Balanced | Medium | Medium | Customer support, general |
| 0.70-0.85 | High recall | Low | High | Creative, brainstorming |
| <0.70 | Too aggressive | Very Low | Very High | Not recommended |

---

## Cache Invalidation

```
TTL-based:     cache.setex(key, ttl, value)
Version-based: key = f"{VERSION}:{hash}"
Event-based:   redis.delete(pattern) on data change
LRU eviction:  cache maxmemory-policy allkeys-lru
```

---

## Cache Hit Ratio Calculation

```
Hit Ratio = Cache Hits / (Cache Hits + Cache Misses)

Example:
  300 hits + 200 misses = 500 total
  Hit ratio = 300/500 = 0.60 (60%)
```

### Typical Hit Ratios

| Strategy | Expected Hit Ratio |
|----------|-------------------|
| No caching | 0% |
| Exact cache | 5-30% |
| Semantic cache | 20-60% |
| Provider KV cache | 80-99% |
| Combined (layered) | 70-95% |

---

## Redis Commands for Caching

```bash
# Basic operations
SET key value
GET key
SETEX key seconds value   # Set with TTL
TTL key                   # Check remaining TTL
DEL key
EXISTS key
KEYS pattern              # Search keys (avoid in production)
SCAN cursor MATCH pattern # Production-safe key search
FLUSHDB                   # Clear all keys

# For semantic cache with RedisVL
FT.CREATE idx ON HASH PREFIX 1 cache: SCHEMA
  query_text TEXT
  response TEXT
  embedding VECTOR HNSW 6 TYPE FLOAT32 DIM 1536 DISTANCE_METRIC COSINE
FT.SEARCH idx "@query_text:(hello world)" RETURN 1 response
FT.SEARCH idx "*=>[KNN 1 @embedding $vec]" PARAMS 2 vec $vector RETURN 1 response
```

---

## Cache Entry Metadata

```python
cache_entry = {
    "response": "...",
    "model": "gpt-4o",
    "timestamp": 1712345678.0,
    "ttl": 3600,
    "hit_count": 42,
    "input_tokens": 1500,
    "output_tokens": 200,
    "latency_ms": 2345,
    "version": 3
}
```

---

## Cost Savings Formula

```
Savings = Total_Input_Tokens × Cacheable_Ratio × Hit_Ratio × 0.5 × Token_Price

Example:
  $120K annual spend
  70% cacheable input tokens
  60% hit ratio
  50% cache discount

  Annual savings = $120K × 0.7 × 0.6 × 0.5 = $25,200
```

---

## Latency Comparison

```
Cache Hit:        <5ms (L1)  → <50ms (L2) → <200ms (semantic)
Cache Miss (LLM): 1-10s (prefill + decode)
Improvement:      20-100x on cache hit
```

---

## Layered Cache Architecture

```
Request → L1 (In-Memory LRU) → L2 (Redis) → L3 (Provider KV) → L4 (Semantic) → LLM
            <1μs               <1ms          <10ms               <50ms           1-10s
```

---

## Cache Stampede Prevention

```python
# Request coalescing pattern
cache_key = build_key(...)

# Check cache
result = cache.get(cache_key)
if result:
    return result

# Acquire compute lock
lock_key = f"lock:{cache_key}"
if redis.setnx(lock_key, "1"):
    redis.expire(lock_key, 10)
    result = call_llm(...)
    cache.set(cache_key, result)
    redis.delete(lock_key)
    return result

# Wait for computing request
for _ in range(100):
    time.sleep(0.05)
    result = cache.get(cache_key)
    if result:
        return result
```

---

## Monitoring Metrics

| Metric | Good | Warning | Critical |
|--------|------|---------|----------|
| Hit ratio | >60% | 30-60% | <30% |
| L1 hit ratio | >30% | 10-30% | <10% |
| Cache P99 latency | <10ms | 10-50ms | >50ms |
| Eviction rate | <10/s | 10-100/s | >100/s |
| Cache memory | <60% | 60-80% | >80% |

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Cache key includes user ID | Hash user ID; don't use raw value |
| No TTL on cache entries | Always set TTL, even if long |
| Caching personalized responses | Include persona/segment in key |
| Not monitoring hit ratio | Add metrics on day one |
| Brute-force semantic search | Use HNSW index (RedisVL, Pinecone) |
| Cache stampede on TTL expiry | Use request coalescing + jitter |
| Cold start after deployment | Warm cache before going live |
