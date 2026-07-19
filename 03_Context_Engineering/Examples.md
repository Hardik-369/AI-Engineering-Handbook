# Code Examples: Context Engineering

All examples use Python. Install dependencies as needed.

---

## 1. Token Counting and Budget Planning

```python
"""Token counting with tiktoken (OpenAI models)"""
import tiktoken

def count_tokens(text: str, model: str = "gpt-4") -> int:
    encoding = tiktoken.encoding_for_model(model)
    return len(encoding.encode(text))

class TokenBudget:
    def __init__(self, model_limit: int = 128_000, output_reserve: int = 4_000):
        self.model_limit = model_limit
        self.output_reserve = output_reserve
        self.available = model_limit - output_reserve

    def allocate(self, system: int, tools: int, retrieval: int,
                 memory: int, history: int) -> dict:
        total = system + tools + retrieval + memory + history
        if total > self.available:
            raise ValueError(f"Over budget by {total - self.available} tokens")

        return {
            "system": {"tokens": system, "pct": system / self.available * 100},
            "tools": {"tokens": tools, "pct": tools / self.available * 100},
            "retrieval": {"tokens": retrieval, "pct": retrieval / self.available * 100},
            "memory": {"tokens": memory, "pct": memory / self.available * 100},
            "history": {"tokens": history, "pct": history / self.available * 100},
            "total_used": total,
            "remaining": self.available - total,
        }

budget = TokenBudget(model_limit=128_000, output_reserve=4_000)
plan = budget.allocate(
    system=500, tools=1000, retrieval=5000,
    memory=500, history=80_000
)

for slot, details in plan.items():
    if isinstance(details, dict):
        print(f"{slot}: {details['tokens']} tokens ({details['pct']:.1f}%)")
    else:
        print(f"{slot}: {details}")
```

---

## 2. Document Chunking

### Fixed-Size Chunking

```python
def fixed_size_chunk(text: str, chunk_size: int = 500,
                     overlap: int = 50) -> list[str]:
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start += chunk_size - overlap
    return chunks

text = "..." * 1000
chunks = fixed_size_chunk(text, chunk_size=200, overlap=20)
print(f"Generated {len(chunks)} chunks")
```

### Semantic Chunking (Recursive)

```python
def recursive_chunk(text: str, chunk_size: int = 500,
                    chunk_overlap: int = 50) -> list[str]:
    separators = ["\n\n", "\n", ". ", " ", ""]
    return _recursive_split(text, separators, chunk_size, chunk_overlap)

def _recursive_split(text: str, separators: list[str],
                     chunk_size: int, overlap: int) -> list[str]:
    if len(text) <= chunk_size:
        return [text]

    separator = separators[0]
    if separator == "":
        # Fallback: fixed-size split
        return fixed_size_chunk(text, chunk_size, overlap)

    if separator in text:
        splits = text.split(separator)
        chunks = []
        current = ""
        for split in splits:
            candidate = current + separator + split if current else split
            if len(candidate) <= chunk_size:
                current = candidate
            else:
                if current:
                    chunks.append(current)
                current = split
        if current:
            chunks.append(current)

        # Recursively split oversized chunks
        if len(separators) > 1:
            result = []
            for chunk in chunks:
                if len(chunk) > chunk_size:
                    result.extend(
                        _recursive_split(chunk, separators[1:],
                                         chunk_size, overlap)
                    )
                else:
                    result.append(chunk)
            return result
        return chunks

    if len(separators) > 1:
        return _recursive_split(text, separators[1:], chunk_size, overlap)

    return fixed_size_chunk(text, chunk_size, overlap)

# Usage
text = "# Introduction\n\nContext engineering is...\n\n## Chunking\n\n..."
chunks = recursive_chunk(text, chunk_size=500)

for i, chunk in enumerate(chunks):
    print(f"Chunk {i}: {len(chunk)} chars — {chunk[:50]}...")
```

### Semantic Chunking (Embedding-Based)

