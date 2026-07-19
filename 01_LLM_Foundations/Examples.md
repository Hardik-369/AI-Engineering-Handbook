# Code Examples — Chapter 01: LLM Foundations

All examples assume you have API keys set in your environment. Install dependencies:

```bash
pip install openai anthropic google-genai tiktoken ollama numpy
```

---

## 1. Tokenizing Text with tiktoken

```python
import tiktoken

# Get tokenizer for a specific model
enc = tiktoken.encoding_for_model("gpt-4o")

text = "Hello, world! How are you today?"

# Encode text → token IDs
token_ids = enc.encode(text)
print(f"Text: {text}")
print(f"Token IDs: {token_ids}")
print(f"Token count: {len(token_ids)}")

# Decode token IDs → text
decoded = enc.decode(token_ids)
print(f"Decoded: {decoded}")

# Decode individual tokens (shows how text is split)
for tid in token_ids:
    token_text = enc.decode([tid]).replace(" ", "·")
    print(f"  ID {tid:6d} → '{token_text}'")

# Count tokens without encoding the whole string
count = len(enc.encode(text))
print(f"\nToken count: {count}")

# Handle special tokens
enc_with_special = tiktoken.encoding_for_model("gpt-4o")
im_start = enc_with_special.encode("<|im_start|>")
im_end = enc_with_special.encode("<|im_end|>")
print(f"<|im_start|> → {im_start}")
print(f"<|im_end|>   → {im_end}")
```

### Wrong Example

```python
# ❌ DON'T: Estimate tokens by splitting on spaces
text = "Hello, world!"
token_estimate = len(text.split())
print(f"Estimated: {token_estimate} tokens")       # 2 tokens
# GPT-4 actual: 4 tokens ["Hello", ",", " world", "!"]
```

### Correct Example

```python
# ✅ DO: Use the official tokenizer
import tiktoken
enc = tiktoken.encoding_for_model("gpt-4")
token_count = len(enc.encode(text))
```

---

## 2. Getting Embeddings with OpenAI

```python
import openai
import numpy as np

client = openai.OpenAI()

def get_embedding(text, model="text-embedding-3-small"):
    text = text.replace("\n", " ")
    resp = client.embeddings.create(input=[text], model=model)
    return np.array(resp.data[0].embedding)

def cosine_similarity(a, b):
    return np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

# Get embeddings for multiple texts
texts = [
    "I love programming in Python",
    "Python is my favorite language",
    "I enjoy eating pizza",
    "The weather today is beautiful",
]

embeddings = [get_embedding(t) for t in texts]

# Compare similarities
for i in range(len(texts)):
    for j in range(i + 1, len(texts)):
        sim = cosine_similarity(embeddings[i], embeddings[j])
        print(f"[{i}] vs [{j}]: {sim:.4f}")
        print(f"  '{texts[i][:40]}...'")
        print(f"  '{texts[j][:40]}...'")

# Matryoshka embeddings: truncate for speed
full_emb = get_embedding("Hello world", model="text-embedding-3-small")
print(f"\nFull embedding dimensions: {len(full_emb)}")

# Use first 256 dimensions (retains ~95% performance)
small_emb = full_emb[:256]
print(f"Truncated dimensions: {len(small_emb)}")
```

---

## 3. Simple Inference Call

### OpenAI

```python
import openai
client = openai.OpenAI()

resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "What is an LLM in one sentence?"},
    ],
    temperature=0.7,
    max_tokens=100,
)

print(resp.choices[0].message.content)
print(f"\nUsage: {resp.usage}")
```

### Anthropic

```python
import anthropic
client = anthropic.Anthropic()

resp = client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=100,
    system="You are a helpful assistant.",
    messages=[
        {"role": "user", "content": "What is an LLM in one sentence?"},
    ],
)

print(resp.content[0].text)
```

### Google Gemini

```python
import google.generativeai as genai
genai.configure(api_key="YOUR_API_KEY")

model = genai.GenerativeModel("gemini-2.0-flash")
resp = model.generate_content("What is an LLM in one sentence?")
print(resp.text)
```

