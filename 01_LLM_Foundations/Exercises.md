# Exercises — Chapter 01: LLM Foundations

## Beginner Exercises

### Exercise 1: Token Counting with tiktoken

Count tokens for various texts using OpenAI's `tiktoken` library.

```python
import tiktoken

# Initialize the tokenizer for GPT-4
enc = tiktoken.encoding_for_model("gpt-4")

# Count tokens for these texts
texts = [
    "Hello, world!",
    "The quick brown fox jumps over the lazy dog.",
    "A" * 1000,
    "I am " + "very " * 100 + "excited about LLMs.",
]

for text in texts:
    tokens = enc.encode(text)
    print(f"Chars: {len(text):6d} | Tokens: {len(tokens):4d} | Ratio: {len(tokens)/len(text):.3f}")
```

**Tasks:**
1. Run the code above and note the character-to-token ratio for each text.
2. Find a text where token count is *larger* than character count (hint: try non-ASCII characters).
3. What happens when you encode "    " (4 spaces)? Does it match your intuition?

### Exercise 2: Token Visualization

Write code to visualize how a sentence is split into tokens with their token IDs:

```python
def visualize_tokens(text):
    enc = tiktoken.encoding_for_model("gpt-4")
    tokens = enc.encode(text)
    decoded_tokens = [enc.decode([t]) for t in tokens]
    
    print(f"Original: {text}")
    print(f"Tokens: {decoded_tokens}")
    print(f"IDs: {tokens}")
    print(f"Count: {len(tokens)}")
    
    # Show the token boundaries
    print("\nVisual:")
    for t in decoded_tokens:
        # Display non-printable chars
        display = t.replace("\n", "\\n").replace("\r", "\\r").replace("\t", "\\t")
        print(f"[{display}]", end="")
    print()

# Test with various inputs
visualize_tokens("I love AI")
visualize_tokens("Hello, how are you today?")
visualize_tokens("unconditionally")
```

**Tasks:**
1. Why does " love" (with space) differ from "love" (without space)?
2. Try visualizing a URL like "https://example.com/path/to/resource". How is it tokenized?
3. Try code snippets in Python/JavaScript. Do keywords tokenize differently from variable names?

### Exercise 3: Embedding Similarity

Use OpenAI embeddings to explore semantic similarity:

```python
import openai
import numpy as np

client = openai.OpenAI()

def get_embedding(text):
    resp = client.embeddings.create(model="text-embedding-3-small", input=text)
    return np.array(resp.data[0].embedding)

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

words = ["king", "queen", "man", "woman", "pancake", "breakfast", "syrup", "nuclear"]
embeddings = {w: get_embedding(w) for w in words}

# Compute similarity matrix
for w1 in words:
    for w2 in words:
        if w1 < w2:
            sim = cosine_similarity(embeddings[w1], embeddings[w2])
            print(f"{w1:12s} × {w2:12s} → {sim:.4f}")
```

**Tasks:**
1. Which pair has the highest similarity? The lowest?
2. Compute `embedding("king") - embedding("man") + embedding("woman")`. Find the closest word to this vector.
3. Group the words into clusters using similarity > 0.3.

### Exercise 4: Temperature Exploration

Experiment with how temperature affects output:

```python
import openai

client = openai.OpenAI()

prompt = "Write a one-sentence tagline for a bakery."

for temperature in [0.0, 0.3, 0.7, 1.0, 1.5, 2.0]:
    # Generate 3 samples at each temperature
    print(f"\n--- Temperature: {temperature} ---")
    for i in range(3):
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            temperature=temperature,
            max_tokens=30,
        )
        print(f"  {i+1}. {resp.choices[0].message.content}")
```

**Tasks:**
1. At temperature 0.0, are all 3 outputs identical? Why?
2. At temperature 2.0, does the output make sense? What goes wrong?
3. For a factual task ("What is the capital of France?"), which temperature is appropriate?

### Exercise 5: Top-P vs Temperature

Compare the effects of top-p and temperature:

```python
import openai

client = openai.OpenAI()
prompt = "List 3 unusual pizza toppings."

configs = [
    {"temp": 0.0, "top_p": 1.0},
    {"temp": 1.0, "top_p": 0.1},
    {"temp": 1.0, "top_p": 0.5},
    {"temp": 1.0, "top_p": 1.0},
    {"temp": 2.0, "top_p": 1.0},
]

for cfg in configs:
    print(f"\n--- temp={cfg['temp']}, top_p={cfg['top_p']} ---")
    for i in range(2):
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            temperature=cfg["temp"],
            top_p=cfg["top_p"],
            max_tokens=100,
        )
        print(f"  {i+1}. {resp.choices[0].message.content}")
```

