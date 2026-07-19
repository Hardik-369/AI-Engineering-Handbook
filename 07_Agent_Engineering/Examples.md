# Examples: Agent Engineering

## Example 1: Simple ReAct Agent

```python
import json
import requests
from openai import OpenAI

client = OpenAI()

def search_web(query: str) -> str:
    """Search the web for information."""
    try:
        resp = requests.get(
            "https://api.duckduckgo.com",
            params={"q": query, "format": "json"},
            timeout=10
        )
        results = resp.json().get("AbstractText", "No results found.")
        return results[:2000]
    except Exception as e:
        return f"Search failed: {e}"

def calculate(expression: str) -> str:
    """Evaluate a mathematical expression."""
    try:
        return str(eval(expression))
    except Exception as e:
        return f"Calculation error: {e}"

TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "search_web",
            "description": "Search the web for current information",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "Search query"}
                },
                "required": ["query"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "calculate",
            "description": "Evaluate a math expression",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {"type": "string", "description": "Math expression"}
                },
                "required": ["expression"]
            }
        }
    }
]

TOOL_MAP = {"search_web": search_web, "calculate": calculate}

def run_agent(task: str, max_steps: int = 10) -> str:
    messages = [
        {"role": "system", "content": "You are a helpful assistant with tools. "
         "Think step by step, use tools when needed, and provide final answers."},
        {"role": "user", "content": task}
    ]

    for step in range(max_steps):
        response = client.chat.completions.create(
            model="gpt-4",
            messages=messages,
            tools=TOOLS,
            tool_choice="auto"
        )

        msg = response.choices[0].message
        messages.append(msg)

        if msg.tool_calls:
            for tc in msg.tool_calls:
                func = tc.function
                args = json.loads(func.arguments)
                result = TOOL_MAP[func.name](**args)
                messages.append({
                    "role": "tool",
                    "tool_call_id": tc.id,
                    "content": result
                })
        else:
            return msg.content

    return "Max steps reached."

# Usage
print(run_agent("What is the population of Japan divided by 2?"))
```

## Example 2: Tool Use with Error Handling

```python
import time
from dataclasses import dataclass
from typing import Any

@dataclass
class ToolResult:
    success: bool
    data: Any = None
    error: str = ""

class RobustToolExecutor:
    def __init__(self, max_retries=3, base_delay=1.0):
        self.max_retries = max_retries
        self.base_delay = base_delay

    def execute(self, tool_func, **kwargs) -> ToolResult:
        for attempt in range(self.max_retries):
            try:
                result = tool_func(**kwargs)
                return ToolResult(success=True, data=result)
            except requests.Timeout:
                delay = self.base_delay * (2 ** attempt)
                print(f"Timeout, retrying in {delay}s (attempt {attempt + 1})")
                time.sleep(delay)
            except requests.HTTPError as e:
                if e.response.status_code == 429:
                    delay = float(e.response.headers.get("Retry-After", self.base_delay))
                    time.sleep(delay)
                elif e.response.status_code >= 500:
                    time.sleep(self.base_delay * (2 ** attempt))
                else:
                    return ToolResult(success=False, error=str(e))
            except Exception as e:
                return ToolResult(success=False, error=str(e))

        return ToolResult(success=False, error="Max retries exceeded")

def fetch_url(url: str) -> str:
    resp = requests.get(url, timeout=10)
    resp.raise_for_status()
    return resp.text[:5000]

executor = RobustToolExecutor()
result = executor.execute(fetch_url, url="https://example.com")
```

## Example 3: Planning Agent

```python
from pydantic import BaseModel
from typing import Optional

class Step(BaseModel):
    id: str
    tool: str
    params: dict
    dependencies: list[str] = []
    status: str = "pending"
    result: Optional[str] = None

class Plan(BaseModel):
    goal: str
    steps: list[Step]
    current_step: int = 0

class PlanningAgent:
    def __init__(self, llm, tools):
        self.llm = llm
        self.tools = tools

    def create_plan(self, goal: str) -> Plan:
        prompt = f"""Create a step-by-step plan to achieve: {goal}

Available tools: {list(self.tools.keys())}

Return a JSON plan with steps containing: id, tool, params, dependencies.
Max 8 steps."""
        response = self.llm.generate(prompt)
        steps_data = json.loads(response)
        return Plan(goal=goal, steps=[Step(**s) for s in steps_data])

    async def execute_plan(self, plan: Plan) -> str:
        while plan.current_step < len(plan.steps):
            step = plan.steps[plan.current_step]

            # Check dependencies
            deps_met = all(
                plan.steps[int(d)].status == "completed"
                for d in step.dependencies
            )
            if not deps_met:
                plan.current_step += 1
                continue

            # Execute
            tool = self.tools.get(step.tool)
            if not tool:
                step.status = "failed"
                step.result = f"Unknown tool: {step.tool}"
                continue

            try:
                result = await tool(**step.params)
                step.status = "completed"
                step.result = str(result)[:1000]
            except Exception as e:
                step.status = "failed"
                step.result = str(e)

            plan.current_step += 1

        return self.synthesize(plan)
```

