# Exercises: Loop Engineering

> **Prerequisites:** Completion of Chapter 04 README
>
> **Difficulty Levels:** 🔵 Beginner | 🟡 Intermediate | 🔴 Advanced

---

## 🔵 Beginner Exercises

### Exercise 1: Simple Retry Loop

Build a function that calls an LLM with retry logic. Simulate failures using a mock LLM that fails randomly.

**Requirements:**
- Max 5 retries
- Exponential backoff (base delay: 1s)
- Jitter (random 0-100ms)
- Return the response or raise an error after all retries fail

**Starter:**
```python
import random, time

def flaky_llm(prompt):
    """Mock LLM that fails 60% of the time."""
    if random.random() < 0.6:
        raise ConnectionError("API timeout")
    return f"Response to: {prompt}"

def retry_llm(prompt, max_retries=5, base_delay=1.0):
    # Your code here
    pass

# Test
result = retry_llm("Hello")
print(result)
```

**Success criteria:** The function successfully returns a response within 5 retries with high probability.

---

### Exercise 2: Basic Reflection Loop

Build a loop where the LLM generates an answer, then reviews its own answer for errors, and provides a corrected version.

**Requirements:**
- Generate initial answer to a math problem
- Use a structured self-review prompt
- If errors found, generate a corrected version
- Max 3 iterations
- Track what changed each iteration

**Test with:**
- "Calculate 15 * 37 + 42 / 2"
- "If a train travels at 60 mph for 2.5 hours, how far does it go?"

**Starter:**
```python
def reflection_loop(question, max_iterations=3):
    # Your code here
    pass

result = reflection_loop("Calculate 15 * 37 + 42 / 2")
print(result)
```

---

### Exercise 3: Validation Loop

Build a loop that generates structured JSON output and validates it against a schema.

**Requirements:**
- Generate a JSON object describing a person (name, age, email, address)
- Validate against a JSON schema
- If validation fails, feed the error back to the LLM
- Max 3 attempts
- Track validation errors per attempt

**Schema:**
```python
schema = {
    "type": "object",
    "required": ["name", "age", "email", "address"],
    "properties": {
        "name": {"type": "string", "minLength": 2},
        "age": {"type": "integer", "minimum": 0, "maximum": 150},
        "email": {"type": "string", "pattern": "^[\\w.-]+@[\\w.-]+\\.\\w{2,}$"},
        "address": {
            "type": "object",
            "required": ["street", "city", "zip"],
            "properties": {
                "street": {"type": "string"},
                "city": {"type": "string"},
                "zip": {"type": "string", "pattern": "^\\d{5}$"}
            }
        }
    }
}
```

---

## 🟡 Intermediate Exercises

### Exercise 4: Code Generation + Test Feedback Loop

Build a system that generates a Python function from a description, runs unit tests, and refines until all tests pass.

**Requirements:**
- Take a function description and unit tests as input
- LLM generates the function
- Run the tests (catch syntax errors, runtime errors, assertion failures)
- Feed errors back to the LLM
- Max 5 iterations
- Track: iterations used, errors encountered, final code

**Test case:**
```python
description = "Function that returns the nth Fibonacci number (0-indexed)"
tests = [
    ("fib(0)", 0),
    ("fib(1)", 1),
    ("fib(2)", 1),
    ("fib(5)", 5),
    ("fib(10)", 55),
    ("fib(20)", 6765),
]
```

**Starter:**
```python
def code_generation_loop(description, tests, max_iterations=5):
    # Your code here
    pass

result = code_generation_loop(description, tests)
print(f"Iterations used: {result['iterations']}")
print(f"Final code:\n{result['code']}")
```

---

### Exercise 5: Critique Loop with Separate Evaluator

Build a two-LLM system: a generator produces content, a critic evaluates it, and the generator refines based on critique.

**Requirements:**
- Generator LLM writes a short argumentative essay on a topic
- Critic LLM evaluates using specific criteria (clarity, evidence, logic, structure)
- Generator revises based on each criterion's feedback
- Max 4 iterations
- Track all critiques and revisions

