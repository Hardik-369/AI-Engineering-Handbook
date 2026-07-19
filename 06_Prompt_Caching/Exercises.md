# Exercises: Prompt Caching

> 15+ exercises ranging from beginner to advanced. Each exercise specifies the technique, expected outcome, and evaluation criteria.

---

## Beginner Exercises

### Exercise 1: Exact Cache Implementation
**Implement an in-memory exact cache** for LLM responses with TTL support.

**Requirements:**
- Cache key includes: model, messages, temperature
- TTL: 300 seconds
- Thread-safe access
- Hit/miss logging

**Starter code:**
```python
class ExactCache:
    def __init__(self, ttl_seconds=300):
        pass

    def get(self, model, messages, temperature=0):
        pass

    def set(self, model, messages, response, temperature=0):
        pass
```

**Evaluation:** Cache returns response on exact match, returns None on miss, expires entries after TTL.

---

### Exercise 2: Semantic Cache with Embeddings
**Build a semantic cache** using cosine similarity and an embedding model.

**Requirements:**
- Use `text-embedding-3-small` (or free alternative)
- Store (query, embedding, response) tuples
- Return cached response if similarity > 0.92
- Handle empty cache gracefully

**Evaluation:** "What's the weather?" and "How's the weather today?" return the same cached response.

---

### Exercise 3: Cache Hit Ratio Calculator
**Build a cache hit ratio monitor** that tracks hits, misses, and overall ratio.

**Requirements:**
- Track total_requests, cache_hits, cache_misses
- Calculate hit_ratio = hits / (hits + misses)
- Log every 100 requests with current ratio
- Export metrics to JSON

**Evaluation:** After 1000 requests with 30% hit rate, reports 0.30 ratio.

---

### Exercise 4: Redis Basic Cache
**Connect to Redis and implement** get/set/delete operations for LLM responses.

**Requirements:**
- Use `redis-py` library
- TTL: 600 seconds
- JSON serialize cache values with metadata
- Handle connection errors gracefully with fallback

**Evaluation:** Successfully stores and retrieves responses; falls back to direct LLM call if Redis is unavailable.

---

### Exercise 5: Cache Key Normalization
**Implement input normalization** to improve exact cache hit rate.

**Requirements:**
- Trim leading/trailing whitespace
- Normalize multiple spaces to single space
- Lowercase the input
- Remove trailing punctuation
- Compare normalized strings

**Evaluation:** "Hello! " and "hello" produce the same cache key.

---

## Intermediate Exercises

### Exercise 6: Anthropic Prompt Caching
**Implement Anthropic's prompt caching** with `cache_control` for a support chatbot.

**Requirements:**
- Cache the system prompt (1000+ tokens) using `cache_control`
- Each user query is appended as a new message
- Log cache creation and cache read tokens
- Compare response time and cost with/without caching

**Evaluation:** Token usage shows `cache_read_input_tokens > 0` on subsequent requests.

---

### Exercise 7: OpenAI Prompt Caching
**Implement OpenAI's automatic prompt caching** and verify it's working.

**Requirements:**
- Use GPT-4o with a 2000+ token system prompt
- Send 5 requests with different user messages
- Parse `usage.prompt_tokens_details.cached_tokens` from response
- Calculate effective cost savings

**Evaluation:** At least 3 of 5 requests show cached tokens > 0.

---

### Exercise 8: Gemini Context Caching
**Implement Gemini's explicit context caching** with configurable TTL.

**Requirements:**
- Use `genai.caching.CachedContent` to create a cache
- Set TTL to 30 minutes
- Generate content from cached context
- Measure latency improvement vs non-cached requests

**Evaluation:** Cached requests complete in <50% of the time of non-cached requests.

---

### Exercise 9: Semantic Cache with RedisVL
**Implement semantic caching using RedisVL** instead of brute-force search.

**Requirements:**
- Install and start Redis (Docker or local)
- Install `redisvl` library
- Create an index with HNSW vector search
- Store 100+ queries with embeddings
- Query with sub-10ms search time

**Evaluation:** Returns cached response in <50ms total for queries with similarity > threshold.

