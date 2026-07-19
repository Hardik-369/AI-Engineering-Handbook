# Code Examples: Prompt Caching

> Working, production-ready code examples for every major caching strategy. All examples use real API calls with the official SDKs and Redis.

---

## Setup

```python
import os
import json
import hashlib
import time
from typing import Optional, List, Tuple, Dict, Any
import numpy as np
from openai import OpenAI
from anthropic import Anthropic
import google.generativeai as genai
import redis

# Configure clients
openai_client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
anthropic_client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
genai.configure(api_key=os.environ["GEMINI_API_KEY"])
redis_client = redis.Redis(host="localhost", port=6379, db=0)
```

---

## 1. In-Memory Exact Cache

```python
class ExactCache:
    def __init__(self, ttl_seconds: int = 300):
        self.cache: Dict[str, Dict[str, Any]] = {}
        self.ttl = ttl_seconds
        self.hits = 0
        self.misses = 0

    def _make_key(self, model: str, messages: list, temperature: float = 0,
                  max_tokens: int = 1024, stop: Optional[List[str]] = None) -> str:
        data = json.dumps({
            "model": model,
            "messages": messages,
            "temperature": temperature,
            "max_tokens": max_tokens,
            "stop": stop
        }, sort_keys=True)
        return hashlib.sha256(data.encode()).hexdigest()

    def _normalize_messages(self, messages: list) -> list:
        """Normalize messages to increase cache hit rate."""
        normalized = []
        for msg in messages:
            if isinstance(msg["content"], str):
                content = msg["content"].strip().lower()
                content = " ".join(content.split())
            else:
                content = msg["content"]
            normalized.append({"role": msg["role"], "content": content})
        return normalized

    def get(self, model: str, messages: list, temperature: float = 0,
            max_tokens: int = 1024, stop: Optional[List[str]] = None) -> Optional[str]:
        normalized = self._normalize_messages(messages)
        key = self._make_key(model, normalized, temperature, max_tokens, stop)
        entry = self.cache.get(key)

        if entry is None:
            self.misses += 1
            return None

        if time.time() - entry["timestamp"] > self.ttl:
            del self.cache[key]
            self.misses += 1
            return None

        self.hits += 1
        return entry["response"]

    def set(self, model: str, messages: list, response: str, temperature: float = 0,
            max_tokens: int = 1024, stop: Optional[List[str]] = None):
        normalized = self._normalize_messages(messages)
        key = self._make_key(model, normalized, temperature, max_tokens, stop)
        self.cache[key] = {"response": response, "timestamp": time.time()}

    def invalidate(self, model: str, messages: list, temperature: float = 0,
                   max_tokens: int = 1024, stop: Optional[List[str]] = None):
        normalized = self._normalize_messages(messages)
        key = self._make_key(model, normalized, temperature, max_tokens, stop)
        self.cache.pop(key, None)

    def hit_ratio(self) -> float:
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0.0

    def size(self) -> int:
        return len(self.cache)


# Usage
cache = ExactCache(ttl_seconds=600)

def cached_completion(model: str, messages: list, **kwargs) -> str:
    cached = cache.get(model, messages, **kwargs)
    if cached:
        print("Cache HIT")
        return cached

    print("Cache MISS — calling LLM")
    response = openai_client.chat.completions.create(
        model=model,
        messages=messages,
        **kwargs
    )
    result = response.choices[0].message.content
    cache.set(model, messages, result, **kwargs)
    return result

# First call — cache miss
response1 = cached_completion("gpt-4o", [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is prompt caching?"}
])

# Second call — cache hit (same messages)
response2 = cached_completion("gpt-4o", [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is prompt caching?"}
])

print(f"Hit ratio: {cache.hit_ratio():.2%}")
print(f"Cache size: {cache.size()}")
```

---

## 2. Semantic Cache

