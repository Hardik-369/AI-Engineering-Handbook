# Project: Build a Multi-Source Research Assistant

Build a research assistant that answers questions using multiple sources with verifiable citations. This project applies all major concepts from Chapter 08: system architecture, pipeline design, context management, retrieval, caching, and error handling.

---

## Learning Objectives

- Design and implement a multi-component AI system architecture
- Build a RAG pipeline with web search and document retrieval
- Implement context management with token budgeting
- Handle errors, rate limits, and failures gracefully
- Add caching for performance and cost optimization
- Implement citation generation and hallucination detection
- Set up monitoring and observability

---

## Specification

### Research Assistant API (`research-assistant`)

Serve as a FastAPI application with the following endpoints:

```bash
# Research a query
POST /research
{"query": "What is the latest research on quantum computing?"}

# Chat with context (multi-turn)
POST /chat
{"session_id": "abc123", "message": "Tell me more about error correction"}

# Ingest a document
POST /ingest
{"file": "paper.pdf", "source": "uploaded"}

# Get system status
GET /health
```

### Response Format

```json
{
  "answer": "Quantum computing leverages quantum mechanical phenomena...",
  "citations": [
    {
      "id": 1,
      "source": "arXiv:2301.04236",
      "title": "Advances in Quantum Error Correction",
      "relevance": 0.92,
      "url": "https://arxiv.org/abs/2301.04236"
    }
  ],
  "sources_used": 4,
  "sources_total": 10,
  "confidence": 0.85,
  "latency_ms": 2340,
  "cost_usd": 0.0032,
  "tokens_used": {"input": 4200, "output": 650}
}
```

### Features

#### 1. Multi-Source Research
- Web search via SerpAPI or Bing API
- Knowledge base (curated internal documents)
- Academic paper search (arXiv API)
- Fallback sources if primary source fails

#### 2. Smart Retrieval
- Hybrid search (keyword + semantic)
- Source credibility ranking
- Recency weighting
- Deduplication of similar sources
- Minimum relevance threshold

#### 3. Citation Generation
- Inline citations in text [1], [2]
- Full reference list at end
- APA/MLA format (configurable)
- Source type labels (Web, Paper, Internal)

#### 4. Context Management
- Multi-turn conversation support
- Session-based memory
- Automatic history summarization
- Token budget enforcement

#### 5. Caching
- Exact match cache for identical queries (T=0)
- Semantic cache for similar queries (threshold 0.95)
- Cache invalidation by TTL

#### 6. Error Handling
- Retry with exponential backoff
- Fallback search providers
- Graceful degradation (partial results)
- Circuit breaker for failing providers

#### 7. Hallucination Detection
- Citation verification (does source exist?)
- Consistency check (compare multiple sources)
- Confidence scoring
- "I cannot find" responses when appropriate

---

## Implementation Plan

### Phase 1: Core Architecture (Required)

**File: `research_assistant.py`**

