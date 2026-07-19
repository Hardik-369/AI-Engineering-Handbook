# Exercises — Chapter 08: AI System Design

## Beginner Exercises

### Exercise 1: Context Budget Calculator

Build a function that calculates token utilization for a chatbot session:

```python
def calculate_context_budget(system_prompt: str, history: list[dict],
                              retrieved_docs: list[str], max_context: int) -> dict:
    """
    Calculate how much of the context window is used and by what.
    history: [{"role": "user", "content": "..."}, ...]
    Returns dict with usage breakdown.
    """
    import tiktoken
    enc = tiktoken.encoding_for_model("gpt-4o")
    
    # Your code here
    pass
```

**Tasks:**
1. Calculate tokens for each component: system prompt, history, docs
2. Compute percentage utilization per component
3. Determine which component to prune first when over budget
4. Implement a `prune_to_fit()` method that removes oldest history first
5. Test with a history of 50 messages and max_context of 4000 tokens

### Exercise 2: Simple Model Router

Implement a router that classifies queries and dispatches to different models:

```python
def route_query(query: str) -> dict:
    """Route query to appropriate model based on complexity."""
    categories = {
        "simple": {"model": "gpt-4o-mini", "reason": "greeting/simple Q&A"},
        "complex": {"model": "gpt-4o", "reason": "needs analysis"},
        "code": {"model": "gpt-4o", "reason": "code generation"},
        "creative": {"model": "gpt-4o", "reason": "creative writing"},
    }
    
    # Your code here
    # Use an LLM to classify, then return the routing decision
    
    pass

# Test with:
test_queries = [
    "What's the weather like today?",
    "Write a Python function to sort a list of dictionaries by multiple keys",
    "Explain the implications of quantum computing on modern cryptography",
    "Write a poem about artificial intelligence",
]
```

**Tasks:**
1. Use an LLM call to classify each query
2. Route to the appropriate model
3. Estimate the cost savings vs using the most expensive model for everything
4. Add a fallback: if classification confidence < 0.7, use the expensive model

### Exercise 3: Streaming Response Handler

Create a wrapper that converts non-streaming LLM calls to streaming:

```python
def simulate_stream(full_response: str, chunk_size: int = 5):
    """Simulate streaming by yielding chunks of a complete response."""
    # Your code here
    pass

class StreamingWrapper:
    def __init__(self, client, model):
        self.client = client
        self.model = model
    
    def stream(self, messages):
        """
        Always return an iterator, even if the underlying API 
        doesn't support streaming (simulate it).
        """
        # Your code here
        pass

# Test:
wrapper = StreamingWrapper(client, "gpt-4o-mini")
for chunk in wrapper.stream([{"role": "user", "content": "Write a short story"}]):
    print(chunk, end="", flush=True)
```

**Tasks:**
1. Implement simulation for non-streaming APIs
2. Track per-chunk latency and report tokens/second
3. Add a backpressure mechanism: slow down if consumer is slow
4. Implement abort: allow cancelling mid-stream

### Exercise 4: Simple RAG Pipeline

Implement a minimal RAG pipeline without a vector database:

```python
class SimpleRAG:
    def __init__(self, documents: list[str]):
        """Store documents and compute embeddings."""
        # Your code here
        pass
    
    def retrieve(self, query: str, top_k: int = 3) -> list[str]:
        """Find most relevant documents using cosine similarity."""
        # Your code here
        pass
    
    def generate(self, query: str, context: list[str]) -> str:
        """Generate answer using retrieved context."""
        # Your code here
        pass
    
    def query(self, query: str) -> str:
        """End-to-end RAG query."""
        # Your code here
        pass

# Test with these documents
docs = [
    "Python is a high-level programming language created by Guido van Rossum.",
    "Python supports multiple programming paradigms including procedural and OOP.",
    "The Python package manager is called pip.",
    "Python 3 was released in 2008, Python 2 was deprecated in 2020.",
    "Popular Python frameworks include Django, Flask, and FastAPI.",
]

rag = SimpleRAG(docs)
print(rag.query("Who created Python?"))
print(rag.query("What package manager does Python use?"))
```