---

## 4. Temperature and Top-P Exploration

```python
import openai
client = openai.OpenAI()

def generate_with_params(prompt, temperature=1.0, top_p=1.0, n=3):
    """Generate n responses with given sampling parameters."""
    responses = []
    for i in range(n):
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": prompt}],
            temperature=temperature,
            top_p=top_p,
            max_tokens=50,
        )
        responses.append(resp.choices[0].message.content)
    return responses

prompt = "Invent a name for a sci-fi alien species."

print("=== Temperature 0 (deterministic) ===")
for r in generate_with_params(prompt, temperature=0.0, n=5):
    print(f"  - {r}")

print("\n=== Temperature 0.7 (balanced) ===")
for r in generate_with_params(prompt, temperature=0.7, n=5):
    print(f"  - {r}")

print("\n=== Temperature 1.5 (creative, risky) ===")
for r in generate_with_params(prompt, temperature=1.5, n=5):
    print(f"  - {r}")

print("\n=== Top-P 0.1 (very focused) ===")
for r in generate_with_params(prompt, temperature=1.0, top_p=0.1, n=5):
    print(f"  - {r}")

print("\n=== Top-P 1.0 (full distribution) ===")
for r in generate_with_params(prompt, temperature=1.0, top_p=1.0, n=5):
    print(f"  - {r}")
```

### Key Observations

| Temperature | Top-P | Behavior |
|-------------|-------|----------|
| 0.0 | 1.0 | Always the same output, highest probability token |
| 0.5 | 1.0 | Slight variation, mostly high-probability tokens |
| 1.0 | 0.1 | Very focused, rarely deviates from top tokens |
| 1.0 | 1.0 | Full model distribution, balanced creativity |
| 2.0 | 1.0 | Near-random, often incoherent |
| 0.7 | 0.9 | Recommended default for creative tasks |

---

## 5. Text Completion vs Chat Completion

### Text Completion (Legacy, GPT-3 era)

```python
# OpenAI text completion (deprecated but illustrative)
import openai
client = openai.OpenAI()

# This was the original API — just complete text
resp = client.completions.create(
    model="gpt-3.5-turbo-instruct",
    prompt="Once upon a time, there was a",
    max_tokens=50,
    temperature=0.8,
)
print("Text completion:")
print(resp.choices[0].text)
```

### Chat Completion (Modern)

```python
import openai
client = openai.OpenAI()

# Chat-based: structured messages with roles
resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "You are a storyteller. Keep responses whimsical."},
        {"role": "user", "content": "Tell me a story about a robot learning to paint."},
    ],
    max_tokens=200,
    temperature=0.8,
)
print("Chat completion:")
print(resp.choices[0].message.content)
```

### Why Chat Completion Won

| Aspect | Text Completion | Chat Completion |
|--------|----------------|-----------------|
| Structure | Unstructured text | System/user/assistant roles |
| Control | Prompt engineering only | System message for behavior control |
| Multi-turn | Manual context management | Automatic history handling |
| Safety | No built-in guardrails | Content filtering built-in |

---

## 6. Local Model with Ollama

```python
import ollama

# List available models
models = ollama.list()
print("Available models:", [m["model"] for m in models["models"]])

# Basic generation
resp = ollama.generate(
    model="llama3.2",
    prompt="What is an LLM?",
)
print(resp["response"])

# Chat interface
resp = ollama.chat(
    model="llama3.2",
    messages=[
        {"role": "user", "content": "Explain transformers in 3 sentences."},
    ],
)
print(resp["message"]["content"])

# Streaming generation
stream = ollama.generate(
    model="llama3.2",
    prompt="Write a haiku about programming.",
    stream=True,
)
for chunk in stream:
    print(chunk["response"], end="", flush=True)
print()

# Structured output with JSON mode
resp = ollama.generate(
    model="llama3.2",
    prompt="List 3 programming languages. Return as JSON.",
    format="json",
)
print(resp["response"])
```

### Wrong Example

