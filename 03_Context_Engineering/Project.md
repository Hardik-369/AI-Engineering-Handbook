# Project: Build a Context Management System

## Overview

Build a system that manages a long conversation with an LLM, implementing sliding windows, summarization, memory retrieval, and adaptive budget allocation. You will measure how different strategies affect response quality over a 50-turn conversation.

**Duration:** 2-3 days (recommended)
**Difficulty:** Advanced
**Prerequisites:** Python, familiarity with LLM APIs, basic ML concepts

---

## Part 1: Foundation — Context Manager Core

### 1.1 Basic Structure

Implement the base `ContextManager` class:

```python
class ContextManager:
    def __init__(self, model_limit: int = 128_000, output_reserve: int = 4_000):
        pass

    def build_context(self, query: str, user_id: str) -> str:
        """Assemble the full context for this query."""
        pass

    def process_response(self, query: str, response: str):
        """Update memory and history after receiving a response."""
        pass
```

### 1.2 Token Accounting

Implement precise token counting. Use `tiktoken` for OpenAI models or the appropriate tokenizer for your model. Track:

- Tokens per message
- Tokens per category (system, history, retrieval, memory)
- Total context utilization percentage

```python
class TokenAccountant:
    def __init__(self, model_limit: int, output_reserve: int):
        pass

    def add_category(self, name: str, content: str) -> int:
        """Add content to a category, return token count."""
        pass

    def get_utilization(self) -> dict:
        """Return breakdown of token usage by category."""
        pass

    def is_over_budget(self) -> bool:
        pass
```

### 1.3 System Prompt Management

Create a system prompt manager that:

- Stores a base system prompt.
- Can inject dynamic instructions.
- Tracks system prompt token count.

```python
class SystemPromptManager:
    def __init__(self, base_prompt: str):
        self.base_prompt = base_prompt
        self.dynamic_instructions = []

    def add_instruction(self, instruction: str):
        self.dynamic_instructions.append(instruction)

    def remove_instruction(self, index: int):
        pass

    def get_prompt(self) -> str:
        return self.base_prompt + "\n" + "\n".join(self.dynamic_instructions)
```

---

## Part 2: Sliding Window Strategies

### 2.1 FIFO Sliding Window

Implement a standard FIFO sliding window. When a new message exceeds the budget, evict the oldest message.

**Interface:**
```python
class FIFOWindow:
    def __init__(self, max_tokens: int):
        self.messages = []

    def add(self, role: str, content: str):
        """Add message, evicting oldest if over budget."""

    def get_messages(self) -> list[dict]:
        """Return messages in chronological order."""
```

### 2.2 Prioritized Sliding Window

Extend the window to support priority levels. System messages are never evicted. Tool outputs are evicted before user messages.

```python
class PrioritizedWindow(FIFOWindow):
    PRIORITY = {"system": 3, "tool": 1, "user": 2, "assistant": 2}

    def evict_candidates(self) -> list[int]:
        """Return indices of evictable messages, sorted by eviction priority."""
```

### 2.3 Summary-Buffer Window

Keep the last N turns verbatim. Summarize everything older into a condensed form.

```python
class SummaryBufferWindow:
    def __init__(self, max_tokens: int, keep_verbatim: int = 5):
        self.summary = ""
        self.buffer = []  # Last N turns, verbatim
        self.archive = []  # Summarized older turns

    def summarize_archive(self):
        """Use LLM to summarize all archived turns."""
```

### 2.4 Importance-Scored Window

Score each message by importance. Evict the lowest-scored messages first.

```python
class ImportanceWindow(FIFOWindow):
    def score(self, message: dict) -> float:
        """Score message importance. Factors: has key terms, query relevance, recency."""
        pass

    def get_eviction_order(self) -> list[int]:
        """Return message indices sorted by eviction priority (lowest score first)."""
```

---

## Part 3: Memory System

### 3.1 Episodic Memory

Store summarized interactions across the conversation. On each new query, retrieve relevant memories.