```python
class SemanticCache:
    def __init__(self, embedding_model: str = "text-embedding-3-small",
                 similarity_threshold: float = 0.95,
                 max_entries: int = 10000):
        self.client = OpenAI()
        self.embedding_model = embedding_model
        self.threshold = similarity_threshold
        self.max_entries = max_entries
        self.entries: List[Dict[str, Any]] = []
        self.hits = 0
        self.misses = 0

    def _get_embedding(self, text: str) -> np.ndarray:
        response = self.client.embeddings.create(
            model=self.embedding_model,
            input=text
        )
        return np.array(response.data[0].embedding)

    def _cosine_similarity(self, a: np.ndarray, b: np.ndarray) -> float:
        return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

    def search(self, query: str) -> Optional[Tuple[str, float]]:
        """Search cache for semantically similar query."""
        query_emb = self._get_embedding(query)
        max_sim = 0.0
        best_response = None

        for entry in self.entries:
            sim = self._cosine_similarity(query_emb, entry["embedding"])
            if sim > max_sim:
                max_sim = sim
                best_response = entry["response"]

        if max_sim >= self.threshold:
            self.hits += 1
            return (best_response, max_sim)

        self.misses += 1
        return None

    def store(self, query: str, response: str):
        """Store query-response pair in cache."""
        embedding = self._get_embedding(query)

        # Evict oldest entry if at capacity
        if len(self.entries) >= self.max_entries:
            self.entries.pop(0)

        self.entries.append({
            "query": query,
            "embedding": embedding,
            "response": response,
            "timestamp": time.time()
        })

    def hit_ratio(self) -> float:
        total = self.hits + self.misses
        return self.hits / total if total > 0 else 0.0


# Usage with LLM fallback
sem_cache = SemanticCache(threshold=0.92)

def semantic_cached_query(user_query: str) -> str:
    # Check semantic cache
    cached = sem_cache.search(user_query)
    if cached:
        response, similarity = cached
        print(f"Semantic cache HIT (similarity: {similarity:.4f})")
        return response

    # Cache miss — call LLM
    print("Semantic cache MISS — calling LLM")
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": user_query}],
        temperature=0
    )
    result = response.choices[0].message.content
    sem_cache.store(user_query, result)
    return result

# Test with semantically similar queries
print(semantic_cached_query("What is the capital of France?"))
print(semantic_cached_query("What's France's capital city?"))  # Should hit cache
print(semantic_cached_query("Capital of France"))  # Should also hit cache
```

---

## 3. Redis Cache with TTL

```python
class RedisLLMCache:
    def __init__(self, host="localhost", port=6379, db=0, ttl=3600):
        self.client = redis.Redis(host=host, port=port, db=db,
                                  decode_responses=True)
        self.ttl = ttl
        self.namespace = "llm_cache:"

    def _key(self, model: str, messages: list, **params) -> str:
        data = json.dumps({
            "model": model,
            "messages": messages,
            **params
        }, sort_keys=True)
        key_hash = hashlib.sha256(data.encode()).hexdigest()
        return f"{self.namespace}{key_hash}"

    def get(self, model: str, messages: list, **params) -> Optional[str]:
        key = self._key(model, messages, **params)
        try:
            value = self.client.get(key)
            if value:
                return value
        except redis.RedisError as e:
            print(f"Redis error: {e}")
        return None

    def set(self, model: str, messages: list, response: str, **params):
        key = self._key(model, messages, **params)
        try:
            self.client.setex(key, self.ttl, response)
        except redis.RedisError as e:
            print(f"Redis error: {e}")

    def delete(self, model: str, messages: list, **params):
        key = self._key(model, messages, **params)
        try:
            self.client.delete(key)
        except redis.RedisError as e:
            print(f"Redis error: {e}")

    def flush(self):
        try:
            for key in self.client.scan_iter(f"{self.namespace}*"):
                self.client.delete(key)
        except redis.RedisError as e:
            print(f"Redis error: {e}")

    def stats(self) -> Dict[str, Any]:
        try:
            keys = list(self.client.scan_iter(f"{self.namespace}*"))
            return {
                "total_keys": len(keys),
                "ttl_seconds": self.ttl,
                "namespace": self.namespace
            }
        except redis.RedisError:
            return {"error": "Could not connect to Redis"}


# Usage
redis_cache = RedisLLMCache(ttl=1800)

def llm_with_redis_cache(model: str, messages: list, **kwargs) -> str:
    cached = redis_cache.get(model, messages, **kwargs)
    if cached:
        print("Redis cache HIT")
        return cached

    print("Redis cache MISS — calling LLM")
    response = openai_client.chat.completions.create(
        model=model,
        messages=messages,
        **kwargs
    )
    result = response.choices[0].message.content
    redis_cache.set(model, messages, result, **kwargs)
    return result
```

---

## 4. RedisVL Semantic Cache

