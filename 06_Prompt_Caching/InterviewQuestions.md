# Interview Questions: Prompt Caching

> 20+ questions covering fundamental concepts, practical implementations, and advanced caching strategies. Questions are grouped by difficulty with answer guidance.

---

## Beginner Questions

### Q1: What is prompt caching, and why is it important?

**Expected answer:** Prompt caching stores the results of LLM computations (KV cache entries, embeddings, or full responses) so they can be reused for future requests with the same or similar prompts. It's important because it reduces API costs (up to 50-80%), lowers latency (from seconds to milliseconds on cache hits), and increases system throughput without additional API capacity.

**Key points:** Avoids redundant computation, reduces costs and latency.

---

### Q2: Explain the difference between exact cache and semantic cache.

**Expected answer:** Exact cache matches prompts character-for-character and returns the cached response only for an identical match. Semantic cache uses embeddings to measure semantic similarity between prompts, returning cached responses for prompts that mean the same thing even with different wording.

**Example:** "What's the capital of France?" and "Capital of France?" — exact cache misses, semantic cache hits.

---

### Q3: What is a KV cache in LLM inference?

**Expected answer:** The KV cache stores the Key and Value vectors computed during the attention mechanism. During the prefill phase, the model computes K and V for every input token. During the decode phase, only the new token's K and V are computed, and the previously stored ones are reused. This avoids O(n²) recomputation of attention for every generated token.

**Key insight:** Without KV cache, generating N tokens would require O(N²) attention computations. With KV cache, it's O(N²) prefill + O(N) per new token.

---

### Q4: How does provider prompt caching (Anthropic/OpenAI) reduce costs?

**Expected answer:** Providers detect repeated prompt prefixes (system prompts, context documents) and store the KV cache for those tokens. When the same prefix appears in a future request, cached tokens are billed at 50% discount. This means the first request pays full price, but subsequent requests with the same prefix pay ~50% less for input tokens.

---

### Q5: What is a cache hit ratio, and how do you calculate it?

**Expected answer:** Cache hit ratio = Cache Hits / (Cache Hits + Cache Misses). It measures the proportion of requests served from cache vs. going to the LLM. A ratio of 0.6 means 60% of requests are cache hits. Higher is better — lower cost and latency.

**Example:** 300 hits + 200 misses = 500 total requests. Hit ratio = 300/500 = 0.6 (60%).

---

### Q6: What is TTL in caching?

**Expected answer:** TTL (Time-To-Live) is the duration a cache entry is considered valid. After the TTL expires, the entry is evicted, and the next request must call the LLM again. TTL balances freshness (short TTL) against cache effectiveness (long TTL). Typical values: 5 minutes for dynamic data, 24 hours for static data.

---

### Q7: What is a cache stampede (thundering herd)?

**Expected answer:** A cache stampede occurs when many concurrent requests miss the cache simultaneously (e.g., after cache invalidation or TTL expiry). All requests hit the LLM at once, causing a spike in API calls, potential rate limiting, and increased latency. Solutions include request coalescing (only one request calls the LLM, others wait) and early recomputation (refresh cache before TTL expires).

---

### Q8: What data would you include in a cache key for response caching?

**Expected answer:** The cache key should include everything that affects the LLM's output: model name, messages array (or system + user prompts), temperature, max_tokens, stop sequences, seed, and any other parameters. It should exclude things that don't affect output: user ID, request ID, metadata.

**Example key:** sha256(json({model, messages, temperature, max_tokens}))

---

## Intermediate Questions

### Q9: Design a semantic cache for a customer support chatbot.

