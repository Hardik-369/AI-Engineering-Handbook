# Interview Questions — Chapter 01: LLM Foundations

## Beginner Questions

### Q1: What is an LLM?
**Answer:** A Large Language Model is a neural network trained on massive text corpora to predict the next token in a sequence. It uses the Transformer architecture (specifically, the decoder-only variant) to process input tokens through multiple layers of self-attention and feed-forward networks. The model assigns a probability distribution over possible next tokens and generates text by sampling from this distribution.

**Key point:** LLMs do not "understand" or "think" — they compute statistical patterns in token sequences.

### Q2: What is tokenization and why does it matter?
**Answer:** Tokenization converts raw text into integers (token IDs) that the model can process. It uses algorithms like Byte Pair Encoding (BPE) to split text into subword units.

**Why it matters:**
- Determines a model's effective vocabulary size
- Directly impacts context window usage (tokens, not words)
- Drives API costs (pricing is per token)
- Affects model performance on different languages and domains

**Example:** "unconditionally" → ["un", "cond", "ition", "ally"] (4 tokens)

### Q3: What is the difference between a token and a word?
**Answer:** A word is a linguistic unit separated by spaces/punctuation. A token is the model's atomic unit of processing — it may be a word, subword, or even a single character.

| Word | Tokens (GPT-4) |
|------|-----------------|
| "hello" | 1 token: "hello" |
| "unconditionally" | 4 tokens: "un" + "cond" + "ition" + "ally" |
| "I'm" | 2 tokens: "I" + "'m" |

### Q4: What is an embedding?
**Answer:** An embedding is a dense vector representation of a token in high-dimensional space (e.g., 1536 dimensions for `text-embedding-3-small`). Semantically similar tokens have nearby vectors.

**Analogy:** Embeddings are like GPS coordinates on a map of meaning. "King" and "Queen" are close; "King" and "pancake" are far apart.

**Key property:** `embedding("king") - embedding("man") + embedding("woman") ≈ embedding("queen")`

### Q5: How do LLMs generate text?
**Answer:** LLMs generate text **autoregressively**: one token at a time, each conditioned on all previously generated tokens.

1. Start with a prompt (input tokens)
2. Compute probability distribution over next token
3. Sample from the distribution (or take argmax for greedy decoding)
4. Append the chosen token to the sequence
5. Repeat until `<eos>` token or max length

---

## Intermediate Questions

### Q6: Explain the attention mechanism in detail.
**Answer:** Scaled dot-product attention computes a weighted sum of Values, where weights are determined by the compatibility between a Query and Keys:

```
Attention(Q, K, V) = softmax(Q × K^T / √d_k) × V
```

- **Q (Query):** "What am I looking for?" — from the current token
- **K (Key):** "What do I contain?" — from all tokens in context
- **V (Value):** "What information do I pass along?" — from all tokens

The scaling factor `1/√d_k` prevents the dot products from growing too large, which would push softmax into regions with extremely small gradients.

Multi-head attention runs this in parallel with different learned projections, allowing the model to attend to different types of relationships simultaneously.

### Q7: What is the Transformer architecture? Why decoder-only?
**Answer:** The Transformer (Vaswani et al., 2017) uses self-attention instead of recurrence. It consists of encoder and decoder stacks, each with multi-head attention and feed-forward networks, plus residual connections and layer normalization.

**Decoder-only** (GPT family) is preferred for language generation because:
1. **Simplicity:** One stack instead of two
2. **Autoregressive:** Naturally suited for next-token prediction
3. **Unified:** Same architecture for pre-training and inference
4. **Scaling:** More parameters for the same compute budget (no encoder)
5. **In-context learning:** Decoder-only models naturally perform few-shot learning

### Q8: What is the KV cache and how does it improve inference?
**Answer:** During autoregressive generation, each step computes attention over all previous tokens. Without caching, this is O(n²) per step because we recompute K and V for every token.

The **KV cache** stores the Key and Value tensors from previous steps. At step n, we only compute:
- Q for the new token
- K and V for the new token
- Attention: Q_new × [K_1, ..., K_n] (using cached K, V)

