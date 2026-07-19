# Curated Paper Reading List

Organized by topic for AI engineers. Each paper includes a summary and explanation of why it matters for practical AI engineering.

---

## How to Use This List

1. **Start with Foundations** — these are the bedrock papers everyone should read.
2. **Pick a topic** relevant to your current project (e.g., RAG if building retrieval, Alignment if safety).
3. **Read the abstract first**, then the implementation details.
4. **Check the "Why It Matters" section** — it contextualizes the paper for engineering decisions.

---

## Foundations

### Attention Is All You Need

| Field | Detail |
|-------|--------|
| **Authors** | Vaswani et al. |
| **Year** | 2017 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:1706.03762](https://arxiv.org/abs/1706.03762) |

**Summary:** Introduced the Transformer architecture, replacing RNNs with self-attention mechanisms. The paper proposed multi-head attention, positional encoding, and the encoder-decoder structure that became the foundation of all modern LLMs.

**Why It Matters:** Every LLM you use (GPT, Claude, Gemini, LLaMA) is a descendant of this architecture. Understanding attention is essential for reasoning about context windows, KV caching, and model behavior. The concepts in this paper directly inform engineering decisions around context management and model selection.

---

### Language Models Are Few-Shot Learners (GPT-3)

| Field | Detail |
|-------|--------|
| **Authors** | Brown et al. |
| **Year** | 2020 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2005.14165](https://arxiv.org/abs/2005.14165) |

**Summary:** Demonstrated that scaling language models to 175B parameters enables in-context learning: models can perform tasks from just a few examples without gradient updates. Introduced the concept of few-shot, one-shot, and zero-shot prompting.

**Why It Matters:** This paper launched the prompt engineering paradigm. The finding that you can "program" LLMs through examples rather than fine-tuning is the core insight behind most LLM applications today. Understanding scaling laws and in-context learning guides decisions about model size, prompt design, and when to fine-tune vs. prompt.

---

### Training Language Models to Follow Instructions (InstructGPT)

| Field | Detail |
|-------|--------|
| **Authors** | Ouyang et al. (OpenAI) |
| **Year** | 2022 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2203.02155](https://arxiv.org/abs/2203.02155) |

**Summary:** Introduced RLHF (Reinforcement Learning from Human Feedback) to align GPT-3 with user intent. The 1.3B InstructGPT model outperformed the 175B GPT-3 despite being 100x smaller. Key insight: alignment makes models more capable and helpful.

**Why It Matters:** RLHF is why modern LLMs are useful. Without it, base models output random continuations. Understanding this paper helps engineers appreciate why model behavior differs between base and instruction-tuned versions, and why fine-tuning for instruction following can dramatically improve performance without scaling.

---

### LLaMA: Open and Efficient Foundation Language Models

| Field | Detail |
|-------|--------|
| **Authors** | Touvron et al. (Meta) |
| **Year** | 2023 |
| **Venue** | arXiv |
| **Link** | [arXiv:2302.13971](https://arxiv.org/abs/2302.13971) |

**Summary:** Meta released LLaMA, showing that smaller models trained on more data can match larger models. LLaMA-13B outperforms GPT-3-175B on most benchmarks. LLaMA became the basis for most open-source LLMs via fine-tuning (Alpaca, Vicuna, etc.).

**Why It Matters:** Kickstarted the open-source LLM ecosystem. Demonstrated that data quality and training efficiency matter more than raw parameter count. Directly led to the current landscape where 7B–70B open models are viable for production.

---

## Prompting

### Chain-of-Thought Prompting Elicits Reasoning in Large Language Models

| Field | Detail |
|-------|--------|
| **Authors** | Wei et al. (Google) |
| **Year** | 2022 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2201.11903](https://arxiv.org/abs/2201.11903) |

**Summary:** Showed that asking models to reason step-by-step ("let's think step by step") dramatically improves performance on arithmetic, commonsense, and symbolic reasoning tasks. CoT is most effective on models with ≥100B parameters.

**Why It Matters:** One of the single most impactful prompt engineering techniques. CoT is the default pattern for any complex reasoning task. Engineers should always consider CoT for math, logic, planning, and multi-step tasks.

---

### Tree of Thoughts: Deliberate Problem Solving with Large Language Models

| Field | Detail |
|-------|--------|
| **Authors** | Yao et al. (Princeton/Google DeepMind) |
| **Year** | 2023 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2305.10601](https://arxiv.org/abs/2305.10601) |

**Summary:** Extends CoT by exploring multiple reasoning paths (branches) and evaluating intermediate states. ToT enables lookahead and backtracking, treating LLM reasoning as a search problem.

**Why It Matters:** For tasks requiring planning or exploration (e.g., game playing, creative writing, complex problem solving), ToT outperforms linear CoT. The tree search pattern is implementable in code and can be combined with tool use.

---

### ReAct: Synergizing Reasoning and Acting in Language Models

| Field | Detail |
|-------|--------|
| **Authors** | Yao et al. (Princeton/Google) |
| **Year** | 2023 |
| **Venue** | ICLR |
| **Link** | [arXiv:2210.03629](https://arxiv.org/abs/2210.03629) |

**Summary:** Interleaves reasoning (thoughts) with actions (tool calls) in a single loop. The model outputs "Thought: ... Action: ... Observation: ..." sequences. ReAct outperforms both reasoning-only and acting-only approaches.

**Why It Matters:** The foundation of most modern agent architectures. LangChain's AgentExecutor, many custom agent loops, and research agent systems all follow the ReAct pattern.

---

### Self-Refine: Iterative Refinement with Self-Feedback

| Field | Detail |
|-------|--------|
| **Authors** | Madaan et al. (CMU/Microsoft) |
| **Year** | 2023 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2303.17651](https://arxiv.org/abs/2303.17651) |

**Summary:** A single LLM generates output, provides feedback on its own output, and refines it — all in a loop without external tools. Works across dialogue, reasoning, code generation, and more.

**Why It Matters:** Self-refinement is a cheap way to improve quality without human feedback. The pattern (generate → critique → refine) is directly implementable and widely used in production systems.

---

### Reflexion: An Autonomous Agent with Dynamic Memory and Self-Reflection

| Field | Detail |
|-------|--------|
| **Authors** | Shinn et al. (Northeastern/Microsoft) |
| **Year** | 2023 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2303.11366](https://arxiv.org/abs/2303.11366) |

**Summary:** Agents store past failures as textual reflections in memory. When encountering similar tasks, they retrieve relevant reflections to guide decision-making. Combines ReAct with episodic memory.

**Why It Matters:** Introduces a practical memory architecture for agents. The reflection-on-failure pattern is easy to implement and significantly improves agent reliability over multiple trials.

---

## Context

### Lost in the Middle: How Language Models Use Long Contexts

| Field | Detail |
|-------|--------|
| **Authors** | Liu et al. (Stanford/UCSD) |
| **Year** | 2023 |
| **Venue** | ACL |
| **Link** | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) |

**Summary:** Found that LLMs perform best when relevant information is at the very beginning or end of the context, and worst when it's in the middle. Performance degrades with longer contexts, regardless of model size.

**Why It Matters:** Critical for RAG system design. Be careful about document ordering: put the most relevant information first and last. This has direct implications for chunking strategies, retrieval ordering, and context window management.

---

### LongBench: A Bilingual, Multitask Benchmark for Long Context Understanding

| Field | Detail |
|-------|--------|
| **Authors** | Bai et al. (THU/Microsoft) |
| **Year** | 2023 |
| **Venue** | arXiv |
| **Link** | [arXiv:2308.14508](https://arxiv.org/abs/2308.14508) |

**Summary:** Comprehensive benchmark for long-context understanding across 21 tasks (single/multi-doc QA, summarization, few-shot, code completion). Evaluated many models, finding significant room for improvement.

**Why It Matters:** Provides a rigorous evaluation framework for long-context performance. When choosing or testing models for long-context use cases, refer to LongBench results to understand real-world capability, not just context window size.

---

### Efficient Transformers: A Survey

| Field | Detail |
|-------|--------|
| **Authors** | Tay et al. (Google) |
| **Year** | 2022 |
| **Venue** | ACM Computing Surveys |
| **Link** | [arXiv:2009.06732](https://arxiv.org/abs/2009.06732) |

**Summary:** Comprehensive survey of efficient transformer variants: sparse attention (Longformer, BigBird), low-rank (Linformer), kernel-based (Performer), recurrent (Transformer-XL), and more.

**Why It Matters:** Understanding efficiency techniques helps engineers reason about context window scaling. FlashAttention, sliding window attention, and KV caching are direct descendants of this work.

---

## RAG

### Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks

| Field | Detail |
|-------|--------|
| **Authors** | Lewis et al. (Facebook AI) |
| **Year** | 2020 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2005.11401](https://arxiv.org/abs/2005.11401) |

**Summary:** Introduced RAG: a parametric memory (the model) combined with non-parametric memory (a retrieval index). Query → retrieve documents → concatenate with query → generate answer. RAG outperforms pure generation on knowledge-intensive tasks.

**Why It Matters:** RAG is the most widely used pattern for grounding LLMs in external knowledge. This paper formalized the architecture that LangChain, LlamaIndex, and most production systems implement. Every AI engineer should understand the retrieve-read-generate pipeline.

---

### Dense Passage Retrieval for Open-Domain Question Answering

| Field | Detail |
|-------|--------|
| **Authors** | Karpukhin et al. (Facebook AI) |
| **Year** | 2020 |
| **Venue** | EMNLP |
| **Link** | [arXiv:2004.04906](https://arxiv.org/abs/2004.04906) |

**Summary:** Proposed DPR: dual-encoder (query encoder + passage encoder) trained to maximize inner product of relevant pairs. First dense retrieval method to outperform BM25 on open-domain QA benchmarks.

**Why It Matters:** DPR is the default retrieval approach in modern RAG systems. Understanding bi-encoder architecture, contrastive learning, and dense vs. sparse retrieval trade-offs is essential for designing RAG pipelines.

---

### GraphRAG: A Graph-Enhanced Retrieval-Augmented Generation Framework

| Field | Detail |
|-------|--------|
| **Authors** | Edge et al. (Microsoft) |
| **Year** | 2024 |
| **Venue** | arXiv |
| **Link** | [arXiv:2404.16130](https://arxiv.org/abs/2404.16130) |

**Summary:** Builds a knowledge graph from documents using LLMs. Answers queries by retrieving relevant graph neighborhoods, combining structured (graph) and unstructured (text) retrieval.

**Why It Matters:** GraphRAG addresses the limitation of flat retrieval for complex, multi-document queries. It's especially useful for datasets with rich entity relationships (e.g., research papers, legal documents, news).

---

## Agents

### Tool-Augmented Language Models

| Field | Detail |
|-------|--------|
| **Authors** | Parisi et al. (Google) |
| **Year** | 2022 |
| **Venue** | arXiv |
| **Link** | [arXiv:2205.12255](https://arxiv.org/abs/2205.12255) |

**Summary:** Comprehensive survey of approaches for augmenting language models with external tools: calculators, search engines, code interpreters, APIs. Covers training methods, prompting strategies, and architectures.

**Why It Matters:** Defines the taxonomy of tool-augmented LLMs. The categories in this survey (fine-tuned tool use, prompted tool use, etc.) map directly to implementation choices engineers face today.

---

### WebGPT: Browser-Assisted Question-Answering with Human Feedback

| Field | Detail |
|-------|--------|
| **Authors** | Nakano et al. (OpenAI) |
| **Year** | 2021 |
| **Venue** | arXiv |
| **Link** | [arXiv:2112.09332](https://arxiv.org/abs/2112.09332) |

**Summary:** Fine-tuned GPT-3 to use a web browser (search, click links, scroll) to answer questions. Used human feedback to train the model to navigate effectively. WebGPT demonstrated that LLMs can learn to interact with complex environments.

**Why It Matters:** Early demonstration of LLMs as web-browsing agents. The lessons (relevance filtering, navigation strategies, feedback loops) are directly applicable to any agents that interact with structured environments.

---

### Generative Agents: Interactive Simulacra of Human Behavior

| Field | Detail |
|-------|--------|
| **Authors** | Park et al. (Stanford/Google) |
| **Year** | 2023 |
| **Venue** | UIST |
| **Link** | [arXiv:2304.03442](https://arxiv.org/abs/2304.03442) |

**Summary:** 25 AI agents with memory, reflection, planning, and social interaction in a simulated town. Agents wake up, plan their day, talk to each other, and evolve their beliefs. Architecture: memory stream → retrieval → reflection → planning → action.

**Why It Matters:** Definitive example of agent architecture with memory. The three-level memory (recent, reflection, summary) and plan-execute loop are adopted in many production agent systems.

---

### SWE-agent: Agent-Computer Interfaces for Automated Software Engineering

| Field | Detail |
|-------|--------|
| **Authors** | Yang et al. (Princeton) |
| **Year** | 2024 |
| **Venue** | ICLR |
| **Link** | [arXiv:2405.15793](https://arxiv.org/abs/2405.15793) |

**Summary:** Agent that fixes GitHub issues end-to-end. Uses a custom agent-computer interface (ACI) — not just plain bash — including file editors, syntax-aware commands, and linters. SWE-agent achieves 12.3% (and with GPT-4, 18.8%) on SWE-bench.

**Why It Matters:** Gold standard for code-fixing agents. Demonstrates that specialized interfaces (not just tool calling) matter as much as the model. The ACI design principles apply to any agents operating in complex environments.

---

## Alignment

### Constitutional AI: Harmlessness from AI Feedback

| Field | Detail |
|-------|--------|
| **Authors** | Bai et al. (Anthropic) |
| **Year** | 2022 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2212.08073](https://arxiv.org/abs/2212.08073) |

**Summary:** Trains models to be harmless using a set of written principles (a "constitution") and AI-generated feedback. Two-stage: supervised (generate revisions based on principles) + RL (prefer constitutional outputs). No human harmlessness labels needed.

**Why It Matters:** Constitutional AI makes alignment scalable and transparent. Engineers working on safety, content moderation, or system-level guardrails should understand how principles-based alignment works. It's the foundation of Claude's harmlessness training.

---

### Learning to Summarize with Human Feedback (RLHF)

| Field | Detail |
|-------|--------|
| **Authors** | Stiennon et al. (OpenAI) |
| **Year** | 2020 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2009.01325](https://arxiv.org/abs/2009.01325) |

**Summary:** Applied RLHF to summarization. Humans compare summaries, a reward model learns preferences, and a policy (LLM) is optimized with PPO. Demonstrates that RLHF improves summary quality over supervised fine-tuning.

**Why It Matters:** The first clear demonstration of RLHF for LLMs. The reward model + PPO pipeline is the basis for ChatGPT, Claude, and most aligned models. Engineers fine-tuning models for preference should understand this pipeline.

---

### Direct Preference Optimization: Your Language Model Is Secretly a Reward Model

| Field | Detail |
|-------|--------|
| **Authors** | Rafailov et al. (Stanford) |
| **Year** | 2023 |
| **Venue** | NeurIPS |
| **Link** | [arXiv:2305.18290](https://arxiv.org/abs/2305.18290) |

**Summary:** DPO simplifies RLHF by directly optimizing the policy using preference pairs, without a separate reward model. The key insight is that the optimal policy can be expressed in closed form as a function of the preference data.

**Why It Matters:** DPO is simpler, more stable, and cheaper than RLHF (PPO). It's quickly becoming the default alignment method for open-source fine-tuning (Zephyr, LLaMA 3). Engineers fine-tuning models should prefer DPO over PPO unless they need the complexity.

---

## Efficiency

### LLMLingua: Compressing Prompts for Accelerated Inference

| Field | Detail |
|-------|--------|
| **Authors** | Jiang et al. (Microsoft) |
| **Year** | 2023 |
| **Venue** | EMNLP |
| **Link** | [arXiv:2310.05736](https://arxiv.org/abs/2310.05736) |

**Summary:** Prompt compression via a small language model (e.g., GPT-2-small) that removes less important tokens. Achieves 2–5x compression with minimal performance loss (<3%).

**Why It Matters:** Prompt compression directly reduces cost and latency. For RAG systems with large contexts, LLMLingua can cut token usage by 50–80%. Understanding compression helps engineers balance cost vs. quality in production.

---

### AutoCompressors: Understanding and Utilizing Compressed Context in LLMs

| Field | Detail |
|-------|--------|
| **Authors** | Mu et al. (CMU/Meta) |
| **Year** | 2023 |
| **Venue** | arXiv |
| **Link** | [arXiv:2310.02508](https://arxiv.org/abs/2310.02508) |

**Summary:** Introduces soft prompt compression: the model learns to compress long contexts into a small number of "summary vectors" that are prepended to future inputs. Works across documents, conversations, and demonstrations.

**Why It Matters:** Alternative to prompt compression that preserves semantic content via learned representations. Points toward architectures where context is compressed and stored, enabling efficient long-term memory.

---

### Efficient Streaming Language Models with Attention Sinks

| Field | Detail |
|-------|--------|
| **Authors** | Xiao et al. (Microsoft) |
| **Year** | 2023 |
| **Venue** | arXiv |
| **Link** | [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) |

**Summary:** Found that transformers focus excessive attention on initial tokens ("attention sinks"). By reserving initial tokens as sinks and using sliding window + KV cache re-computation, models can stream indefinitely with stable quality.

**Why It Matters:** Enables models to operate on arbitrarily long sequences without OOM or performance degradation. StreamingLLM is used in production for chat systems, code completion, and any application requiring >128K effective context.

---

## Index

| Topic | Papers |
|-------|--------|
| **Foundations** | Attention Is All You Need, GPT-3, InstructGPT, LLaMA |
| **Prompting** | Chain-of-Thought, Tree of Thoughts, ReAct, Self-Refine, Reflexion |
| **Context** | Lost in the Middle, LongBench, Efficient Transformers Survey |
| **RAG** | Retrieval-Augmented Generation, Dense Passage Retrieval, GraphRAG |
| **Agents** | Tool-Augmented LMs, WebGPT, Generative Agents, SWE-agent |
| **Alignment** | Constitutional AI, RLHF, DPO |
| **Efficiency** | LLMLingua, AutoCompressors, Streaming LLM |

---

## Quick Start: Papers by Use Case

| You're working on... | Start with... |
|-------------------|---------------|
| Improving prompt quality | CoT Prompting → Self-Refine → Tree of Thoughts |
| Building a RAG pipeline | RAG (Lewis) → DPR → Lost in the Middle |
| Building an agent | ReAct → Reflexion → SWE-agent → Generative Agents |
| Reducing cost/latency | LLMLingua → Streaming LLM → AutoCompressors |
| Safety / alignment | Constitutional AI → RLHF → DPO |
| Understanding context limits | Lost in the Middle → LongBench → Efficient Transformers |
| Choosing a model | Attention → LLaMA → InstructGPT |
| Fine-tuning | InstructGPT → DPO → Constitutional AI |

---

## Contributing

See the [contribution guide](../.github/CONTRIBUTING.md) for adding papers. Priority will be given to papers that:
- Have direct engineering impact
- Are widely cited (>100 citations)
- Represent significant architecture/technique shifts
- Offer reproducible or implementable insights
