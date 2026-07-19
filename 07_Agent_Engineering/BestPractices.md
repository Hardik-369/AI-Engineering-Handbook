# Best Practices: Agent Engineering

## Prompting for Agents

### System Prompt Design
- **Define the agent's persona clearly**: "You are a helpful research assistant with access to web search, document reading, and data analysis tools."
- **Describe the loop**: Explain how the agent should think, act, and observe. Include a step-by-step instruction for the ReAct pattern.
- **List all tools with precise usage guidelines**: For each tool, explain what it does, when to use it, and what returns.
- **Set boundaries**: Specify what the agent should NOT do, and when to ask for help.
- **Include output format requirements**: "Always provide final answers in markdown format."
- **Provide examples**: Include 1-3 examples of good agent behavior.

### Tool Description Best Practices
- **Use descriptive names**: `search_web` not `func1` or `tool_a`.
- **Write clear, detailed descriptions**: "Search the web for current information. Use for facts, news, recent events, and general knowledge queries. Returns title, URL, and snippet for each result."
- **Specify parameter formats**: Include formats, defaults, and constraints.
- **Mention edge cases**: "Returns empty results for very specific queries — try broader terms."
- **Indicate cost/time**: "This tool costs ~$0.01 per call and takes 1-3 seconds."
- **Describe return structure**: "Returns a list of dicts with keys: title, url, snippet, source."

### Few-Shot Examples
- Include examples in the system prompt showing the agent handling similar scenarios.
- Show both successful and unsuccessful tool calls with corrections.
- Demonstrate the thinking process: "Thought: I need to find X. Action: search('X')."

## Architecture Best Practices

### Design Principles
- **Start simple, add complexity gradually**: Begin with a basic ReAct loop, then add memory, planning, and multi-agent coordination.
- **Separate concerns**: Keep planning, execution, memory, and evaluation as distinct modules.
- **Make the agent observable**: Log every thought, action, and observation. This is critical for debugging.
- **Design for failure**: Assume tools will fail, LLMs will hallucinate, and plans will need revision.
- **Test incrementally**: Test each component (tools, memory, planner) before integrating.

### State Management
- **Never mutate state in place**: Always return new state objects or use immutable patterns.
- **Checkpoint critical states**: Save state before and after major tool calls for recovery.
- **Serialize state carefully**: Use JSON-compatible types. Handle non-serializable objects (e.g., file handles).
- **Version your state schema**: As your agent evolves, state format will change. Handle migrations.

### Error Handling
- **Catch every tool failure**: Wrap every tool call in try/except with structured error responses.
- **Provide useful error messages to the LLM**: Include what went wrong and how to fix it.
- **Implement retry logic with backoff**: Network errors and rate limits are common.
- **Have a fallback strategy**: If the primary tool fails, try a different approach.
- **Set maximum retries per tool**: Prevent infinite retry loops.

## Memory Best Practices

### Context Management
- **Monitor token usage continuously**: Track both input and output tokens per step.
- **Summarize aggressively**: When approaching the context limit, compress earlier exchanges.
- **Keep the system prompt and recent messages**: These are usually the most important.
- **Use structured summaries**: Extract key facts, decisions, and pending actions.
- **Store full trajectories externally**: Don't rely on the context window for long-term memory.

### Vector Store Usage
- **Chunk documents appropriately**: 500-1000 token chunks with overlap.
- **Store metadata alongside vectors**: Task type, timestamp, success status.
- **Use hybrid search**: Combine vector similarity with keyword filtering.
- **Retrieve with diversity**: Use MMR (Maximum Marginal Relevance) to avoid redundant results.
- **Update embeddings periodically**: As models improve, re-embed stored content.

## Tool Design Best Practices

### Creating Tools
- **Make tools focused**: Each tool should do one thing well.
- **Return structured data**: Prefer JSON over plain text for tool results.
- **Include error information in results**: Don't just raise exceptions.
- **Design for streaming**: Tools that take a while should yield intermediate progress.
- **Add idempotency keys**: For tools that trigger side effects, prevent duplicate execution.

### Tool Descriptions
- **Specify exactly when to use**: "Use this tool when you need current information from the web."
- **Also specify when NOT to use**: "Do NOT use this tool for mathematical calculations — use calculate() instead."
- **Provide query examples**: "Good query: 'latest AI developments 2025'. Poor query: 'stuff'."
- **Mention limitations**: "This tool only returns text content, not images or videos."

## Safety Best Practices

### By Risk Level

| Risk Level | Examples | Required Safeguards |
|------------|----------|---------------------|
| Low | Text search, date lookup | Input validation, output filtering |
| Medium | File reading, data processing | Approval gates, rate limiting |
| High | Code execution, file writing, API calls | Human-in-the-loop, sandboxing, budget limits |
| Critical | Database writes, financial transactions, external communications | Multi-party approval, audit trails, rollback capability |

### Guardrails
- **Validate all inputs**: Check for prompt injection attempts.
- **Validate all outputs**: Ensure responses don't contain harmful content.
- **Limit tool access by scope**: Only provide tools the agent actually needs.
- **Set hard timeouts**: Both per-step and per-task.
- **Budget limits**: Set per-task and per-session API budgets.
- **Human approval gates**: Require human approval for sensitive operations.