**Speedup:** O(n) per step instead of O(n²). Total inference goes from O(n³) to O(n²).

**Memory cost:** ~2 bytes × d_model × num_layers × seq_len for each token (with quantization).

### Q9: Explain temperature, top-k, and top-p sampling.
**Answer:** These control how the model selects the next token from the probability distribution:

**Temperature (T):** Sharpens or flattens the distribution.
- T=0: Always pick highest probability token (deterministic)
- T=1: Sample according to learned distribution
- T>1: Distribution becomes more uniform (more random, potentially incoherent)

```
softmax(x/T) where T is temperature
```

**Top-K:** Only sample from the K tokens with highest probabilities.

**Top-P (nucleus):** Sample from the smallest set of tokens whose cumulative probability exceeds P.

**Best practice:** Use top_p=0.9 and temperature=0.7 in combination for creative tasks. Use temperature=0 for factual tasks.

### Q10: What is the difference between pre-training and fine-tuning?
**Answer:**

| Aspect | Pre-training | Fine-tuning |
|--------|--------------|-------------|
| Data | Internet-scale (trillions of tokens) | Task-specific (thousands to millions) |
| Objective | Next token prediction | Instruction following / task completion |
| Cost | Millions of dollars | Hundreds to thousands of dollars |
| Duration | Weeks to months | Hours to days |
| Outcome | Base model (text completer) | Instruct model (assistant) |
| Parameters | All parameters updated | All or subset (LoRA) |

### Q11: What is RLHF and why do we need it?
**Answer:** Reinforcement Learning from Human Feedback aligns LLMs with human preferences. It's how GPT-3 became ChatGPT.

**Three steps:**
1. **SFT:** Fine-tune on high-quality instruction-response pairs
2. **Reward Model:** Train a separate model to predict human preference (which response is better)
3. **RL Fine-tuning:** Optimize the LLM to maximize reward while staying close to the original (KL penalty)

**Why needed:** Next-token prediction optimizes for plausible text, not helpful/honest/harmless responses. RLHF injects human values and preferences.

### Q12: What is DPO and how does it differ from RLHF?
**Answer:** Direct Preference Optimization (DPO) skips the reward model step. Instead of training a reward model and then running RL, DPO directly optimizes the LLM using a binary preference loss:

```
L_DPO = -log σ(β(log π(y_w|x) - π_ref(y_w|x) - log π(y_l|x) + π_ref(y_l|x)))
```

Where y_w is the preferred response and y_l is the dispreferred.

**Advantages:** Simpler (no reward model), more stable (no RL), cheaper to train. **Disadvantage:** Less flexible than RLHF with a separately trained reward model.

### Q13: How do you choose between prompting, RAG, and fine-tuning?
**Answer:** Follow this decision tree:

1. **Simple task?** → Prompt. Start with zero-shot, escalate to few-shot.
2. **Need specific factual knowledge?** → RAG. Especially for changing/evolving information.
3. **Need new behavior or output format?** → Fine-tune. Especially for domain-specific writing styles or structured outputs.
4. **Combination?** → RAG for facts + prompt for behavior. Only fine-tune if both fail.

**Rule of thumb:** Prompt > RAG > Fine-tune. Each step is more expensive and less flexible.

### Q14: What are the scaling laws for LLMs?
**Answer:** Kaplan et al. (2020) found that model performance follows a power-law relationship with:
- Model size (parameters)
- Dataset size (tokens)
- Compute budget (FLOPs)

**Key findings:**
- Doubling compute → predictable improvement
- Larger models are more sample-efficient (need fewer tokens per parameter)
- Optimal is to scale model and data together

**Chinchilla scaling (Hoffmann et al., 2022):** For a given compute budget, the optimal model size and training tokens are equal in magnitude. Most models were undertrained — they should use more data and fewer parameters.

---

## Advanced Questions

### Q15: What is the difference between causal attention and bidirectional attention?
**Answer:**