```python
import numpy as np
from redisvl import Index
from redisvl.schema import IndexSchema

class RedisVLSemanticCache:
    def __init__(self, redis_url="redis://localhost:6379",
                 embedding_dims=1536, threshold=0.92):
        self.threshold = threshold
        self.client = OpenAI()

        schema = IndexSchema.from_dict({
            "index": {"name": "semantic_cache_v2", "prefix": "sc:"},
            "fields": [
                {"name": "query_text", "type": "text"},
                {"name": "response", "type": "text"},
                {
                    "name": "embedding",
                    "type": "vector",
                    "attrs": {
                        "dims": embedding_dims,
                        "algorithm": "hnsw",
                        "distance_metric": "cosine",
                        "ef_construction": 200,
                        "m": 16
                    }
                },
                {"name": "timestamp", "type": "numeric"},
                {"name": "hit_count", "type": "numeric"}
            ]
        })

        self.index = Index(schema)
        self.index.connect(redis_url)

    def _get_embedding(self, text: str) -> list:
        response = self.client.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return response.data[0].embedding

    def search(self, query: str) -> Optional[str]:
        query_emb = self._get_embedding(query)

        results = self.index.query(
            vector=query_emb,
            top_k=1,
            return_fields=["response", "hit_count"],
            num_results=1
        )

        if results:
            distance = results[0].distance
            similarity = 1 - distance

            if similarity >= self.threshold:
                # Increment hit counter
                self.index.update(
                    results[0].id,
                    {"hit_count": results[0].hit_count + 1}
                )
                return results[0].response

        return None

    def store(self, query: str, response: str):
        embedding = self._get_embedding(query)
        self.index.load([
            {
                "query_text": query,
                "response": response,
                "embedding": embedding,
                "timestamp": time.time(),
                "hit_count": 0
            }
        ])

    def get_popular(self, top_k: int = 10) -> List[Dict]:
        """Get most frequently hit cache entries."""
        results = self.index.query(
            vector=[0] * 1536,  # dummy vector for sorting
            top_k=top_k,
            return_fields=["query_text", "hit_count"],
            sort_by="hit_count",
            sort_order="desc"
        )
        return [
            {"query": r.query_text, "hits": r.hit_count}
            for r in results
        ]


# Usage
vl_cache = RedisVLSemanticCache()

def semantic_cached_chat(query: str) -> str:
    result = vl_cache.search(query)
    if result:
        return f"[CACHED] {result}"

    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": query}]
    )
    result = response.choices[0].message.content
    vl_cache.store(query, result)
    return result
```

---

## 5. Anthropic Prompt Caching

```python
def anthropic_with_caching(user_query: str, long_context: str):
    """
    Demonstrate Anthropic's prompt caching with cache_control.
    The long context document is cached for reuse across requests.
    """
    response = anthropic_client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1000,
        system=[
            {
                "type": "text",
                "text": "You are a document analysis assistant. "
                        "Answer questions based on the provided document.",
                "cache_control": {"type": "ephemeral"}
            }
        ],
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": f"Here is the document to analyze:\n\n{long_context}",
                        "cache_control": {"type": "ephemeral"}
                    },
                    {
                        "type": "text",
                        "text": user_query
                    }
                ]
            }
        ]
    )

    usage = response.usage
    print(f"Input tokens: {usage.input_tokens}")
    print(f"Output tokens: {usage.output_tokens}")
    print(f"Cache creation tokens: {usage.cache_creation_input_tokens}")
    print(f"Cache read tokens: {usage.cache_read_input_tokens}")

    # Calculate effective cost
    uncached_cost = usage.input_tokens * 3.0 / 1_000_000
    cached_cost = (
        usage.cache_read_input_tokens * 1.5 / 1_000_000 +
        usage.cache_creation_input_tokens * 3.75 / 1_000_000 +
        (usage.input_tokens - usage.cache_read_input_tokens
         - usage.cache_creation_input_tokens) * 3.0 / 1_000_000
    )

    print(f"Uncached cost: ${uncached_cost:.6f}")
    print(f"Cached cost: ${cached_cost:.6f}")
    print(f"Savings: ${uncached_cost - cached_cost:.6f}")

    return response.content[0].text


# First call — creates cache
long_doc = "..." * 500  # 1500+ tokens
response1 = anthropic_with_caching("Summarize the document.", long_doc)

# Second call — reads from cache
response2 = anthropic_with_caching(
    "What are the key findings?", long_doc
)
```

---

## 6. OpenAI Prompt Caching