```python
#!/usr/bin/env python3
"""
Multi-Source Research Assistant
A production-grade AI system with RAG, caching, and error handling.

Usage:
    uvicorn research_assistant:app --reload
    curl -X POST http://localhost:8000/research -H "Content-Type: application/json" -d '{"query": "..."}'
"""

import os
import time
import json
import hashlib
import asyncio
import logging
from dataclasses import dataclass, field
from typing import Optional
from datetime import datetime, timedelta

import openai
import numpy as np
import httpx
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

# ─── Configuration ────────────────────────────────────────────────────────

@dataclass
class Config:
    openai_api_key: str = os.getenv("OPENAI_API_KEY", "")
    serpapi_key: str = os.getenv("SERPAPI_KEY", "")
    arxiv_enabled: bool = True
    max_sources: int = 8
    min_relevance: float = 0.3
    cache_ttl: int = 3600  # 1 hour
    max_retries: int = 3
    request_timeout: int = 30
    primary_model: str = "gpt-4o-mini"
    heavy_model: str = "gpt-4o"
    budget_per_session: float = 0.50  # $0.50 per session

config = Config()

# ─── Caching ──────────────────────────────────────────────────────────────

class ExactCache:
    def __init__(self, ttl=3600):
        self.cache = {}
        self.ttl = ttl
    
    def get(self, key: str) -> Optional[dict]:
        if key in self.cache:
            entry = self.cache[key]
            if time.time() - entry["timestamp"] < self.ttl:
                return entry["data"]
            del self.cache[key]
        return None
    
    def set(self, key: str, data: dict):
        self.cache[key] = {"data": data, "timestamp": time.time()}
    
    def make_key(self, query: str) -> str:
        return hashlib.md5(query.encode()).hexdigest()

class SemanticCache:
    def __init__(self, threshold=0.95, ttl=3600):
        self.entries = []  # [(embedding, data, timestamp)]
        self.threshold = threshold
        self.ttl = ttl
    
    def get(self, query_emb: list) -> Optional[dict]:
        now = time.time()
        for emb, data, ts in self.entries:
            if now - ts > self.ttl:
                continue
            similarity = np.dot(emb, query_emb) / (
                np.linalg.norm(emb) * np.linalg.norm(query_emb)
            )
            if similarity > self.threshold:
                return data
        return None
    
    def set(self, embedding: list, data: dict):
        self.entries.append((embedding, data, time.time()))

# ─── Source Credibility ───────────────────────────────────────────────────

DOMAIN_RANKINGS = {
    "arxiv.org": 1.0, ".edu": 0.95, ".gov": 0.95,
    "wikipedia.org": 0.85, "nature.com": 0.95, "sciencedirect.com": 0.9,
    "ieee.org": 0.9, "acm.org": 0.9, "medium.com": 0.4,
    "github.com": 0.6, "stackoverflow.com": 0.6,
}

def rank_source(source: dict) -> float:
    url = source.get("url", "")
    domain_score = 0.3
    for domain, score in DOMAIN_RANKINGS.items():
        if domain in url:
            domain_score = score
            break
    
    recency_score = 0.3
    if "date" in source:
        try:
            days_old = (datetime.now() - datetime.fromisoformat(source["date"])).days
            recency_score = max(0, 1 - days_old / 365)
        except:
            pass
    
    relevance_score = source.get("relevance", 0.5)
    
    return 0.4 * domain_score + 0.3 * recency_score + 0.3 * relevance_score

# ─── Search Providers ─────────────────────────────────────────────────────

class SearchProvider:
    async def search(self, query: str) -> list[dict]:
        raise NotImplementedError

class WebSearch(SearchProvider):
    async def search(self, query: str) -> list[dict]:
        async with httpx.AsyncClient(timeout=10) as client:
            resp = await client.get(
                "https://serpapi.com/search",
                params={"q": query, "api_key": config.serpapi_key, "num": 5},
            )
            results = resp.json().get("organic_results", [])
            return [{
                "title": r.get("title", ""),
                "url": r.get("link", ""),
                "snippet": r.get("snippet", ""),
                "source_type": "web",
                "date": r.get("date", ""),
            } for r in results]

class ArxivSearch(SearchProvider):
    async def search(self, query: str) -> list[dict]:
        if not config.arxiv_enabled:
            return []
        async with httpx.AsyncClient(timeout=10) as client:
            resp = await client.get(
                "http://export.arxiv.org/api/query",
                params={
                    "search_query": f"all:{query}",
                    "start": 0,
                    "max_results": 5,
                },
            )
            # Parse Atom XML (simplified)
            import xml.etree.ElementTree as ET
            results = []
            try:
                root = ET.fromstring(resp.text)
                ns = {"atom": "http://www.w3.org/2005/Atom"}
                for entry in root.findall("atom:entry", ns):
                    title = entry.find("atom:title", ns)
                    summary = entry.find("atom:summary", ns)
                    results.append({
                        "title": title.text.strip() if title is not None else "",
                        "url": entry.find("atom:id", ns).text if entry.find("atom:id", ns) is not None else "",
                        "snippet": summary.text[:300] if summary is not None else "",
                        "source_type": "paper",
                        "date": "",
                    })
            except:
                pass
            return results

# ─── Research Engine ──────────────────────────────────────────────────────

class ResearchEngine:
    def __init__(self):
        self.client = openai.OpenAI(api_key=config.openai_api_key)
        self.search_providers = [WebSearch(), ArxivSearch()]
        self.exact_cache = ExactCache(config.cache_ttl)
        self.semantic_cache = SemanticCache()
        self.circuit_breaker = {"failures": 0, "last_failure": 0}
    
    async def research(self, query: str, session_id: Optional[str] = None) -> dict:
        start = time.time()
        
        # Check exact cache
        cache_key = self.exact_cache.make_key(query)
        cached = self.exact_cache.get(cache_key)
        if cached:
            cached["cached"] = True
            cached["latency_ms"] = (time.time() - start) * 1000
            return cached
        
        try:
            # Step 1: Query understanding
            analysis = await self.understand_query(query)
            
            # Step 2: Multi-source search
            all_sources = []
            for provider in self.search_providers:
                try:
                    results = await provider.search(analysis["search_query"])
                    all_sources.extend(results)
                except Exception as e:
                    logging.warning(f"Search provider failed: {e}")
            
            if not all_sources:
                return {
                    "answer": "I could not find any relevant sources to answer this question.",
                    "citations": [],
                    "sources_used": 0,
                    "sources_total": 0,
                    "confidence": 0,
                    "latency_ms": (time.time() - start) * 1000,
                    "cost_usd": 0,
                    "tokens_used": {"input": 0, "output": 0},
                    "error": "No sources found",
                }
            
            # Step 3: Rank and filter sources
            ranked = sorted(all_sources, key=rank_source, reverse=True)
            top_sources = ranked[:config.max_sources]
            
            # Step 4: Deduplicate
            deduped = self.deduplicate(top_sources)
            
            # Step 5: Generate answer with citations
            result = await self.synthesize(query, deduped)
            
            # Step 6: Hallucination check
            result = await self.verify_answer(result)
            
            result["latency_ms"] = (time.time() - start) * 1000
            result["sources_total"] = len(all_sources)
            result["sources_used"] = len(deduped)
            
            # Cache the result
            self.exact_cache.set(cache_key, result)
            
            return result
            
        except Exception as e:
            logging.error(f"Research failed: {e}")
            return {
                "answer": "An error occurred while processing your request.",
                "error": str(e),
                "latency_ms": (time.time() - start) * 1000,
                "cost_usd": 0,
                "confidence": 0,
            }
    
    async def understand_query(self, query: str) -> dict:
        """Analyze and decompose the research query."""
        resp = self.client.chat.completions.create(
            model=config.primary_model,
            messages=[{
                "role": "user",
                "content": f"""Analyze this research query:

Query: {query}

Return JSON with:
- search_query: optimized search query
- sub_questions: list of sub-questions to research
- entities: key entities mentioned
- intent: factual | comparative | exploratory | how_to"""
            }],
            response_format={"type": "json_object"},
            temperature=0,
        )
        return json.loads(resp.choices[0].message.content)
    
    def deduplicate(self, sources: list[dict]) -> list[dict]:
        seen_urls = set()
        deduped = []
        for source in sources:
            url = source.get("url", "")
            if url and url not in seen_urls:
                seen_urls.add(url)
                deduped.append(source)
        return deduped
    
    async def synthesize(self, query: str, sources: list[dict]) -> dict:
        """Generate answer with citations from sources."""
        context = "\n\n".join(
            f"[{i+1}] {s['title']}\nURL: {s.get('url', '')}\n{s.get('snippet', '')}"
            for i, s in enumerate(sources)
        )
        
        resp = self.client.chat.completions.create(
            model=config.heavy_model,
            messages=[{
                "role": "system",
                "content": f"""You are a research assistant. Synthesize an answer using ONLY the provided sources.

Rules:
1. Use inline citations [1], [2], etc. for every factual claim
2. If sources disagree, note the disagreement
3. If the answer isn't in the sources, say "I cannot find sufficient information"
4. Include a References section at the end
5. Be objective and balanced"""
            }, {
                "role": "user",
                "content": f"Query: {query}\n\nSources:\n{context}"
            }],
            temperature=0,
        )
        
        content = resp.choices[0].message.content
        usage = resp.usage
        
        # Verify citations exist in sources
        citations = self.extract_citations(content, sources)
        
        return {
            "answer": content,
            "citations": citations,
            "confidence": min(0.9, 0.5 + 0.1 * len(citations)),
            "cost_usd": (usage.prompt_tokens * 2.50 + usage.completion_tokens * 10.00) / 1_000_000,
            "tokens_used": {
                "input": usage.prompt_tokens,
                "output": usage.completion_tokens,
            },
        }
    
    def extract_citations(self, answer: str, sources: list[dict]) -> list[dict]:
        import re
        refs = re.findall(r'\[(\d+)\]', answer)
        citations = []
        seen = set()
        for ref in refs:
            idx = int(ref) - 1
            if idx < len(sources) and idx not in seen:
                seen.add(idx)
                source = sources[idx]
                citations.append({
                    "id": idx + 1,
                    "source": source.get("url", ""),
                    "title": source.get("title", ""),
                    "relevance": rank_source(source),
                })
        return citations
    
    async def verify_answer(self, result: dict) -> dict:
        """Check for hallucination using a second LLM call."""
        resp = self.client.chat.completions.create(
            model=config.primary_model,
            messages=[{
                "role": "user",
                "content": f"""Fact-check this answer. Identify any claims NOT supported by citations.

Answer: {result['answer']}

Return JSON:
{{"verified": true/false, "issues": [], "confidence_score": 0.85}}"""
            }],
            response_format={"type": "json_object"},
            temperature=0,
        )
        verification = json.loads(resp.choices[0].message.content)
        
        if not verification.get("verified", True):
            result["confidence"] = min(result["confidence"], verification.get("confidence_score", 0.5))
            result["warnings"] = verification.get("issues", [])
        
        return result

# ─── FastAPI Application ──────────────────────────────────────────────────

app = FastAPI(title="Multi-Source Research Assistant")
engine = ResearchEngine()

class ResearchRequest(BaseModel):
    query: str
    session_id: Optional[str] = None

class ResearchResponse(BaseModel):
    answer: str
    citations: list = []
    sources_used: int = 0
    sources_total: int = 0
    confidence: float = 0
    latency_ms: float = 0
    cost_usd: float = 0
    tokens_used: dict = {}
    cached: bool = False
    warnings: list = []
    error: Optional[str] = None

@app.post("/research", response_model=ResearchResponse)
async def research_endpoint(req: ResearchRequest):
    if not req.query:
        raise HTTPException(status_code=400, detail="Query is required")
    return await engine.research(req.query, req.session_id)

@app.get("/health")
async def health():
    return {
        "status": "healthy",
        "cache_size": len(engine.exact_cache.cache),
        "model": config.primary_model,
    }

# ─── Main ─────────────────────────────────────────────────────────────────

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

### Phase 2: Session Management (Required)

Build session-based multi-turn conversation support:

```python
# session_manager.py
import uuid
import time
import tiktoken
from typing import List, Dict