**Causal (masked) attention** (decoder models like GPT):
- Each token can only attend to itself and previous tokens
- Uses an upper-triangular mask filled with -inf
- Enables autoregressive generation

**Bidirectional attention** (encoder models like BERT):
- Each token can attend to all tokens (past and future)
- No masking
- Better for understanding tasks (classification, NER, QA)

**Some models (T5, XLNet):** Use a combination or permutation-based approaches.

### Q16: Explain the role of positional encodings in Transformers.
**Answer:** Self-attention is permutation-invariant — it doesn't know the order of tokens. Positional encodings inject sequence order information.

**Types:**

1. **Sinusoidal (Vaswani et al.):** Fixed, pre-defined sine/cosine waves at different frequencies.
   ```
   PE(pos, 2i) = sin(pos / 10000^(2i/d_model))
   PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
   ```

2. **Learned:** Embedding layer that learns position representations.

3. **RoPE (Rotary Position Embedding):** Multiplies queries and keys by a rotation matrix based on position. Used in LLaMA, PaLM, GPT-NeoX. Performs better at longer sequences and supports for extrapolation.

### Q17: What causes hallucinations in LLMs and how do you mitigate them?
**Answer:** Hallucinations occur when the model generates plausible but incorrect information.

**Causes:**
1. **Training objective:** Next-token prediction optimizes for plausible text, not true text
2. **Knowledge cutoff:** Model doesn't know what it doesn't know
3. **Sampling:** Creative sampling (high temperature) can invent facts
4. **Compression:** The model compresses trillions of tokens into weights — information loss is inevitable

**Mitigations:**
- **RAG:** Ground generation in retrieved documents
- **Temperature control:** Use T=0 for factual queries
- **Prompt engineering:** "Say you don't know. Only answer if you're certain."
- **Prompting:** Ask model to cite sources / show work
- **Verification:** Cross-check with external tools (search, calculator, code execution)
- **Fine-tuning:** Train on hallucination-free data with rejection sampling

### Q18: How do reasoning models work (o1, DeepSeek R1)?
**Answer:** Reasoning models (o1, o3, DeepSeek R1) use test-time compute to improve accuracy on multi-step problems.

**Key techniques:**
1. **Chain-of-thought:** Model generates intermediate reasoning steps
2. **Self-verification:** Model checks its own reasoning for errors
3. **Backtracking:** Model explores different reasoning paths
4. **Reinforcement learning:** Trained with RL to discover effective reasoning strategies
5. **Thinking tokens:** Internal tokens used for reasoning, hidden in the final output

**Trade-off:** Improved accuracy (10-30% on math/code benchmarks) at the cost of 5-50x more tokens and latency.

### Q19: What is the context window and what are the challenges with long contexts?
**Answer:** The context window is the maximum number of tokens the model can process in a single forward pass.

**Challenges with long contexts:**
1. **Quadratic attention:** O(n²) compute and memory. Flash Attention mitigates this to almost O(n).
2. **"Lost in the middle"** (Liu et al., 2023): Models perform worse on information in the middle of long contexts.
3. **Positional encoding extrapolation:** Models trained on 4K tokens may not generalize to 128K.
4. **Key-value cache size:** For a 70B model, a 128K context requires ~800GB of memory for the KV cache.

**Solutions:** RoPE extrapolation, ALiBi, sliding window attention, hierarchical attention.

### Q20: Explain the full training pipeline for a modern LLM.
**Answer:**

```
Pre-training → SFT → Reward Modeling → RLHF → Evaluation → Deployment
```

1. **Data collection and filtering:** Curate trillions of tokens from the internet. Deduplicate, filter for quality and safety.

2. **Tokenization training:** Train BPE tokenizer on the corpus. Typical vocab size: 32K-128K.

3. **Pre-training:** Train on next-token prediction. Uses:
   - AdamW optimizer
   - Cosine learning rate schedule with warmup
   - Gradient clipping
   - Mixed precision (BF16)
   - Distributed training over thousands of GPUs
   - ZeRO optimization or FSDP for sharding