## Production Best Practices

### Observability
- **Log every step**: Thought, action, observation, and result. Include timestamps and token counts.
- **Use structured logging**: JSON format with consistent fields (correlation_id, agent_id, step_number, etc.).
- **Set up metrics**: Step count, latency, cost, success rate, tool usage frequency.
- **Implement tracing**: Trace across LLM calls, tool executions, and memory operations.
- **Create dashboards**: Visualize agent performance over time.

### Performance
- **Cache tool results**: If the same tool is called with the same params, return cached result.
- **Use cheaper models for simple tasks**: Route simple queries to cheaper/faster models.
- **Parallelize independent tool calls**: Use asyncio to execute multiple tool calls concurrently.
- **Stream outputs**: Show progress to users rather than making them wait.
- **Pre-warm connections**: Keep persistent connections to APIs and databases.

### Reliability
- **Implement circuit breakers**: If a tool repeatedly fails, stop calling it temporarily.
- **Use dead-letter queues**: Store failed tasks for later analysis and retry.
- **Graceful degradation**: If the primary LLM is unavailable, fall back to a cheaper model.
- **Checkpoint agent state**: Save state at key points for recovery.
- **Version your agents**: Use semantic versioning for agent definitions.

## Testing Best Practices

### Unit Tests
- Test each tool in isolation with valid and invalid inputs.
- Test memory operations (store, retrieve, summarize).
- Test planning logic with controlled inputs.
- Test error handling code paths.

### Integration Tests
- Test the complete agent loop with mock LLM responses.
- Test multi-step scenarios that require planning and re-planning.
- Test memory persistence across sessions.
- Test safety guardrails with adversarial inputs.

### Evaluation Tests
- Create a benchmark dataset of real tasks with expected outcomes.
- Run the agent on each task and compute success rate.
- Analyze failures systematically to find common patterns.
- Track metrics over time to detect regressions.

## Common Pitfalls to Avoid

1. **Over-tooling**: Giving the agent too many tools leads to confusion. Start with 3-5 tools.
2. **Weak tool descriptions**: Vague descriptions cause the LLM to misuse tools.
3. **Ignoring context limits**: Long agent trajectories hit context limits. Plan for it.
4. **No error recovery**: Assuming tools always work leads to brittle agents.
5. **Over-engineering**: Start with a simple ReAct loop. Add complexity only when needed.
6. **No observability**: Trying to debug a black-box agent is nearly impossible.
7. **Unbounded costs**: Without budget limits, agent costs can spiral.
8. **Testing only sunny-day scenarios**: Test failures, edge cases, and adversarial inputs.
9. **Single-threaded execution**: Failing to parallelize independent tool calls wastes time.
10. **Ignoring safety**: Safety should be designed in from the start, not bolted on later.

## When to Use Each Pattern

| Pattern | When to Use | When NOT to Use |
|---------|-------------|-----------------|
| **ReAct** | Simple tasks, quick prototyping | Complex tasks requiring deep planning |
| **Plan-ahead** | Tasks with clear, sequential steps | Dynamic environments with frequent changes |
| **Hierarchical** | Very complex tasks, large codebases | Simple, single-step tasks |
| **Single agent** | Well-scoped, focused tasks | Tasks requiring diverse expertise |
| **Multi-agent** | Tasks needing multiple specialities | Simple tasks where one agent suffices |
| **Reflexion** | Tasks where learning from mistakes helps | One-shot tasks with no repeat opportunity |
| **Human-in-loop** | High-risk operations | Low-risk, high-volume tasks |

## Code Organization

```
agents/
├── core/
│   ├── agent.py          # Base agent class
│   ├── loop.py           # Agent execution loop
│   ├── state.py          # State management
│   └── types.py          # Common types
├── tools/
│   ├── base.py           # Tool base class
│   ├── web_search.py
│   ├── code_executor.py
│   └── file_operations.py
├── memory/
│   ├── base.py           # Memory interface
│   ├── buffer.py         # Short-term (conversation buffer)
│   ├── vector.py         # Long-term (vector store)
│   └── episodic.py       # Episodic memory
├── planning/
│   ├── react.py          # ReAct planner
│   ├── plan_ahead.py     # Plan-ahead planner
│   └── hierarchical.py   # Hierarchical planner
├── safety/
│   ├── guardrails.py     # Input/output validation
│   ├── budget.py         # Cost management
│   └── human_loop.py     # Human-in-the-loop
├── evaluation/
│   ├── metrics.py        # Evaluation metrics
│   ├── benchmark.py      # Benchmark runner
│   └── analysis.py       # Failure analysis
├── multi_agent/
│   ├── supervisor.py     # Supervisor pattern
│   ├── router.py         # Router pattern
│   ├── debate.py         # Debate pattern
│   └── shared_memory.py  # Shared memory
├── production/
│   ├── logging.py        # Structured logging
│   ├── metrics.py        # Metrics collection
│   ├── tracing.py        # Distributed tracing
│   └── recovery.py       # Error recovery
└── config/
    ├── settings.py       # Configuration
    └── models.py         # Model definitions
```
