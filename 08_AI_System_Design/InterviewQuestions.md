# Interview Questions — Chapter 08: AI System Design

## Beginner Questions

### Q1: What are the key differences between a traditional software system and an AI system?

**Answer:** AI systems have several unique characteristics:

| Aspect | Traditional | AI System |
|--------|-------------|-----------|
| Output correctness | Deterministic (pass/fail) | Probabilistic (continuous quality) |
| Failure modes | Crash, wrong logic | Hallucination, plausible but wrong |
| Testing | Assert exact output | Evaluate on distribution, no ground truth |
| Cost | Compute + storage | Per-token LLM costs dominate |
| Latency | Predictable | Variable (depends on output length) |
| Security | SQL injection, XSS | Prompt injection, jailbreaks |
| Observability | Stack traces, logs | Need to trace reasoning paths |

**Key insight:** AI systems are traditional distributed systems with an LLM as a new, unreliable compute primitive. All the standard principles (reliability, scalability, maintainability) still apply, plus new concerns specific to LLMs.

### Q2: How would you design a chatbot that remembers past conversations?

**Answer:** Implement a multi-tier memory system:

1. **Ephemeral memory** (in-memory, current session): Stores recent messages for immediate context
2. **Working memory** (Redis/TTL): Last N sessions, auto-expiring
3. **Long-term memory** (Database): User preferences, key facts extracted from conversations
4. **Summary memory** (LLM-generated): Compressed history for very long conversations

When the context window is full:
- Summarize oldest messages into a compressed form
- Store the summary in long-term memory
- Retrieve relevant memories via embedding similarity for new conversations

**Key challenge:** Balancing between providing enough context and staying within token budgets.

### Q3: What is the difference between stateless and stateful chatbot architectures?

**Answer:**

| Aspect | Stateless | Stateful |
|--------|-----------|----------|
| Context | Sent with every request | Stored server-side |
| Scaling | Simple round-robin (any server) | Session affinity needed |
| Failure recovery | Transparent (retry any server) | Session lost on failure |
| Request size | Grows with conversation length | Fixed size (just query) |
| Implementation | Stateless HTTP | WebSocket or persistent connection |
| Use case | Simple Q&A, API integrations | Long conversations, complex assistants |

**Best practice:** Hybrid approach — store session metadata server-side but manage context window client-side. Use a session ID to retrieve long-term memory on demand.

### Q4: Explain the concept of context window management.

**Answer:** Context window management is the process of deciding what fits in the LLM's limited context (e.g., 128K tokens). Key strategies:

1. **Token budgeting**: Allocate fixed token counts for system prompt, conversation history, retrieved docs, etc.
2. **Pruning**: Remove oldest/densest messages first when over budget
3. **Summarization**: Compress multiple messages into a short summary
4. **Retrieval**: Only include relevant context (not everything)
5. **Sliding window**: Keep the most recent N messages and summarize the rest

```
Budget allocation example:
System prompt:    500 tokens  (0.4%)
Conversation:   30,000 tokens (23.4%)
Retrieved docs: 20,000 tokens (15.6%)
Safety margin:  77,500 tokens (60.6%) ← For future conversation turns
```

### Q5: What is streaming and why is it important in AI systems?

**Answer:** Streaming sends LLM output token-by-token as it's generated, rather than waiting for the complete response. It's important because:

1. **Perceived latency**: Users see content in 200-500ms instead of waiting 2-10s for full response
2. **Progressive rendering**: UI can show content as it arrives
3. **Early cancellation**: Users can stop generation mid-response
4. **Real-time experience**: Essential for chat, voice, and interactive applications

**Implementation:** Server-Sent Events (SSE) or WebSockets. The backend sends each token as a separate event, and the frontend appends tokens to the UI.

---

## Intermediate Questions

### Q6: Design a customer support system that uses AI for auto-responses but escalates to humans when needed.

**Answer:**
```
Architecture:
1. Ticket Ingestion → 2. Classification → 3. Sentiment Analysis →
4. Knowledge Retrieval → 5. Response Generation → 6. Confidence Check →
7. Auto-reply (high confidence) or Escalate (low confidence)
```