4. **Supervised Fine-Tuning (SFT):** Train on (instruction, response) pairs. Usually with a lower learning rate and fewer steps.

5. **Reward Modeling:** Train a separate model to predict human preference judgments.

6. **RLHF:** Use PPO or REINFORCE to optimize the policy (LLM) against the reward model, with a KL penalty to prevent reward hacking.

7. **Evaluation:** Benchmark on MMLU, HumanEval, GSM8K, HELM, etc.

8. **Safety tuning:** Red-teaming, refusal training, content filtering.

9. **Deployment:** Quantization (int8, FP8), KV cache optimization, serving infrastructure.

### Q21: What is the difference between sparse and dense attention?
**Answer:**

**Dense attention:** Every token attends to every other token (or all previous tokens for causal). O(n²) complexity.

**Sparse attention:** Only attend to a subset of tokens.
- **Sliding window:** Attend only to L nearest neighbors (O(n × L))
- **Dilated:** Attend to every k-th token (long-range with less compute)
- **Global + local:** Some tokens (e.g., [CLS]) attend to everything; others attend locally
- **Strided:** Mix of sliding window and dilated patterns

**Used in:** Longformer, BigBird, Mistral (sliding window), GPT-3 (dense for most layers, sparse for efficiency).

### Q22: How does model quantization work and why use it?
**Answer:** Quantization reduces the precision of model weights (e.g., FP32 → INT4), dramatically reducing memory and increasing speed.

**Types:**
- **Post-training quantization (PTQ):** Quantize weights after training
- **Quantization-aware training (QAT):** Simulate quantization during training
- **GPTQ:** Optimal brain quantization for LLMs
- **AWQ:** Activation-aware weight quantization
- **GGUF/GGML:** Quantization for local inference (llama.cpp)

**Memory savings:**
- FP32: 4 bytes per parameter
- FP16/BF16: 2 bytes
- INT8: 1 byte
- INT4: 0.5 bytes

A 70B model in FP32: 280GB → INT4: 35GB (runs on a single GPU).

**Trade-off:** Slight accuracy degradation, especially at INT4 for complex reasoning.

### Q23: What is the Mixture of Experts (MoE) architecture?
**Answer:** MoE divides the model into multiple "expert" sub-networks. Each input token is routed to a subset of experts (typically top-2), so only a fraction of parameters are activated per token.

**Example — Mixtral 8x7B:**
- 8 experts, each ~7B parameters
- Total parameters: ~47B
- Active per token: ~13B (2 experts)
- Performance matches a ~30B dense model

**Advantages:** Better compute efficiency (more parameters with same FLOPs per token). **Disadvantages:** More memory (all experts must be loaded), routing overhead, expert load imbalance.

### Q24: How do you handle rate limits and API errors in production?
**Answer:**

1. **Exponential backoff:** Wait 2^n seconds after the n-th retry, with jitter
2. **Retry with fallback:** Full retry → same model → smaller model → cached response
3. **Token bucket algorithm:** Maintain a steady request rate
4. **Request batching:** Combine multiple prompts into one API call
5. **Queue system:** Buffer requests and process at allowed rate
6. **Provisioned throughput:** Reserve capacity for production workloads

### Q25: Compare proprietary vs. open-source LLMs for production.
**Answer:**

| Factor | Proprietary (GPT-4, Claude) | Open-Source (Llama, Mistral) |
|--------|---------------------------|------------------------------|
| Quality | State-of-the-art | 80-95% of SOTA |
| Cost | Pay-per-token (OpEx) | Infrastructure (CapEx) |
| Latency | 100-500ms (API) | Variable (depends on hardware) |
| Privacy | Data sent to API | Fully local |
| Customization | Fine-tuning available | Full control |
| Reliability | SLA, uptime | Self-managed |
| Context window | 128K-200K | 8K-128K |

**Decision:** Use proprietary for rapid prototyping, variable workloads, and SOTA quality. Use open-source for fixed workloads, privacy-sensitive data, and cost optimization at scale.