```python
class EpisodicMemory:
    def __init__(self):
        self.memories = []  # List of {"timestamp": ..., "summary": ..., "key_facts": [...]}

    def add_memory(self, summary: str, key_facts: list[str]):
        pass

    def retrieve(self, query: str, k: int = 3) -> list[str]:
        """Retrieve memories relevant to the query."""
        pass

    def consolidate(self):
        """Merge related memories to reduce redundancy."""
        pass
```

### 3.2 Semantic Memory

Extract and store factual knowledge from the conversation. Use embeddings for retrieval.

```python
class SemanticMemory:
    def __init__(self, embedding_model="all-MiniLM-L6-v2"):
        self.facts = []
        self.embeddings = None

    def extract_facts(self, conversation_turn: dict) -> list[str]:
        """Use an LLM to extract factual statements from a conversation turn."""
        pass

    def store(self, facts: list[str]):
        """Embed and store new facts."""
        pass

    def query(self, question: str, k: int = 5) -> list[str]:
        """Retrieve most relevant facts using embedding similarity."""
        pass
```

### 3.3 Memory Summarization

Implement the summarization logic that converts raw conversation into compressed memory entries.

```python
class MemorySummarizer:
    def __init__(self, model="gpt-4o-mini"):
        pass

    def summarize_turns(self, turns: list[dict]) -> str:
        """Summarize a block of conversation turns."""
        pass

    def extract_key_facts(self, text: str) -> list[str]:
        """Extract factual statements from text."""
        pass

    def update_running_summary(self, old_summary: str, new_turns: list[dict]) -> str:
        """Incrementally update an existing summary with new turns."""
        pass
```

---

## Part 4: Retrieval and Context Packing

### 4.1 Document Retrieval

Implement a simple retrieval system (can use a local vector store or simulate with a small document set).

```python
class Retriever:
    def __init__(self):
        self.documents = []
        self.index = None

    def add_documents(self, docs: list[str]):
        pass

    def retrieve(self, query: str, k: int = 5) -> list[str]:
        pass
```

### 4.2 Context Packer

Assemble all components into a final context string. Must handle the lost in the middle problem through strategic ordering.

```python
class ContextPacker:
    def __init__(self, max_tokens: int):
        pass

    def pack(self, system: str, memories: list[str],
             docs: list[str], history: list[dict],
             query: str) -> str:
        """
        Pack everything into a single context string.
        - Place system at start (protected position)
        - Place most relevant docs at start and end
        - Place history before query
        - Truncate from middle if over budget
        """
        pass
```

---

## Part 5: Full Integration

### 5.1 Complete System

Combine all components into a working system:

```python
class ContextManagementSystem:
    def __init__(self, config: dict):
        self.config = config
        self.token_accountant = TokenAccountant(...)
        self.system_prompts = SystemPromptManager(...)
        self.window = PrioritizedWindow(...)
        self.episodic_memory = EpisodicMemory(...)
        self.semantic_memory = SemanticMemory(...)
        self.summarizer = MemorySummarizer(...)
        self.retriever = Retriever(...)
        self.packer = ContextPacker(...)

    def query(self, user_query: str) -> str:
        # 1. Retrieve documents
        # 2. Retrieve relevant episodic memories
        # 3. Query semantic memory
        # 4. Pack context
        # 5. Check budget — compress if needed
        # 6. Send to LLM
        # 7. Update sliding window
        # 8. Update memory (summarize if threshold reached)
        # 9. Return response
        pass
```

### 5.2 Configuration

```python
config = {
    "model": "gpt-4o",
    "model_limit": 128_000,
    "output_reserve": 4_000,
    "window_strategy": "priority",  # "fifo" | "priority" | "summary_buffer" | "importance"
    "keep_verbatim": 5,
    "summarize_every_n_turns": 5,
    "max_retrieved_docs": 5,
    "memory_top_k": 3,
    "include_semantic_memory": True,
    "include_episodic_memory": True,
}
```

---

## Part 6: Evaluation

### 6.1 Baseline Test

Run a 50-turn conversation with:
1. No context management (just FIFO window).
2. Your full system.

For each turn, record:
- Model response
- Context utilization %
- Memory size
- Number of evictions