```python
import numpy as np
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

def semantic_chunk(text: str, threshold: float = 0.7) -> list[str]:
    sentences = text.replace('\n', ' ').split('. ')
    sentences = [s.strip() + '.' for s in sentences if s.strip()]

    embeddings = model.encode(sentences)

    chunks = []
    current_chunk = [sentences[0]]

    for i in range(1, len(sentences)):
        sim = np.dot(embeddings[i], embeddings[i-1]) / (
            np.linalg.norm(embeddings[i]) * np.linalg.norm(embeddings[i-1])
        )

        if sim >= threshold:
            current_chunk.append(sentences[i])
        else:
            chunks.append(' '.join(current_chunk))
            current_chunk = [sentences[i]]

    if current_chunk:
        chunks.append(' '.join(current_chunk))

    return chunks

chunks = semantic_chunk(text, threshold=0.65)
for i, chunk in enumerate(chunks):
    print(f"Semantic chunk {i}: {len(chunk.split())} words")
```

---

## 3. Embedding-Based Retrieval

```python
import numpy as np
from sentence_transformers import SentenceTransformer

class DenseRetriever:
    def __init__(self, model_name: str = 'all-MiniLM-L6-v2'):
        self.model = SentenceTransformer(model_name)
        self.documents = []
        self.embeddings = None

    def index(self, documents: list[str]):
        self.documents = documents
        self.embeddings = self.model.encode(documents, show_progress_bar=True)

    def retrieve(self, query: str, k: int = 5) -> list[tuple[str, float]]:
        query_emb = self.model.encode([query])[0]
        scores = np.dot(self.embeddings, query_emb) / (
            np.linalg.norm(self.embeddings, axis=1) * np.linalg.norm(query_emb)
        )
        top_indices = np.argsort(scores)[-k:][::-1]
        return [(self.documents[i], float(scores[i])) for i in top_indices]

retriever = DenseRetriever()
retriever.index([
    "Context engineering manages what goes into an LLM's context window.",
    "Chunking breaks documents into pieces for retrieval.",
    "The lost in the middle problem affects long context performance.",
    "Memory systems provide persistence across conversation turns.",
    "Sliding windows discard old messages when context is full.",
])

results = retriever.retrieve("How do I manage conversation history?", k=2)
for doc, score in results:
    print(f"Score: {score:.4f} — {doc}")
```

---

## 4. Reranking with Cross-Encoders

```python
from sentence_transformers import CrossEncoder

class Reranker:
    def __init__(self, model_name: str = 'cross-encoder/ms-marco-MiniLM-L-6-v2'):
        self.model = CrossEncoder(model_name)

    def rerank(self, query: str, documents: list[str],
               top_k: int = 5) -> list[tuple[str, float]]:
        pairs = [[query, doc] for doc in documents]
        scores = self.model.predict(pairs)
        ranked = sorted(zip(documents, scores), key=lambda x: x[1], reverse=True)
        return ranked[:top_k]

retriever = DenseRetriever()
retriever.index(all_documents)
initial_results = retriever.retrieve("chunking strategies", k=20)

reranker = Reranker()
final_results = reranker.rerank("chunking strategies",
                                [doc for doc, _ in initial_results],
                                top_k=5)

for doc, score in final_results:
    print(f"Re-ranked Score: {score:.4f} — {doc[:80]}...")
```

---

## 5. Memory Summarization

```python
import openai  # or anthropic, etc.

class MemorySummarizer:
    def __init__(self, max_turns_before_summary: int = 5):
        self.turns = []
        self.summary = ""
        self.max_turns = max_turns_before_summary

    def add_turn(self, user: str, assistant: str):
        self.turns.append({"user": user, "assistant": assistant})

        if len(self.turns) >= self.max_turns:
            self._summarize()

    def _summarize(self):
        conversation = "\n".join(
            f"User: {t['user']}\nAssistant: {t['assistant']}"
            for t in self.turns
        )

        prompt = f"""Summarize the following conversation concisely.
Keep important facts, decisions, and user preferences.

Previous summary: {self.summary}

New conversation:
{conversation}

Updated summary:"""

        response = openai.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            max_tokens=300,
        )

        self.summary = response.choices[0].message.content
        self.turns = []

    def get_context(self) -> str:
        context = f"## Conversation Summary\n{self.summary}\n\n"
        context += "## Recent Messages\n"
        for t in self.turns:
            context += f"User: {t['user']}\nAssistant: {t['assistant']}\n"
        return context

memory = MemorySummarizer(max_turns_before_summary=3)
memory.add_turn("What is RAG?", "RAG stands for Retrieval-Augmented Generation...")
memory.add_turn("How does chunking work?", "Chunking breaks documents into pieces...")
memory.add_turn("What are embedding models?", "Embedding models convert text to vectors...")
print(memory.summary)
```