## Example 4: Memory-Augmented Agent

```python
import chromadb
from sentence_transformers import SentenceTransformer

class AgentMemory:
    def __init__(self):
        self.embedder = SentenceTransformer("all-MiniLM-L6-v2")
        self.client = chromadb.Client()
        self.collection = self.client.create_collection(
            name="agent_memory",
            embedding_function=self.embedder.encode
        )
        self.conversation_history = []

    def store_experience(self, task: str, action: str, result: str, success: bool):
        self.collection.add(
            documents=[f"Task: {task}\nAction: {action}\nResult: {result}"],
            metadatas=[{"success": str(success), "task": task}],
            ids=[f"exp_{int(time.time())}"]
        )

    def retrieve_similar(self, query: str, k: int = 3) -> list[str]:
        results = self.collection.query(
            query_texts=[query],
            n_results=k
        )
        return results["documents"][0] if results["documents"] else []

    def get_recent_context(self, n: int = 5) -> list[dict]:
        return self.conversation_history[-n:]

    def compress_history(self, llm) -> str:
        if len(self.conversation_history) > 20:
            text = json.dumps(self.conversation_history[:-10])
            summary = llm.generate(
                f"Summarize this conversation history concisely:\n{text}"
            )
            self.conversation_history = (
                [{"role": "system", "content": f"Previous context: {summary}"}]
                + self.conversation_history[-10:]
            )

class MemoryAgent:
    def __init__(self, llm, tools):
        self.llm = llm
        self.tools = tools
        self.memory = AgentMemory()

    async def run(self, task: str) -> str:
        # Retrieve relevant past experiences
        similar = self.memory.retrieve_similar(task)
        context = "\n".join(similar)

        messages = [
            {"role": "system", "content": (
                "You are an agent with access to past experiences."
                f"Relevant past experiences:\n{context}"
            )},
            *self.memory.get_recent_context(),
            {"role": "user", "content": task}
        ]

        response = await self.llm.generate(messages, tools=self.tools)
        self.memory.conversation_history.append(
            {"role": "assistant", "content": response}
        )

        self.memory.compress_history(self.llm)
        return response
```

## Example 5: Multi-Agent System with LangGraph

```python
from typing import TypedDict, Literal
from langgraph.graph import StateGraph, END
from langgraph.checkpoint import MemorySaver
import operator

class AgentState(TypedDict):
    messages: list
    task: str
    research_results: list
    code: str
    review_comments: list
    next_agent: str

# Define agent nodes
async def researcher(state: AgentState) -> AgentState:
    """Research agent that gathers information."""
    task = state["task"]
    # Simulate research
    results = [
        {"source": "web", "content": f"Research findings for: {task}"},
        {"source": "docs", "content": "Documentation reference..."}
    ]
    state["research_results"] = results
    state["next_agent"] = "coder"
    return state

async def coder(state: AgentState) -> AgentState:
    """Coding agent that writes implementation."""
    task = state["task"]
    research = state.get("research_results", [])
    # Generate code based on research
    state["code"] = (
        f"# Implementation for: {task}\n"
        f"# Based on {len(research)} research sources\n"
        "def solution():\n    pass"
    )
    state["next_agent"] = "reviewer"
    return state

async def reviewer(state: AgentState) -> AgentState:
    """Review agent that checks quality."""
    code = state.get("code", "")
    comments = [
        {"line": 1, "comment": "Add docstring"},
        {"line": 3, "comment": "Consider error handling"}
    ]
    state["review_comments"] = comments

    if comments:
        state["next_agent"] = "coder"  # Send back for fixes
    else:
        state["next_agent"] = END
    return state

# Build the graph
def create_multi_agent_system():
    workflow = StateGraph(AgentState)

    workflow.add_node("researcher", researcher)
    workflow.add_node("coder", coder)
    workflow.add_node("reviewer", reviewer)

    workflow.set_entry_point("researcher")

    workflow.add_conditional_edges(
        "researcher",
        lambda s: s["next_agent"],
        {"coder": "coder"}
    )
    workflow.add_conditional_edges(
        "coder",
        lambda s: s["next_agent"],
        {"reviewer": "reviewer", END: END}
    )
    workflow.add_conditional_edges(
        "reviewer",
        lambda s: s["next_agent"],
        {"coder": "coder", END: END}
    )

    return workflow.compile(checkpointer=MemorySaver())

# Usage
async def main():
    system = create_multi_agent_system()
    initial_state = {
        "messages": [],
        "task": "Build a Python function to sort a list of dictionaries by a key",
        "research_results": [],
        "code": "",
        "review_comments": [],
        "next_agent": "researcher"
    }
    result = await system.ainvoke(initial_state)
    print(f"Final code:\n{result['code']}")

# Run: asyncio.run(main())
```