---

### Exercise 10: Cache Invalidation Strategies
**Implement three invalidation strategies** and compare their behavior.

**Requirements:**
- TTL-based: automatic expiry after N seconds
- Version-based: global version counter in cache key
- Event-based: explicit invalidation endpoint
- Log each invalidation event
- Test with 10 queries before and after invalidation

**Evaluation:** After invalidation, stale responses are not returned. Each strategy has different trade-offs you can demonstrate.

---

### Exercise 11: Cache Warm-Up
**Implement a cache warm-up system** that pre-populates the cache before user traffic arrives.

**Requirements:**
- Load queries from historical logs
- Generate responses for top-100 queries
- Store in cache with longer TTL
- Log warming progress
- Compare hit ratio with/without warm-up

**Evaluation:** Hit ratio is 20+ percentage points higher with warm-up during the first hour.

---

### Exercise 12: Cache Hit Ratio Optimization
**Given a dataset of 10,000 queries**, optimize the cache configuration for maximum hit ratio.

**Requirements:**
- Experiment with different similarity thresholds (0.80 to 0.99)
- Measure precision, recall, and hit ratio at each threshold
- Find the optimal threshold for accuracy vs hit rate
- Report findings with a chart

**Evaluation:** Student identifies the best threshold for their specific accuracy requirements.

---

### Exercise 13: Multi-Layer Cache
**Build a layered cache** with L1 (memory), L2 (Redis), and L3 (LLM).

**Requirements:**
- L1: Python dict with LRU eviction (max 1000 entries)
- L2: Redis with TTL
- L3: LLM API call
- Return from L1 on hit; promote L2 hit to L1
- Log which layer served each request

**Evaluation:** Hot queries served from L1 (<1ms); warm queries from L2 (<10ms); cold queries from LLM (2-10s).

---

### Exercise 14: Cost Analysis Dashboard
**Build a dashboard** that compares API costs with and without caching.

**Requirements:**
- Track: total tokens, cached tokens, cost without cache, cost with cache
- Calculate savings percentage
- Show daily/weekly/monthly trends
- Display current cache hit ratio
- Export report as CSV

**Evaluation:** Dashboard shows at least 40% cost savings with realistic usage patterns.

---

## Advanced Exercises

### Exercise 15: Request Coalescing (Cache Stampede Prevention)
**Implement request coalescing** to prevent the thundering herd problem.

**Requirements:**
- If 100 requests arrive simultaneously for the same uncached key, only 1 calls the LLM
- Other 99 wait for the first to complete and then get the cached result
- Use `asyncio.Lock` or `threading.Lock`
- Measure time savings vs 100 parallel LLM calls
- Handle error propagation (if LLM call fails, all waiters get the error)

**Evaluation:** With 50 concurrent requests for the same uncached prompt, only 1 LLM call is made.

---

### Exercise 16: Semantic Cache with Negative Lookups
**Extend the semantic cache** to cache "no match" results to avoid redundant embedding comparisons.

**Requirements:**
- Track which queries have no semantic match
- Cache the negative result with a shorter TTL
- Skip embedding lookup for known mismatches
- Measure reduction in embedding API calls

**Evaluation:** Recurring unique queries don't repeatedly hit the embedding API.

---

### Exercise 17: Context-Aware Cache Eviction
**Implement an ML-powered cache eviction policy** that predicts which entries will be accessed again.

**Requirements:**
- Track access patterns: frequency, recency, time-of-day patterns
- Score each cache entry for future access probability
- Evict lowest-scored entries when cache is full
- Compare hit ratio vs LRU eviction

**Evaluation:** ML-based eviction beats LRU by 10+ percentage points on hit ratio.

---

### Exercise 18: Distributed Cache Synchronization
**Build a distributed cache** that stays consistent across multiple application instances.

**Requirements:**
- Use Redis pub/sub for invalidation messages
- When one instance invalidates a key, all instances know
- Handle network partitions gracefully
- Measure eventual consistency timing

**Evaluation:** After invalidation on node A, node B stops serving stale data within 500ms.
