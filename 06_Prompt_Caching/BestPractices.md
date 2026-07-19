# Best Practices: Production Prompt Caching

> A comprehensive guide to designing, implementing, and operating caching systems for LLM applications in production.

---

## 1. Caching Strategy Selection

### Decision Framework

```
1. Is the prompt deterministic (temperature=0)?
   YES → Cache aggressively (exact + semantic)
   NO  → Cache prefix only (KV cache) or skip

2. Is the prompt repeated frequently?
   YES → Prioritize exact + KV caching
   NO  → Prioritize semantic caching (wider net)

3. Is latency critical?
   YES → L1 (in-memory) + L2 (Redis)
   NO  → L2 (Redis) + L3 (provider KV) may suffice

4. Is cost reduction the primary goal?
   YES → Maximize KV cache usage (50% discount)
   NO  → Balance hit ratio vs. infrastructure cost
```

### Strategy Selection Matrix

| Use Case | Primary Strategy | Secondary Strategy | Cache Key |
|----------|-----------------|-------------------|-----------|
| Customer support FAQ | Semantic cache | Exact cache | Normalized query |
| Code generation | Exact cache + version | Provider KV cache | Prompt + dependency versions |
| Document Q&A | Provider KV cache | Semantic cache | Document ID + query |
| Translation | Exact cache | Response cache | Source text + target language |
| Classification | Embedding cache | Response cache | Text hash |
| Chatbot | Provider KV cache | Semantic cache | Session prefix + query |

---

## 2. Cache Key Design

### Key Components

Always include in cache key:
- `model` — Different models give different outputs
- `messages` or normalized prompt — The actual input
- `temperature` — Affects output diversity
- `max_tokens` — Affects response length
- `stop_sequences` — Affects where generation stops

Conditionally include:
- `seed` — For reproducible outputs
- `top_p`, `frequency_penalty`, `presence_penalty` — If non-default
- `response_format` — JSON mode vs. text

Never include in cache key:
- `user_id` — Unless personalization is part of the prompt
- `request_id` — Unique per request
- Timestamps — Would make every key unique

### Key Format

```python
def build_cache_key(model: str, messages: list, **params) -> str:
    """
    Build a deterministic, collision-resistant cache key.
    """
    canonical = {
        "model": model,
        "messages": normalize_messages(messages),
        "temperature": params.get("temperature", 0),
        "max_tokens": params.get("max_tokens", 1024),
        "stop": params.get("stop", None),
        "seed": params.get("seed", None),
        "response_format": params.get("response_format", None)
    }
    serialized = json.dumps(canonical, sort_keys=True, ensure_ascii=False)
    return hashlib.sha256(serialized.encode()).hexdigest()
```

### Key Normalization

Increase exact cache hit rate by normalizing inputs:

```python
def normalize_messages(messages: list) -> list:
    normalized = []
    for msg in messages:
        content = msg.get("content", "")
        if isinstance(content, str):
            # Trim whitespace
            content = content.strip()
            # Collapse multiple spaces
            content = re.sub(r'\s+', ' ', content)
            # Remove trailing punctuation (optional)
            content = content.rstrip('.,!?;:')
        normalized.append({"role": msg["role"], "content": content})
    return normalized
```

---

## 3. TTL Configuration

### Recommended TTLs by Use Case

| Use Case | TTL | Rationale |
|----------|-----|-----------|
| System prompt (provider cache) | N/A (sliding) | Provider manages — 5-10 min inactivity |
| FAQ responses | 24 hours | Answers change rarely |
| Code generation | 1 hour | Libraries update, best practices evolve |
| Translation pairs | 7 days | Language is stable |
| News/current events | 5 minutes | Rapidly changing |
| Personalized content | 0 (no cache) | Per-user customization |
| Embeddings | 24-48 hours | Embedding model rarely changes |

### Adaptive TTL

```python
class AdaptiveTTL:
    """Adjust TTL based on query frequency and recency."""

    def __init__(self, min_ttl=60, max_ttl=86400, decay_factor=0.5):
        self.min_ttl = min_ttl
        self.max_ttl = max_ttl
        self.decay_factor = decay_factor
        self.access_counts = {}

    def get_ttl(self, key: str) -> int:
        freq = self.access_counts.get(key, 0)
        # More frequent queries get longer TTL
        ttl = min(self.max_ttl, self.min_ttl * (1 + freq * self.decay_factor))
        return int(ttl)

    def record_access(self, key: str):
        self.access_counts[key] = self.access_counts.get(key, 0) + 1
```

---

## 4. Semantic Cache Tuning

### Threshold Calibration