enc = tiktoken.encoding_for_model("gpt-4o")

class Session:
    def __init__(self, max_context_tokens=32000):
        self.id = str(uuid.uuid4())
        self.history: List[Dict] = []
        self.max_context = max_context_tokens
        self.created_at = time.time()
        self.last_active = time.time()
        self.cost_spent = 0.0
    
    def add_message(self, role: str, content: str):
        self.history.append({"role": role, "content": content, "timestamp": time.time()})
        self.last_active = time.time()
        self._prune_if_needed()
    
    def _prune_if_needed(self):
        total = sum(len(enc.encode(m["content"])) for m in self.history)
        while total > self.max_context and len(self.history) > 2:
            removed = self.history.pop(0)
            total -= len(enc.encode(removed["content"]))

class SessionManager:
    def __init__(self, session_timeout=3600):
        self.sessions: Dict[str, Session] = {}
        self.timeout = session_timeout
    
    def get_or_create(self, session_id: Optional[str] = None) -> Session:
        if session_id and session_id in self.sessions:
            session = self.sessions[session_id]
            if time.time() - session.last_active < self.timeout:
                return session
        session = Session()
        self.sessions[session.id] = session
        return session
    
    def cleanup(self):
        now = time.time()
        expired = [sid for sid, s in self.sessions.items() if now - s.last_active > self.timeout]
        for sid in expired:
            del self.sessions[sid]