### 6.2 Quality Metrics

Define and measure:

1. **Factual consistency** — Does the model remember facts from earlier turns? Test by asking about turn 3 at turn 30.
2. **Context relevance** — Are retrieved documents and memories actually relevant to the query?
3. **Token efficiency** — Tokens consumed per response.
4. **Information loss** — After compression, can the model still answer questions about compressed content?

### 6.3 Ablation Study

Disable components one at a time and measure quality differences:

- Without semantic memory
- Without episodic memory
- Without summarization (raw history only)
- Without document retrieval

**Hypothesis:** Each component contributes to a different failure mode. Which component is most critical?

### 6.4 Strategy Comparison

Compare the four sliding window strategies on:

1. **Fact retention** — How many turns back can the model recall?
2. **Token efficiency** — Tokens per query.
3. **Response quality** — Human or LLM-as-judge scoring.
4. **Latency** — Time to pack context and generate response.

---

## Part 7: Analysis and Documentation

### 7.1 What to Report

Write up your findings:

1. **System architecture** — Diagram and explain your system.
2. **Token usage profiles** — Show how token allocation changes over 50 turns for each strategy.
3. **Error analysis** — Find 3 conversations where your system failed. Why?
4. **Strategy comparison** — Which sliding window strategy performed best? Under what conditions?
5. **Memory analysis** — How did memory size grow? Were retrievals useful?
6. **Lost in the middle** — Did your context packing strategy mitigate the effect?

### 7.2 Visualization

Create plots:

```
1. Context utilization over time (line chart, x=turn, y=% used)
2. Token allocation by category (stacked area chart)
3. Fact retention vs turn distance (scatter plot)
4. Strategy performance comparison (bar chart)
5. Memory growth over conversation (line chart)
```

### 7.3 Production Recommendations

Based on your experiments, write a one-page recommendation for:

1. Which sliding window strategy to use in production.
2. When to summarize and how aggressively.
3. How to set token budgets for each category.
4. What monitoring metrics to track.

---

## Part 8: Extension Challenges (Optional)

### 8.1 Multi-Session Memory

Extend the system to persist memory across multiple sessions. Use a JSON file or SQLite as persistent storage.

### 8.2 Adaptive Budget Allocation

Implement a classifier that determines whether a query is:
- **Fact-seeking** → Allocate more to retrieval
- **Conversational** → Allocate more to history
- **Instructional** → Allocate more to system/tools

Adjust budget allocation dynamically based on the classifier output.

### 8.3 Lost in the Middle Mitigation

Implement and evaluate at least 3 strategies for mitigating lost in the middle:
1. Structured XML formatting
2. Critical information duplication (mention key facts at start and end)
3. Recency-based reordering
4. Explicit "pay attention to X" instructions

### 8.4 Streaming Context Updates

Implement a system where the context is updated incrementally as new information arrives, without rebuilding from scratch each turn.

### 8.5 Compression Benchmark

Compare three compression strategies on the same conversation:
1. LLM-based summarization (gpt-4o-mini)
2. Extractive (textrank)
3. Learned compression (LLMLingua or simulated)

Measure compression ratio, response quality, and latency for each.

---

## Deliverables

1. **Working code** — Complete Python implementation of the context management system.
2. **Test conversations** — Logs from at least 3 different 50-turn conversations.
3. **Evaluation data** — Token usage, response quality, memory growth metrics.
4. **Analysis report** — Findings, comparisons, and recommendations (2-3 pages).
5. **Configuration guide** — How to configure the system for different use cases.

## Evaluation Rubric

| Criterion | Excellent | Good | Needs Work |
|-----------|-----------|------|------------|
| Code quality | Modular, tested, documented | Functional but messy | Incomplete or broken |
| Context engineering | All strategies implemented, compared | Basic implementation only | Missing key components |
| Evaluation | Rigorous metrics, multiple runs | Some metrics, single run | No evaluation |
| Analysis | Insightful findings, clear recommendations | Basic observations | No analysis |
| Documentation | Clear, complete, well-structured | Some documentation | None or minimal |