```python
def find_optimal_threshold(cache: SemanticCache,
                           queries: List[str],
                           expected_responses: List[str],
                           min_accuracy: float = 0.95) -> float:
    """
    Find the highest threshold that maintains minimum accuracy.
    Lower threshold = higher hit rate but lower accuracy.
    """
    from sklearn.metrics import accuracy_score

    for threshold in np.arange(0.99, 0.70, -0.01):
        cache.threshold = threshold
        predictions = []

        for query in queries:
            cached = cache.search(query)
            if cached:
                predictions.append(cached)
            else:
                predictions.append(None)

        # Only evaluate queries that would be cache hits
        hit_indices = [i for i, p in enumerate(predictions) if p is not None]
        if not hit_indices:
            continue

        hit_accuracy = accuracy_score(
            [expected_responses[i] for i in hit_indices],
            [predictions[i] for i in hit_indices]
        )

        if hit_accuracy >= min_accuracy:
            hit_rate = len(hit_indices) / len(queries)
            print(f"Threshold {threshold:.2f}: "
                  f"Accuracy {hit_accuracy:.3f}, Hit rate {hit_rate:.3f}")
            return threshold

    return 0.95  # Safe default
```

### Similarity Score Distribution

Monitor the distribution of similarity scores to understand cache behavior:

```python
def plot_similarity_distribution(cache: SemanticCache):
    """Plot histogram of similarity scores from recent queries."""
    scores = [entry["similarity"] for entry in cache.recent_scores]
    plt.figure(figsize=(10, 5))
    plt.hist(scores, bins=50, alpha=0.7)
    plt.axvline(x=cache.threshold, color='r', linestyle='--',
                label=f"Threshold: {cache.threshold}")
    plt.xlabel("Similarity Score")
    plt.ylabel("Frequency")
    plt.title("Semantic Cache Similarity Score Distribution")
    plt.legend()
    plt.savefig("similarity_distribution.png")
```

---

## 5. Error Handling & Resilience

### Graceful Degradation

```python
class ResilientCache:
    """Cache wrapper that never blocks the LLM call."""

    def __init__(self, fallback_fn):
        self.cache = {}
        self.fallback_fn = fallback_fn

    def get(self, key: str) -> Optional[str]:
        try:
            return self.cache.get(key)
        except Exception as e:
            # Log error, return None to trigger LLM call
            logger.warning(f"Cache read failed: {e}")
            return None

    def set(self, key: str, value: str):
        try:
            self.cache[key] = value
        except Exception as e:
            logger.warning(f"Cache write failed: {e}")
```

### Circuit Breaker for Cache Backend

```python
class CacheCircuitBreaker:
    """Prevents cascading failures when Redis is down."""

    def __init__(self, failure_threshold=5, recovery_timeout=30):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.last_failure_time = 0
        self.state = "closed"  # closed, open, half-open

    def is_open(self) -> bool:
        if self.state == "open":
            if time.time() - self.last_failure_time > self.recovery_timeout:
                self.state = "half-open"
                return False
            return True
        return False

    def record_success(self):
        self.failure_count = 0
        self.state = "closed"

    def record_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = "open"
            logger.warning("Cache circuit breaker OPEN — skipping cache")
```

---

## 6. Monitoring & Observability

### Metrics to Track

| Metric | What It Measures | Alert Threshold |
|--------|-----------------|-----------------|
| Cache hit ratio | Effectiveness | < 20% |
| Cache miss ratio | How often we call LLM | > 80% |
| L1 hit ratio | Local cache effectiveness | < 10% |
| L2 hit ratio | Redis cache effectiveness | < 30% |
| Cache size | Memory usage | > 80% of capacity |
| Eviction rate | How often entries are removed | > 100/second |
| Cache latency (P50/P95/P99) | How fast the cache responds | P99 > 50ms |
| LLM call rate | How often we hit the API | > rate limit |
| Cost savings | $$ saved vs. no caching | < $100/day |

### Structured Logging

```python
import structlog
logger = structlog.get_logger()

def log_cache_operation(operation: str, cache_layer: str,
                        hit: bool, latency_ms: float, key: str):
    logger.info(
        "cache_operation",
        operation=operation,
        cache_layer=cache_layer,
        hit=hit,
        latency_ms=round(latency_ms, 2),
        key_prefix=key[:8] + "...",
    )

# Usage
log_cache_operation("get", "l1", True, 0.5, cache_key)
```

### Health Check Endpoint

```python
from flask import Flask, jsonify
app = Flask(__name__)

@app.route("/health/cache")
def cache_health():
    """Cache health check endpoint for monitoring."""
    try:
        redis_client.ping()
        redis_ok = True
    except:
        redis_ok = False

    return jsonify({
        "status": "healthy" if redis_ok else "degraded",
        "redis": "connected" if redis_ok else "disconnected",
        "l1_size": len(l1_cache),
        "l2_size": redis_client.dbsize(),
        "hit_ratio": cache_monitor.overall_hit_ratio(),
        "p95_latency": cache_monitor.p95_latency(),
        "eviction_count": eviction_counter.value
    })
```

