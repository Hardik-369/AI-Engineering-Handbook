# Resources: Loop Engineering

> Curated collection of research papers, articles, tools, and communities for deep diving into loop engineering.

---

## Research Papers

### Foundational Papers

| Paper | Authors | Year | Relevance |
|---|---|---|---|
| [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903) | Wei et al. | 2022 | Foundational for multi-step reasoning that loops build upon |
| [Self-Consistency Improves Chain of Thought Reasoning in Language Models](https://arxiv.org/abs/2203.11171) | Wang et al. | 2022 | Parallel sampling + voting, precursor to exploration loops |
| [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) | Yao et al. | 2022 | **The** agent loop paper — Thought → Action → Observation |
| [Tree of Thoughts: Deliberate Problem Solving with Large Language Models](https://arxiv.org/abs/2305.10601) | Yao et al. | 2023 | Tree search over reasoning paths, multi-branch exploration |
| [Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366) | Shinn et al. | 2023 | Reflection loop with episodic memory for agents |
| [Self-Refine: Iterative Refinement with Self-Feedback](https://arxiv.org/abs/2303.17651) | Madaan et al. | 2023 | Generate → Self-feedback → Refine loop architecture |

### Advanced Papers

| Paper | Authors | Year | Relevance |
|---|---|---|---|
| [Improving Factuality and Reasoning in Language Models through Multiagent Debate](https://arxiv.org/abs/2305.14325) | Du et al. | 2023 | Multiple LLMs debate, critique loops between agents |
| [CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing](https://arxiv.org/abs/2305.11738) | Gou et al. | 2023 | Self-correction using external tools as critics |
| [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073) | Bai et al. | 2022 | Rules-based self-critique and revision |
| [Q*: Improving Multi-step Reasoning in LLMs](https://arxiv.org/abs/2306.00982) | Various | 2023 | Search-guided reasoning loops |
| [RAP: Retrieval-Augmented Planning for Long-Horizon Tasks](https://arxiv.org/abs/2402.01825) | Various | 2024 | Planning loops with retrieval |
| [Eureka: Human-Level Reward Design via Coding Large Language Models](https://arxiv.org/abs/2310.12931) | Ma et al. | 2023 | Code generation + evaluation loop for reward design |
| [Voyager: An Open-Ended Embodied Agent with Large Language Models](https://arxiv.org/abs/2305.16291) | Wang et al. | 2023 | Agent loop for Minecraft with skill discovery |

### Analysis Papers

| Paper | Authors | Year | Relevance |
|---|---|---|---|
| [Large Language Models Cannot Self-Correct Reasoning Yet](https://arxiv.org/abs/2310.01798) | Huang et al. | 2023 | Critical analysis of self-correction limitations |
| [Rethinking the Bounds of LLM Reasoning: Are Multi-Agent Discussions the Key?](https://arxiv.org/abs/2402.18272) | Various | 2024 | Analysis of multi-agent debate effectiveness |
| [Does Self-Correction Require External Feedback?](https://arxiv.org/abs/2310.01477) | Various | 2023 | When self-correction works and when it doesn't |
| [Understanding the Limitations of Self-Correction](https://arxiv.org/abs/2310.00851) | Various | 2023 | Taxonomy of self-correction failure modes |

---

## Articles & Blog Posts

### Practical Guides

- **[Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents)** — Anthropic's guide to production agent loops
- **[The Practical Guide to LLM Loops](https://hamel.dev/blog/posts/llm-loops/)** — Hamel Husain's practical loop patterns
- **[A Survey of Techniques for Maximizing LLM Performance](https://www.semianalysis.com/p/a-survey-of-techniques-for-maximizing)** — SemiAnalysis on LLM optimization
- **[LLM Loops: The Key to Reliable AI](https://cobusgreyling.medium.com/llm-loops-the-key-to-reliable-ai-8f7a8e8d8c8a)** — Overview of loop patterns

### Architecture & Design

- **[The ReAct Pattern: Building AI Agents](https://www.promptingguide.ai/techniques/react)** — Prompting Guide's ReAct explanation
- **[Reflexion: A Framework for Self-Reflection in AI Agents](https://www.promptingguide.ai/techniques/reflexion)** — Prompting Guide's Reflexion explainer
- **[LLM Agents: A Comprehensive Guide](https://www.mlq.ai/llm-agents/)** — Overview of agent loop architectures
- **[The Anatomy of an AI Agent](https://cobusgreyling.medium.com/the-anatomy-of-an-ai-agent-6f8e7e7f8e7)** — Agent loop breakdown

### Production

- **[Monitoring LLM Systems in Production](https://www.honeyhive.ai/blog/monitoring-llm-systems)** — Monitoring loops and agents
- **[Cost Optimization for LLM Applications](https://www.vellum.ai/blog/cost-optimization-for-llm-applications)** — Strategies including loop cost management
- **[Testing LLM Loops](https://hamel.dev/blog/posts/testing-llm-loops/)** — Testing strategies for iterative systems
- **[Reliability Patterns for LLM Systems](https://www.buildt.ai/blog/reliability-patterns-for-llm-systems)** — Retry and reliability patterns

---

## Tools & Frameworks

### Agent Frameworks (ReAct Loops)

| Tool | Description |
|---|---|
| [LangChain](https://www.langchain.com/) | Agent loop implementation, tool integration, memory |
| [LlamaIndex](https://www.llamaindex.ai/) | RAG loops, agent loops, query planning |
| [AutoGen](https://github.com/microsoft/autogen) | Multi-agent conversations, critique loops |
| [CrewAI](https://www.crewai.com/) | Multi-agent orchestration with role-based loops |
| [Semantic Kernel](https://github.com/microsoft/semantic-kernel) | Microsoft's agent framework with planning |
| [Haystack](https://haystack.deepset.ai/) | RAG pipelines with validation loops |

### Evaluation & Testing

| Tool | Description |
|---|---|
| [LangSmith](https://www.langchain.com/langsmith) | Tracing, evaluation, monitoring for LLM loops |
| [Weights & Biases Prompts](https://wandb.ai/site/prompts) | LLM experiment tracking and loop analysis |
| [HoneyHive](https://www.honeyhive.ai/) | LLM evaluation and monitoring platform |
| [Arize Phoenix](https://www.arize.com/phoenix/) | LLM observability including loop tracing |
| [DeepEval](https://github.com/confident-ai/deepeval) | Unit testing for LLM outputs |

### Prompt Management

| Tool | Description |
|---|---|
| [LangSmith Hub](https://smith.langchain.com/hub) | Prompt versioning for loop prompts |
| [PromptLayer](https://www.promptlayer.com/) | LLM call logging and prompt management |
| [Agenta](https://github.com/agenta-ai/agenta) | LLM app platform with iteration tracking |

---

## Code Repositories

### Reference Implementations

- **[ReAct Implementation](https://github.com/ysymyth/ReAct)** — Original ReAct paper code
- **[Reflexion Implementation](https://github.com/noahshinn/reflexion)** — Original Reflexion paper code
- **[Self-Refine Implementation](https://github.com/madaan/self-refine)** — Original Self-Refine paper code
- **[Tree of Thoughts Implementation](https://github.com/princeton-nlp/tree-of-thought-llm)** — Original ToT paper code

### Production Examples

- **[OpenAI Evals](https://github.com/openai/evals)** — Evaluation loops and test framework
- **[LangChain Templates](https://github.com/langchain-ai/langchain/tree/master/templates)** — Production loop templates
- **[LlamaIndex Examples](https://github.com/run-llama/llama_index/tree/main/docs/examples)** — RAG loop examples

---

## Communities & Discussions

| Community | Best For |
|---|---|
| [r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/) | Loop engineering discussion and patterns |
| [LangChain Discord](https://discord.gg/langchain) | Agent loop implementation help |
| [Hugging Face Discord](https://discord.gg/huggingface) | Research paper discussions |
| [LLM Agents Group](https://groups.google.com/g/llm-agents) | Academic discussion of agent loops |

---

## Related Chapters

- **Chapter 02 (Prompt Engineering):** Prompt patterns used inside each LLM call within a loop. Chain-of-thought, few-shot, and structured output prompts are especially relevant.

- **Chapter 03 (Context Management):** Managing state across loop iterations. Context window limits, sliding windows, and summarization of accumulated history.

- **Chapter 07 (Agents):** The ReAct agent loop is one of the most important production loop patterns. Agent design extends loop engineering with tool use, multi-agent coordination, and environment interaction.

---

## Citation Guide

When citing loop engineering concepts in your work, use these canonical references:

```bibtex
# ReAct
@article{yao2022react,
  title={ReAct: Synergizing Reasoning and Acting in Language Models},
  author={Yao, Shunyu and Zhao, Jeffrey and Yu, Dian and Du, Nan and Shafran, Izhak and Narasimhan, Karthik and Cao, Yuan},
  journal={arXiv preprint arXiv:2210.03629},
  year={2022}
}

# Reflexion
@article{shinn2023reflexion,
  title={Reflexion: Language Agents with Verbal Reinforcement Learning},
  author={Shinn, Noah and Cassano, Federico and Gopinath, Ashwin and Narasimhan, Karthik and Yao, Shunyu},
  journal={arXiv preprint arXiv:2303.11366},
  year={2023}
}

# Self-Refine
@article{madaan2023selfrefine,
  title={Self-Refine: Iterative Refinement with Self-Feedback},
  author={Madaan, Aman and Tandon, Niket and Gupta, Prakhar and Hallinan, Skyler and Gao, Luyu and Wiegreffe, Sarah and Alon, Uri and Dziri, Nouha and Prabhumoye, Shrimai and Yang, Yiming and others},
  journal={arXiv preprint arXiv:2303.17651},
  year={2023}
}

# Self-Correction Limitations
@article{huang2023selfcorrection,
  title={Large Language Models Cannot Self-Correct Reasoning Yet},
  author={Huang, Jie and Chen, Xinyun and Mishra, Swaroop and Zheng, Huanyu and Yu, Adams Wei and Song, Xinying and Zhou, Denny},
  journal={arXiv preprint arXiv:2310.01798},
  year={2023}
}
```
