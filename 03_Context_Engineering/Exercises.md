# Exercises: Context Engineering

## Beginner

### Exercise 1: Calculate Token Usage

Given a GPT-4o model with a 128K token limit, plan the context budget for a customer support agent:

- System prompt: 420 tokens
- Tool definitions: 890 tokens
- Retrieved documents: up to 4,000 tokens
- Output reservation: 2,000 tokens
- Conversation history: remaining tokens

If each conversation turn averages 150 tokens (user + assistant combined), how many turns of history can you fit before you need summarization?

**Task:** Calculate the budget and determine the threshold at which history summarization must trigger.

---

### Exercise 2: Fixed-Size Chunking

Take the following text and chunk it into pieces of 100 characters with 20 characters of overlap:

```
Artificial intelligence is transforming every industry. From healthcare to finance, 
AI systems are being deployed to automate complex tasks that previously required 
human expertise. However, with great power comes great responsibility. 
Ethical considerations around bias, fairness, and transparency must be at the 
forefront of any AI deployment strategy. Organizations that fail to address 
these concerns risk damaging their reputation and eroding user trust.
```

**Task:** Write out the chunks manually or with a script. Identify where sentences are broken. Would semantic chunking produce better results?

---

### Exercise 3: Identify the Lost in the Middle

You have 10 documents to place in context. Document A is the most critical (must be answered correctly), Document B is second most critical. Where do you place each document to maximize accuracy?

**Task:** Order the documents and explain your strategy.

---

### Exercise 4: Token Counting Practice

Using any tokenizer (tiktoken, Hugging Face tokenizers), count the tokens in these strings:

```python
strings = [
    "Hello, world!",
    "The quick brown fox jumps over the lazy dog.",
    "Context engineering is the discipline of managing what information goes into an LLM's context window.",
    "a" * 1000,
]
```

**Task:** How many tokens does each string contain? What's the ratio of tokens to characters for each?

---

### Exercise 5: Fixed vs Semantic Chunking

Take a 500-word article. Chunk it using:
1. Fixed-size: 200 characters, 20 character overlap
2. Semantic: Split at paragraph boundaries

Compare the quality of chunks. Which preserves meaning better?

---

## Intermediate

### Exercise 6: BM25 Retrieval Implementation

Implement BM25 from scratch (no libraries) for a small collection of 5-10 documents.

**Task:**
1. Compute IDF for each term across the corpus.
2. For a given query, compute BM25 scores.
3. Return the top 3 documents.
4. Test with queries that have exact matches and partial matches.

Discuss when BM25 fails (e.g., synonyms).

---

### Exercise 7: Embedding-Based Retrieval

Using sentence-transformers (`all-MiniLM-L6-v2`), implement a dense retrieval system:

```python
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer('all-MiniLM-L6-v2')

documents = [
    "Context engineering manages what goes into an LLM's context window.",
    "Chunking breaks documents into pieces for retrieval.",
    "The lost in the middle problem affects long context performance.",
    "Memory systems provide persistence across conversation turns.",
    "Sliding windows discard old messages when context is full.",
]

queries = [
    "How do I manage conversation history?",
    "What is document chunking?",
    "Why do models forget information in the middle?",
]
```

**Task:**
1. Embed all documents and queries.
2. Compute cosine similarity.
3. Return top-2 documents for each query.
4. Identify any false positives or misses.
5. Try with a query that uses synonyms not present in any document.

---

### Exercise 8: Chunk Overlap Experiment

Implement a document chunker with configurable overlap. Test with a 2000-token document:

- Chunk size: 256 tokens
- Overlaps: 0, 32, 64, 128 tokens

**Task:**
1. Measure total tokens stored at each overlap level.
2. Measure the number of chunks.
3. For a given query, does higher overlap improve recall?
4. What's the overlap sweet spot for this document?

---

### Exercise 9: Reranking Pipeline

Build a two-stage retrieval pipeline:

```
BM25/Dense → Top 20 → Cross-encoder reranker → Top 5
```

Use a small cross-encoder model (`cross-encoder/ms-marco-MiniLM-L-6-v2`).

**Task:**
1. Retrieve top 20 with BM25 or dense.
2. Score each with the cross-encoder.
3. Re-rank and select top 5.
4. Compare precision@5 with and without reranking.

---

### Exercise 10: Context Packing Experiment

You have 6 retrieved documents (D1-D6) with relevance scores:
D1: 0.95, D2: 0.90, D3: 0.80, D4: 0.75, D5: 0.60, D6: 0.50

Your context window can fit 4 documents.

**Task:**
1. Order by relevance descending: D1, D2, D3, D4
2. Order by relevance ascending: D4, D3, D2, D1
3. Put most relevant at start and end: D1, D3, D4, D2
4. Test with an LLM — which ordering produces the best answer?

---

### Exercise 11: Conversation History Management

Simulate a 20-turn conversation. Each turn is ~100 tokens. Your context budget for history is 1000 tokens.

**Task:**
1. Implement FIFO sliding window.
2. Implement a summary-buffer: keep last 3 turns verbatim, summarize the rest.
3. Implement relevance-weighted eviction: score each past turn by relevance to the current query.
4. Compare the quality of responses under each strategy.

---

## Advanced

### Exercise 12: Memory System with Summarization

Build a complete memory system:

```python
class MemorySystem:
    def __init__(self, max_turns=10):
        self.working_memory = []  # Current conversation
        self.episodic_memory = []  # Summarized past
        self.semantic_memory = {}  # Extracted knowledge

    def add_turn(self, user_msg, assistant_msg):
        pass

    def summarize_episodic(self):
        pass

    def retrieve_relevant(self, query, k=3):
        pass

    def get_context(self):
        pass
```