---

## 7. Production Deployment

### Cache Warming Checklist

- [ ] Load historical query logs (last 30 days)
- [ ] Identify top-K most frequent queries (Pareto analysis)
- [ ] Generate LLM responses for top-K queries
- [ ] Store in cache with 2x normal TTL
- [ ] Verify cache hit ratio > 50% after warm-up
- [ ] Schedule periodic re-warm (daily/weekly)
- [ ] Monitor cold start after deployment

### Rollout Strategy

```
Phase 1: Shadow mode (1 week)
  - Run cache in parallel, no serving
  - Measure hit ratio, accuracy, latency
  - Cache vs. LLM response comparison

Phase 2: Canary (1 week, 5% traffic)
  - Serve cache hits to 5% of users
  - Monitor accuracy, latency, user feedback

Phase 3: Gradual rollout (2 weeks)
  - Increase by 25% each week
  - Monitor cost savings and P95 latency

Phase 4: Full production
  - 100% of traffic
  - Continuous monitoring and tuning
```

### Anti-Patterns

| Anti-Pattern | Problem | Solution |
|-------------|---------|----------|
| Caching personalized responses | User A sees User B's data | Include session/user hash in key |
| Infinite TTL | Stale responses forever | Always set TTL, even if long |
| No invalidation strategy | Can't update cache on data change | Version-based + event-based invalidation |
| Cache-everything approach | Memory blowup for diverse queries | LRU eviction + size limits |
| Ignoring caching headers | Provider cache not utilized | Always set cache_control when supported |
| Not normalizing inputs | Low exact cache hit rate | Normalize: trim, lowercase, collapse spaces |
| Embedding on cache hot path | Embedding call adds latency | Async embedding, batch processing |

---

## 8. Cost Optimization

### Maximize Provider KV Cache Usage

```python
class CostOptimizedPrompter:
    """Optimize prompts for maximum KV cache reuse."""

    def build_cached_prompt(self, system_prompt: str,
                            context: str,
                            user_query: str) -> list:
        """
        Structure prompt so shared prefix is maximized.
        Provider KV cache works on repeated prefixes.
        """
        return [
            {
                "role": "system",
                "content": system_prompt
                # Set cache_control if using Anthropic
            },
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": f"Context:\n{context}",
                        # Set cache_control if using Anthropic
                    },
                    {
                        "type": "text",
                        "text": f"Query: {user_query}"
                        # No cache_control — this changes per request
                    }
                ]
            }
        ]

    def estimate_savings(self, usage):
        """Estimate cost savings from caching."""
        total_input = usage.prompt_tokens
        cached = getattr(usage, 'prompt_tokens_details', {}).get('cached_tokens', 0)
        savings_ratio = cached / total_input if total_input > 0 else 0
        cost_without = total_input * self.input_price / 1_000_000
        cost_with = (cached * self.input_price * 0.5 +
                     (total_input - cached) * self.input_price) / 1_000_000
        return {
            "savings_ratio": savings_ratio,
            "cost_without": cost_without,
            "cost_with": cost_with,
            "savings": cost_without - cost_with
        }
```

### Tiered Caching for Cost Efficiency

| Cache Layer | Cost | Benefit | When to Use |
|-------------|------|---------|-------------|
| L1 (in-memory) | Free (already paid RAM) | <1μs, zero API cost | Always |
| L2 (Redis) | $50-200/month | <1ms, shared across services | 100+ QPS |
| L3 (Provider KV) | 50% discount | Latency reduction + cost cut | System prompts >1024 tokens |
| L4 (Semantic) | Embedding API cost | Broader cache hits | Diverse query patterns |
| L5 (Full response) | Storage cost | Zero LLM cost on hit | Deterministic, repeated queries |

---

## Summary

| Principle | Practice |
|-----------|----------|
| Cache the shared prefix | Use provider KV caching for system prompts and context |
| Normalize before caching | Trim, lowercase, collapse spaces for higher hit rate |
| Set appropriate TTL | Match TTL to data freshness requirements |
| Monitor hit ratio | Track hits/misses per layer; alert on drops |
| Graceful degradation | Never let cache failure block the LLM call |
| Warm the cache | Pre-populate before deployment |
| Tune semantic thresholds | Start high (0.95), measure, adjust down |
| Version your cache | Use version keys for atomic invalidation |
| Test cache accuracy | Verify cached responses match fresh LLM output |
| Consider ROI | Only build cache layers that pay for themselves |