**Key components:**
- **Classifier**: Routes to billing, technical, account, or general
- **Sentiment detector**: Flags angry/frustrated customers for priority
- **Knowledge retriever**: Semantic search over support docs
- **Response generator**: LLM generates response using retrieved knowledge
- **Confidence scorer**: Ensures response quality exceeds threshold (e.g., 0.7)
- **Escalation manager**: Routes low-confidence, angry, or sensitive tickets to humans

**Escalation triggers:**
- Response confidence < threshold
- Sentiment = angry/frustrated
- Keyword match: "refund", "legal", "cancel", "manager"
- VIP customer tier
- Multi-failure: same issue escalated twice

### Q7: How would you build a research assistant that provides answers with verifiable citations?

**Answer:**

```mermaid
graph LR
    Query → Analyze → Search → Rank → Extract → Synthesize → Cite
```

1. **Query analysis**: Decompose into sub-questions, identify entities
2. **Multi-source search**: Web search, knowledge base, academic papers
3. **Source ranking**: Score by credibility (domain authority), recency, relevance
4. **Content extraction**: Extract relevant passages from top sources
5. **Synthesis**: LLM generates answer using ONLY extracted content
6. **Citation**: Inline citations [1], [2] with full references at bottom

**Key design decisions:**
- Minimum relevance threshold for retrieval (no threshold = hallucination risk)
- Source credibility scoring (.edu/.gov > .com > blog > forum)
- Dedicated citation formatting stage (APA/MLA per user preference)
- "I cannot find information about this" as a valid answer
- Cross-validation: fact-checker LLM verifies claims against sources

### Q8: Design a system that converts natural language questions into SQL queries.

**Answer:**
```
Components:
1. Schema Understanding → 2. SQL Generation → 3. Validation →
4. Execution → 5. Self-Correction → 6. Result Interpretation → 7. Visualization
```

**Safety mechanisms:**
- Only allow SELECT statements (block INSERT, UPDATE, DELETE, DROP)
- Add LIMIT to all queries
- Validate syntax before execution
- Execute in read-only connection
- Log all generated SQL for audit

**Schema understanding:**
- Maintain schema embeddings for relevant table identification
- Include foreign key relationships in context
- Add sample rows for column value understanding
- Filter schema to only relevant tables based on question

**Self-correction:**
```
SQL → Execute → Error? → LLM fixes SQL → Re-execute (max 3 times)
```

**Visualization selection:**
- Analyze result structure to determine chart type
- Bar chart for categorical comparisons
- Line chart for time series
- Pie chart for distributions
- Table for detailed data

### Q9: What are the four fundamental architecture patterns for AI systems? When would you use each?

**Answer:**

| Pattern | Mental Model | Best For | Trade-off |
|---------|-------------|----------|-----------|
| **Orchestrator** | One brain, many hands | Complex reasoning (coding agent) | Cost, latency |
| **Pipeline** | Assembly line | Fixed workflows (document Q&A) | Rigidity |
| **Router** | Switchboard | Heterogeneous requests (support) | Classification accuracy |
| **Ensemble** | Wisdom of crowds | High-stakes decisions | 3-5x cost |

**Decision tree:**
1. Fixed workflow? → Pipeline
2. Multiple request types? → Router
3. Need reliability above all? → Ensemble
4. Complex adaptive reasoning? → Orchestrator

### Q10: How would you handle LLM failures in production?

**Answer:** Multi-layer resilience:

1. **Retry with backoff**: Exponential backoff (2^n + jitter) for transient failures (rate limits, 5xx)
2. **Fallback models**: Cheaper model → more expensive model → cached response
3. **Circuit breaker**: After N failures, skip provider for M minutes
4. **Caching**: T=0 responses are deterministic and cachable (exact + semantic)
5. **Graceful degradation**: If LLM unavailable, use template responses or queue requests
6. **Monitoring**: Alert on error rate > 2%, P95 latency > 5s

```python
fallback_chain = [
    ("gpt-4o-mini", 0.15),   # Cheap, try first
    ("gpt-4o", 2.50),        # Powerful, try second
    ("claude-sonnet", 3.00), # Different provider, try third
    ("cached", 0.0),         # Best-effort cached response
]
```

### Q11: Design a system that can answer questions about documents (PDF, Word, images).

**Answer:**

```mermaid
graph LR
    Document → Parse → Extract (text, tables, images) →
    Chunk → Embed → Index → Retrieve → Rerank → Answer
```