**Expected answer:**
1. **Embedding model:** `text-embedding-3-small` for query embeddings.
2. **Vector database:** RedisVL for sub-10ms similarity search.
3. **Threshold:** 0.92 (start high, tune based on accuracy needs).
4. **TTL:** 24 hours (support answers don't change frequently).
5. **Cache key:** Normalized query text + embedding vector.
6. **Fallback:** On cache miss, call LLM, store response in cache.
7. **Invalidation:** Version-based when support docs are updated.
8. **Monitoring:** Hit ratio dashboard, similarity score distribution.

**Edge cases:** Multi-turn conversations (cache per turn vs. per session), personalization (include user context in key), PII in queries (hash or exclude).

---

### Q10: Compare Redis and in-memory caching for LLM applications.

| Aspect | In-Memory (dict/LRU) | Redis |
|--------|-------------------|-------|
| Latency | <1μs | <1ms |
| Capacity | RAM limited (GB) | RAM + disk (TB) |
| Distribution | Single process | Multi-node cluster |
| Persistence | None | Optional (RDB/AOF) |
| Data structures | Dict-like | Strings, hashes, vectors, sorted sets |
| TTL | Manual | Built-in key expiry |
| Vector search | Brute force | HNSW (via RedisVL) |
| Best for | L1 cache, single server | L2 cache, distributed systems |

---

### Q11: How would you implement cache invalidation for an LLM-powered code generator?

**Expected answer:**
1. **TTL-based:** Short TTL (5-10 min) for development, longer for stable APIs.
2. **Version-based:** Increment a CACHE_VERSION whenever the library/framework versions change.
3. **Event-based:** Hook into CI/CD pipeline — when code is pushed, invalidate cache entries related to changed modules.
4. **Selective invalidation:** Tag cache entries by module name; invalidate only affected tags.
5. **Warm-up:** After invalidation, regenerate cache for the most common code generation patterns.

---

### Q12: What are the trade-offs of aggressive caching in LLM applications?

**Trade-offs:**
- **Freshness vs. hit rate:** Longer TTL increases hit rate but risks stale responses.
- **Precision vs. recall (semantic):** Lower threshold increases hit rate but may return incorrect responses for dissimilar queries.
- **Memory vs. coverage:** Larger cache improves hit rate but consumes more memory.
- **Complexity vs. return:** Multi-layer caching adds complexity; ensure ROI justifies the engineering cost.
- **Cold start:** Cache is empty after deployment; requires warm-up strategy.

---

### Q13: How would you measure the ROI of implementing a caching layer?

**Expected answer:**
1. **Current cost:** Track API spend for 1 month (total tokens × token price).
2. **Cacheable portion:** Estimate percentage of input tokens that are repeated (system prompts, context docs).
3. **Expected hit rate:** Based on query diversity and caching strategy.
4. **Cost savings:** Current cost × cacheable portion × hit rate × 50% discount.
5. **Infrastructure cost:** Redis cluster + embedding API + engineering time.
6. **ROI:** (Annual savings - infrastructure cost) / infrastructure cost.

**Example:** $120K annual spend, 60% cacheable, 70% hit rate → $25K savings vs $5K infra cost → 400% ROI.

---

### Q14: Explain how Anthropic's cache_control parameter works.

**Expected answer:** `cache_control: {"type": "ephemeral"}` marks a content block (system prompt, document, or message) as cacheable. When the same block is sent again in a different request, Anthropic returns cached KV values instead of recomputing. The cache lasts for 5 minutes of inactivity (sliding window). The API response includes `cache_creation_input_tokens` (first write) and `cache_read_input_tokens` (subsequent reads). Pricing: write is 25% more expensive per token, read is 50% cheaper.

---

### Q15: How does OpenAI's automatic prompt caching differ from Anthropic's explicit caching?

**Expected answer:** OpenAI caches automatically without any code changes — it detects repeated prompt prefixes and applies caching transparently. Anthropic requires explicit `cache_control` markers on content blocks. OpenAI's cache TTL is 5-10 minutes of inactivity (not documented precisely). Anthropic's is 5 minutes (sliding). Both offer roughly 50% discount on cached tokens. OpenAI's approach is simpler (no code changes) but offers less control. Anthropic's approach requires code changes but allows fine-grained control over what gets cached.

---

### Q16: What is cache warming, and when is it necessary?

**Expected answer:** Cache warming pre-populates the cache with known queries before handling real user traffic. It's necessary after:
- New deployment (cache starts empty — cold start).
- Major cache invalidation event.
- Launching a new feature with predictable query patterns.

**How:** Load historical query logs, find top-K most frequent queries, pre-generate responses, and store in cache with longer initial TTL.

---

## Advanced Questions

### Q17: Design a distributed caching system for an LLM application serving 10M requests/day across 3 data centers.

**Expected answer:**
1. **Architecture:**
   - L1: Node-local LRU cache (10K entries, <1μs)
   - L2: Redis Cluster (3 data centers, cross-region replication)
   - L3: Semantic cache (RedisVL, HNSW index)
   - L4: Provider KV cache

2. **Consistency:**
   - Write-through for cache updates (write to L2, then L1)
   - Redis pub/sub for invalidation across nodes
   - CRDT-based conflict resolution for concurrent writes

3. **Data partitioning:**
   - Cache key hash → Redis hash slot
   - Consistent hashing for node distribution

4. **Resilience:**
   - Redis Sentinel for automatic failover
   - Circuit breaker: if Redis is down, fall back to L1 only, then L3
   - Rate limiting for LLM API calls on mass cache miss

5. **Monitoring:**
   - Per-layer hit ratio, latency P50/P95/P99
   - Cache size, eviction rate, memory usage
   - Alert on hit ratio drop > 10%

6. **Cost optimization:**
   - Cache most popular queries (Pareto: 20% of queries = 80% of traffic)
   - Adaptive TTL based on query frequency
   - Automatic warm-up based on traffic patterns

---

### Q18: How would you implement request coalescing to prevent cache stampede in a distributed system?

**Expected answer:**
1. **Local lock (per process):** Use `asyncio.Lock` or `threading.Lock` with a key-specific approach — only the first request computes, others wait.
2. **Distributed lock (across processes):** Use Redis SETNX with TTL — the first request acquires a lock for the cache key and computes; others poll for the cached value.
3. **Optimistic approach:** Allow all requests to compute, but use a "compute token" — only one finishes and writes to cache; others' results are discarded.
4. **Circuit breaker:** If TTL expires and cache miss rate spikes, activate coalescing mode with longer TTL.

```python
def get_with_coalescing(key, compute_fn, redis_client, ttl=60):
    # Check cache
    cached = redis_client.get(key)
    if cached:
        return cached

    # Try to acquire compute lock
    lock_key = f"lock:{key}"
    lock_acquired = redis_client.setnx(lock_key, "1")
    redis_client.expire(lock_key, 10)

    if lock_acquired:
        result = compute_fn()
        redis_client.setex(key, ttl, result)
        redis_client.delete(lock_key)
        return result

    # Wait for the computing request to finish
    for _ in range(100):
        time.sleep(0.05)
        cached = redis_client.get(key)
        if cached:
            return cached

    raise TimeoutError("Cache compute timeout")
```

---

### Q19: How would you handle personalization in a cached LLM system?

**Expected answer:** Personalization (user-specific responses) makes caching harder because the same prompt might need different responses for different users.

**Strategies:**
1. **Selective caching:** Cache only the non-personalized portion of the response. Cache key includes user persona/segment, not user ID.
2. **Template-based caching:** Cache the base response, then fill in personalized variables (name, preferences) client-side.
3. **User-group caching:** Assign users to groups (segments) and cache per group. Key = {prompt, group_id}.
4. **Hybrid approach:** Cache the system prompt + context (KV cache), but generate the user-specific response fresh.
5. **Blind caching:** Cache as usual, but mask PII in cache keys (hash user IDs).

**Best practice:** Cache the shared prefix (system prompt, context documents) via provider KV cache. Only personalize the final user message.

---

### Q20: Design a caching strategy for an LLM that generates code. The output must always use the latest library versions.

**Expected answer:**
1. **Version-based cache invalidation:**
   - Cache key includes `{prompt, library_versions_hash}`.
   - When a library is updated, increment the global version counter.
   - All cache entries with the old version become invalid.

2. **TTL with sliding window:**
   - Default TTL: 1 hour.
   - If a cache entry is accessed frequently, extend its TTL.
   - Force refresh if a security patch is released.

3. **Selective invalidation by dependency:**
   - Maintain an index: `library_name → [cache_keys]`.
   - When `numpy` is updated, invalidate only entries that use numpy.

4. **Dependency graph awareness:**
   - Parse the user's import statements to determine which libraries are relevant.
   - Cache key includes resolved dependency tree hash.

5. **Testing layer:**
   - Before returning cached code, run a syntax check or unit test.
   - If the test fails (e.g., deprecated API), regenerate.

6. **Warm-up:**
   - After a library version bump, regenerate code for the top-100 most common code generation patterns.

---

### Q21: How do you handle non-deterministic LLM outputs (temperature > 0) in a cache?

**Expected answer:**
1. **Include temperature in cache key:** Exact match only for same temperature. `T=0` responses are deterministic and highly cacheable. `T>0` responses vary — reduce cache effectiveness.
2. **Cache temperature=0 only:** Only cache deterministic outputs. For creative tasks, skip caching or use very short TTL.
3. **Seed-based caching:** If the API supports `seed` parameter, include it in cache key. Same seed + same prompt = same output.
4. **Semantic caching for creative tasks:** Cache the gist (semantic meaning) rather than exact text. Use semantic cache with moderation threshold.
5. **Best practice:** Cache temperature=0 responses aggressively. For temperature>0, only cache the shared prefix (KV cache), not the full response.

---

### Q22: What are the privacy implications of caching LLM responses?

**Expected answer:**
1. **Data leakage:** Cached responses may contain sensitive information (PII, proprietary data). Another user's cache hit could leak data meant for a different user.
2. **Cache key hashing:** Hash user-specific data in cache keys. Never store raw PII in keys.
3. **User segregation:** Use separate cache namespaces per tenant in multi-tenant systems.
4. **Encryption at rest:** Encrypt cache entries containing sensitive data.
5. **Access control:** Cache should not be directly accessible to end users.
6. **Compliance:** GDPR/SOC2 considerations — right to erasure means you must be able to delete specific cache entries on request.
7. **Audit logging:** Log cache access for sensitive queries.

**Best practices:** Never cache PII in response content. Hash user IDs in cache keys. Use tenant isolation. Implement cache entry deletion API.

---

### Q23: Compare caching strategies for a RAG pipeline.

| Strategy | What It Caches | Benefit | Trade-off |
|----------|---------------|---------|-----------|
| Document embedding cache | Document chunk embeddings | Avoid re-embedding chunks | Storage cost for embeddings |
| Query embedding cache | User query embeddings | Avoid recomputing same query embedding | Only helps if queries repeat |
| Retrieval result cache | Retrieved chunks for a query | Skip vector DB search on cache hit | Stale results if documents change |
| Generation cache | Final LLM response | Skip LLM call entirely | Stale responses, no personalization |
| KV cache (provider) | Shared prompt tokens | 50% discount on input tokens | Requires shared prefix |

**Recommended:** Use all five layers with appropriate TTLs. Document embeddings: cache permanently. Query embeddings: short TTL (minutes). Retrieval results: version-based invalidation. Generation: short TTL with personalization exclusion.

---

### Q24: How do you test a caching system?

**Expected answer:**
1. **Unit tests:**
   - Cache write + read returns correct value.
   - TTL expiry returns None.
   - Invalidation removes entry.
   - Concurrent access is thread-safe.

2. **Integration tests:**
   - Cache hit reduces latency (measure with/without).
   - Cache hit returns same response as direct LLM call.
   - Semantic cache returns correct response for similar queries.
   - Cache miss correctly calls LLM.

3. **Load tests:**
   - Cache withstands high concurrency (1000+ QPS).
   - No cache stampede under burst traffic.
   - Memory doesn't grow unbounded (LRU works).

4. **Resilience tests:**
   - Redis failure falls back to LLM (graceful degradation).
   - Network partition doesn't corrupt cache.
   - Cache rebuild after crash succeeds.

5. **Accuracy tests:**
   - Semantic cache: measure precision and recall at different thresholds.
   - Response cache: verify cached and fresh responses are semantically equivalent.
   - No hallucination amplification from stale cached content.

---

### Q25: What's the future of prompt caching in AI systems?

**Expected answer:** 
1. **Agent-level caching:** Cache entire agent trajectories (not just single LLM calls) — if an agent found the same answer through a chain of tool calls, cache the full trajectory.
2. **Cross-model caching:** Shared KV cache representations across different models from the same family.
3. **Learned cache policies:** ML models that predict which queries to cache and when to invalidate.
4. **Hardware KV cache:** GPU manufacturers building dedicated KV cache memory to support longer contexts without OOM.
5. **Standardized caching protocols:** Cross-provider caching standards (similar to HTTP caching headers).
6. **Federated caching:** Cache shared across organizations for common queries.
7. **Cache as a service:** Specialized caching layers offered by cloud providers (like Cloudflare for LLMs).
