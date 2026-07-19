# Resources: Agent Engineering

## Foundational Papers

1. **ReAct: Synergizing Reasoning and Acting in Language Models** - Yao et al., 2022
   https://arxiv.org/abs/2210.03629
   The paper that introduced the ReAct pattern — interleaving reasoning traces with task-specific actions.

2. **Reflexion: Language Agents with Verbal Reinforcement Learning** - Shinn et al., 2023
   https://arxiv.org/abs/2303.11366
   Introduces the reflexion pattern where agents store verbal reflections in episodic memory.

3. **Toolformer: Language Models Can Teach Themselves to Use Tools** - Schick et al., 2023
   https://arxiv.org/abs/2302.04761
   Shows how LLMs can learn to use tools through self-supervised learning.

4. **Tree of Thoughts: Deliberate Problem Solving with Large Language Models** - Yao et al., 2023
   https://arxiv.org/abs/2305.10601
   Extends chain-of-thought to explore multiple reasoning paths over a tree structure.

5. **Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models** - Wang et al., 2023
   https://arxiv.org/abs/2305.04091
   A planning-based approach to prompting LLMs for complex reasoning tasks.

6. **LLM-as-a-Judge** - Zheng et al., 2023
   https://arxiv.org/abs/2306.05685
   Using LLMs as evaluators for agent and LLM outputs.

7. **Tool Learning with Foundation Models** - Qin et al., 2023
   https://arxiv.org/abs/2304.08354
   Comprehensive survey on tool learning paradigms.

8. **Communicative Agents for Software Development** - Qian et al., 2023
   https://arxiv.org/abs/2307.07924
   CHATDEV: Multi-agent system for software development.

9. **AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation** - Wu et al., 2023
   https://arxiv.org/abs/2308.08155
   Microsoft's framework for multi-agent conversations.

10. **AgentBench: Evaluating LLMs as Agents** - Liu et al., 2023
    https://arxiv.org/abs/2308.03688
    Comprehensive benchmark for evaluating LLM-based agents.

## Frameworks & Libraries

### Agent Frameworks

| Framework | Description | Link |
|-----------|-------------|------|
| **LangChain** | Most popular agent framework with tool use, memory, and chains | https://github.com/langchain-ai/langchain |
| **LangGraph** | Graph-based agent orchestration for complex workflows | https://github.com/langchain-ai/langgraph |
| **AutoGen** | Multi-agent conversation framework by Microsoft | https://github.com/microsoft/autogen |
| **CrewAI** | Multi-agent orchestration with role-based agents | https://github.com/crewAIInc/crewAI |
| **Semantic Kernel** | Enterprise agent framework by Microsoft | https://github.com/microsoft/semantic-kernel |
| **DSPy** | Programming with foundation models | https://github.com/stanfordnlp/dspy |
| **Vercel AI SDK** | Streaming-first AI SDK with tool use | https://github.com/vercel/ai |
| **OpenAI Agents SDK** | Lightweight agent framework by OpenAI | https://github.com/openai/openai-agents-python |
| **LlamaIndex** | Data framework with agent capabilities | https://github.com/run-llama/llama_index |
| **Ag2 (formerly AutoGen)** | Community fork of AutoGen | https://github.com/ag2ai/ag2 |

### Tool & Function Calling APIs

| Provider | Documentation |
|----------|---------------|
| **OpenAI** | https://platform.openai.com/docs/guides/function-calling |
| **Anthropic** | https://docs.anthropic.com/en/docs/build-with-claude/tool-use |
| **Google Gemini** | https://ai.google.dev/gemini-api/docs/function-calling |
| **Mistral** | https://docs.mistral.ai/capabilities/function_calling/ |
| **Together AI** | https://docs.together.ai/docs/function-calling |

### Memory & Vector Databases

| Tool | Description | Link |
|------|-------------|------|
| **Chroma** | Lightweight, in-process vector DB | https://github.com/chroma-core/chroma |
| **Pinecone** | Managed vector database | https://www.pinecone.io/ |
| **Weaviate** | Open-source vector search engine | https://weaviate.io/ |
| **Qdrant** | Vector similarity search engine | https://qdrant.tech/ |
| **pgvector** | PostgreSQL extension for vectors | https://github.com/pgvector/pgvector |
| **Redis Stack** | Redis with vector search | https://redis.io/docs/stack/search/ |
| **Milvus** | Distributed vector database | https://milvus.io/ |