**Tasks:**
1. What happens with top_p=0.1? Why are outputs so similar?
2. What combination gives the most creative (but still coherent) outputs?
3. Explain the interaction between temperature and top_p.

---

## Intermediate Exercises

### Exercise 6: Cosine Similarity from Scratch

Implement cosine similarity and find the most similar pair in a set of sentences:

```python
import numpy as np

def cosine_similarity(a, b):
    """Compute cosine similarity between two vectors."""
    dot_product = sum(ai * bi for ai, bi in zip(a, b))
    norm_a = sum(ai ** 2 for ai in a) ** 0.5
    norm_b = sum(bi ** 2 for bi in b) ** 0.5
    return dot_product / (norm_a * norm_b + 1e-10)
```

**Tasks:**
1. Test your implementation against `sklearn.metrics.pairwise.cosine_similarity`
2. Find the most semantically similar pair from a list of 10 movie reviews
3. Implement a function `most_similar(word, candidates, embeddings)` that returns the top-3 most similar words

### Exercise 7: Simple Attention Implementation

Implement scaled dot-product attention from scratch using numpy:

```python
import numpy as np

def softmax(x, axis=-1):
    exp_x = np.exp(x - np.max(x, axis=axis, keepdims=True))
    return exp_x / np.sum(exp_x, axis=axis, keepdims=True)

def scaled_dot_product_attention(Q, K, V):
    """
    Q: (batch, seq_len_q, d_k)
    K: (batch, seq_len_k, d_k)
    V: (batch, seq_len_v, d_k) — note: seq_len_v = seq_len_k
    """
    d_k = Q.shape[-1]
    
    # Step 1: Compute Q @ K^T
    scores = np.matmul(Q, K.transpose(0, 2, 1))  # (batch, q_len, k_len)
    
    # Step 2: Scale
    scores = scores / np.sqrt(d_k)
    
    # Step 3: Apply softmax
    weights = softmax(scores)
    
    # Step 4: Weighted sum of values
    output = np.matmul(weights, V)  # (batch, q_len, d_v)
    
    return output, weights

# Test with random vectors
batch_size, seq_len, d_k = 2, 4, 8
np.random.seed(42)
Q = np.random.randn(batch_size, seq_len, d_k)
K = np.random.randn(batch_size, seq_len, d_k)
V = np.random.randn(batch_size, seq_len, d_k)

output, weights = scaled_dot_product_attention(Q, K, V)
print(f"Output shape: {output.shape}")
print(f"Attention weights shape: {weights.shape}")
print(f"\nAttention weights for first batch, first query:")
print(weights[0, 0].round(3))
```

**Tasks:**
1. Verify that attention weights sum to 1 for each query
2. What happens if you remove the scaling factor (√d_k)?
3. Implement a causal mask (upper triangular = -inf) for decoder-style attention

### Exercise 8: Multi-Head Attention

Extend your attention implementation to support multiple heads:

```python
def multi_head_attention(Q, K, V, num_heads=8):
    """
    Q, K, V: (batch, seq_len, d_model)
    Returns: (batch, seq_len, d_model)
    """
    batch_size, seq_len, d_model = Q.shape
    d_k = d_model // num_heads
    
    # Project and reshape to (batch, heads, seq_len, d_k)
    # (Simplified: in reality you'd use learned projection matrices)
    
    def split_heads(x):
        return x.reshape(batch_size, seq_len, num_heads, d_k).transpose(0, 2, 1, 3)
    
    Q_split = split_heads(Q)
    K_split = split_heads(K)
    V_split = split_heads(V)
    
    # Apply attention to each head
    output, weights = scaled_dot_product_attention(Q_split, K_split, V_split)
    
    # Concatenate heads
    output = output.transpose(0, 2, 1, 3).reshape(batch_size, seq_len, d_model)
    return output
```

**Tasks:**
1. Verify that the output shape matches the input shape
2. Compare single-head vs multi-head attention outputs
3. Add learned projection matrices (W_Q, W_K, W_V) to make it a proper implementation

### Exercise 9: Tokenization — BPE Algorithm from Scratch

Implement a simplified BPE tokenizer:

```python
from collections import Counter
import re

def get_stats(vocab):
    """Count all adjacent pairs in the vocabulary."""
    pairs = Counter()
    for word, freq in vocab.items():
        symbols = word.split()
        for i in range(len(symbols) - 1):
            pairs[(symbols[i], symbols[i + 1])] += freq
    return pairs

def merge_vocab(pair, vocab):
    """Merge the most frequent pair in vocabulary."""
    merged = {}
    bigram = " ".join(pair)
    replacement = "".join(pair)
    for word, freq in vocab.items():
        new_word = word.replace(bigram, replacement)
        merged[new_word] = freq
    return merged

# Create initial vocabulary (word-level with character splits)
text = "low low low low low low low lower lower newest newest newest newest newest newest widest widest widest"
vocab = {word: freq for word, freq in Counter(text.split()).items()}

# Split each word into characters
vocab = {" ".join(word): freq for word, freq in vocab.items()}

print("Initial vocab:", dict(vocab))

# BPE training loop
num_merges = 5
for i in range(num_merges):
    pairs = get_stats(vocab)
    if not pairs:
        break
    best_pair = pairs.most_common(1)[0][0]
    vocab = merge_vocab(best_pair, vocab)
    print(f"Merge {i+1}: {best_pair} → {''.join(best_pair)}")
    print(f"  Vocab size: {len(vocab)}")
```

**Tasks:**
1. What are the first 5 merges? Do they make intuitive sense?
2. Change the training text to include numbers. How do the merges change?
3. Add a function to encode new words using the learned merges.

### Exercise 10: Perplexity Calculation

Implement perplexity and compare model confidence:

```python
import numpy as np
import openai

client = openai.OpenAI()

def get_logprobs(prompt, continuation):
    """Get log probabilities of continuation given prompt."""
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "Complete the following."},
            {"role": "user", "content": prompt},
        ],
        max_tokens=30,
        logprobs=True,
        top_logprobs=5,
    )
    
    logprobs = []
    for token in response.choices[0].logprobs.content:
        logprobs.append(token.logprob)
    return logprobs

def perplexity(logprobs):
    avg_logprob = np.mean(logprobs)
    return np.exp(-avg_logprob)

# Test with expected vs unexpected continuations
prompts = [
    ("The sky is ", "blue"),
    ("The sky is ", "purple"),
    ("2 + 2 = ", "4"),
    ("2 + 2 = ", "5"),
]

for prompt, continuation in prompts:
    lp = get_logprobs(prompt, continuation)
    ppl = perplexity(lp)
    print(f"'{prompt}{continuation}' → perplexity: {ppl:.2f}")
```

**Tasks:**
1. Which continuation has lower perplexity (higher confidence)?
2. Add completions with different lengths. How does perplexity normalize for length?
3. What happens with grammatically correct but semantically unlikely sentences?

### Exercise 11: Greedy vs Sampling Decoding

Compare different decoding strategies:

```python
import numpy as np

# Simulated logits for next token prediction
logits = np.array([2.0, 1.5, 0.5, 0.0, -0.5, -1.0, -2.0, -3.0])

def softmax(x, temperature=1.0):
    x = x / temperature
    exp_x = np.exp(x - np.max(x))
    return exp_x / exp_x.sum()

def greedy_decoding(logits):
    return np.argmax(logits)

def sampling_decoding(logits, temperature=1.0):
    probs = softmax(logits, temperature)
    return np.random.choice(len(probs), p=probs)

def top_k_sampling(logits, k=3, temperature=1.0):
    top_k_indices = np.argsort(logits)[-k:]
    top_k_logits = logits[top_k_indices]
    probs = softmax(top_k_logits, temperature)
    chosen = np.random.choice(len(probs), p=probs)
    return top_k_indices[chosen]

# Demonstrate
print("Greedy:", greedy_decoding(logits))
print("\nSampling (10 runs):")
for temp in [0.5, 1.0, 2.0]:
    results = [sampling_decoding(logits, temp) for _ in range(20)]
    print(f"  temp={temp}: {results}")

print("\nTop-K sampling (10 runs, k=3):")
results = [top_k_sampling(logits, k=3, temperature=1.0) for _ in range(20)]
print(f"  {results}")
```

**Tasks:**
1. Why does greedy decoding always pick the same token?
2. At temperature 2.0, do you see tokens with negative logits being selected?
3. Implement top-p (nucleus) sampling.

---

## Advanced Exercises

### Exercise 12: Tiny Transformer Forward Pass

Build a minimal Transformer block forward pass:

```python
import numpy as np

class LayerNorm:
    def __init__(self, d_model, eps=1e-5):
        self.gamma = np.ones(d_model)
        self.beta = np.zeros(d_model)
        self.eps = eps
    
    def __call__(self, x):
        mean = x.mean(axis=-1, keepdims=True)
        var = x.var(axis=-1, keepdims=True)
        return self.gamma * (x - mean) / np.sqrt(var + self.eps) + self.beta

class FeedForward:
    def __init__(self, d_model, d_ff):
        self.W1 = np.random.randn(d_model, d_ff) / np.sqrt(d_model)
        self.b1 = np.zeros(d_ff)
        self.W2 = np.random.randn(d_ff, d_model) / np.sqrt(d_ff)
        self.b2 = np.zeros(d_model)
    
    def __call__(self, x):
        return x @ self.W1 @ self.W2 + self.b1 @ self.W2 + self.b2
        # Approximate: x @ W1 + b1 → ReLU → (x @ W1 + b1) @ W2 + b2

class TransformerBlock:
    def __init__(self, d_model, d_ff, num_heads):
        self.attention = MultiHeadAttention(d_model, num_heads)
        self.ffn = FeedForward(d_model, d_ff)
        self.norm1 = LayerNorm(d_model)
        self.norm2 = LayerNorm(d_model)
    
    def __call__(self, x):
        # Attention with residual
        attn_out = self.attention(x)
        x = x + self.norm1(attn_out)
        # FFN with residual
        ffn_out = self.ffn(x)
        x = x + self.norm2(ffn_out)
        return x

# Test
d_model, d_ff, num_heads = 64, 256, 8
batch, seq_len = 2, 10
x = np.random.randn(batch, seq_len, d_model)
block = TransformerBlock(d_model, d_ff, num_heads)
output = block(x)
print(f"Input shape: {x.shape}, Output shape: {output.shape}")
```

**Tasks:**
1. Verify that the input and output shapes match
2. Remove residual connections. How does training behavior change?
3. Add dropout between sublayers

### Exercise 13: KV Cache Implementation

Implement a simple KV cache for efficient autoregressive generation:

```python
import numpy as np

class KVCache:
    def __init__(self, max_seq_len, d_model, num_heads):
        self.max_seq_len = max_seq_len
        self.d_k = d_model // num_heads
        self.num_heads = num_heads
        self.cache = None  # Will store (K, V) tuples per layer
    
    def update(self, layer_idx, K_new, V_new):
        """
        K_new, V_new: (batch, num_heads, 1, d_k) — single new token
        """
        if self.cache is None:
            self.cache = {}
        
        if layer_idx not in self.cache:
            # First token — initialize cache
            self.cache[layer_idx] = (K_new, V_new)
            return K_new, V_new
        
        # Append to existing cache
        K_old, V_old = self.cache[layer_idx]
        K = np.concatenate([K_old, K_new], axis=2)
        V = np.concatenate([V_old, V_new], axis=2)
        self.cache[layer_idx] = (K, V)
        return K, V
    
    def get(self, layer_idx):
        if self.cache and layer_idx in self.cache:
            return self.cache[layer_idx]
        return None

# Demonstrate KV cache savings
def simulate_generation(seq_len, use_cache=True):
    cache = KVCache(seq_len, 64, 8)
    
    # Without KV cache: full recomputation
    flops_without = sum(i * 64 for i in range(1, seq_len + 1))
    
    # With KV cache: only compute for new tokens
    flops_with = seq_len * 64  # Only compute Q for new tokens
    
    savings = (1 - flops_with / flops_without) * 100
    print(f"Sequence length: {seq_len}")
    print(f"FLOPS without cache: {flops_without}")
    print(f"FLOPS with cache: {flops_with}")
    print(f"Savings: {savings:.1f}%")

simulate_generation(100)
simulate_generation(1000)
```

**Tasks:**
1. How do savings scale with sequence length?
2. Memory requirements: how much memory does the cache use for a 70B model?
3. Implement cache eviction strategies for very long sequences

### Exercise 14: Training Loop Simulation

Simulate a minimal training loop with gradient descent:

```python
import numpy as np

class TinyLM:
    def __init__(self, vocab_size, d_model):
        self.embed = np.random.randn(vocab_size, d_model) * 0.01
        self.W_out = np.random.randn(d_model, vocab_size) * 0.01
    
    def forward(self, token_ids):
        """Simple bigram model: predict next token from current."""
        # token_ids: (batch, seq_len)
        batch, seq_len = token_ids.shape
        embeddings = self.embed[token_ids]  # (batch, seq_len, d_model)
        
        # Use last token's embedding to predict next
        last_emb = embeddings[:, -1, :]  # (batch, d_model)
        logits = last_emb @ self.W_out  # (batch, vocab_size)
        
        # Softmax
        logits -= logits.max(axis=-1, keepdims=True)
        probs = np.exp(logits) / np.exp(logits).sum(axis=-1, keepdims=True)
        return probs, (embeddings, last_emb)
    
    def compute_loss(self, probs, target_ids):
        """Cross-entropy loss."""
        batch = probs.shape[0]
        correct_logprobs = -np.log(probs[np.arange(batch), target_ids] + 1e-10)
        return correct_logprobs.mean()

# Generate synthetic data
vocab_size, d_model = 20, 16
model = TinyLM(vocab_size, d_model)

# Simple training data: sequences of random token IDs
np.random.seed(42)
data = np.random.randint(0, vocab_size, (100, 5))
inputs = data[:, :-1]
targets = data[:, -1]

# Training loop
learning_rate = 0.01
for step in range(50):
    probs, cache = model.forward(inputs)
    loss = model.compute_loss(probs, targets)
    
    # Simple gradient descent (approximate)
    grad = probs
    grad[np.arange(len(targets)), targets] -= 1
    grad = grad / len(targets)
    
    model.W_out -= learning_rate * cache[1].T @ grad
    
    if step % 10 == 0:
        print(f"Step {step}: loss = {loss:.4f}")
```

**Tasks:**
1. How does the loss change over time?
2. Increase vocab_size to 100. How does this affect training?
3. Add a proper backward pass with full gradients

### Exercise 15: Chain-of-Thought Demonstration

Compare standard vs. chain-of-thought prompting for reasoning:

```python
import openai

client = openai.OpenAI()

problems = [
    "If a store has a 25% off sale and then an additional 10% off the sale price, what is the total discount percentage?",
    "Alice is twice as old as Bob was when Alice was as old as Bob is now. If Bob is 30, how old is Alice?",
    "How many Rs are in the word 'strawberry'?",
]

for problem in problems:
    print(f"\n=== Problem: {problem} ===")
    
    # Standard prompt
    resp_standard = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": problem}],
        temperature=0,
    )
    print(f"\nStandard:\n{resp_standard.choices[0].message.content}")
    
    # CoT prompt
    resp_cot = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "user", "content": f"{problem}\n\nLet's think through this step by step:"}
        ],
        temperature=0,
    )
    print(f"\nChain-of-Thought:\n{resp_cot.choices[0].message.content}")
```

**Tasks:**
1. Does CoT give better answers? For which problems?
2. Try temperature > 0 with CoT. Does it help or hurt?
3. Implement self-consistency: run CoT 5 times and take the majority answer

### Exercise 16: Model Comparison Benchmark

Write a function to compare multiple models on the same task:

```python
import openai
from anthropic import Anthropic
import time

def benchmark_models(prompt, models):
    """
    Compare models on latency, output length, and quality.
    Note: You'll need API keys for each provider.
    """
    results = []
    
    for model_name, client, kwargs in models:
        start = time.time()
        
        try:
            resp = client.chat.completions.create(
                model=model_name,
                messages=[{"role": "user", "content": prompt}],
                max_tokens=100,
                temperature=0,
            )
            latency = time.time() - start
            output = resp.choices[0].message.content
            tokens_out = resp.usage.completion_tokens
            
            results.append({
                "model": model_name,
                "latency": latency,
                "output_length": len(output),
                "tokens": tokens_out,
                "tokens_per_sec": tokens_out / latency,
                "output": output[:200],
            })
        except Exception as e:
            results.append({"model": model_name, "error": str(e)})
    
    return results

# Usage (requires API keys):
"""
results = benchmark_models(
    "Explain what an LLM is in 2 sentences.",
    [
        ("gpt-4o-mini", openai_client, {}),
        ("gpt-4o", openai_client, {}),
        ("claude-3-5-sonnet-20241022", anthropic_client, {}),
    ]
)

for r in results:
    print(f"{r['model']}: {r.get('latency', 0):.2f}s | {r.get('tokens_per_sec', 0):.1f} tok/s")
"""
```

**Tasks:**
1. Compare GPT-4o-mini vs GPT-4o on factual accuracy
2. Measure latency over 10 runs and compute statistics
3. Create a leaderboard: cost per token vs quality score