## Example 6: Reflexion Agent

```python
class ReflexionAgent:
    def __init__(self, llm, tools, max_attempts=5):
        self.llm = llm
        self.tools = tools
        self.max_attempts = max_attempts
        self.memories = []  # Store reflections

    async def run(self, task: str) -> str:
        context = self._build_context()

        for attempt in range(self.max_attempts):
            trajectory = []

            # Act
            response, actions = await self._act(task, context)
            trajectory.extend(actions)

            # Evaluate
            evaluation = await self._evaluate(task, response)

            if evaluation["success"]:
                return response

            # Reflect
            reflection = await self._reflect(task, trajectory, evaluation)
            self.memories.append(reflection)
            context = self._build_context()

        return f"Failed after {self.max_attempts} attempts."

    async def _act(self, task, context):
        prompt = f"{context}\nTask: {task}\n"
        actions = []
        response = ""

        for _ in range(10):
            result = await self.llm.generate(
                prompt, tools=self.tools
            )
            if hasattr(result, "tool_calls") and result.tool_calls:
                for tc in result.tool_calls:
                    tool_result = await self.tools[tc.name](**tc.args)
                    actions.append({"call": tc, "result": tool_result})
                    prompt += f"\nObservation: {tool_result}\n"
            else:
                response = result
                break

        return response, actions

    async def _evaluate(self, task, response):
        prompt = f"""Task: {task}
Response: {response}

Evaluate: Did this response correctly complete the task?
Respond with JSON: {{"success": bool, "issues": [str], "score": float}}"""
        evaluation = await self.llm.generate(prompt)
        return json.loads(evaluation)

    async def _reflect(self, task, trajectory, evaluation):
        prompt = f"""Task: {task}
What was attempted: {json.dumps(trajectory, indent=2)}
Issues: {evaluation['issues']}

Generate a reflection on what went wrong and how to improve.
Be specific about what to do differently."""
        reflection = await self.llm.generate(prompt)
        return {"task": task, "reflection": reflection}

    def _build_context(self):
        if not self.memories:
            return ""
        memories_text = "\n".join(
            f"Past reflection: {m['reflection']}" for m in self.memories[-3:]
        )
        return f"Previous lessons:\n{memories_text}\n"
```

## Example 7: Research Agent

```python
class ResearchAgent:
    def __init__(self, llm):
        self.llm = llm
        self.search_tool = SearchTool()
        self.read_tool = ReadTool()

    async def research(self, topic: str, depth: int = 3) -> dict:
        # Phase 1: Formulate queries
        queries = await self._generate_queries(topic)

        # Phase 2: Search
        all_results = []
        for query in queries:
            results = await self.search_tool.search(query, num=5)
            all_results.extend(results)

        # Deduplicate
        unique_results = self._deduplicate(all_results)

        # Phase 3: Read top results
        documents = []
        for result in unique_results[:depth]:
            content = await self.read_tool.read(result.url)
            documents.append({
                "url": result.url,
                "title": result.title,
                "content": content[:5000]
            })

        # Phase 4: Synthesize
        report = await self._synthesize(topic, documents)

        # Phase 5: Fact-check
        verified_report = await self._fact_check(report, documents)

        return {
            "topic": topic,
            "sources": [d["url"] for d in documents],
            "report": verified_report,
            "queries_used": queries,
            "documents_analyzed": len(documents)
        }

    async def _generate_queries(self, topic: str) -> list[str]:
        prompt = f"""Generate 3 search queries to research: {topic}
Return as JSON list of strings."""
        response = await self.llm.generate(prompt)
        return json.loads(response)

    async def _synthesize(self, topic, documents) -> str:
        context = "\n\n".join(
            f"Source {i+1} ({d['url']}):\n{d['content'][:2000]}"
            for i, d in enumerate(documents)
        )
        prompt = f"""Synthesize the following information about "{topic}" into
a well-structured report with sections: Overview, Key Findings, and Conclusion.

Sources:
{context}

Report:"""
        return await self.llm.generate(prompt)

    async def _fact_check(self, report, documents) -> str:
        prompt = f"""Fact-check this report against the original sources.
Report: {report}
Sources: {json.dumps([d['content'][:1000] for d in documents])}

Verify claims and note any inconsistencies:"""
        verification = await self.llm.generate(prompt)
        return f"{report}\n\n## Fact Check\n{verification}"
```