### Observability & Monitoring

| Tool | Description | Link |
|------|-------------|------|
| **LangSmith** | LangChain's observability platform | https://smith.langchain.com/ |
| **LangFuse** | Open-source observability for LLM apps | https://langfuse.com/ |
| **Weights & Biases** | ML platform with LLM tracing | https://wandb.ai/ |
| **Phoenix (Arize)** | Open-source AI observability | https://github.com/Arize-AI/phoenix |
| **Helicone** | LLM observability platform | https://www.helicone.ai/ |
| **OpenTelemetry** | Open-source observability framework | https://opentelemetry.io/ |

### Evaluation

| Tool | Description | Link |
|------|-------------|------|
| **GAIA Benchmark** | General AI assistants benchmark | https://huggingface.co/gaia-benchmark |
| **SWE-bench** | Software engineering benchmark | https://www.swebench.com/ |
| **WebArena** | Web task benchmark | https://webarena.dev/ |
| **ToolBench** | Tool use benchmark | https://github.com/THUDM/ToolBench |
| **AgentBench** | Multi-domain agent benchmark | https://github.com/THUDM/AgentBench |

## Tutorials & Courses

- **LangChain Agent Tutorial**: https://python.langchain.com/docs/tutorials/agents/
- **OpenAI Agent Cookbook**: https://cookbook.openai.com/examples/introduction_to_agents
- **Anthropic Agent Cookbook**: https://github.com/anthropics/anthropic-cookbook
- **Building Effective Agents (Anthropic)**: https://docs.anthropic.com/en/docs/build-with-claude/agentic
- **Hugging Face Agent Course**: https://huggingface.co/learn/agents-course/
- **DeepLearning.AI Short Courses**:
  - "Building Systems with the ChatGPT API"
  - "LangChain for LLM Application Development"
  - "Multi AI Agent Systems with CrewAI"

## Blogs & Articles

- **"What Is an AI Agent?" - IBM**: https://www.ibm.com/think/topics/ai-agents
- **"The Rise of AI Agents" - Lilian Weng**: https://lilianweng.github.io/posts/2023-06-23-agent/
- **"Practical Tips for LLM Agents" - Jason Liu**: https://jason-liu.medium.com/practical-tips-for-llm-agents-1234
- **"Building Effective Agents" - Anthropic**: https://www.anthropic.com/engineering/building-effective-agents
- **"Agentic Design Patterns" - Matt Shumer**: https://matt.sh/agentic-design-patterns
- **"The Agent Journey" - Hamel Husain**: https://hamel.dev/blog/posts/agent/

## Communities

- **LangChain Discord**: https://discord.gg/langchain
- **Hugging Face Discord**: https://discord.gg/huggingface
- **r/LocalLLaMA**: Reddit community for LLM development
- **r/MachineLearning**: Reddit ML community
- **AI Agent Hackers**: https://discord.gg/ai-agent-hackers

## Conferences & Workshops

- **NeurIPS Agent Workshops**: Annual workshop on LLM agents
- **ICLR Agent Papers**: Leading conference for agent research
- **AI Engineer Summit**: Conference focused on AI engineering
- **LangChain Conference**: Dedicated to agent frameworks

## Design Patterns References

- **ReAct Pattern**: https://www.promptingguide.ai/techniques/react
- **Plan-and-Execute Pattern**: https://blog.langchain.dev/plan-and-execute-agents/
- **Reflexion Pattern**: https://github.com/noahshinn/reflexion
- **Multi-Agent Patterns**: https://blog.langchain.dev/cooperative-agents/
- **Agent Evaluation**: https://www.evidentlyai.com/blog/llm-agents-evaluation

## Safety Resources

- **Anthropic Safety**: https://www.anthropic.com/safety
- **OpenAI Safety**: https://openai.com/safety/
- **Responsible AI Practices**: https://ai.google/responsibility/
- **OWASP LLM Top 10**: https://owasp.org/www-project-top-10-for-llm-applications/

## Tools for Building Sandboxed Environments

- **Docker**: Container-based sandboxing
- **E2B**: Cloud-based sandbox for AI agents
- **Modal**: Serverless sandboxed compute
- **gVisor**: Google's container sandbox
- **Firecracker**: Lightweight VM for sandboxing
