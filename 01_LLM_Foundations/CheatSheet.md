# Cheat Sheet — Chapter 01: LLM Foundations

## Tokenization Quick Reference

| Concept | Detail |
|---------|--------|
| Algorithm | Byte Pair Encoding (BPE), WordPiece, Unigram, SentencePiece |
| 1 token ≈ | 4 characters (English), 1 word (most), 3/4 word (average) |
| GPT-4 vocab | ~100,000 tokens |
| LLaMA vocab | 32,000 tokens |
| Tiktoken | `tiktoken.encoding_for_model("gpt-4")` |
| HuggingFace | `AutoTokenizer.from_pretrained("model-name")` |
| Special tokens | `<|endoftext|>`, `<|im_start|>`, `<|im_end|>`, `<s>`, `</s>`, `[CLS]`, `[SEP]` |

### Token Count Estimates

| Text | Approx. Tokens |
|------|----------------|
| "Hello" | 1 |
| 1 sentence (15 words) | ~20 |
| 1 paragraph (100 words) | ~130 |
| 1 page (500 words) | ~650 |
| 1 email | ~200-500 |
| 1 chat message | ~50-200 |
| This cheat sheet | ~800 |

---

## Context Windows

| Model | Max Context |
|-------|-------------|
| GPT-4o / GPT-4o-mini | 128K tokens |
| o1 / o3 | 200K tokens |
| Claude 3.5 Haiku / Sonnet | 200K tokens |
| Claude Opus 4 | 200K tokens |
| Gemini 2.0 Flash | 1M tokens |
| Gemini 2.5 Pro | 1M tokens |
| LLaMA 4 | 128K tokens |
| Mistral 7B | 32K tokens |
| Mistral Large | 128K tokens |
| DeepSeek V3 / R1 | 128K tokens |
| Qwen 3 72B | 131K tokens |
| Phi-4 | 16K tokens |

### Context Window Math

```
Available = model max - system_prompt - user_input
Leave 20% buffer: Available * 0.8
```

---

## Sampling Parameters

| Parameter | Range | Effect | Recommended |
|-----------|-------|--------|-------------|
| Temperature | 0.0 - 2.0+ | Randomness | 0 (facts), 0.7 (creative) |
| Top-P | 0.0 - 1.0 | Nucleus sampling | 0.9 |
| Top-K | 1 - vocab_size | Top K tokens | 40-50 |
| Frequency Penalty | -2.0 - 2.0 | Discourage repetition | 0.1-0.3 |
| Presence Penalty | -2.0 - 2.0 | Encourage new topics | 0.1-0.3 |
| Max Tokens | 1 - context_limit | Output length limit | Task-dependent |
| Stop Sequences | List of strings | Early stopping | ["\n\n", "Human:"] |
| Seed | Integer | Reproducibility | Same seed + T=0 |

### Common Combinations

| Use Case | Temp | Top-P | Top-K |
|----------|------|-------|-------|
| Factual Q&A | 0.0 | 1.0 | — |
| Code generation | 0.2 | 0.9 | — |
| Creative writing | 0.8 | 0.95 | 40 |
| Brainstorming | 1.0 | 1.0 | — |
| Chatbot | 0.7 | 0.9 | — |
| Translation | 0.0 | 1.0 | — |

---

## Model Capabilities Matrix

| Capability | GPT-4o | Claude 3<sup>.</sup>5 Sonnet | Gemini 2<sup>.</sup>0 Flash | LLaMA 4 70B | DeepSeek V3 |
|-----------|--------|---------------------------|----------------------------|-------------|-------------|
| Chat | ★★★★★ | ★★★★★ | ★★★★ | ★★★★ | ★★★★ |
| Coding | ★★★★ | ★★★★★ | ★★★★ | ★★★★ | ★★★★ |
| Reasoning | ★★★★ | ★★★★ | ★★★★ | ★★★ | ★★★★ |
| Long context | ★★★★ | ★★★★★ | ★★★★★ | ★★★★ | ★★★★ |
| Multilingual | ★★★★★ | ★★★★ | ★★★★★ | ★★★ | ★★★★★ |
| Cost efficiency | ★★★ | ★★★ | ★★★★★ | ★★★★ | ★★★★★ |
| Speed | ★★★★★ | ★★★★ | ★★★★★ | ★★★ | ★★★★ |
| Safety | ★★★★★ | ★★★★★ | ★★★★ | ★★★ | ★★★ |