```python
def openai_with_caching(user_query: str, long_system_prompt: str):
    """
    OpenAI automatically caches repeated prefix tokens.
    Just ensure the shared prefix is long enough (>1024 tokens).
    """
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": long_system_prompt},
            {"role": "user", "content": user_query}
        ],
        temperature=0
    )

    usage = response.usage
    details = usage.prompt_tokens_details

    print(f"Total input tokens: {usage.prompt_tokens}")
    print(f"Cached tokens: {details.cached_tokens if details else 0}")
    print(f"Output tokens: {usage.completion_tokens}")

    if details and details.cached_tokens > 0:
        savings = details.cached_tokens * 2.5 / 1_000_000 / 2
        print(f"Savings from caching: ${savings:.6f}")

    return response.choices[0].message.content


# Build a system prompt >1024 tokens
system_prompt = "You are an expert analyst.\n" * 200  # ~800 tokens
system_prompt += "You specialize in: " + ", ".join([
    "technology", "finance", "healthcare", "energy",
    "education", "transportation", "retail", "media"
] * 20)  # total > 1024 tokens

# First call — warm cache
resp1 = openai_with_caching("Analyze Q3 trends.", system_prompt)

# Second call — cached
resp2 = openai_with_caching("What are the risks?", system_prompt)
```

---

## 7. Gemini Context Caching

```python
import datetime

def gemini_with_caching():
    """
    Gemini requires explicit context cache creation.
    Create once, reuse across multiple queries.
    """
    long_context = "..." * 11000  # > 32K tokens

    # Create cached content
    cache = genai.caching.CachedContent.create(
        model='models/gemini-1.5-pro',
        display_name='analysis-context',
        system_instruction={
            "parts": [{
                "text": "You are an expert document analyst. "
                        "Answer questions based on the provided context."
            }]
        },
        contents=[{
            "parts": [{"text": long_context}]
        }],
        ttl=datetime.timedelta(minutes=30)
    )

    print(f"Cache name: {cache.name}")
    print(f"Cache expire time: {cache.expire_time}")

    # Use cached content for generation
    model = genai.GenerativeModel.from_cached_content(cached_content=cache)

    # Multiple queries using the same cached context
    queries = [
        "Summarize the main arguments.",
        "What evidence supports claim X?",
        "What are the counterarguments?"
    ]

    for query in queries:
        start = time.time()
        response = model.generate_content(query)
        latency = time.time() - start

        usage = response.usage_metadata
        print(f"Query: {query[:50]}...")
        print(f"  Latency: {latency:.2f}s")
        print(f"  Cached tokens: {usage.cached_content_token_count}")
        print(f"  Total tokens: {usage.total_token_count}")

    # Clean up
    cache.delete()
```

---

## 8. KV Cache Visualization

```python
def kv_cache_simulation():
    """
    Simulate how KV cache works during LLM inference.
    This is a conceptual demonstration, not actual GPU code.
    """
    import numpy as np

    class KVCache:
        def __init__(self, num_layers=32, num_heads=16, head_dim=64):
            self.num_layers = num_layers
            self.num_heads = num_heads
            self.head_dim = head_dim
            # Shape: (layers, 2, batch, heads, seq_len, head_dim)
            # 2 = (K, V)
            self.cache = None
            self.seq_len = 0

        def prefill(self, tokens: int):
            """Simulate prefill phase — compute and store KV for all tokens."""
            if self.cache is None:
                self.cache = np.zeros((
                    self.num_layers, 2, 1, self.num_heads,
                    self.seq_len + tokens, self.head_dim
                ))
            self.seq_len += tokens
            print(f"Prefilled {tokens} tokens. Total cached: {self.seq_len}")

        def decode(self, token: int = 1):
            """Simulate decode phase — only compute for new token."""
            self.seq_len += token
            print(f"Decoded {token} token(s). Total cached: {self.seq_len}")

        def reset(self):
            self.cache = None
            self.seq_len = 0
            print("KV cache cleared")

    # Simulate two requests with shared prefix
    cache = KVCache(num_layers=8, num_heads=4, head_dim=32)

    print("=== Request 1 ===")
    cache.prefill(1024)  # System prompt
    cache.prefill(50)    # User input
    cache.decode(200)    # Generate response

    print("\n=== Request 2 (shared prefix) ===")
    cache.reset()
    cache.prefill(1024)  # System prompt (could be restored from cache)
    cache.prefill(75)    # Different user input
    cache.decode(150)    # Generate response

kv_cache_simulation()
```

---

## 9. Layered Cache (L1 + L2 + L3)