---

## 6. Sliding Window Implementation

```python
class SlidingWindow:
    def __init__(self, max_tokens: int = 100_000,
                 token_counter=None):
        self.max_tokens = max_tokens
        self.token_counter = token_counter or (lambda x: len(x.split()))
        self.messages = []
        self.token_count = 0

    def add(self, role: str, content: str) -> bool:
        tokens = self.token_counter(content)
        entry = {"role": role, "content": content, "tokens": tokens}

        # If a single message exceeds budget, error
        if tokens > self.max_tokens:
            raise ValueError("Message exceeds max context budget")

        # Evict until we have room
        while self.token_count + tokens > self.max_tokens:
            if not self.messages:
                return False
            evicted = self.messages.pop(0)
            self.token_count -= evicted["tokens"]

        self.messages.append(entry)
        self.token_count += tokens
        return True

    def get_context(self) -> list[dict]:
        return [{"role": m["role"], "content": m["content"]}
                for m in self.messages]

    def __len__(self):
        return len(self.messages)

    def utilization(self) -> float:
        return self.token_count / self.max_tokens

window = SlidingWindow(max_tokens=1000)
window.add("system", "You are a helpful assistant.")
window.add("user", "Hello!")
window.add("assistant", "Hi! How can I help?")

for _ in range(20):
    window.add("user", "What is context engineering?" * 5)
    window.add("assistant", "Context engineering is..." * 5)

print(f"Messages in window: {len(window)}")
print(f"Utilization: {window.utilization():.1%}")
```

### Prioritized Sliding Window

```python
class PrioritizedSlidingWindow(SlidingWindow):
    def __init__(self, max_tokens: int = 100_000, priority_roles=None):
        super().__init__(max_tokens)
        self.priority_roles = set(priority_roles or ["system"])

    def add(self, role: str, content: str) -> bool:
        tokens = self.token_counter(content)
        is_priority = role in self.priority_roles

        if tokens > self.max_tokens:
            raise ValueError("Message exceeds max context budget")

        while self.token_count + tokens > self.max_tokens:
            if not self.messages:
                return False

            # Find lowest-priority, non-priority message to evict
            evict_candidates = [
                (i, m) for i, m in enumerate(self.messages)
                if m["role"] not in self.priority_roles
            ]

            if not evict_candidates:
                # Only priority messages remain — evict oldest
                evicted = self.messages.pop(0)
            else:
                # Evict the oldest non-priority message
                idx = evict_candidates[0][0]
                evicted = self.messages.pop(idx)

            self.token_count -= evicted["tokens"]

        self.messages.append(
            {"role": role, "content": content, "tokens": tokens}
        )
        self.token_count += tokens
        return True

pwindow = PrioritizedSlidingWindow(max_tokens=1000)
pwindow.add("system", "You are a helpful assistant. Always be concise.")
pwindow.add("user", "Hello!")
pwindow.add("assistant", "Hi!")

for _ in range(20):
    pwindow.add("user", "What is context engineering?" * 5)
    pwindow.add("assistant", "Context engineering is..." * 5)

# System prompt should still be present
context = pwindow.get_context()
print(f"First message role: {context[0]['role']}")
print(f"Messages in window: {len(pwindow)}")
```

---

## 7. Lost in the Middle Experiment