**Test with:** "Should remote work be mandatory?"

**Starter:**
```python
def critique_loop(topic, max_iterations=4):
    # Your code here
    pass

result = critique_loop("Should remote work be mandatory?")
for i, (critique, revision) in enumerate(zip(result["critiques"], result["revisions"])):
    print(f"\n--- Iteration {i+1} ---")
    print(f"Critique: {critique[:100]}...")
    print(f"Revision: {revision[:100]}...")
```

---

### Exercise 6: Retry with Parameter Variation

Build a retry loop that varies model and temperature on each retry attempt.

**Requirements:**
- Try different models in sequence: `["gpt-4o", "claude-3-opus", "gpt-4o-mini"]`
- Vary temperature: `[0.1, 0.3, 0.5, 0.7]`
- Stop on first success
- Log which combination succeeded
- Handle model-specific errors

**Starter:**
```python
def retry_with_variation(prompt, models, temperatures):
    # Your code here
    pass
```

---

### Exercise 7: Convergence Detection

Build a loop that detects when output has stopped meaningfully changing and stops early.

**Requirements:**
- Generate a product description
- Evaluate quality each iteration
- Track similarity between consecutive outputs (use simple text overlap or embedding similarity)
- Stop when similarity > 95% for 2 consecutive iterations
- Track all outputs for analysis

**Starter:**
```python
def convergence_loop(task, max_iterations=10, similarity_threshold=0.95):
    # Your code here
    pass
```

---

### Exercise 8: Multi-Stage Validation Pipeline

Build a validation loop with three stages: schema → business rules → factual consistency.

**Requirements:**
- Generate a product listing (JSON with title, description, price, category, tags)
- Stage 1: Validate JSON schema
- Stage 2: Validate business rules (price > 0, title_length < 100, etc.)
- Stage 3: Validate factual consistency with a source document
- Each stage can request regeneration with specific feedback
- Track which stage fails and why

---

### Exercise 9: Cost-Tracking Loop

Extend any loop from previous exercises to track and report costs.

**Requirements:**
- Track token usage per iteration
- Calculate cost using a per-model pricing table
- Stop if cost exceeds a budget
- Report: total cost, cost per iteration, tokens used
- Visualize cost vs. quality improvement

**Starter:**
```python
PRICING = {
    "gpt-4o": {"input": 0.01, "output": 0.03},  # per 1K tokens
    "gpt-4o-mini": {"input": 0.001, "output": 0.002},
}

class CostTrackingLoop:
    def __init__(self, budget=0.10):
        self.budget = budget
        self.total_cost = 0.0
        self.iteration_costs = []
    
    def call_llm(self, prompt, model="gpt-4o"):
        # Track tokens and cost
        pass
```

---

## 🔴 Advanced Exercises

### Exercise 10: Recursive Summarization

Build a system that recursively summarizes a very long document (100K+ tokens) by splitting into chunks, summarizing each, and combining.

**Requirements:**
- Split text into chunks of ~3000 tokens
- Recursively summarize each chunk (with depth limit)
- Combine chunk summaries and summarize again if needed
- Handle the base case (small enough to summarize directly)
- Track: depth of recursion, number of LLM calls, total tokens processed
- Preserve key facts across summarization levels

**Test with:** Generate a long synthetic document using the LLM (e.g., "Write a 50-page detailed report on...").

**Starter:**
```python
def recursive_summarize(text, max_chunk_tokens=3000, max_depth=5):
    # Your code here
    pass
```

---

### Exercise 11: Planning Loop with Replanning

Build a loop where the LLM generates a plan, executes steps, monitors progress, and replans on failure.

**Requirements:**
- Task: "Research and write a 500-word report on [topic]"
- Plan: decompose into 4-6 research steps
- Execute each step (simulate research with LLM calls)
- After each step, evaluate if the plan is still on track
- If a step fails 3 times, replan from that point
- Track: original plan, executed steps, failures, replans, final report