```python
class LayeredCache:
    """
    Multi-layer cache:
    L1: In-memory LRU (fastest, smallest)
    L2: Redis (fast, larger)
    L3: LLM API (slow, expensive)
    """

    def __init__(self, redis_host="localhost", redis_port=6379,
                 l1_size=1000, l1_ttl=60, l2_ttl=3600):
        # L1: In-memory LRU
        self.l1: Dict[str, Dict[str, Any]] = {}
        self.l1_size = l1_size
        self.l1_ttl = l1_ttl
        self.l1_access: List[str] = []  # LRU tracking

        # L2: Redis
        self.l2 = redis.Redis(host=redis_host, port=redis_port,
                              decode_responses=True)
        self.l2_ttl = l2_ttl

        # L3: LLM call (no storage needed)

        # Stats
        self.stats = {"l1_hits": 0, "l2_hits": 0, "l3_misses": 0}

    def _key(self, model: str, messages: list) -> str:
        data = json.dumps({"model": model, "messages": messages}, sort_keys=True)
        return hashlib.sha256(data.encode()).hexdigest()

    def _evict_l1_if_needed(self):
        while len(self.l1) >= self.l1_size:
            oldest = self.l1_access.pop(0)
            del self.l1[oldest]

    def get(self, model: str, messages: list) -> Optional[str]:
        key = self._key(model, messages)

        # L1 check
        if key in self.l1:
            entry = self.l1[key]
            if time.time() - entry["ts"] < self.l1_ttl:
                self.stats["l1_hits"] += 1
                # Move to end of LRU list
                self.l1_access.remove(key)
                self.l1_access.append(key)
                return entry["response"]
            else:
                del self.l1[key]
                self.l1_access.remove(key)

        # L2 check
        l2_value = self.l2.get(f"llm:{key}")
        if l2_value:
            self.stats["l2_hits"] += 1
            # Promote to L1
            self._evict_l1_if_needed()
            self.l1[key] = {"response": l2_value, "ts": time.time()}
            self.l1_access.append(key)
            return l2_value

        self.stats["l3_misses"] += 1
        return None

    def set(self, model: str, messages: list, response: str):
        key = self._key(model, messages)

        # Store in L1
        self._evict_l1_if_needed()
        self.l1[key] = {"response": response, "ts": time.time()}
        self.l1_access.append(key)

        # Store in L2
        self.l2.setex(f"llm:{key}", self.l2_ttl, response)

    def report(self) -> Dict[str, Any]:
        total = sum(self.stats.values())
        return {
            **self.stats,
            "l1_size": len(self.l1),
            "hit_ratio": (self.stats["l1_hits"] + self.stats["l2_hits"]) / total
            if total > 0 else 0
        }


# Usage
def llm_with_layered_cache(model: str, messages: list) -> str:
    layered = LayeredCache()

    cached = layered.get(model, messages)
    if cached:
        return cached

    response = openai_client.chat.completions.create(
        model=model,
        messages=messages
    )
    result = response.choices[0].message.content
    layered.set(model, messages, result)
    return result
```

---

## 10. Cache Hit Ratio Monitor

```python
import threading
from collections import deque
import matplotlib.pyplot as plt

class CacheMonitor:
    def __init__(self, window_size: int = 1000):
        self.hits = deque(maxlen=window_size)
        self.misses = deque(maxlen=window_size)
        self.total_hits = 0
        self.total_misses = 0
        self.lock = threading.Lock()

    def record_hit(self):
        with self.lock:
            self.hits.append(1)
            self.misses.append(0)
            self.total_hits += 1

    def record_miss(self):
        with self.lock:
            self.hits.append(0)
            self.misses.append(1)
            self.total_misses += 1

    def windowed_hit_ratio(self) -> float:
        with self.lock:
            total = len(self.hits)
            if total == 0:
                return 0.0
            return sum(self.hits) / total

    def overall_hit_ratio(self) -> float:
        total = self.total_hits + self.total_misses
        return self.total_hits / total if total > 0 else 0.0

    def get_stats(self) -> Dict[str, Any]:
        return {
            "windowed_hit_ratio": self.windowed_hit_ratio(),
            "overall_hit_ratio": self.overall_hit_ratio(),
            "total_hits": self.total_hits,
            "total_misses": self.total_misses,
            "window_size": len(self.hits)
        }

    def plot_ratio_trend(self, ratios: List[float]):
        plt.figure(figsize=(10, 5))
        plt.plot(ratios, label="Hit Ratio (windowed)")
        plt.axhline(y=sum(ratios)/len(ratios), color='r',
                    linestyle='--', label="Average")
        plt.xlabel("Time window")
        plt.ylabel("Hit ratio")
        plt.title("Cache Hit Ratio Over Time")
        plt.legend()
        plt.grid(True)
        plt.savefig("cache_hit_ratio.png")


# Usage with caching system
monitor = CacheMonitor(window_size=500)

def monitored_cached_call(llm_call_fn, *args, **kwargs):
    """Wrap any cached LLM call with monitoring."""
    # Simulating: replace with actual cache check
    import random
    is_hit = random.random() < 0.6  # 60% hit rate

    if is_hit:
        monitor.record_hit()
        return "[CACHED] response"
    else:
        monitor.record_miss()
        return llm_call_fn(*args, **kwargs)

# Simulate requests
for i in range(1000):
    result = monitored_cached_call(
        openai_client.chat.completions.create,
        model="gpt-4o",
        messages=[{"role": "user", "content": f"Query {i}"}]
    )

print(json.dumps(monitor.get_stats(), indent=2))
```