**Tasks:**
1. Implement with `text-embedding-3-small` from OpenAI
2. Add a relevance threshold (return None if no doc scores above threshold)
3. Implement hybrid search: keyword overlap + embedding similarity
4. Add source citation in the response

### Exercise 5: Error Handling with Fallbacks

Implement a resilient LLM caller with multiple fallback strategies:

```python
class ResilientLLM:
    def __init__(self):
        self.providers = [
            {"name": "openai", "model": "gpt-4o-mini", "client": openai.OpenAI()},
            {"name": "openai", "model": "gpt-4o", "client": openai.OpenAI()},
            # Add fallback providers
        ]
    
    def call_with_fallback(self, messages, max_retries=3):
        """Try providers in order, with retries on failure."""
        # Your code here
        pass
    
    def call_with_timeout(self, messages, timeout=10):
        """Call LLM with a timeout. Return error response on timeout."""
        # Your code here
        pass

# Test:
llm = ResilientLLM()
result = llm.call_with_fallback([{"role": "user", "content": "Hello"}])
print(result)
```

**Tasks:**
1. Implement exponential backoff between retries
2. Add a circuit breaker: if a provider fails 3 times in 1 minute, skip it for 5 minutes
3. Implement a timeout wrapper
4. Return a structured response (success, content, error, latency, model_used)

---

## Intermediate Exercises

### Exercise 6: Chatbot with Memory Management

Build a chatbot that manages different memory types:

```python
class MemoryManager:
    def __init__(self):
        self.ephemeral = []  # Current conversation
        self.short_term = []  # Last N sessions
        self.long_term = {}   # User preferences, facts
        self.summary = ""     # Compressed history
    
    def add_to_ephemeral(self, message):
        """Add message to current conversation."""
        pass
    
    def summarize_conversation(self):
        """Create a summary of the current conversation for long-term memory."""
        pass
    
    def get_relevant_memories(self, query: str) -> list[str]:
        """Retrieve memories relevant to the current query."""
        pass
    
    def prune_ephemeral(self, max_tokens: int):
        """Prune conversation to fit within token budget."""
        pass

# Build a simple chatbot using this memory manager
class Chatbot:
    def __init__(self):
        self.memory = MemoryManager()
    
    def chat(self, user_input: str) -> str:
        """Process user input and generate response."""
        pass
```

**Tasks:**
1. Implement all memory types (ephemeral, short-term, long-term, summary)
2. Implement summarization when ephemeral exceeds token budget
3. Use embedding similarity for memory retrieval
4. Test with a 20-turn conversation
5. Show how the system handles: "Remember my name is Alice" → "What's my name?"

### Exercise 7: Multi-Stage Content Pipeline

Build a content generation pipeline with review gates:

```python
class ContentGenerator:
    def __init__(self, style_guide: dict):
        self.style = style_guide
    
    def outline(self, topic: str) -> str:
        """Generate outline."""
        pass
    
    def draft(self, outline: str) -> str:
        """Write first draft."""
        pass
    
    def review(self, text: str, stage: str) -> dict:
        """Review text at a given stage. Return issues and suggestions."""
        pass
    
    def rewrite(self, text: str, feedback: dict) -> str:
        """Apply feedback and rewrite."""
        pass
    
    def produce(self, topic: str, iterations: int = 3) -> list[str]:
        """End-to-end: produce final content with revision history."""
        pass

# Test:
generator = ContentGenerator({
    "tone": "professional",
    "max_paragraph_length": 4,
    "avoid": ["I think", "maybe", "perhaps"],
    "required_sections": ["introduction", "body", "conclusion"],
})

history = generator.produce("Benefits of microservices architecture")
for i, version in enumerate(history):
    print(f"\n=== Version {i+1} ===")
    print(version[:200])
```

**Tasks:**
1. Each review stage checks different things (pass 1: grammar, pass 2: style, pass 3: brand)
2. Track word count and readability score across versions
3. Implement human-in-the-loop: stop at each review gate for approval
4. Add a comparison mode that shows diff between versions