**Starter:**
```python
def planning_loop(task, max_plan_attempts=3, max_step_attempts=3):
    # Your code here
    pass
```

---

### Exercise 12: ReAct Agent Loop

Build a simple ReAct agent that can use tools (calculator, web search, database lookup) to answer questions.

**Requirements:**
- Implement the Thought → Action → Observation loop
- Tools: `calculator(expression)`, `search(query)`, `current_time()`
- Parse the model's action output to determine which tool to call
- Feed observation back into the context
- Stop when the model outputs a Final answer
- Max 10 steps
- Handle tool errors gracefully

**Test with:**
- "What is the current time in New York plus 5 hours?"
- "Calculate the square root of 144 multiplied by 3"

**Starter:**
```python
TOOLS = {
    "calculator": lambda expr: eval(expr),  # Use with caution in prod
    "current_time": lambda: "2024-01-15 14:30:00 UTC",
    "search": lambda q: f"Simulated result for: {q}",
}

def react_agent(question, max_steps=10):
    # Your code here
    pass
```

---

### Exercise 13: Self-Correction with Rollback

Build a loop that tracks output quality and rolls back to the best-so-far if quality decreases.

**Requirements:**
- Generate answers to a coding problem
- Score each answer (using a quality heuristic or LLM evaluator)
- If current score < best score, roll back to best answer
- Add a penalty for rolling back (to prevent oscillation)
- Track: all answers, scores, rollbacks
- Visualize quality over iterations

**Starter:**
```python
def self_correction_with_rollback(problem, max_iterations=10):
    # Your code here
    pass
```

---

### Exercise 14: Parallel Exploration Loop

Build a loop that generates multiple candidate solutions in parallel, evaluates all, then refines the top candidates.

**Requirements:**
- Generate N candidates in parallel (use async or threads)
- Evaluate all N candidates
- Select top K candidates (K < N)
- Generate refinements of top K in parallel
- Repeat for M rounds
- Track: all candidates, scores, selection rationale

**Visual:**
```
Round 1: [A][B][C][D][E]  →  Select A, C, E
Round 2: [A'][C'][E']      →  Select A', E'
Round 3: [A''][E'']        →  Select A''
Final: A''
```

---

### Exercise 15: Multi-Agent Debate

Build a debating system where multiple LLM agents discuss a topic and converge on a consensus answer.

**Requirements:**
- 3 agents with different system prompts (optimist, pessimist, analyst)
- Round 1: Each agent gives their answer with reasoning
- Round 2+: Each agent sees all others' answers and can update their position
- Track: positions per round, convergence, final consensus
- Max 5 rounds
- Detect if consensus is reached early

**Test with:** "Will AI replace software engineers by 2030?"

---

### Exercise 16: Full Production Code Generation Loop

Build a complete code generation system that handles the full pipeline: spec → code → test → lint → fix → documentation.

**Requirements:**
- Take a specification with acceptance criteria
- Generate code
- Run unit tests (provided or generated)
- Run linter (simulated or real)
- Generate documentation
- If any stage fails, feed all errors back for regeneration
- Track: quality score, iterations per stage, total cost
- Implement early stopping if quality plateaus
- Graceful degradation: return best attempt if max iterations reached

**Evaluation rubric:**
- Code correctness (tests passing): 40%
- Code quality (lint score): 25%
- Documentation quality: 20%
- Efficiency (iterations used): 15%

---

## Bonus: Open-Ended Challenges

1. **Loop Monitor:** Build a dashboard that visualizes loop execution in real-time (iterations, cost, quality, decisions).

2. **Adaptive Loop:** Build a loop that dynamically adjusts its parameters (model, temperature, max iterations) based on task difficulty.

3. **Loop Optimizer:** Given a task and budget, automatically determine the optimal loop configuration (type, parameters, stopping conditions).

4. **Feedback Quality Analyzer:** Build a system that evaluates whether feedback is actually useful (specific, actionable, correct) before feeding it back into the loop.

5. **Cross-Loop Memory:** Build a system where insights from one loop execution are stored and reused in future loop executions (learning from experience).