```python
# ❌ DON'T: Assume local models have the same capabilities as API models
# Local 7B models often struggle with:
# - Complex reasoning
# - Following instructions precisely
# - Long context
# - Multilingual tasks
```

### Correct Example

```python
# ✅ DO: Match model to task
# - Local 7B-8B: Summarization, simple Q&A, classification
# - Local 30B-70B: General tasks (with good GPU)
# - API models: Complex reasoning, multilingual, long documents
```

---

## 7. Complete Pipeline: Tokenize → Embed → Analyze

```python
import tiktoken
import openai
import numpy as np

client = openai.OpenAI()
enc = tiktoken.encoding_for_model("gpt-4o")

def analyze_text(text):
    """Full pipeline: count tokens, get embedding, compute statistics."""
    
    # Step 1: Tokenize
    token_ids = enc.encode(text)
    tokens = [enc.decode([t]) for t in token_ids]
    
    # Step 2: Get embedding
    embedding = np.array(
        client.embeddings.create(
            input=[text], model="text-embedding-3-small"
        ).data[0].embedding
    )
    
    return {
        "text": text,
        "char_count": len(text),
        "token_count": len(token_ids),
        "char_per_token": len(text) / len(token_ids),
        "tokens": list(zip(token_ids, tokens)),
        "embedding": embedding,
        "embedding_dim": len(embedding),
    }

# Analyze some texts
texts = [
    "Short text.",
    "A medium-length sentence about artificial intelligence and machine learning.",
    "The quick brown fox jumps over the lazy dog. " * 10,
]

for t in texts:
    result = analyze_text(t)
    print(f"\nText ({result['char_count']} chars): {t[:60]}...")
    print(f"  Tokens: {result['token_count']} ({result['char_per_token']:.2f} chars/token)")
    print(f"  Embedding dim: {result['embedding_dim']}")
```

---

## 8. Error Handling and Retry Logic

```python
import openai
import time
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(stop=stop_after_attempt(3), wait=wait_exponential(multiplier=1, min=2, max=10))
def safe_chat_completion(client, **kwargs):
    return client.chat.completions.create(**kwargs)

client = openai.OpenAI()

try:
    resp = safe_chat_completion(
        client=client,
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": "Hello!"}],
        max_tokens=10,
    )
    print(resp.choices[0].message.content)
    
except openai.RateLimitError:
    print("Rate limited — implement backoff")
except openai.APIError as e:
    print(f"API error: {e}")
except Exception as e:
    print(f"Unexpected error: {e}")
```

---

## 9. Batching for Efficiency

```python
import openai
client = openai.OpenAI()

# Single request (slower for many texts)
texts = ["Hello world", "AI is cool", "Python rocks"]

# Batch embedding request (faster!)
resp = client.embeddings.create(
    input=texts,
    model="text-embedding-3-small",
)

for i, data in enumerate(resp.data):
    embedding = data.embedding
    print(f"Text {i}: '{texts[i]}' → embedding dim {len(embedding)}")

# Batch chat completions (API-dependent)
from concurrent.futures import ThreadPoolExecutor

def chat(prompt):
    return client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        max_tokens=20,
    ).choices[0].message.content

prompts = [
    "Say hello in French",
    "Say hello in Spanish",
    "Say hello in German",
]

with ThreadPoolExecutor(max_workers=3) as pool:
    results = list(pool.map(chat, prompts))

for prompt, result in zip(prompts, results):
    print(f"{prompt}: {result}")
```

---

## 10. Logprob Inspection

```python
import openai
client = openai.OpenAI()

resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "user", "content": "The capital of France is"}
    ],
    max_tokens=5,
    logprobs=True,
    top_logprobs=5,
)

# Inspect top token probabilities at each position
for i, token_data in enumerate(resp.choices[0].logprobs.content):
    print(f"\nPosition {i}: '{token_data.token}' (logprob: {token_data.logprob:.3f})")
    print("  Top alternatives:")
    for alt in token_data.top_logprobs:
        prob = round(alt.logprob, 3)
        token_str = alt.token.replace("\n", "\\n")
        print(f"    '{token_str}' → logprob: {prob}")
```