**Document processing pipeline:**
1. **Parse**: Extract text from PDF/Word, handle scanned docs with OCR
2. **Layout analysis**: Identify headings, paragraphs, tables, images
3. **Table extraction**: Convert tables to markdown, preserve structure
4. **Image understanding**: Caption images using vision LLM
5. **Chunking**: Split by section boundary with overlap (10-20%)
6. **Indexing**: Embed chunks with metadata (page number, section, doc name)

**Retrieval:**
- Hybrid search (keyword + semantic) for better recall
- Rerank top-20 results with cross-encoder
- Filter by document source or date

**Answer generation:**
- Include source citations with page numbers
- "I cannot find this in the documents" when no relevant content
- Quote relevant passages verbatim when appropriate

### Q12: How do you estimate and control costs in an AI system?

**Answer:** Costs are dominated by LLM API calls. Key strategies:

**Cost estimation:**
```
Cost per request = (input_tokens × input_price + output_tokens × output_price) / 1,000,000
Monthly cost = daily_queries × cost_per_request × 30
```

**Cost control mechanisms:**

1. **Model routing**: Use cheap model for 80% of queries
2. **Caching**: T=0 responses are deterministic; semantic cache catches similar queries
3. **Context compression**: Remove unnecessary tokens from context
4. **Output length limits**: Set strict max_tokens
5. **Budget guard**: Reject or downgrade requests when daily budget exceeded
6. **Monitoring**: Real-time cost tracking with alerts at 50/80/100% of budget
7. **Batch processing**: Combine multiple requests into one API call

**Example savings:** Routing 80% of queries to GPT-4o-mini ($0.0006/req) vs GPT-4o ($0.01/req) saves ~95% on 80% of traffic.

---

## Advanced Questions

### Q13: Design a coding agent that can understand a large codebase and make multi-file changes.

**Answer:**

**Architecture:**
```mermaid
graph TB
    Task → ProblemAnalyzer → RepoExplorer → CodeGraph →
    ArchitecturePlanner → ChangeSpec → CodeGenerator →
    FileWriter → TestRunner → Debugger → CodeReviewer → Output
```

**Key components:**

1. **Repo Explorer**: Builds a structured understanding of the codebase:
   - File tree with module relationships
   - Dependency graph between files
   - Entry points and configuration files
   - Recent git history for context
   - Test file mappings

2. **Change Planner**: Determines what files to create/modify/delete:
   - Uses repo structure to plan minimal changes
   - Identifies all files that need updates
   - Handles imports and exports across files

3. **Code Generator**: Generates code per file:
   - Reads existing files for context (not full files, just relevant sections)
   - Generates complete file content for new files
   - Generates surgical diffs for existing files

4. **Validation Loop**:
   - Syntax check (AST parse for Python, tsc for TypeScript)
   - Lint check (flake8, ESLint)
   - Test runner (pytest, jest)
   - Debug loop: run → fail → fix (max 3 iterations)

**Storage**: Use a file-based approach. Never hold the entire codebase in context. Read files on demand using semantic search over file contents.

**Multi-file editing**: Generate an edit plan (list of file paths + operations) before generating any code. Apply changes transactionally (all succeed or all rollback).

### Q14: How would you build a voice AI system with low-latency requirements?

**Answer:**

**Latency budget (target: < 1.2s end-to-end):**
- Voice Activity Detection: 200ms
- Speech-to-Text: 300ms
- LLM first token: 300ms
- Text-to-Speech: 200ms
- Network: 100ms
- Buffer: 100ms

**Architecture:**

1. **VAD (Voice Activity Detection)**: Silero or WebRTC VAD, detects when user starts/stops speaking
2. **STT (Speech-to-Text)**: Whisper API or faster-distil-whisper, streaming transcription
3. **Endpointing**: Detect when user has finished speaking (silence > 500ms)
4. **LLM**: Process text and start streaming response immediately
5. **TTS (Text-to-Speech)**: ElevenLabs or Cartesia for low-latency, stream audio as tokens arrive
6. **Turn Management**: Coordinate who speaks when

**Interruption handling:**
- If user speaks while AI is responding, stop TTS immediately
- Do not process the partial AI response as context
- The new user utterance becomes the new query