### Exercise 17: Embedding — Vector Algebra

Demonstrate the famous word vector arithmetic:

```python
import numpy as np
import openai

client = openai.OpenAI()

def get_embedding(text):
    resp = client.embeddings.create(model="text-embedding-3-small", input=text)
    return np.array(resp.data[0].embedding)

def find_closest(vector, candidates, n=5):
    """Find the n closest words to a given vector."""
    similarities = []
    for word, emb in candidates.items():
        sim = np.dot(vector, emb) / (np.linalg.norm(vector) * np.linalg.norm(emb))
        similarities.append((word, sim))
    similarities.sort(key=lambda x: x[1], reverse=True)
    return similarities[:n]

words = ["king", "queen", "man", "woman", "boy", "girl", "prince", "princess",
         "uncle", "aunt", "paris", "france", "rome", "italy", "tokyo", "japan"]

embeddings = {w: get_embedding(w) for w in words}

# King - Man + Woman ≈ ?
result = embeddings["king"] - embeddings["man"] + embeddings["woman"]
print("Closest to king - man + woman:")
for word, sim in find_closest(result, embeddings):
    print(f"  {word}: {sim:.4f}")

# Paris - France + Italy ≈ ?
result2 = embeddings["paris"] - embeddings["france"] + embeddings["italy"]
print("\nClosest to paris - france + italy:")
for word, sim in find_closest(result2, embeddings):
    print(f"  {word}: {sim:.4f}")
```

**Tasks:**
1. Do the analogies work? (king - man + woman ≈ queen?)
2. Try non-royal analogies: "doctor" - "man" + "woman"
3. What happens with abstract concepts (love, hate, fear)?

### Exercise 18: Streaming Response Handler

Implement a streaming response handler for real-time applications:

```python
import openai

client = openai.OpenAI()

def stream_chat(prompt, model="gpt-4o-mini"):
    """Stream a chat response token by token."""
    stream = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}],
        stream=True,
    )
    
    collected = []
    for chunk in stream:
        if chunk.choices[0].delta.content:
            token = chunk.choices[0].delta.content
            collected.append(token)
            # In a real app, yield to UI
            print(token, end="", flush=True)
    
    print()
    return "".join(collected)

# Test
output = stream_chat("Write a haiku about artificial intelligence.")
print(f"\n\nTotal length: {len(output)} chars")
```

**Tasks:**
1. Modify to track per-token timing and report tokens/sec
2. Add a callback mechanism for real-time UI updates
3. Implement a backpressure mechanism for rate-limited APIs

---

## Bonus Challenge

### Exercise 19: Build a Tiny GPT from Scratch

Follow Andrej Karpathy's `gpt.py` pattern. Implement:
1. Token and position embeddings
2. One transformer block (attention + FFN + residual + layernorm)
3. A language model head
4. Training on a tiny dataset (e.g., Shakespeare, Linux source code)

Hint: Start with Karpathy's "Let's build GPT" tutorial and simplify to ~100 lines.

### Exercise 20: Token-Level Cost Estimator

Build a function that estimates the cost of generating text with different models:

```python
MODEL_PRICING = {
    "gpt-4o": {"input": 2.50, "output": 10.00},
    "gpt-4o-mini": {"input": 0.15, "output": 0.60},
    "claude-3-5-sonnet": {"input": 3.00, "output": 15.00},
    "claude-3-5-haiku": {"input": 0.25, "output": 1.25},
    "gemini-2.0-flash": {"input": 0.10, "output": 0.40},
}

def estimate_cost(text, output_length_tokens=500, model="gpt-4o"):
    """Estimate the cost of a single API call (in cents)."""
    import tiktoken
    enc = tiktoken.encoding_for_model(model)
    input_tokens = len(enc.encode(text))
    
    pricing = MODEL_PRICING.get(model, {"input": 1.00, "output": 2.00})
    cost = (input_tokens / 1_000_000 * pricing["input"] + 
            output_length_tokens / 1_000_000 * pricing["output"])
    
    return {
        "input_tokens": input_tokens,
        "output_tokens": output_length_tokens,
        "cost_usd": cost,
        "model": model,
    }

# Tasks:
# 1. Compare cost of analyzing a 10-page document (30K tokens) with GPT-4o vs GPT-4o-mini
# 2. How many queries can you run for $10 with each model?
# 3. Add token caching (if input repeats, some providers discount)
```