```

### Phase 3: Dashboard & Monitoring (Optional but Recommended)

Add a monitoring dashboard:

```python
# monitor.py
import time
from collections import deque
from dataclasses import dataclass

@dataclass
class RequestMetrics:
    timestamp: float
    query: str
    latency_ms: float
    cost_usd: float
    confidence: float
    sources_used: int
    success: bool
    cached: bool

class Monitor:
    def __init__(self, window_minutes=60):
        self.metrics = deque(maxlen=10000)
        self.window = window_minutes * 60
    
    def record(self, metrics: RequestMetrics):
        self.metrics.append(metrics)
    
    def summary(self) -> dict:
        recent = [m for m in self.metrics if time.time() - m.timestamp < self.window]
        
        if not recent:
            return {"status": "no_data"}
        
        latencies = [m.latency_ms for m in recent]
        costs = [m.cost_usd for m in recent]
        success_rate = sum(1 for m in recent if m.success) / len(recent)
        cache_rate = sum(1 for m in recent if m.cached) / len(recent)
        
        return {
            "requests": len(recent),
            "p50_latency_ms": sorted(latencies)[len(latencies) // 2],
            "p95_latency_ms": sorted(latencies)[int(len(latencies) * 0.95)],
            "avg_cost": sum(costs) / len(costs),
            "total_cost": sum(costs),
            "success_rate": success_rate,
            "cache_hit_rate": cache_rate,
            "avg_confidence": sum(m.confidence for m in recent if m.success) / max(sum(1 for m in recent if m.success), 1),
        }
```

### Phase 4: Advanced Features (Optional)

Add these features to extend the system:

**1. Rate Limiting:**
```python
from fastapi import Request, HTTPException
import asyncio

rate_limits = {}

async def rate_limit(request: Request):
    client_ip = request.client.host
    now = time.time()
    
    if client_ip not in rate_limits:
        rate_limits[client_ip] = []
    
    rate_limits[client_ip] = [t for t in rate_limits[client_ip] if now - t < 60]
    
    if len(rate_limits[client_ip]) >= 20:  # 20 requests per minute
        raise HTTPException(status_code=429, detail="Rate limit exceeded")
    
    rate_limits[client_ip].append(now)
```

**2. Feedback Collection:**
```python
@app.post("/feedback")
async def feedback(fb: Feedback):
    """Collect user feedback on responses."""
    # Store feedback for analysis
    # Use for evaluation and fine-tuning
    pass
```

**3. Source Content Extraction:**
```python
async def extract_content(url: str) -> str:
    """Fetch and extract main content from a URL."""
    async with httpx.AsyncClient() as client:
        resp = await client.get(url, timeout=10)
        # Use newspaper3k or readability for content extraction
        from newspaper import Article
        article = Article(url)
        article.download()
        article.parse()
        return article.text
```

**4. Parallel Search with Timeout:**
```python
async def search_all_providers(query: str) -> list[dict]:
    tasks = []
    for provider in search_providers:
        task = asyncio.create_task(provider.search(query))
        tasks.append(task)
    
    done, pending = await asyncio.wait(
        tasks, timeout=15.0,
        return_when=asyncio.ALL_COMPLETED,
    )
    
    # Cancel remaining
    for task in pending:
        task.cancel()
    
    results = []
    for task in done:
        try:
            results.extend(task.result())
        except:
            pass
    return results
```

---

## Testing

```python
# test_research_assistant.py
import pytest
from httpx import AsyncClient, ASGITransport
from research_assistant import app, ResearchEngine

@pytest.fixture
def client():
    transport = ASGITransport(app=app)
    return AsyncClient(transport=transport, base_url="http://test")

@pytest.mark.asyncio
async def test_research_endpoint(client):
    response = await client.post("/research", json={"query": "What is Python?"})
    assert response.status_code == 200
    data = response.json()
    assert "answer" in data
    assert len(data["answer"]) > 0

@pytest.mark.asyncio
async def test_empty_query(client):
    response = await client.post("/research", json={"query": ""})
    assert response.status_code == 400

@pytest.mark.asyncio
async def test_health(client):
    response = await client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"

class TestResearchEngine:
    def setup_method(self):
        self.engine = ResearchEngine()
    
    def test_deduplicate(self):
        sources = [
            {"url": "https://example.com/1", "title": "A"},
            {"url": "https://example.com/1", "title": "B"},  # duplicate
            {"url": "https://example.com/2", "title": "C"},
        ]
        deduped = self.engine.deduplicate(sources)
        assert len(deduped) == 2
        assert deduped[0]["url"] == "https://example.com/1"
    
    def test_rank_source(self):
        source = {"url": "https://arxiv.org/abs/2301.04236", "relevance": 0.9}
        score = rank_source(source)
        assert 0 <= score <= 1
        assert score > 0.8  # arXiv should score high
    
    def test_extract_citations(self):
        answer = "According to recent research [1], quantum computing is advancing rapidly [2]."
        sources = [
            {"url": "https://arxiv.org/abs/2301.04236", "title": "Paper 1"},
            {"url": "https://arxiv.org/abs/2302.04236", "title": "Paper 2"},
        ]
        citations = self.engine.extract_citations(answer, sources)
        assert len(citations) == 2
        assert citations[0]["id"] == 1
```

---

## Extension Ideas

| Feature | Difficulty | Description |
|---------|------------|-------------|
| Multi-language support | Medium | Translate queries and sources |
| PDF ingestion | Medium | Parse uploaded PDFs as sources |
| Custom source weighting | Easy | Allow users to prioritize certain sources |
| Export citations | Easy | Export in BibTeX, RIS, or Zotero format |
| Answer comparison | Medium | Show multiple perspectives on controversial topics |
| Trend analysis | Hard | Track how answers change over time (source updates) |
| Collaborative research | Hard | Multiple users share sessions and sources |
| Voice interface | Medium | Add STT input and TTS output |
| Source summarization | Easy | Generate concise summaries of each source |
| Bias detection | Hard | Identify potential bias in sources and answers |

---

## Deliverables

1. **`research_assistant.py`** — FastAPI application with core research engine
2. **`session_manager.py`** — Session management for multi-turn conversations
3. **`monitor.py`** — Monitoring and metrics collection
4. **`test_research_assistant.py`** — Unit tests with >80% coverage
5. **`requirements.txt`** — Dependencies
6. **`README.md`** (in project directory) — Setup, API docs, architecture overview

### Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Correctness | 25% | Accurate answers with proper citations |
| Architecture | 25% | Clean separation of concerns, modular design |
| Error handling | 15% | Graceful degradation, retries, fallbacks |
| Performance | 15% | Caching, parallel searches, latency optimization |
| Testing | 10% | Comprehensive test coverage |
| Monitoring | 5% | Metrics, logging, observability |
| Extensions | 5% | Bonus features implemented |

---

## Submission

Zip the project directory and submit. Include:

```
requirements.txt:
openai>=1.0.0
fastapi>=0.104.0
uvicorn>=0.24.0
httpx>=0.25.0
numpy>=1.24.0
tiktoken>=0.5.0
pytest>=7.4.0
pydantic>=2.0.0
sse-starlette>=1.8.0
```

---

## Hints

1. **Start simple**: Get the basic FastAPI server running with a hardcoded response before adding search
2. **Test with mock data**: Use fake search results during development to avoid API costs
3. **Cache aggressively**: The exact cache will save you time and money during testing
4. **Handle failures early**: A failing search provider should never crash the entire request
5. **Use async throughout**: httpx.AsyncClient, asyncio.gather for parallel searches
6. **Monitor costs**: Track both token usage and dollar cost of every request
7. **Session cleanup**: Implement periodic cleanup of expired sessions to prevent memory leaks
8. **Log everything**: Every search, every LLM call, every cache hit/miss should be logged