**Optimizations:**
- Pre-warm connections to all services
- Use WebSocket for persistent connections
- Stream audio in chunks (don't wait for full TTS)
- Cache common phrases (greetings, confirmations)
- Use smaller, faster models for VAD and STT

### Q15: Design an A/B testing framework for AI systems.

**Answer:**

**Challenges specific to AI A/B testing:**
- Output is non-deterministic (even at T=0, due to sampling)
- Quality is subjective and continuous
- Different prompts = different behavior (hard to isolate variables)
- Cost varies between test and control

**Framework components:**

```python
class ABTest:
    def __init__(self, name, control_fn, variant_fn, metrics):
        self.name = name
        self.control = control_fn
        self.variant = variant_fn
        self.metrics = metrics  # e.g., ["latency", "cost", "user_satisfaction", "accuracy"]
        self.allocations = {"control": 0.5, "variant": 0.5}
    
    def route(self, request):
        if random.random() < self.allocations["control"]:
            return "control", self.control(request)
        return "variant", self.variant(request)
```

**What to test:**
1. **Model comparison**: GPT-4o-mini vs GPT-4o for the same task
2. **Prompt variations**: Different system prompts, few-shot examples
3. **Architecture patterns**: Pipeline vs orchestrator for the same task
4. **Chunking strategies**: Chunk size, overlap, strategy
5. **Temperature settings**: T=0 vs T=0.3 for creative tasks

**Metrics to track:**
- Quality score (LLM-as-judge or human evaluation)
- Latency (P50, P95)
- Cost per request
- Error rate
- User satisfaction (thumbs up/down)
- Hallucination rate

**Statistical significance:**
- Use chi-squared test for categorical metrics
- Use t-test for continuous metrics
- Run until statistical power > 0.8
- Monitor for Simpson's paradox (segmentation effects)

### Q16: How do you handle hallucinations in production AI systems?

**Answer:** Multi-layer defense:

**1. Input layer:**
- Clear system prompt: "Only answer if you are certain. Say 'I don't know' otherwise."
- Add strict guardrails and output constraints

**2. Retrieval layer:**
- Ground every response in retrieved documents (RAG)
- Set minimum relevance threshold for retrieval
- If no relevant docs found, refuse to answer

**3. Generation layer:**
- Use low temperature (T=0) for factual queries
- Require citations in structured format
- Limit output length

**4. Verification layer:**
- **Self-verification**: Ask LLM to verify its own answer
- **Cross-verification**: Use a second LLM call to fact-check
- **Tool verification**: Check claims against web search or knowledge base
- **Consistency check**: Ask the same question in different ways, compare answers

**5. Output layer:**
- Parse citations and verify they exist in the context
- Check for logical consistency
- Reject responses that make unsupported claims

**Detection metrics:**
- Citation confidence (are sources cited appropriately?)
- Factual consistency score (LLM-as-judge comparing claims to sources)
- User feedback (correction rate)

### Q17: Design a content generation system that maintains consistent brand voice.

**Answer:**

**Pipeline:**
```
Brief → Research → Outline → Draft → Style Check → Fact Check → 
Edit Pass 1 → Edit Pass 2 → Edit Pass 3 → Final Review → Format → Publish
```

**Brand voice enforcement:**

1. **Style guide as structured data:**
```python
STYLE_GUIDE = {
    "voice": {
        "tone": "Professional but approachable",
        "perspective": "Second person (you/your)",
        "contractions": "Avoid",
        "jargon": "Define on first use"
    },
    "formatting": {
        "max_paragraph_sentences": 3,
        "heading_case": "Sentence case",
        "bullet_style": "Capitalize, no period"
    },
    "vocabulary": {
        "preferred": {"use": "utilize", "help": "assist"},
        "forbidden": ["revolutionary", "game-changing", "synergy"]
    }
}
```

2. **Multi-pass editing**: Each pass has a different focus:
   - Pass 1: Grammar, clarity, sentence structure
   - Pass 2: Brand voice, tone consistency
   - Pass 3: Flow, transitions, call-to-action

3. **Automated checks**: After generation, validate:
   - No forbidden terms present
   - Tone matches target (via LLM-as-judge)
   - Structure matches outline
   - Reading level appropriate for audience

4. **Human-in-the-loop**: Optional review gates at outline, draft, and final stages

### Q18: How would you design a caching system for LLM responses?

**Answer:** Multi-level caching:

**Level 1 — Exact match cache (In-memory, Redis):**
- Key: Hash of (prompt + temperature + model)
- TTL: 5-60 minutes (depends on volatility)
- Hit rate: ~20% for production workloads
- Only for T=0 requests (deterministic)

**Level 2 — Semantic cache (Vector DB):**
- Embed the query, find similar embedded queries
- Threshold: cosine similarity > 0.95
- Only return cached response if highly confident
- Handles: "What's the weather in NYC?" and "Weather New York City"
- Must verify semantic equivalence before returning

**Level 3 — Prefix cache (Provider built-in):**
- Repeated system prompts get automatic caching
- OpenAI/KV cache: 50% discount on cached tokens
- Anthropic prompt caching: automatic for repeated prefixes
- No implementation needed, just use the provider's feature

**Invalidation strategy:**
- Time-based TTL for all caches
- Explicit invalidation for specific content types
- No invalidation for system-level caches (model updates handled by providers)

**Cache warming:**
- Pre-compute responses for common queries during off-peak
- Seed cache with expected queries from traffic analysis

### Q19: Build a monitoring and observability system for an AI application.

**Answer:**

**Three pillars of AI observability:**

**1. Tracing (why did this happen?)**
- Trace ID per request, propagated across all components
- Span per: retrieval, LLM call, tool execution, validation
- Record: input, output, latency, cost, model used

```python
{
    "trace_id": "abc123",
    "spans": [
        {"name": "retrieval", "duration_ms": 150, "input": "query", "output": ["chunk1", "chunk2"]},
        {"name": "llm_call", "duration_ms": 1200, "model": "gpt-4o-mini", 
         "input_tokens": 2500, "output_tokens": 300, "cost_usd": 0.002},
        {"name": "validation", "duration_ms": 50, "passed": True},
    ],
    "total_duration_ms": 1400,
    "total_cost_usd": 0.002,
    "user_satisfaction": "positive",
}
```

**2. Metrics (what is happening?)**
- Request volume, latency (P50/P95/P99), error rate
- Cost per request, daily/monthly spend
- Cache hit rate, retrieval relevance scores
- Hallucination rate, escalation rate
- User satisfaction score

**3. Logging (what happened?)**
- Every LLM call logged with: timestamp, model, tokens, latency, cost
- Every retrieval logged with: query, top_k results, scores
- Every error logged with: stack trace, context, recovery action

**Alerting thresholds:**
- P95 latency > 5s
- Error rate > 2%
- Cost spike > 20% above baseline
- Hallucination rate > 5%
- User satisfaction drop > 10%

### Q20: Design a multi-tenant AI system with per-customer customization.

**Answer:**

**Architecture:**
```
Gateway → Tenant Router → Custom Prompt Loader → LLM → Response
```

**Per-tenant configuration:**
```python
TENANT_CONFIGS = {
    "acme_corp": {
        "system_prompt": "You are Acme Corp's support agent...",
        "knowledge_base_id": "kb_acme",
        "model": "gpt-4o-mini",
        "rate_limit": 100,  # requests/minute
        "custom_tools": ["check_inventory", "create_order"],
        "budget_limit": 500,  # $/month
    },
}
```

**Isolation strategies:**
1. **Logical isolation** (shared infrastructure, per-tenant prompts/KB): Most cost-effective
2. **Physical isolation** (dedicated model deployment): For compliance-heavy tenants
3. **Hybrid**: Shared cheap model for simple queries, dedicated for complex

**Customization points:**
- System prompt (per-tenant brand voice, policies)
- Knowledge base (per-tenant documents)
- Tools/Function calling (per-tenant APIs)
- Model selection (tenant chooses price/quality trade-off)
- Rate limits and budgets (per-tenant)

**Data isolation:**
- Tenant ID in all database queries
- Encrypted per-tenant vector stores
- No cross-tenant data leakage in prompts
- Separate audit logs per tenant

**Challenges:**
- Context window: cannot fit all tenants' data equally
- Cost attribution: which tenant consumed what
- Model versioning: tenant pinned to specific model version
- Cold start: first request for a tenant needs prompt building
