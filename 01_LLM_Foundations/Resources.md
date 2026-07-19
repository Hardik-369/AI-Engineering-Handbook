# Resources — Chapter 01: LLM Foundations

---

## Foundational Papers

### Must-Read

| Paper | Year | Why It Matters |
|-------|------|----------------|
| [Attention Is All You Need](https://arxiv.org/abs/1706.03762) | 2017 | Introduced the Transformer architecture. The paper that changed NLP forever. |
| [Language Models Are Few-Shot Learners](https://arxiv.org/abs/2005.14165) (GPT-3) | 2020 | Demonstrated scaling laws and in-context learning. 175B parameters. |
| [Training Language Models to Follow Instructions with Human Feedback](https://arxiv.org/abs/2203.02155) (InstructGPT) | 2022 | RLHF paper. How GPT-3 became ChatGPT. |
| [Scaling Laws for Neural Language Models](https://arxiv.org/abs/2001.08361) | 2020 | Power-law relationships between compute, data, and model size. |
| [Training Compute-Optimal Large Language Models](https://arxiv.org/abs/2203.15556) (Chinchilla) | 2022 | Optimal model size vs data scaling. Most models were undertrained. |

### Architecture & Training

| Paper | Key Contribution |
|-------|-----------------|
| [BERT: Pre-training of Deep Bidirectional Transformers](https://arxiv.org/abs/1810.04805) | Encoder-only Transformer, masked language modeling |
| [GPT-2: Language Models are Unsupervised Multitask Learners](https://cdn.openai.com/better-language-models/language_models_are_unsupervised_multitask_learners.pdf) | Zero-shot transfer, scaling decoder-only |
| [LLaMA: Open and Efficient Foundation Language Models](https://arxiv.org/abs/2302.13971) | Open-source, efficient scaling, SwiGLU + RoPE |
| [PaLM: Scaling Language Modeling with Pathways](https://arxiv.org/abs/2204.02311) | 540B model, parallel attention+FFN |
| [Mixtral of Experts](https://arxiv.org/abs/2401.04088) | Mixture of Experts for efficient LLMs |

### Fine-Tuning & Alignment

| Paper | Key Contribution |
|-------|-----------------|
| [LoRA: Low-Rank Adaptation of Large Language Models](https://arxiv.org/abs/2106.09685) | Parameter-efficient fine-tuning |
| [Direct Preference Optimization (DPO)](https://arxiv.org/abs/2305.18290) | RLHF without reward model |
| [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073) | Self-supervised safety training |

### Reasoning & Inference

| Paper | Key Contribution |
|-------|-----------------|
| [Chain-of-Thought Prompting Elicits Reasoning](https://arxiv.org/abs/2201.11903) | Step-by-step reasoning |
| [Tree of Thoughts: Deliberate Problem Solving](https://arxiv.org/abs/2305.10601) | Multi-path reasoning with search |
| [Flash Attention: Fast and Memory-Efficient Exact Attention](https://arxiv.org/abs/2205.14135) | IO-aware attention algorithm |
| [Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172) | Position bias in long documents |

---

## Documentation & APIs

### Official API Docs

| Provider | Documentation | Key Features |
|----------|--------------|--------------|
| [OpenAI API](https://platform.openai.com/docs) | Chat completions, embeddings, fine-tuning, vision |
| [Anthropic API](https://docs.anthropic.com/) | Claude, extended thinking, prompt caching |
| [Google AI (Gemini)](https://ai.google.dev/docs) | Gemini models, 1M context, multimodal |
| [Mistral AI](https://docs.mistral.ai/) | Open-weight models, API, Le Chat |
| [DeepSeek](https://platform.deepseek.com/) | V3, R1 reasoning, competitive pricing |

### Open-Source Libraries

| Library | Purpose |
|---------|---------|
| [Hugging Face Transformers](https://huggingface.co/docs/transformers) | Model hub, tokenizers, training |
| [Tiktoken](https://github.com/openai/tiktoken) | Fast BPE tokenizer for OpenAI models |
| [LLaMA.cpp](https://github.com/ggerganov/llama.cpp) | Local inference (CPU/GPU), quantization |
| [Ollama](https://ollama.com/) | Local model runner (user-friendly) |
| [vLLM](https://github.com/vllm-project/vllm) | High-throughput inference server |
| [LangChain](https://python.langchain.com/) | Application framework (Chains, Agents, RAG) |
| [Haystack](https://haystack.deepset.ai/) | NLP pipeline framework |
| [Weights & Biases](https://wandb.ai/) | Training experiment tracking |

---

## Courses & Tutorials

### Video Courses

| Resource | Creator | Length | What You'll Learn |
|----------|---------|--------|-------------------|
| [Let's Build GPT from Scratch](https://www.youtube.com/watch?v=kCc8FmEb1nY) | Andrej Karpathy | 2h | Implement a GPT from scratch in Python |
| [Attention in Transformers](https://www.youtube.com/watch?v=eMlx5fFNoYc) | 3Blue1Brown | 30 min | Visual explanation of attention |
| [CS224N: NLP with Deep Learning](https://web.stanford.edu/class/cs224n/) | Stanford | Full course | Comprehensive NLP course |
| [Neural Networks: Zero to Hero](https://karpathy.ai/zero-to-hero.html) | Andrej Karpathy | Series | From basics to GPT |

### Written Guides

| Resource | Description |
|----------|-------------|
| [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) | Best visual guide to Transformers |
| [The Illustrated GPT-2](https://jalammar.github.io/illustrated-gpt2/) | Visual guide to decoder-only models |
| [LLM Visualization](https://bbycroft.net/llm) | Interactive 3D visualization of LLM internals |
| [Anthropic's Transformer Circuits](https://transformer-circuits.pub/) | Mechanistic interpretability research |

### Practical Workshops

| Resource | Description |
|----------|-------------|
| [OpenAI Cookbook](https://cookbook.openai.com/) | Practical examples and patterns |
| [Hugging Face NLP Course](https://huggingface.co/learn/nlp-course) | Hands-on with transformers library |
| [Fast.ai Practical Deep Learning](https://course.fast.ai/) | Top-down approach to deep learning |

---

## Tools & Platforms

| Tool | Use Case |
|------|----------|
| [ChatGPT](https://chat.openai.com/) | Chat interface for GPT-4o |
| [Claude.ai](https://claude.ai/) | Chat interface for Claude |
| [Gemini](https://gemini.google.com/) | Chat interface for Gemini |
| [Perplexity](https://www.perplexity.ai/) | Search + LLM (with citations) |
| [Hugging Face Spaces](https://huggingface.co/spaces) | Demo apps for open models |
| [Pinecone](https://www.pinecone.io/) | Vector database for embeddings |
| [Weaviate](https://weaviate.io/) | Open-source vector database |
| [LMSYS Chatbot Arena](https://chat.lmsys.org/) | Compare models side-by-side |

---

## Books

| Title | Author | Focus |
|-------|--------|-------|
| [Speech and Language Processing](https://web.stanford.edu/~jurafsky/slp3/) | Jurafsky & Martin | Comprehensive NLP textbook |
| [Deep Learning](https://www.deeplearningbook.org/) | Goodfellow, Bengio, Courville | Deep learning foundations |
| [Understanding Deep Learning](https://udlbook.github.io/udlbook/) | Prince | Modern deep learning text |

---

## Communities

| Platform | Where to Go |
|----------|-------------|
| [r/LocalLLaMA](https://reddit.com/r/LocalLLaMA) | Open-source LLMs and local deployment |
| [r/MachineLearning](https://reddit.com/r/MachineLearning) | General ML discussion |
| [Hugging Face Discord](https://discord.gg/huggingface) | Open-source model community |
| [LM Arena Discord](https://discord.gg/lmsys) | Model comparison and discussion |
| [AI Stack Exchange](https://ai.stackexchange.com/) | Q&A for AI/ML |

---

## Chapter Cross-References

| Chapter | Connection |
|---------|------------|
| [Chapter 02: Prompt Engineering] | Building on your understanding of how models process input |
| [Chapter 03: RAG] | Using embeddings and retrieval to ground LLMs |
| [Chapter 05: Deployment] | Inference optimization, quantization, serving |
| [Chapter 06: LLMOps] | Training infrastructure, monitoring, evaluation |
| [Chapter 07: Advanced Reasoning] | Tree-of-thought, self-consistency, tool use |
