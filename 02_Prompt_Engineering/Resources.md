# Resources: Prompt Engineering

> Curated list of official guides, academic papers, tools, frameworks, and communities for prompt engineering.

---

## Official Prompt Engineering Guides

### OpenAI
- **OpenAI Prompt Engineering Guide** — https://platform.openai.com/docs/guides/prompt-engineering
  Six strategies: write clear instructions, provide reference text, split complex tasks, give GPT time to think, use external tools, test systematically.
- **OpenAI Best Practices for GPT-4o** — https://platform.openai.com/docs/guides/prompt-engineering/gpt-4o-specific
  Model-specific tips for GPT-4o including structured outputs and function calling.
- **OpenAI Function Calling Guide** — https://platform.openai.com/docs/guides/function-calling
  Comprehensive guide to tool definitions, parallel calls, and streaming.
- **OpenAI Structured Outputs Guide** — https://platform.openai.com/docs/guides/structured-outputs
  JSON schema mode, response_format, and best practices.

### Anthropic
- **Anthropic Prompt Engineering Guide** — https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering
  Covers XML prompting, role prompting, and Claude-specific patterns.
- **Anthropic Tool Use Guide** — https://docs.anthropic.com/en/docs/build-with-claude/tool-use
  Tool calling with Claude models, including `tool_choice` strategies.
- **Anthropic System Prompt Guide** — https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/system-prompts
  How to write effective system prompts for Claude.

### Google (Gemini)
- **Gemini Prompt Engineering Guide** — https://ai.google.dev/gemini-api/docs/prompting
  Covers Gemini-specific techniques including role prompting and chain of thought.
- **Gemini Structured Output** — https://ai.google.dev/gemini-api/docs/structured-outputs
  JSON mode and schema configuration for Gemini.

---

## Academic Papers

### Foundational Papers

