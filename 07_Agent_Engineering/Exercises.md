# Exercises: Agent Engineering

## Beginner

### Exercise 1: Handcrafted ReAct Loop
Implement a ReAct agent manually (without any framework). Create a simple agent that has access to `calculate` and `get_current_time` tools, and can answer math questions involving dates.

**Tools needed:**
- `calculate(expression: str) -> str`: Evaluate math expressions
- `get_current_time() -> str`: Get current date/time

**Test with:** "What day of the week will it be 30 days from now?"

### Exercise 2: Tool Descriptions
Write tool descriptions for three tools: `search_database`, `send_email`, and `generate_image`. For each tool, write:
- A clear name and description
- Input parameters with types and descriptions
- Return value description
- A note about when to (and when NOT to) use this tool

Compare your descriptions with a partner and discuss which is clearer.

### Exercise 3: Simple Tool Executor
Build a class `ToolExecutor` that takes a tool definition and a handler function, and safely executes tool calls with error handling. Include:
- Parameter validation against the JSON schema
- Timeout support (30 seconds)
- Retry logic (3 attempts with exponential backoff)
- Structured error responses

### Exercise 4: Context Window Manager
Write a function `manage_context_window(messages, max_tokens)` that:
- Takes a list of messages and a token limit
- Trims old messages if the total exceeds the limit
- Preserves the system prompt and the most recent user message
- Returns the trimmed message list

Test with a mock conversation of 50 exchanges.

### Exercise 5: Agent State Tracker
Implement a class `AgentState` that tracks:
- The current task description
- Steps taken (with timestamps)
- Current plan (list of steps)
- Errors encountered
- Total tokens used

Add methods: `add_step()`, `add_error()`, `summary()`, and `to_dict()`.

## Intermediate

### Exercise 6: ReAct Agent with Memory
Extend the ReAct agent from Exercise 1 to include memory:
- Store successful trajectories in a vector store (use Chroma or a simple dict)
- Before acting, retrieve similar past trajectories
- Use retrieved examples to inform decisions
- Test with: "What's 15% of 340?" after having answered "What's 10% of 200?"

### Exercise 7: Plan-and-Execute Agent
Build a plan-and-execute agent that:
1. Takes a high-level goal like "Plan a 3-day trip to Paris"
2. Generates a step-by-step plan
3. Executes each step, using tools as needed
4. Can re-plan if a step fails
5. Returns a final synthesized result

Tools: `search_web`, `get_weather`, `book_flight` (mock), `get_hotel_info` (mock)

### Exercise 8: Self-Reflecting Agent
Implement an agent with reflexion capabilities:
- Run a task, track the trajectory
- Evaluate the outcome (success/failure)
- Generate a textual reflection on what went wrong
- Store the reflection
- On the next attempt, include past reflections in the prompt

Test on a task that requires multiple attempts (e.g., "Write code that divides by zero, then fix it").

### Exercise 9: Agent with Episodic Memory
Build a `TrajectoryStore` that:
- Records every action the agent takes (tool, params, result, timestamp)
- Stores trajectories indexed by task similarity
- Can retrieve past trajectories for similar tasks
- Can summarize a trajectory into a "lesson learned"

Use for an agent that performs data analysis tasks.

### Exercise 10: Multi-Agent Debate System
Create a debate system with 3 agents:
1. Each agent generates an initial answer to a question
2. Over 3 rounds, agents see each other's answers and revise their own
3. A judge agent synthesizes the final answer
4. Track how answers converge (or diverge) over rounds

Test on: "What is the most effective approach to reduce carbon emissions?"

### Exercise 11: Router Agent
Build a router agent that:
- Classifies user input into categories (coding, research, creative, analysis)
- Routes to specialized sub-agents
- The sub-agents return results
- The router formats a unified response

Create mock sub-agents that return plausible outputs for each category.

### Exercise 12: Tool Calling with Parallel Execution
Implement an agent that can call multiple tools in parallel:
- The LLM returns multiple tool calls in one response
- Execute all calls concurrently using asyncio
- Return all results to the LLM at once
- Handle partial failures (some tools succeed, others fail)

Test with: "Search for 'AI news' and 'Python tutorials' and calculate 15 * 27"

### Exercise 13: Agent with Summarization
Build an agent that automatically summarizes long conversations:
- Track token usage per message
- When approaching context limit, compress older exchanges
- Use a secondary LLM call to generate summaries
- Preserve key facts and decisions in the summary
- Verify the summary retains all critical information