## Example 8: Supervisor Agent Pattern

```python
class SupervisorAgent:
    def __init__(self, llm):
        self.llm = llm
        self.agents = {
            "research": ResearchAgent(llm),
            "analysis": AnalysisAgent(llm),
            "writing": WritingAgent(llm),
        }

    async def run(self, task: str) -> str:
        # Decompose
        subtasks = await self._decompose(task)

        results = {}
        for subtask in subtasks:
            agent = self._select_agent(subtask)
            result = await agent.run(subtask.description)
            results[subtask.id] = result

        # Synthesize
        return await self._synthesize(task, results)

    async def _decompose(self, task: str) -> list:
        prompt = f"""Decompose this task into subtasks with required agent type
(research, analysis, or writing): {task}
Return as JSON list: [{{"id": str, "agent": str, "description": str}}]"""
        response = await self.llm.generate(prompt)
        return json.loads(response)

    def _select_agent(self, subtask):
        return self.agents.get(subtask["agent"])

    async def _synthesize(self, task, results):
        context = "\n".join(
            f"Subtask {k}: {v}" for k, v in results.items()
        )
        prompt = f"""Synthesize these results into a final answer for the task.

Task: {task}
Results:
{context}

Final answer:"""
        return await self.llm.generate(prompt)
```

## Example 9: Human-in-the-Loop Agent

```python
class HumanInTheLoopAgent:
    def __init__(self, llm, tools, risk_policy=None):
        self.llm = llm
        self.tools = tools
        self.risk_policy = risk_policy or self._default_policy()
        self.pending_approvals = []

    async def run(self, task, auto_approve=False):
        messages = [{"role": "user", "content": task}]

        for step in range(30):
            response = await self.llm.generate(messages, tools=self.tools)

            if response.tool_calls:
                for tc in response.tool_calls:
                    risk = self._assess_risk(tc.name, tc.args)
                    if risk > 0.5 and not auto_approve:
                        approved = await self._request_approval(tc, risk)
                        if not approved:
                            continue

                    result = await self._safe_execute(tc)
                    messages.append({
                        "role": "tool",
                        "content": str(result)
                    })
            else:
                return response.content

        return "Max steps"

    def _assess_risk(self, tool_name, args):
        policy = self.risk_policy.get(tool_name, {"risk": 0.1})
        return policy["risk"]

    async def _request_approval(self, tool_call, risk):
        print(f"\n[APPROVAL REQUIRED] Risk: {risk:.1f}")
        print(f"Tool: {tool_call.name}")
        print(f"Args: {json.dumps(tool_call.args, indent=2)}")
        response = input("Approve? (y/n): ")
        return response.lower() == "y"

    async def _safe_execute(self, tool_call):
        try:
            return await self.tools[tool_call.name](**tool_call.args)
        except Exception as e:
            return {"error": str(e)}

    def _default_policy(self):
        return {
            "search_web": {"risk": 0.1},
            "read_file": {"risk": 0.3},
            "write_file": {"risk": 0.7},
            "execute_code": {"risk": 0.9},
            "delete_file": {"risk": 1.0},
            "send_email": {"risk": 0.8},
            "api_call": {"risk": 0.4},
        }
```

## Example 10: Streaming Agent Output

```python
from typing import AsyncGenerator

class StreamingAgent:
    def __init__(self, llm, tools):
        self.llm = llm
        self.tools = tools

    async def run(self, task: str) -> AsyncGenerator[dict, None]:
        yield {"type": "status", "content": "Starting agent..."}
        messages = [{"role": "user", "content": task}]

        for step in range(10):
            yield {"type": "thought", "step": step, "content": "Processing..."}

            # Stream the LLM response
            full_response = ""
            async for chunk in self.llm.stream(messages, self.tools):
                if chunk.type == "token":
                    full_response += chunk.content
                    yield {"type": "token", "content": chunk.content}
                elif chunk.type == "tool_call_start":
                    yield {
                        "type": "tool_call",
                        "name": chunk.name,
                        "params": chunk.params
                    }
                elif chunk.type == "tool_call_end":
                    yield {
                        "type": "tool_result",
                        "name": chunk.name,
                        "content": chunk.result[:500]
                    }
                    messages.append({
                        "role": "tool",
                        "content": str(chunk.result)
                    })

            if not hasattr(full_response, "tool_calls"):
                yield {"type": "final", "content": full_response}
                return

            messages.append({"role": "assistant", "content": full_response})

        yield {"type": "error", "content": "Max steps reached"}
```