| Paper | Year | Authors | Key Contribution | Link |
|-------|------|---------|-----------------|------|
| "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models" | 2022 | Wei et al. | Introduced CoT prompting | [arXiv](https://arxiv.org/abs/2201.11903) |
| "Self-Consistency Improves Chain of Thought Reasoning in Language Models" | 2022 | Wang et al. | Self-consistency with CoT | [arXiv](https://arxiv.org/abs/2203.11171) |
| "Tree of Thoughts: Deliberate Problem Solving with Large Language Models" | 2023 | Yao et al. | Introduced ToT | [arXiv](https://arxiv.org/abs/2305.10601) |
| "ReAct: Synergizing Reasoning and Acting in Language Models" | 2022 | Yao et al. | Introduced ReAct | [arXiv](https://arxiv.org/abs/2210.03629) |
| "Language Models are Few-Shot Learners" | 2020 | Brown et al. | GPT-3, in-context learning | [arXiv](https://arxiv.org/abs/2005.14165) |
| "The Power of Scale for Parameter-Efficient Prompt Tuning" | 2021 | Lester et al. | Prompt tuning | [arXiv](https://arxiv.org/abs/2104.08691) |

### Advanced Papers

| Paper | Year | Authors | Key Contribution | Link |
|-------|------|---------|-----------------|------|
| "Automatic Prompt Engineering" | 2022 | Zhou et al. | Automated prompt generation | [arXiv](https://arxiv.org/abs/2211.01910) |
| "DSPy: Compiling Declarative Language Model Calls into Self-Improving Pipelines" | 2023 | Khattab et al. | Programmatic prompt optimization | [arXiv](https://arxiv.org/abs/2310.03714) |
| "LLMLingua: Compressing Prompts for Accelerated Inference" | 2023 | Jiang et al. | Prompt compression | [arXiv](https://arxiv.org/abs/2310.05736) |
| "Graph of Thoughts: Solving Elaborate Problems with Large Language Models" | 2023 | Bessa et al. | Extends ToT with graph structure | [arXiv](https://arxiv.org/abs/2308.09687) |
| "Constitutional AI: Harmlessness from AI Feedback" | 2022 | Bai et al. | Constraint-based safety | [arXiv](https://arxiv.org/abs/2212.08073) |
| "Take a Step Back: Evoking Reasoning via Abstraction in Large Language Models" | 2023 | Zhou et al. | Step-back prompting | [arXiv](https://arxiv.org/abs/2310.06117) |
| "WizardLM: Empowering Large Language Models to Follow Complex Instructions" | 2023 | Xu et al. | Instruction evolution | [arXiv](https://arxiv.org/abs/2304.12244) |

### Survey Papers

- "Pre-train, Prompt, and Predict: A Systematic Survey of Prompting Methods in Natural Language Processing" — Liu et al. (2021) — [arXiv](https://arxiv.org/abs/2107.13586)
- "A Survey of Prompt Engineering Methods in Large Language Models" — Sahoo et al. (2024) — [arXiv](https://arxiv.org/abs/2401.14443)
- "The Prompt Report: A Systematic Survey of Prompting Techniques" — White et al. (2023) — [arXiv](https://arxiv.org/abs/2306.06608)

---

## Frameworks & Tools

### Prompt Management

| Tool | Description | Link |
|------|-------------|------|
| **LangChain Prompt Hub** | Versioned prompt registry with sharing | https://smith.langchain.com/hub |
| **Portkey** | Prompt management, testing, monitoring | https://portkey.ai |
| **Helicone** | LLM observability and prompt tracking | https://helicone.ai |
| **Agenta** | Open-source prompt management platform | https://github.com/Agenta-AI/agenta |
| **PromptLayer** | Prompt logging, versioning, and analysis | https://promptlayer.com |

### Prompt Optimization

| Tool | Description | Link |
|------|-------------|------|
| **DSPy** | Programmatic prompt optimization framework | https://github.com/stanfordnlp/dspy |
| **TextGrad** | Automatic differentiation through text | https://github.com/zou-group/textgrad |
| **PromptPerfect** | Automated prompt optimization service | https://promptperfect.jina.ai |
| **LMQL** | SQL-like language for LLM interactions | https://lmql.ai |
| **Guidance** | Domain-specific language for LLM control | https://github.com/guidance-ai/guidance |

### Prompt Compression

| Tool | Description | Link |
|------|-------------|------|
| **LLMLingua** | Prompt compression via trained models | https://github.com/microsoft/LLMLingua |
| **Selective Context** | Context selection for long documents | https://github.com/openai/selective-context |
| **AutoCompress** | Automatic prompt compression | Variant of LLMLingua |

### Evaluation & Testing

| Tool | Description | Link |
|------|-------------|------|
| **LangSmith** | LLM evaluation and testing platform | https://smith.langchain.com |
| **DeepEval** | Open-source LLM evaluation framework | https://github.com/confident-ai/deepeval |
| **RAGAS** | RAG evaluation framework | https://github.com/explodinggradients/ragas |
| **EleutherAI LM Eval** | Standardized LLM evaluation harness | https://github.com/EleutherAI/lm-evaluation-harness |
| **Continuous Eval** | Continuous evaluation for LLM apps | https://github.com/relari-ai/continuous-eval |

---

## Courses & Tutorials

### Free Courses
- **OpenAI Prompt Engineering (DeepLearning.AI)** — https://www.deeplearning.ai/short-courses/chatgpt-prompt-engineering-for-developers/
  Short course by Isa Fulford (OpenAI) and Andrew Ng.
- **Anthropic Prompt Engineering Interactive Tutorial** — https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview
  Interactive tutorial built into Anthropic docs.
- **Google Prompt Engineering Course** — https://ai.google.dev/gemini-api/docs/prompting
  Gemini-specific techniques and best practices.
- **Coursera: Prompt Engineering Specialization** — https://www.coursera.org/specializations/prompt-engineering
  Comprehensive specialization by DeepLearning.AI and AWS.

### Books
- **"The Prompt Engineering Guide"** by DAIR.AI — https://promptingguide.ai
  Comprehensive open-source guide covering all major techniques.
- **"A Practitioner's Guide to Prompt Engineering"** — O'Reilly (2024)
  Practical guide with production patterns.

---

## Communities

- **r/PromptEngineering** — https://reddit.com/r/PromptEngineering
  Active Reddit community for discussion and sharing.
- **OpenAI Community** — https://community.openai.com
  Official OpenAI forums with prompt engineering section.
- **Anthropic Discord** — Official Discord with prompt engineering channels
- **Hugging Face Prompt Engineering** — https://huggingface.co/tasks/prompt-engineering
  Community-contributed prompt templates and discussions.
- **LangChain Discord** — Active community for LLM development and prompting.

---

## Benchmarks & Datasets

### Prompt-Specific Benchmarks

| Benchmark | Description | Link |
|-----------|-------------|------|
| **BIG-bench** | 204+ tasks for LLM evaluation | https://github.com/google/BIG-bench |
| **MMLU** | Multi-task language understanding | https://github.com/hendrycks/test |
| **GSM8K** | Grade school math problems (CoT) | https://github.com/openai/grade-school-math |
| **HotpotQA** | Multi-hop QA (ReAct) | https://hotpotqa.github.io |
| **HumanEval** | Code generation benchmark | https://github.com/openai/human-eval |
| **HELM** | Holistic evaluation of language models | https://crfm.stanford.edu/helm/latest/ |

### Few-Shot Datasets

- **CrossFit** — 160 NLP tasks for few-shot learning
- **FewGLUE** — Few-shot version of GLUE benchmark
- **MetaICL** — In-context learning benchmark

---

## Prompt Injection & Safety Resources

- **OWASP LLM Top 10** — https://llmtop10.com
  Security risks for LLM applications, including prompt injection.
- **Prompt Injection Survey** — https://github.com/liu67321/Prompt-Injection
  Curated list of prompt injection papers and resources.
- **Gandalf (Prompt Injection Game)** — https://gandalf.lakera.ai
  Interactive game to learn prompt injection defense.
- **Learn Prompt Injection** — https://learnprompting.org/docs/prompt_injection
  Tutorial and examples of prompt injection attacks.

---

## Model-Specific Resources

### GPT-4o / OpenAI
- **OpenAI Cookbook** — https://cookbook.openai.com
  Practical recipes for prompt engineering, function calling, RAG.
- **OpenAI Examples** — https://platform.openai.com/examples
  Curated example prompts for various tasks.

### Claude / Anthropic
- **Claude Prompt Library** — https://docs.anthropic.com/en/prompt-library
  Ready-to-use prompts for Claude.
- **Claude Cookbook** — https://github.com/anthropics/anthropic-cookbook
  Code examples for Claude including prompt engineering.

### Gemini / Google
- **Gemini Cookbook** — https://github.com/google-gemini/generative-ai-python
  Python examples for Gemini API including structured output.

---

## Newsletters & Blogs

- **The Batch (Andrew Ng)** — https://www.deeplearning.ai/the-batch/
  Weekly AI newsletter covering prompt engineering developments.
- **Import AI (Jack Clark)** — https://importai.substack.com
  In-depth analysis of AI research including prompting.
- **Latent Space** — https://www.latent.space
  AI engineering podcast and newsletter.
- **Interconnects (Nathan Lambert)** — https://www.interconnects.ai
  AI research analysis including alignment and prompting.

---

## Related Chapters

- [Chapter 01: Foundations](../01_Foundations/README.md) — How LLMs work, tokenization, generation
- [Chapter 03: Context Engineering](../03_Context_Engineering/README.md) — Context windows, RAG, long-context
- [Chapter 07: Safety](../07_Safety/README.md) — Prompt injection, output filtering, alignment
- [Chapter 10: Agents](../10_Agents/README.md) — ReAct agents, multi-agent systems, tool use