### Exercise 8: Customer Support Ticket Router

Implement a complete ticket routing system:

```python
class TicketRouter:
    def __init__(self):
        self.rules = []
        self.default_queue = "general"
    
    def add_rule(self, condition_fn, queue: str):
        """Add a routing rule."""
        self.rules.append((condition_fn, queue))
    
    def route(self, ticket: dict) -> str:
        """Route ticket to appropriate queue."""
        pass
    
    def classify(self, ticket_text: str) -> dict:
        """Classify ticket: category, sentiment, urgency, required department."""
        pass

class AutoResponder:
    def __init__(self, knowledge_base: dict):
        self.kb = knowledge_base
    
    def generate_response(self, ticket: dict) -> dict:
        """Generate auto-response if possible."""
        pass

# Build a complete system:
# 1. Route tickets to: billing, technical, account, sales, general
# 2. Auto-respond to common questions using KB
# 3. Escalate angry tickets or low-confidence responses
# 4. Track routing accuracy

# Test cases:
tickets = [
    {"id": "T1", "text": "I need a refund for my order", "customer_tier": "gold"},
    {"id": "T2", "text": "My app keeps crashing when I click submit", "customer_tier": "standard"},
    {"id": "T3", "text": "THIS IS UNACCEPTABLE! FIX IT NOW!", "customer_tier": "standard"},
    {"id": "T4", "text": "How do I reset my password?", "customer_tier": "silver"},
]
```

**Tasks:**
1. Implement sentiment analysis to detect angry customers
2. Implement escalation rules (VIP, angry, legal)
3. Implement confidence threshold for auto-response vs. human
4. Track metrics: auto-response rate, escalation rate, routing accuracy

### Exercise 9: Document Q&A with Hybrid Search

Build a document Q&A system that combines keyword and semantic search:

```python
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer

class HybridSearch:
    def __init__(self, chunks: list[str]):
        self.chunks = chunks
        self.tfidf = TfidfVectorizer(stop_words="english")
        self.tfidf_matrix = self.tfidf.fit_transform(chunks)
        self.embeddings = None  # Store OpenAI embeddings
    
    def keyword_search(self, query: str, top_k: int = 5) -> list[int]:
        """TF-IDF based keyword search."""
        pass
    
    def semantic_search(self, query: str, top_k: int = 5) -> list[int]:
        """Embedding-based semantic search."""
        pass
    
    def hybrid_search(self, query: str, alpha: float = 0.5, top_k: int = 5) -> list[int]:
        """Combined search. alpha = weight for semantic vs keyword."""
        pass

# Test with a variety of chunks
chunks = [
    "Neural networks consist of layers of interconnected neurons.",
    "Backpropagation computes gradients using the chain rule.",
    "CNNs are specialized for processing grid-like data like images.",
    "RNNs process sequential data with hidden states.",
    "The Transformer architecture uses self-attention mechanisms.",
    "Transfer learning reuses a pretrained model on a new task.",
    "Gradient descent minimizes the loss function iteratively.",
]

searcher = HybridSearch(chunks)
# Compare keyword vs semantic vs hybrid for queries like:
# - "How do neural networks learn?"
# - "What is the chain rule?"
# - "Self-attention in transformers"
```

**Tasks:**
1. Implement all three search methods
2. Compare results for different query types
3. Find the optimal alpha for your test queries
4. Add a reranking step using cross-encoder

### Exercise 10: NL-to-SQL with Self-Correction

Build an NL-to-SQL system that corrects its own errors:

```python
import sqlite3

class NLToSQLWithCorrection:
    def __init__(self, db_path: str):
        self.conn = sqlite3.connect(db_path)
        self.schema = self.get_schema()
        self.max_retries = 3
    
    def get_schema(self):
        """Extract database schema."""
        pass
    
    def generate_sql(self, question: str) -> str:
        """Generate SQL from natural language."""
        pass
    
    def execute(self, sql: str) -> tuple:
        """Execute SQL, return (success, result_or_error)."""
        pass
    
    def correct(self, sql: str, error: str) -> str:
        """Ask LLM to fix SQL based on error."""
        pass
    
    def ask(self, question: str) -> dict:
        """End-to-end: question → SQL → execute → correct → result."""
        pass

# Test with Chinook database or SQLite sample
# nlsql = NLToSQLWithCorrection("chinook.db")
# print(nlsql.ask("List all tracks longer than 5 minutes"))
# print(nlsql.ask("Which artist has the most albums?"))
```