## Advanced

### Exercise 14: Supervisor Agent for Code Review
Create a multi-agent system for code review:
1. **Supervisor**: Receives code, delegates to reviewers
2. **Style Reviewer**: Checks code style, naming, formatting
3. **Logic Reviewer**: Checks for bugs, edge cases, correctness
4. **Security Reviewer**: Checks for vulnerabilities
5. **Supervisor**: Aggregates all feedback, prioritizes issues

Each reviewer should return structured feedback. The supervisor produces a final report.

### Exercise 15: Agent with Safety Guardrails
Implement a safety layer for an agent that can write files and execute code:
- **Input guard**: Reject malicious or harmful requests
- **Action validator**: Check every tool call against policies
- **Output guard**: Ensure outputs don't contain harmful content
- **Human-in-the-loop**: Require approval for high-risk actions
- **Rate limiter**: Limit tool call frequency

Test with: "Delete all files in the home directory" and "Send 1000 emails"

### Exercise 16: Agent Evaluation Framework
Build a framework to evaluate agent performance:
- Create a test dataset of 10 tasks with expected outcomes
- Run your agent on each task
- Record trajectory, cost, steps, and outcome for each
- Compute metrics: success rate, average steps, average cost
- Generate a failure analysis report
- Visualize results with a table or chart

### Exercise 17: Hierarchical Planning Agent
Implement hierarchical planning:
- High-level planner creates a plan with abstract steps
- Each abstract step is delegated to a sub-planner
- Sub-planners create detailed plans with tool calls
- Handle failures at any level with re-planning
- Support 3 levels of hierarchy minimum

Test on: "Organize a virtual conference with 5 speakers and 200 attendees"

### Exercise 18: Agent with Dynamic Tool Creation
Build an agent that can create new tools dynamically:
- Agent can write Python functions as new tools
- New tools are registered and available immediately
- Tool descriptions are auto-generated from docstrings
- Agent can compose tools (use one tool's output as another's input)
- Implement a simple sandbox for code execution

### Exercise 19: Production Agent with Observability
Take any agent from a previous exercise and add:
- Structured logging (JSON format with correlation IDs)
- Metrics collection (step count, latency, cost)
- Distributed tracing across tool calls
- Health check endpoint
- Graceful shutdown on errors
- Configuration via environment variables
- Circuit breaker for external API calls

### Exercise 20: Agent with Budget Management
Build an agent that operates within a budget:
- Track costs: LLM tokens (input/output), API calls, tool execution time
- Set per-task and per-session budgets
- Before each action, estimate the cost
- If approaching budget, switch to cheaper models
- If budget exceeded, gracefully degrade (e.g., use simpler tools)
- Log all costs for analysis

**Bonus:** Implement a "budget negotiation" where the agent suggests trade-offs to the user.

### Exercise 21: Collaborative Multi-Agent System
Build a system with at least 4 specialized agents that collaborate on a complex task:
- Agents communicate through a shared message bus
- Each agent has a specific capability and can't do others
- Agents can request help from other agents
- Implement a priority system for task scheduling
- Handle agent failures (an agent goes down mid-task)

Test on: "Build a simple web app with user authentication"

### Exercise 22: Agent That Learns from Feedback
Implement an agent that improves from user feedback:
- After each response, the user can provide feedback (correct/incorrect)
- Store feedback in episodic memory
- Automatically detect patterns in feedback
- Adjust behavior based on learned patterns
- Generate a weekly improvement report

### Exercise 23: Streaming Agent with Real-Time UI
Create a streaming agent that emits events for each stage:
- `status`: "Searching for information..."
- `thought`: "I think I need to look up X..."
- `tool_call`: "Calling search_web with query='...'"
- `tool_result`: "Found 5 results"
- `token`: Individual response tokens

Build a simple CLI or web UI that renders these events differently.

### Exercise 24: Agent Security Challenge
Build an agent and try to break it:
- Attempt prompt injection to override instructions
- Try to make the agent call dangerous tools
- Attempt to exfiltrate data through tool outputs
- Try to make the agent exceed its budget
- Implement defenses against each attack

Document each attack, whether it succeeded, and how you fixed it.

### Exercise 25: Multi-Turn Research Agent
Build an agent that can conduct multi-turn research:
- Takes a research question
- Generates search queries
- Reads and analyzes results
- Identifies gaps in the information
- Generates follow-up queries
- Produces a comprehensive report with citations
- Allows the user to ask follow-up questions about the research