**Task:**
1. Implement the full system.
2. After every 5 turns, summarize working memory into episodic memory.
3. On each new query, retrieve relevant episodic memories.
4. Return a packed context with: system prompt + retrieved memories + working memory summary + current input.
5. Test with a 20-turn conversation and measure context utilization.

---

### Exercise 13: Sliding Window with Importance Scoring

Implement a non-FIFO sliding window:

```python
class ImportanceSlidingWindow:
    def __init__(self, max_tokens=4000):
        self.messages = []
        self.max_tokens = max_tokens

    def score_importance(self, message):
        # Score based on: contains key terms, is recent, was from system, etc.
        pass

    def add_message(self, message):
        pass

    def evict(self):
        pass
```

**Task:**
1. Define an importance scoring function.
2. When budget is exceeded, evict the lowest-importance messages.
3. Test with a mixed conversation (system instructions, user questions, tool outputs).
4. Compare against FIFO — does importance scoring improve response quality?

---

### Exercise 14: Lost in the Middle Replication

Replicate the core finding from Liu et al. (2023):

```python
import random

documents = [f"Document {i}: The capital of {country} is {capital}." 
             for i, (country, capital) in enumerate(countries)]

for position in range(len(documents)):
    # Place the target document at each position
    # Ask: "What is the capital of X?"
    # Measure accuracy at each position
    pass
```

**Task:**
1. Create 20 documents with trivia facts.
2. For each position (0-19), place the target document at that position.
3. Query the LLM and record accuracy.
4. Plot accuracy vs position — do you see the U-shaped curve?
5. Try with structured formats (XML) — does the curve flatten?

---

### Exercise 15: Token Budget Optimizer

Build a system that dynamically allocates context budget based on the query:

```python
class BudgetOptimizer:
    def __init__(self, model_limit=128000, output_reserve=4000):
        pass

    def classify_query(self, query):
        # "factual" → allocate more to retrieval
        # "conversational" → allocate more to history
        # "instructional" → allocate more to system prompt
        pass

    def allocate(self, query_type):
        pass
```

**Task:**
1. Classify queries into at least 3 types.
2. For each type, define an optimal budget allocation.
3. Test with 10 queries per type.
4. Measure whether dynamic allocation improves response quality over fixed allocation.

---

### Exercise 16: Recursive Chunking on Real Documents

Implement a recursive chunker that tries separators in order:

```python
def recursive_chunk(text, chunk_size=500, chunk_overlap=50):
    separators = ["\n\n", "\n", ".", " ", ""]
    # Try each separator, splitting the text
    # If any chunk is too large, recursively split with the next separator
    pass
```

**Task:**
1. Implement the function.
2. Test on a long markdown or HTML document.
3. Compare against fixed-size chunking.
4. Measure chunk coherence (do all sentences in a chunk belong to the same topic?).

---

### Exercise 17: Cross-Encoder Reranker from Scratch

Implement a simplified reranker using a pre-trained BERT model:

**Task:**
1. Load a BERT-based cross-encoder.
2. For each (query, document) pair, produce a relevance score.
3. Implement batched scoring for efficiency.
4. Compare speed vs accuracy tradeoffs: scoring 100 pairs vs scoring 10 pairs.

---

### Exercise 18: Multi-Turn Memory Retrieval

Build a system that carries information across sessions:

```
Session 1: User discusses Python projects
  → Memory: "User is building an AI handbook, prefers Python"

Session 2: User asks about chunking
  → Retrieve: "User is building an AI handbook"
  → Response tailored to handbook context

Session 3: User asks about deployment
  → Retrieve: previous technical discussions
  → Maintain consistency
```

**Task:**
1. Implement cross-session memory retrieval.
2. Use embeddings to match current query to past sessions.
3. Include relevant past context in the current prompt.
4. Test with 5 simulated sessions.

---

### Exercise 19: Context Compression Comparison

Implement three compression strategies and compare them:

1. **Extractive:** Keep top-50% of sentences by relevance score.
2. **Abstractive:** Use an LLM to summarize the document in 1/3 the tokens.
3. **Learned:** Use a perplexity-based filter (simulate with a small LM).

**Task:**
1. Apply each to the same 1000-token document.
2. Measure compression ratio.
3. Measure information retention (ask an LLM questions about the compressed version).
4. Which strategy gives the best efficiency-retention tradeoff?

---

### Exercise 20: Full Context Management System

Combine everything into a production-quality context manager:

```python
class ContextManager:
    def __init__(self, model="gpt-4o"):
        self.retriever = HybridRetriever()
        self.reranker = CrossEncoderReranker()
        self.memory = MemorySystem()
        self.window = SlidingWindow(max_tokens=100000)
        self.compressor = ContextCompressor()

    def build_context(self, query, user_id):
        # 1. Retrieve relevant documents
        docs = self.retriever.retrieve(query, top_k=20)
        # 2. Rerank
        docs = self.reranker.rerank(query, docs, top_k=5)
        # 3. Load memory
        memories = self.memory.retrieve(query)
        # 4. Pack context
        context = self.pack_context(query, docs, memories)
        # 5. Compress if needed
        if count_tokens(context) > self.window.max_tokens:
            context = self.compressor.compress(context)
        return context
```

**Task:**
1. Implement all components.
2. Test with a long-running conversation (20+ turns).
3. Profile token usage over time.
4. Measure response quality at turn 1, 10, and 20.
5. Document what degrades and why.