```python
"""Reproduce the Lost in the Middle effect"""

import random
import openai

def run_lost_in_middle_experiment(model="gpt-4o-mini"):
    facts = [
        ("France", "Paris"), ("Japan", "Tokyo"), ("Brazil", "Brasilia"),
        ("Australia", "Canberra"), ("Egypt", "Cairo"), ("Canada", "Ottawa"),
        ("India", "New Delhi"), ("Germany", "Berlin"), ("Italy", "Rome"),
        ("Spain", "Madrid"), ("South Korea", "Seoul"), ("Argentina", "Buenos Aires"),
        ("Sweden", "Stockholm"), ("Norway", "Oslo"), ("Nigeria", "Abuja"),
        ("Kenya", "Nairobi"), ("Chile", "Santiago"), ("Vietnam", "Hanoi"),
        ("Thailand", "Bangkok"), ("Poland", "Warsaw"),
    ]

    def make_document(country, capital):
        return f"Document: The country {country} has a capital city called {capital}."

    def make_context(target_idx, num_docs=20):
        docs = [make_document(c, cap) for c, cap in facts]
        target = docs[target_idx]
        remaining = [d for i, d in enumerate(docs) if i != target_idx]
        random.shuffle(remaining)

        context_docs = remaining[:target_idx] + [target] + remaining[target_idx:]
        context_docs = context_docs[:num_docs]

        # Ensure target is at correct position
        target_pos = context_docs.index(target)
        if target_pos != target_idx:
            context_docs.remove(target)
            context_docs.insert(target_idx, target)

        return "\n\n".join(
            f"<doc id={i+1}>{d}</doc>"
            for i, d in enumerate(context_docs)
        )

    results = []
    for position in range(20):
        country, capital = facts[position]
        context = make_context(position, num_docs=20)

        prompt = f"""Read the documents below and answer the question.

Documents:
{context}

Question: What is the capital of {country}?
Answer with just the city name."""

        response = openai.chat.completions.create(
            model=model,
            messages=[{"role": "user", "content": prompt}],
            max_tokens=10,
            temperature=0,
        )

        answer = response.choices[0].message.content.strip().lower()
        correct = capital.lower() in answer
        results.append({"position": position, "correct": correct, "answer": answer})
        print(f"Position {position:2d}: {capital:15s} → {answer:15s} {'✓' if correct else '✗'}")

    accuracy_by_position = {}
    for r in results:
        pos_group = r["position"] // 4  # Group into 5 quartiles
        if pos_group not in accuracy_by_position:
            accuracy_by_position[pos_group] = []
        accuracy_by_position[pos_group].append(r["correct"])

    print("\nAccuracy by quartile:")
    for group in sorted(accuracy_by_position.keys()):
        acc = sum(accuracy_by_position[group]) / len(accuracy_by_position[group])
        print(f"  Positions {group*4}-{group*4+3}: {acc:.0%}")

    return results

# results = run_lost_in_middle_experiment()
```

---

## 8. Hybrid Retrieval (BM25 + Dense)

