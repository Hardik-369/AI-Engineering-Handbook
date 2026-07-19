# Cheat Sheet: Agent Engineering

## Core Agent Loop

```
┌─────────┐    ┌─────────┐    ┌─────────┐
│ Perceive│───→│  Think  │───→│   Act   │
└─────────┘    └─────────┘    └─────────┘
     ↑                              │
     └─────────── Observe ──────────┘
```

## ReAct Pattern

```
Thought: What do I need to do?
Action: tool_name(param1="value1")
Observation: <result from tool>
Thought: I have the information I need.
Answer: <final response>
```

## Agent Architecture Components

| Component | Role | Example |
|-----------|------|---------|
| **LLM** | Brain — reasoning engine | GPT-4, Claude 3.5, Gemini |
| **Tools** | Hands — interact with world | search, calculate, read_file |
| **Memory** | State — store & retrieve | context buffer, vector DB |
| **Planner** | Strategy — decide next steps | ReAct, Plan-ahead, Hierarchical |
| **Executor** | Doer — run tool calls | loop manager, tool dispatcher |
| **Evaluator** | Critic — assess quality | success check, self-reflection |

## Memory Types

```
Short-term:  Context window (~4K-200K tokens)
Long-term:   Vector DB, KV store, relational DB
Episodic:    Past trajectories, success/failure records
Semantic:    Knowledge base, learned patterns
```

## Context Management Strategy

```
if total_tokens > max_tokens:
    keep = system_prompt + recent_n_exchanges
    compress middle with summary
```

## Tool Definition Format

```python
{
    "type": "function",
    "function": {
        "name": "tool_name",
        "description": "What this tool does and when to use it",
        "parameters": {
            "type": "object",
            "properties": {
                "param1": {
                    "type": "string",
                    "description": "Description of param1"
                }
            },
            "required": ["param1"]
        }
    }
}
```

## Planning Patterns

| Pattern | Flow | Best For |
|---------|------|----------|
| **ReAct** | Thought → Action → Observation → loop | Simple Q&A, quick tasks |
| **Plan-ahead** | Generate plan → Execute → Synthesize | Multi-step procedures |
| **Hierarchical** | High-level plan → Sub-plans → Execute | Complex, decomposable tasks |
| **Re-planning** | Execute → Error → New plan → Retry | Dynamic/unpredictable tasks |

## Multi-Agent Patterns

| Pattern | Description |
|---------|-------------|
| **Supervisor** | One agent delegates to specialists |
| **Router** | Classify input → route to specialist |
| **Debate** | Multiple agents critique each other |
| **Pipeline** | Agents in sequence, handoff style |
| **Marketplace** | Agents bid on tasks |

## Safety Guardrails

```
Input Validation → Action Validation → Human Approval → Output Validation
```

### Risk Assessment

| Risk | Actions | Safeguards |
|------|---------|------------|
| Low | search, read | Input/output filters |
| Medium | write, update | Approval gates, rate limits |
| High | delete, execute code | Human-in-loop, sandbox |
| Critical | financial, email | Multi-party approval, audit |

## Agent Evaluation Metrics

| Metric | Formula | Target |
|--------|---------|--------|
| Success Rate | successes / total tasks | >80% |
| Steps per Task | total steps / total tasks | <10 |
| Cost per Task | total cost / total tasks | <$0.50 |
| Latency | total time / total tasks | <30s |
| Tool Accuracy | correct tool uses / total tool uses | >90% |

## Production Checklist

- [ ] Structured logging (JSON, correlation IDs)
- [ ] Metrics collection (steps, cost, latency)
- [ ] Distributed tracing
- [ ] Circuit breakers
- [ ] Timeouts (per-step and per-task)
- [ ] Rate limiting
- [ ] Budget controls
- [ ] Human-in-the-loop
- [ ] Graceful degradation
- [ ] State checkpointing
- [ ] Failure analysis pipeline
- [ ] A/B testing framework

## Cost Estimation

```python
cost = (input_tokens/1000 * input_rate) 
     + (output_tokens/1000 * output_rate)
     + tool_api_calls * tool_cost
```

## Common Tool Categories

| Category | Examples |
|----------|----------|
| **Search** | web_search, vector_search, db_query |
| **Read** | read_file, fetch_url, get_document |
| **Write** | write_file, send_email, create_record |
| **Code** | execute_python, run_shell, compile |
| **Data** | calculate, transform, analyze |
| **System** | list_files, get_time, sleep |

## Error Recovery Strategies

```
Tool timeout    → retry with backoff (2^n * 5s)
Tool 429        → wait Retry-After header
Tool 5xx        → retry with backoff
Bad params      → ask LLM to fix parameters
Tool missing    → try alternative tool
Plan stuck      → replan from current state
Context full    → summarize and continue
Budget exceeded → switch to cheaper model
```

## Useful Libraries

| Library | Purpose |
|---------|---------|
| LangChain/LangGraph | Agent frameworks |
| AutoGen | Multi-agent systems |
| CrewAI | Multi-agent orchestration |
| OpenAI/Anthropic SDK | LLM + tool calling |
| Chroma/Pinecone/Weaviate | Vector databases |
| Pydantic | Schema validation |
| structlog | Structured logging |
| OpenTelemetry | Distributed tracing |

## Quick Reference: Agent Types

```python
# Simple ReAct
while not done:
    thought = llm.think(state)
    if thought.has_action():
        result = execute_tool(thought.action)
        state.add_observation(result)
    else:
        done = True

# Plan-and-Execute
plan = planner.generate(task)
for step in plan:
    result = executor.run(step)
    if result.failed:
        plan = planner.replan(plan, error=result.error)

# Reflection
for attempt in range(max_attempts):
    result = agent.run(task)
    if evaluator.is_success(result):
        return result
    reflection = reflector.reflect(task, trajectory, result)
    memory.store(reflection)

# Multi-Agent Supervisor
subtasks = supervisor.decompose(task)
results = {}
for subtask in subtasks:
    agent = supervisor.select_agent(subtask)
    results[subtask.id] = await agent.run(subtask)
final = supervisor.synthesize(results)
```