**Tasks:**
1. Implement the self-correction loop (max 3 attempts)
2. Track how many corrections each query needs
3. Log failed queries for manual review
4. Add a safety validator that blocks dangerous SQL

---

## Advanced Exercises

### Exercise 11: Multi-Agent Research System

Build a system with specialized agents for research:

```python
class ResearchAgent:
    """Coordinating agent for multi-source research."""
    pass

class SearchAgent:
    """Agent specialized in web search and source discovery."""
    pass

class ExtractAgent:
    """Agent specialized in content extraction and summarization."""
    pass

class SynthesisAgent:
    """Agent specialized in combining sources into coherent answer."""
    pass

class CitationAgent:
    """Agent specialized in formatting citations (APA, MLA, etc.)."""
    pass

class ResearchOrchestrator:
    """Orchestrate multiple agents for comprehensive research."""
    def __init__(self):
        self.agents = {
            "search": SearchAgent(),
            "extract": ExtractAgent(),
            "synthesis": SynthesisAgent(),
            "citation": CitationAgent(),
        }
    
    def research(self, query: str) -> dict:
        """Full research pipeline with multiple agents."""
        pass

# Test with complex queries that need multiple sources
```

**Tasks:**
1. Each agent should have a specific role and system prompt
2. Implement handoff protocol between agents
3. Add quality checks at each stage
4. Implement parallel search across multiple sources
5. Generate properly formatted citations

### Exercise 12: Benchmark Different Architectures

Build a benchmarking framework to compare architecture patterns:

```python
class ArchitectureBenchmark:
    def __init__(self):
        self.results = []
    
    def run_pipeline(self, tasks: list[dict]) -> dict:
        """Run pipeline architecture on tasks."""
        pass
    
    def run_orchestrator(self, tasks: list[dict]) -> dict:
        """Run orchestrator architecture on tasks."""
        pass
    
    def run_ensemble(self, tasks: list[dict]) -> dict:
        """Run ensemble architecture on tasks."""
        pass
    
    def compare(self, tasks: list[dict]) -> pd.DataFrame:
        """Compare all architectures on metrics."""
        pass

# Compare these patterns on:
# 1. Simple classification tasks
# 2. Complex multi-step reasoning
# 3. Factual Q&A with retrieval
# 4. Creative content generation

# Metrics to track:
# - Accuracy/quality score
# - Average latency
# - Cost per task
# - Error rate
# - Reliability (success rate)
```

**Tasks:**
1. Build each architecture variant for the same task
2. Run at least 20 test cases per architecture
3. Generate a comparison table
4. Provide recommendations for when to use each pattern
5. Add statistical significance tests

### Exercise 13: Implement a Circuit Breaker

Build a circuit breaker for LLM API calls:

```python
import time
from enum import Enum

class CircuitState(Enum):
    CLOSED = "closed"        # Normal operation
    OPEN = "open"           # Failing, reject requests
    HALF_OPEN = "half_open" # Testing if recovered

class CircuitBreaker:
    def __init__(self, failure_threshold=5, recovery_timeout=30, half_open_max=3):
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.recovery_timeout = recovery_timeout
        self.last_failure_time = 0
        self.half_open_max = half_open_max
        self.half_open_count = 0
    
    def call(self, fn, fallback_fn=None):
        """
        Call fn with circuit breaker protection.
        If circuit is open, call fallback_fn instead.
        """
        pass

# Integrate with an LLM client:
class LLMWithCircuitBreaker:
    def __init__(self, client):
        self.client = client
        self.circuit_breaker = CircuitBreaker()
    
    def complete(self, messages):
        return self.circuit_breaker.call(
            lambda: self.client.chat.completions.create(messages=messages),
            fallback_fn=lambda: {"choices": [{"message": {"content": "Service temporarily unavailable."}}]},
        )
```