```python
import numpy as np
import math
from collections import Counter
from sentence_transformers import SentenceTransformer

class BM25:
    def __init__(self, k1=1.5, b=0.75):
        self.k1 = k1
        self.b = b
        self.documents = []
        self.doc_lengths = []
        self.avgdl = 0
        self.idf = {}

    def fit(self, documents):
        self.documents = documents
        self.doc_lengths = [len(doc.split()) for doc in documents]
        self.avgdl = sum(self.doc_lengths) / len(documents)

        N = len(documents)
        term_doc_freq = Counter()
        for doc in documents:
            terms = set(doc.lower().split())
            term_doc_freq.update(terms)

        self.idf = {
            term: math.log((N - freq + 0.5) / (freq + 0.5) + 1)
            for term, freq in term_doc_freq.items()
        }

    def score(self, query: str, doc_idx: int) -> float:
        doc = self.documents[doc_idx]
        doc_len = self.doc_lengths[doc_idx]
        query_terms = query.lower().split()
        doc_terms = doc.lower().split()
        doc_term_counts = Counter(doc_terms)

        score = 0.0
        for term in query_terms:
            if term in self.idf:
                tf = doc_term_counts.get(term, 0)
                score += (
                    self.idf[term]
                    * tf * (self.k1 + 1)
                    / (tf + self.k1 * (1 - self.b + self.b * doc_len / self.avgdl))
                )
        return score

    def retrieve(self, query: str, k: int = 5) -> list[tuple[str, float]]:
        scores = [(i, self.score(query, i)) for i in range(len(self.documents))]
        scores.sort(key=lambda x: x[1], reverse=True)
        return [(self.documents[i], s) for i, s in scores[:k]]

class HybridRetriever:
    def __init__(self, alpha=0.5):
        self.alpha = alpha
        self.bm25 = BM25()
        self.dense = DenseRetriever()
        self.documents = []

    def index(self, documents):
        self.documents = documents
        self.bm25.fit(documents)
        self.dense.index(documents)

    def retrieve(self, query: str, k: int = 5) -> list[tuple[str, float]]:
        bm25_results = dict(self.bm25.retrieve(query, k=k*2))
        dense_results = dict(self.dense.retrieve(query, k=k*2))

        all_docs = set(list(bm25_results.keys()) + list(dense_results.keys()))

        # Normalize scores
        bm25_scores = np.array(list(bm25_results.values()))
        dense_scores = np.array(list(dense_results.values()))

        bm25_norm = (bm25_scores - bm25_scores.min()) / (
            bm25_scores.max() - bm25_scores.min() + 1e-8
        ) if len(bm25_scores) > 0 else np.array([])

        dense_norm = (dense_scores - dense_scores.min()) / (
            dense_scores.max() - dense_scores.min() + 1e-8
        ) if len(dense_scores) > 0 else np.array([])

        # Reciprocal Rank Fusion fallback
        hybrid = {}
        for doc in all_docs:
            rank_bm25 = (
                list(bm25_results.keys()).index(doc)
                if doc in bm25_results else len(bm25_results)
            )
            rank_dense = (
                list(dense_results.keys()).index(doc)
                if doc in dense_results else len(dense_results)
            )
            hybrid[doc] = (1 / (60 + rank_bm25)) + (1 / (60 + rank_dense))

        ranked = sorted(hybrid.items(), key=lambda x: x[1], reverse=True)
        return ranked[:k]

retriever = HybridRetriever(alpha=0.5)
retriever.index(["Document 1...", "Document 2...", "Document 3..."])
results = retriever.retrieve("query", k=3)
```

---

## 9. Full Context Packing

```python
class ContextPacker:
    def __init__(self, model_limit: int = 128_000, output_reserve: int = 4_000):
        self.model_limit = model_limit
        self.output_reserve = output_reserve
        self.input_budget = model_limit - output_reserve

    def pack(self, system_prompt: str, retrieved_docs: list[str],
             conversation_history: list[dict], memory_summary: str,
             current_query: str) -> str:

        # 1. Always include system prompt
        context = f"<system>\n{system_prompt}\n</system>\n\n"

        # 2. Memory summary (if relevant)
        if memory_summary:
            context += f"<memory>\n{memory_summary}\n</memory>\n\n"

        # 3. Retrieved documents (starting with most relevant)
        context += "<documents>\n"
        for i, doc in enumerate(retrieved_docs):
            context += f"  <doc id={i+1}>{doc}</doc>\n"
        context += "</documents>\n\n"

        # 4. Conversation history (recent at end)
        context += "<history>\n"
        for msg in conversation_history[-10:]:
            context += f"<{msg['role']}>{msg['content']}</{msg['role']}>\n"
        context += "</history>\n\n"

        # 5. Current query
        context += f"<query>{current_query}</query>"

        # 6. Truncate if needed
        tokens = count_tokens(context)
        if tokens > self.input_budget:
            # Strategy: truncate from the middle (history) first
            return self._truncate_from_middle(context)

        return context

    def _truncate_from_middle(self, context: str) -> str:
        """Remove history entries from the middle until under budget"""
        parts = context.split("<history>")
        if len(parts) != 2:
            return context[:int(len(context) * self.input_budget / count_tokens(context))]

        before_history = parts[0]
        after_history = parts[1].split("</history>", 1)
        history_content = after_history[0]
        after = after_history[1]

        history_lines = history_content.strip().split("\n")

        while count_tokens(f"{before_history}<history>{history_content}</history>{after}") > self.input_budget:
            if len(history_lines) <= 2:
                break
            # Remove middle history entries
            mid = len(history_lines) // 2
            history_lines = history_lines[:mid-1] + history_lines[mid+1:]

        return f"{before_history}<history>\n" + "\n".join(history_lines) + f"\n</history>{after}"
```