---

## 11. Request Coalescing

```python
import asyncio
import threading

class CoalescingCache:
    """
    Prevents cache stampede: when N concurrent requests miss cache,
    only one calls the LLM; others wait for the result.
    """

    def __init__(self, ttl: int = 300):
        self.cache: Dict[str, str] = {}
        self.in_flight: Dict[str, asyncio.Future] = {}
        self.ttl = ttl
        self.lock = asyncio.Lock()

    async def get_or_compute(self, key: str, compute_fn) -> str:
        # Check cache
        if key in self.cache:
            return self.cache[key]

        async with self.lock:
            # Double-check after acquiring lock
            if key in self.cache:
                return self.cache[key]

            # Check if another request is already computing this
            if key in self.in_flight:
                future = self.in_flight[key]
            else:
                # We're the first — start computing
                future = asyncio.Future()
                self.in_flight[key] = future

        if not future.done():
            # We're computing — do the LLM call
            try:
                result = await compute_fn()
                self.cache[key] = result
                future.set_result(result)
            except Exception as e:
                future.set_exception(e)
                raise
            finally:
                async with self.lock:
                    del self.in_flight[key]

        # Wait for result (either from our computation or another request's)
        return await future

    def invalidate(self, key: str):
        self.cache.pop(key, None)


# Usage
async def example():
    cache = CoalescingCache()
    llm_call_count = 0

    async def expensive_llm_call(query: str) -> str:
        nonlocal llm_call_count
        llm_call_count += 1
        await asyncio.sleep(2)  # Simulate LLM latency
        return f"Response to: {query}"

    # Simulate 50 concurrent requests for the same key
    query = "What is prompt caching?"
    tasks = [
        cache.get_or_compute(query, lambda: expensive_llm_call(query))
        for _ in range(50)
    ]
    results = await asyncio.gather(*tasks)

    print(f"LLM calls made: {llm_call_count}")  # Should be 1
    print(f"Unique responses: {len(set(results))}")  # Should be 1
    print(f"All same: {all(r == results[0] for r in results)}")  # Should be True


# Run: asyncio.run(example())
```

---

## 12. Cache Warm-Up Script

```python
def cache_warm_up(cache, historical_queries: List[str],
                  llm_fn, top_k: int = 100,
                  concurrency: int = 5):
    """
    Pre-populate cache with most popular queries.
    """
    import concurrent.futures
    from collections import Counter

    # Find most popular queries
    query_counts = Counter(historical_queries)
    popular = [q for q, _ in query_counts.most_common(top_k)]

    print(f"Warming cache with {len(popular)} queries...")

    def warm_single(query: str) -> str:
        if cache.get(query):
            return f"Already cached: {query[:50]}"

        response = llm_fn(query)
        cache.set(query, response)
        return f"Cached: {query[:50]}"

    with concurrent.futures.ThreadPoolExecutor(max_workers=concurrency) as pool:
        results = list(pool.map(warm_single, popular))

    print(f"Warm-up complete. {sum(1 for r in results if 'Cached:' in r)} new entries.")


# Usage
historical_logs = [
    "What is prompt caching?",
    "How to reduce LLM costs?",
    "What is prompt caching?",
    "Best practices for LLM",
    "How to reduce LLM costs?",
    # ... 10,000+ entries
    "What is prompt caching?",
]

cache_warm_up(
    ExactCache(ttl=3600),
    historical_logs,
    lambda q: openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": q}]
    ).choices[0].message.content,
    top_k=50
)
```