---

## Pricing (per 1M tokens, USD)

| Model | Input | Output | Output (reasoning) |
|-------|-------|--------|-------------------|
| GPT-4o | $2.50 | $10.00 | — |
| GPT-4o-mini | $0.15 | $0.60 | — |
| o1-mini | $1.10 | $4.40 | — |
| o3 | $10.00 | $40.00 | $40.00 |
| Claude Haiku | $0.25 | $1.25 | — |
| Claude Sonnet | $3.00 | $15.00 | — |
| Claude Opus 4 | $15.00 | $75.00 | $75.00 |
| Gemini Flash | $0.10 | $0.40 | — |
| Gemini Pro | $1.25 | $10.00 | — |
| DeepSeek V3 | $0.50 | $2.00 | — |
| DeepSeek R1 | $0.55 | $2.19 | $2.19 |
| Mistral Large | $2.00 | $6.00 | — |
| Qwen 3 72B | $0.90 | $2.70 | — |

---

## Embedding Models

| Model | Dimensions | Max Input | Cost/1M tokens | Use Case |
|-------|-----------|-----------|----------------|----------|
| text-embedding-3-small | 1536 (256+) | 8K tokens | $0.02 | General purpose (cheap) |
| text-embedding-3-large | 3072 | 8K tokens | $0.13 | High quality |
| voyage-2 | 1024 | 8K tokens | $0.09 | Code-aware |
| cohere-embed-v3 | 1024 | 8K tokens | $0.10 | Multilingual |
| BGE (BAAI/bge-base-en) | 768 | 512 tokens | Free (local) | Open source |

---

## Key Formulas

### Cosine Similarity
```
cos_sim(a, b) = (a · b) / (||a|| × ||b||)
```

### Scaled Dot-Product Attention
```
Attention(Q, K, V) = softmax(QK^T / √d_k) · V
```

### Cross-Entropy Loss
```
L = -∑ P(target) × log(P(predicted))
```

### Perplexity
```
PPL = exp(L)  where L is average cross-entropy loss
```

### Temperature-Softmax
```
P(i) = exp(x_i / T) / ∑ exp(x_j / T)
```

---

## Hardware Requirements (Local Inference)

| Model Size | FP16 VRAM | INT4 VRAM | Typical GPU |
|-----------|-----------|-----------|-------------|
| 1B | 2 GB | 0.5 GB | CPU / Any |
| 7B | 14 GB | 4 GB | RTX 3090 / 4090 |
| 8B | 16 GB | 5 GB | RTX 3090 / 4090 |
| 13B | 26 GB | 7 GB | A100 40GB |
| 30B | 60 GB | 16 GB | A100 80GB |
| 70B | 140 GB | 35 GB | 2× A100 |
| 180B | 360 GB | 90 GB | 4× H100 |

---

## Common Gotchas

| Gotcha | Explanation |
|--------|-------------|
| Tokens ≠ Words | "I love AI" = 3 tokens. "unconditionally" = 5 tokens |
| T=0 is deterministic | Same input → same output every time |
| T>1 is risky | Output may be gibberish |
| Streaming ≠ faster | Perceived speed improves; total time same |
| Context window is tokens | Not characters, not words |
| "Lost in the middle" | Model pays less attention to middle of context |
| API pricing is per token | Both input and output tokens cost |
| System prompt matters | It sets behavior; don't waste it on examples |
| Fine-tuning ≠ knowledge injection | Use RAG for facts, fine-tune for format/behavior |

---

## Quick API Patterns

### OpenAI
```python
client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Hello"}],
    temperature=0,
)
```

### Anthropic
```python
client.messages.create(
    model="claude-3-5-sonnet-20241022",
    max_tokens=100,
    messages=[{"role": "user", "content": "Hello"}],
)
```

### Gemini
```python
model = genai.GenerativeModel("gemini-2.0-flash")
model.generate_content("Hello")
```

### Ollama (Local)
```python
ollama.generate(model="llama3.2", prompt="Hello")
```