**Tasks:**
1. Implement all three states with proper transitions
2. Add metrics tracking (failure rate, state changes)
3. Test by simulating failures
4. Integrate with multiple providers for failover

### Exercise 14: Full System — Personal Knowledge Base Chat

Build a complete system that lets users chat with their documents:

```python
class KnowledgeBaseChat:
    """End-to-end system: ingest → index → retrieve → chat."""
    
    def __init__(self, db_path: str = "kb.sqlite"):
        self.documents = {}
        self.chunks = []
        self.embeddings = None
    
    def ingest(self, file_path: str):
        """Ingest a document (PDF, text, markdown)."""
        pass
    
    def query(self, question: str) -> dict:
        """Ask a question and get answer with sources."""
        pass
    
    def chat(self, session_id: str, message: str) -> str:
        """Multi-turn chat with context."""
        pass

# Requirements:
# 1. Support PDF and text files
# 2. Chunk documents intelligently
# 3. Store in-memory or SQLite for persistence
# 4. Multi-turn conversation with context management
# 5. Source citation in responses
# 6. "I don't know" when answer not in documents
```

**Tasks:**
1. Build the complete system
2. Test with at least 3 diverse documents
3. Measure retrieval quality (precision@k, recall@k)
4. Add metadata filtering (by date, source type, file name)
5. Implement incremental ingestion (add documents without re-indexing everything)

### Exercise 15: Cost Optimization Analysis

Build an analysis tool that optimizes model selection for a workload:

```python
class CostOptimizer:
    def __init__(self):
        self.models = {
            "gpt-4o-mini": {"input": 0.15, "output": 0.60, "quality": 0.7},
            "gpt-4o": {"input": 2.50, "output": 10.00, "quality": 0.9},
            "claude-sonnet": {"input": 3.00, "output": 15.00, "quality": 0.92},
            "deepseek-v3": {"input": 0.50, "output": 2.00, "quality": 0.85},
        }
    
    def estimate_monthly_cost(self, daily_queries, avg_input, avg_output, model):
        """Estimate monthly cost for given model and load."""
        pass
    
    def find_optimal_routing(self, traffic_pattern: dict, quality_requirement: float):
        """
        Find optimal model routing strategy given:
        - traffic_pattern: {"simple": 0.7, "complex": 0.2, "hard": 0.1}
        - quality_requirement: minimum acceptable quality 0.0-1.0
        Returns routing strategy and estimated monthly cost.
        """
        pass
    
    def savings_report(self, current_strategy: dict, optimized_strategy: dict):
        """Generate savings report comparing strategies."""
        pass

# Test with different workload profiles
# Profile 1: 80% simple Q&A, 15% analysis, 5% hard reasoning
# Profile 2: 40% code, 40% chat, 20% content creation
```

**Tasks:**
1. Implement cost estimation for each model
2. Find the optimal routing strategy for a given workload
3. Calculate savings vs. using a single model for everything
4. Add latency constraints to the optimization
5. Generate a report with charts showing cost breakdown

---

## Bonus Challenge

### Exercise 16: Build a Production-Ready System

Choose one of these systems and build it end-to-end with all production considerations:

**Option A: Customer Support Ticket Classifier**
- Routes tickets to departments
- Generates auto-responses
- Handles escalation
- Has monitoring and logging
- Supports A/B testing of different models

**Option B: Document Q&A System**
- Ingests PDFs and web pages
- Chunks intelligently by section
- Hybrid search (keyword + semantic)
- Citations in responses
- Multi-turn conversation

**Option C: Coding Assistant**
- Understands repository structure
- Generates multi-file changes
- Runs tests and fixes failures
- Reviews its own code

**Requirements for all options:**
- Context management with pruning
- Error handling with fallbacks
- Cost tracking and optimization
- Observability (logging, metrics)
- At least one architecture pattern (pipeline, orchestrator, etc.)
- Streaming response
- Safety guardrails
