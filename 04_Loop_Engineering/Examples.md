# Code Examples: Loop Engineering

> All examples are production-ready patterns. Adapt them to your specific LLM provider and error handling needs.

---

## 1. Retry with Exponential Backoff

```python
import time
import random
import logging
from functools import wraps

logger = logging.getLogger(__name__)

class RetryableError(Exception):
    """Base class for retryable errors."""
    pass

class RateLimitError(RetryableError):
    pass

class TimeoutError(RetryableError):
    pass

class APIError(RetryableError):
    pass

def retry_with_backoff(
    max_retries=5,
    base_delay=1.0,
    max_delay=60.0,
    jitter_factor=0.1,
    backoff_factor=2.0,
):
    """Decorator for retrying LLM calls with exponential backoff."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except RetryableError as e:
                    last_exception = e
                    
                    if attempt == max_retries - 1:
                        logger.error(f"All {max_retries} retries exhausted")
                        raise
                    
                    delay = min(base_delay * (backoff_factor ** attempt), max_delay)
                    jitter = random.uniform(0, delay * jitter_factor)
                    sleep_time = delay + jitter
                    
                    logger.warning(
                        f"Attempt {attempt + 1}/{max_retries} failed: {e}. "
                        f"Retrying in {sleep_time:.2f}s"
                    )
                    time.sleep(sleep_time)
                    
            raise last_exception
        return wrapper
    return decorator


# Usage
@retry_with_backoff(max_retries=5, base_delay=1.0)
def call_llm(prompt, model="gpt-4o"):
    """Make an LLM API call with automatic retry."""
    # Your actual API call here
    # raise RateLimitError("429 Too Many Requests") if rate limited
    # raise TimeoutError("Request timed out") if timeout
    return f"Response to: {prompt}"


# Non-decorator version for more control
def retry_llm_call(llm_func, max_retries=5, base_delay=1.0, max_delay=60.0):
    """Retry an LLM call with exponential backoff and jitter."""
    for attempt in range(max_retries):
        try:
            return llm_func()
        except RetryableError as e:
            if attempt == max_retries - 1:
                raise
            
            delay = min(base_delay * (2 ** attempt), max_delay)
            jitter = random.uniform(0, 0.1 * delay)
            time.sleep(delay + jitter)
    
    return None  # Should not reach here
```

---

## 2. Reflection Loop (Generate → Review → Improve)

```python
def reflection_loop(task, max_iterations=3):
    """
    A loop where the LLM generates output, reviews its own output,
    and improves based on self-critique.
    """
    state = {
        "task": task,
        "current_output": None,
        "previous_output": None,
        "iterations": 0,
        "history": []
    }
    
    while state["iterations"] < max_iterations:
        # Step 1: Generate
        if state["current_output"] is None:
            prompt = f"Task: {task}\n\nProvide a thorough response:"
        else:
            prompt = (
                f"Task: {task}\n\n"
                f"Previous version:\n{state['current_output']}\n\n"
                f"Self-review feedback:\n{state['history'][-1]['feedback']}\n\n"
                f"Please provide an improved version addressing all feedback:"
            )
        
        state["previous_output"] = state["current_output"]
        state["current_output"] = call_llm(prompt)
        
        # Step 2: Self-review
        review_prompt = (
            f"Task: {task}\n\n"
            f"Response to review:\n{state['current_output']}\n\n"
            f"Review the response for errors and improvement opportunities.\n"
            f"Format your review as:\n"
            f"ERRORS:\n"
            f"- [Severity: CRITICAL/MODERATE/MINOR] Description\n"
            f"SUGGESTIONS:\n"
            f"- Description of improvement\n"
            f"OVERALL ASSESSMENT: Pass / Needs Improvement"
        )
        
        review = call_llm(review_prompt)
        
        # Step 3: Decision
        is_pass = "PASS" in review and "NEEDS IMPROVEMENT" not in review.upper()
        
        state["history"].append({
            "iteration": state["iterations"],
            "output": state["current_output"],
            "feedback": review,
            "passed": is_pass
        })
        
        if is_pass:
            break
        
        state["iterations"] += 1
    
    return {
        "final_output": state["current_output"],
        "iterations_used": state["iterations"] + 1,
        "history": state["history"]
    }


# Usage
result = reflection_loop("Explain quantum computing to a 10-year-old")
print(f"Final output ({result['iterations_used']} iterations):")
print(result["final_output"])
```

---

## 3. Critique Loop with Separate Evaluator

```python
def critique_loop(task, generator_model="gpt-4o", critic_model="gpt-4o-mini", max_iterations=5):
    """
    A loop where a generator LLM produces output and a separate critic LLM evaluates it.
    The generator refines based on the critic's feedback.
    """
    state = {
        "task": task,
        "output": None,
        "critiques": [],
        "iterations": 0,
        "history": []
    }
    
    # Initial generation
    gen_prompt = f"Task: {task}\n\nProvide a detailed, high-quality response:"
    state["output"] = call_llm(gen_prompt, model=generator_model)
    
    for i in range(max_iterations):
        # Critic evaluates
        critic_prompt = (
            f"Task: {task}\n\n"
            f"Response to evaluate:\n{state['output']}\n\n"
            f"Evaluate this response on:\n"
            f"1. Accuracy (0-10): Are all claims correct?\n"
            f"2. Completeness (0-10): Does it cover all aspects?\n"
            f"3. Clarity (0-10): Is it well-structured and easy to follow?\n"
            f"4. Specific issues: List specific errors or omissions\n"
            f"5. Improvement suggestions: Specific, actionable feedback\n\n"
            f"Final verdict: APPROVE or REVISE"
        )
        
        critique = call_llm(critic_prompt, model=critic_model)
        state["critiques"].append(critique)
        
        # Parse verdict
        is_approved = "APPROVE" in critique.upper() and "REVISE" not in critique.upper()
        
        state["history"].append({
            "iteration": i,
            "output": state["output"],
            "critique": critique,
            "approved": is_approved
        })
        
        if is_approved:
            break
        
        # Generator refines
        gen_prompt = (
            f"Task: {task}\n\n"
            f"Your previous response:\n{state['output']}\n\n"
            f"Critic's feedback:\n{critique}\n\n"
            f"Please provide a revised version that addresses all feedback. "
            f"Focus on correcting errors and improving quality:"
        )
        
        state["output"] = call_llm(gen_prompt, model=generator_model)
        state["iterations"] += 1
    
    return {
        "final_output": state["output"],
        "iterations_used": state["iterations"] + 1,
        "history": state["history"]
    }


# Usage
result = critique_loop(
    "Explain the pros and cons of renewable energy sources",
    generator_model="gpt-4o",
    critic_model="gpt-4o-mini"
)
```

---

## 4. Code Generation with Test Feedback

```python
import sys
import io
import traceback
import ast

def code_generation_loop(specification, test_cases, max_iterations=5):
    """
    Generate code from a specification, run tests, and refine until all pass.
    
    Args:
        specification: Description of the function to implement
        test_cases: List of (call_expression, expected_result) tuples
        max_iterations: Maximum refinement cycles
    
    Returns:
        dict with keys: code, iterations, errors, tests_passed
    """
    state = {
        "spec": specification,
        "code": None,
        "tests": test_cases,
        "errors": [],
        "iterations": 0,
        "test_results": []
    }
    
    for i in range(max_iterations):
        # Generate code
        if state["code"] is None:
            prompt = (
                f"Write a Python function for this specification:\n{specification}\n\n"
                f"Return ONLY the function code, no explanations.\n"
                f"Make sure the function handles edge cases."
            )
        else:
            errors_text = "\n".join(state["errors"])
            prompt = (
                f"Fix this Python function:\n\n{state['code']}\n\n"
                f"The following errors occurred:\n{errors_text}\n\n"
                f"Return ONLY the corrected function code, no explanations."
            )
        
        state["code"] = call_llm(prompt)
        state["code"] = extract_code(state["code"])
        
        # Run tests
        passed = 0
        failed = 0
        errors = []
        test_results = []
        
        for expr, expected in test_cases:
            try:
                # Create a safe namespace
                namespace = {}
                exec(state["code"], namespace)
                
                # Find the function name and call it
                result = eval(expr, namespace)
                
                if result == expected:
                    passed += 1
                    test_results.append({"test": expr, "passed": True})
                else:
                    failed += 1
                    error_msg = f"Test {expr}: Expected {expected}, got {result}"
                    errors.append(error_msg)
                    test_results.append({
                        "test": expr, "passed": False,
                        "error": error_msg
                    })
                    
            except SyntaxError as e:
                failed += 1
                error_msg = f"SyntaxError: {e}"
                errors.append(error_msg)
                test_results.append({"test": expr, "passed": False, "error": error_msg})
                
            except Exception as e:
                failed += 1
                error_msg = f"{type(e).__name__}: {e}"
                errors.append(error_msg)
                test_results.append({"test": expr, "passed": False, "error": error_msg})
        
        state["test_results"] = test_results
        state["errors"] = errors
        
        if failed == 0:
            break
        
        state["iterations"] += 1
    
    return {
        "code": state["code"],
        "iterations_used": state["iterations"] + 1,
        "tests_passed": passed,
        "tests_total": len(test_cases),
        "test_results": state["test_results"],
        "all_passed": failed == 0
    }


def extract_code(text):
    """Extract Python code from LLM response, handling markdown fences."""
    if "```python" in text:
        start = text.index("```python") + 10
        end = text.index("```", start)
        return text[start:end].strip()
    elif "```" in text:
        start = text.index("```") + 3
        end = text.index("```", start)
        return text[start:end].strip()
    return text.strip()


# Usage
spec = "Function that returns True if a number is prime, False otherwise"
tests = [
    ("is_prime(2)", True),
    ("is_prime(3)", True),
    ("is_prime(4)", False),
    ("is_prime(17)", True),
    ("is_prime(1)", False),
    ("is_prime(-5)", False),
]

result = code_generation_loop(spec, tests)
print(f"Iterations: {result['iterations_used']}")
print(f"Tests passed: {result['tests_passed']}/{result['tests_total']}")
print(f"\nFinal code:\n{result['code']}")
```

---

## 5. Planning and Execution Loop

```python
import json
import time

def planning_loop(task, max_plan_attempts=3, max_step_attempts=3):
    """
    A planning loop that generates a plan, executes steps,
    monitors progress, and replans on failure.
    """
    state = {
        "task": task,
        "plan": None,
        "completed_steps": [],
        "failed_steps": [],
        "step_results": [],
        "plan_attempts": 0,
        "output": None
    }
    
    def generate_plan(task, previous_plan=None, failed_step=None, results=None):
        """Ask LLM to generate or revise a plan."""
        if previous_plan is None:
            prompt = (
                f"Task: {task}\n\n"
                f"Create a step-by-step plan to complete this task. "
                f"Each step should be specific and actionable.\n"
                f"Format as JSON array:\n"
                f'[{{"step": "description", "criteria": "success criteria"}}]'
            )
        else:
            results_text = json.dumps(results or [], indent=2)
            prompt = (
                f"Task: {task}\n\n"
                f"Previous plan failed at step: {failed_step}\n"
                f"Completed steps and results:\n{results_text}\n\n"
                f"Revise the remaining plan to achieve the task goal. "
                f"Format as JSON array:\n"
                f'[{{"step": "description", "criteria": "success criteria"}}]'
            )
        
        response = call_llm(prompt)
        return json.loads(extract_json(response))
    
    def execute_step(step):
        """Execute a single plan step."""
        prompt = (
            f"Task: {state['task']}\n\n"
            f"Current step: {step['step']}\n"
            f"Success criteria: {step['criteria']}\n\n"
            f"Execute this step and provide the result:"
        )
        return call_llm(prompt)
    
    def verify_step(result, criteria):
        """Check if step result meets criteria."""
        prompt = (
            f"Step criteria: {criteria}\n\n"
            f"Step result:\n{result}\n\n"
            f"Does this result meet the criteria? Answer YES or NO, and explain why."
        )
        response = call_llm(prompt)
        return response.upper().startswith("YES")
    
    # Generate initial plan
    state["plan"] = generate_plan(task)
    state["plan_attempts"] = 1
    
    while state["plan_attempts"] <= max_plan_attempts:
        plan = state["plan"]
        results = list(state["step_results"])
        
        for step_idx, step in enumerate(plan):
            if step_idx < len(state["completed_steps"]):
                continue  # Already completed
            
            step_successful = False
            
            for attempt in range(max_step_attempts):
                result = execute_step(step)
                
                if verify_step(result, step["criteria"]):
                    state["completed_steps"].append(step_idx)
                    state["step_results"].append({
                        "step": step["step"],
                        "result": result,
                        "attempts": attempt + 1
                    })
                    step_successful = True
                    break
                else:
                    time.sleep(0.5)  # Brief pause before retry
            
            if not step_successful:
                state["failed_steps"].append({
                    "step": step["step"],
                    "attempts": max_step_attempts
                })
                
                # Replan
                state["plan"] = generate_plan(
                    task, state["plan"],
                    step["step"],
                    state["step_results"]
                )
                state["plan_attempts"] += 1
                break  # Restart with new plan
        
        else:
            # All steps completed successfully
            break
    
    # Synthesize final output
    if state["completed_steps"]:
        results_text = "\n".join([
            f"Step {r['step']}: {r['result']}"
            for r in state["step_results"]
        ])
        
        state["output"] = call_llm(
            f"Task: {task}\n\n"
            f"Completed work:\n{results_text}\n\n"
            f"Synthesize a final report summarizing the completed work:"
        )
    
    return {
        "final_output": state["output"],
        "plan_attempts": state["plan_attempts"],
        "steps_completed": len(state["completed_steps"]),
        "step_results": state["step_results"],
        "failed_steps": state["failed_steps"]
    }


def extract_json(text):
    """Extract JSON from LLM response."""
    if "```json" in text:
        start = text.index("```json") + 7
        end = text.index("```", start)
        return text[start:end].strip()
    # Try to find JSON array in text
    start = text.index("[")
    end = text.rindex("]") + 1
    return text[start:end]


# Usage
result = planning_loop(
    "Research and summarize the key events of the French Revolution",
    max_plan_attempts=2,
    max_step_attempts=2
)
```

---

## 6. ReAct Agent Loop

```python
import json
import re

class ReActAgent:
    """
    A ReAct (Reasoning + Acting) agent that thinks, acts, and observes
    in a loop until it produces a final answer.
    """
    
    def __init__(self, tools=None, max_steps=10):
        self.tools = tools or {}
        self.max_steps = max_steps
        self.thoughts = []
        self.actions = []
        self.observations = []
    
    def add_tool(self, name, func, description=""):
        self.tools[name] = {"func": func, "description": description}
    
    def run(self, question):
        """Run the ReAct loop on a question."""
        context = f"Question: {question}\n\n"
        
        for step in range(self.max_steps):
            # Thought step
            thought_prompt = (
                f"{context}\n"
                f"Available tools: {list(self.tools.keys())}\n\n"
                f"Think about what to do next. Format:\n"
                f"Thought: your reasoning about what to do\n"
                f"Action: tool_name(tool_input)\n"
                f"or\n"
                f"Final: your final answer"
            )
            
            response = call_llm(thought_prompt)
            self.thoughts.append(response)
            
            # Parse response
            if "Final:" in response:
                final_answer = response.split("Final:")[-1].strip()
                return {
                    "answer": final_answer,
                    "steps": step + 1,
                    "thoughts": self.thoughts,
                    "actions": self.actions,
                    "observations": self.observations
                }
            
            # Extract action
            action_match = re.search(r"Action:\s*(\w+)\(([^)]*)\)", response)
            if not action_match:
                context += f"Thought: {response}\n"
                context += "Observation: Invalid format. Use Action: tool(input)\n"
                continue
            
            tool_name = action_match.group(1)
            tool_input = action_match.group(2).strip().strip('"')
            
            self.actions.append({"tool": tool_name, "input": tool_input})
            
            # Execute action
            if tool_name in self.tools:
                try:
                    observation = self.tools[tool_name]["func"](tool_input)
                except Exception as e:
                    observation = f"Error executing {tool_name}: {e}"
            else:
                observation = f"Unknown tool: {tool_name}"
            
            self.observations.append(observation)
            
            # Update context
            context += (
                f"Thought: {response}\n"
                f"Action: {tool_name}(\"{tool_input}\")\n"
                f"Observation: {observation}\n"
            )
        
        # Max steps reached, force final answer
        final_prompt = (
            f"{context}\n"
            f"Maximum steps reached. Provide your final answer based on what you've learned:"
        )
        final_answer = call_llm(final_prompt)
        
        return {
            "answer": final_answer,
            "steps": self.max_steps,
            "thoughts": self.thoughts,
            "actions": self.actions,
            "observations": self.observations,
            "truncated": True
        }


# Tools
def calculator(expression):
    """Evaluate a mathematical expression."""
    # Sandboxed evaluation - use a proper expression parser in production
    allowed = set("0123456789+-*/(). ")
    if not all(c in allowed for c in expression):
        return "Error: Invalid characters in expression"
    try:
        return str(eval(expression))
    except Exception as e:
        return f"Error: {e}"

def search(query):
    """Simulate a web search."""
    # In production, call a real search API
    return f"Simulated search results for '{query}': [result 1, result 2, result 3]"

def current_time():
    """Get the current time."""
    import datetime
    return datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")


# Usage
agent = ReActAgent(max_steps=10)
agent.add_tool("calculator", calculator, "Evaluate math expressions")
agent.add_tool("search", search, "Search for information")
agent.add_tool("current_time", current_time, "Get current date and time")

result = agent.run("What is the current time in New York? Calculate what time it will be in 5 hours.")
print(f"Answer: {result['answer']}")
print(f"Steps used: {result['steps']}")
```

---

## 7. Recursive Summarization

```python
import tiktoken

def count_tokens(text, model="gpt-4"):
    """Count tokens in text for a given model."""
    try:
        enc = tiktoken.encoding_for_model(model)
        return len(enc.encode(text))
    except Exception:
        return len(text) // 4  # Rough estimate

def split_into_chunks(text, max_tokens, overlap_tokens=100):
    """Split text into chunks of approximately max_tokens."""
    chunks = []
    sentences = text.replace("\n", " ").split(". ")
    current_chunk = []
    current_tokens = 0
    
    for sentence in sentences:
        sentence_tokens = count_tokens(sentence)
        
        if current_tokens + sentence_tokens > max_tokens and current_chunk:
            chunks.append(". ".join(current_chunk) + ".")
            # Keep last few sentences for overlap
            overlap = []
            overlap_tokens_count = 0
            for s in reversed(current_chunk):
                t = count_tokens(s)
                if overlap_tokens_count + t > overlap_tokens:
                    break
                overlap.insert(0, s)
                overlap_tokens_count += t
            
            current_chunk = overlap
            current_tokens = overlap_tokens_count
        
        current_chunk.append(sentence)
        current_tokens += sentence_tokens
    
    if current_chunk:
        chunks.append(". ".join(current_chunk) + ".")
    
    return chunks


def recursive_summarize(text, max_chunk_tokens=3000, max_depth=5, depth=0):
    """
    Recursively summarize a document.
    
    Base case: Text fits in one chunk → summarize directly.
    Recursive case: Split into chunks, summarize each, combine.
    """
    stats = {"calls": 0, "depth": depth}
    
    if depth >= max_depth:
        return {"summary": text[:1000] + "... (truncated at max depth)", "stats": stats}
    
    if count_tokens(text) <= max_chunk_tokens:
        # Base case
        prompt = (
            f"Summarize the following text, preserving key facts and details:\n\n{text}\n\n"
            f"Summary:"
        )
        stats["calls"] += 1
        return {"summary": call_llm(prompt), "stats": stats}
    
    # Recursive case
    chunks = split_into_chunks(text, max_chunk_tokens)
    
    chunk_summaries = []
    for i, chunk in enumerate(chunks):
        result = recursive_summarize(chunk, max_chunk_tokens, max_depth, depth + 1)
        chunk_summaries.append(result["summary"])
        stats["calls"] += result["stats"]["calls"]
        stats["depth"] = max(stats["depth"], result["stats"]["depth"])
    
    # Combine chunk summaries
    combined = "\n\n".join(chunk_summaries)
    
    if count_tokens(combined) > max_chunk_tokens:
        # Summarize the combined summaries
        result = recursive_summarize(combined, max_chunk_tokens, max_depth, depth + 1)
        stats["calls"] += result["stats"]["calls"]
        return {"summary": result["summary"], "stats": stats}
    else:
        prompt = (
            f"Synthesize these section summaries into a coherent overall summary:\n\n{combined}\n\n"
            f"Overall summary:"
        )
        stats["calls"] += 1
        return {"summary": call_llm(prompt), "stats": stats}


# Usage
long_text = "Your very long document here... " * 10000  # ~500K chars
result = recursive_summarize(long_text, max_chunk_tokens=3000, max_depth=5)
print(f"Summary length: {len(result['summary'])} chars")
print(f"LLM calls: {result['stats']['calls']}")
print(f"Max depth: {result['stats']['depth']}")
print(f"\nSummary:\n{result['summary']}")
```

---

## 8. Parallel Execution in Loops

```python
import asyncio
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

# --- Async Version ---

async def call_llm_async(prompt, model="gpt-4o"):
    """Async LLM call placeholder."""
    await asyncio.sleep(0.5)  # Simulate API call
    return f"Response to: {prompt[:50]}..."

async def parallel_refinement_loop(task, num_candidates=3, num_rounds=2):
    """
    Generate multiple candidates in parallel, evaluate all, refine top K.
    """
    state = {
        "task": task,
        "candidates": [],
        "rounds": []
    }
    
    for round_num in range(num_rounds):
        # Generate candidates in parallel
        if round_num == 0:
            prompts = [
                f"Task: {task}\n\nGenerate a solution (version {i+1}):"
                for i in range(num_candidates)
            ]
        else:
            prompts = []
            for i, candidate in enumerate(state["candidates"]):
                feedback = state["rounds"][-1]["feedback"][i]
                prompts.append(
                    f"Task: {task}\n\n"
                    f"Previous version {i+1}:\n{candidate}\n\n"
                    f"Feedback:\n{feedback}\n\n"
                    f"Provide an improved version:"
                )
        
        tasks = [call_llm_async(p) for p in prompts]
        candidates = await asyncio.gather(*tasks)
        
        # Evaluate all in parallel
        eval_prompts = [
            f"Rate this response on quality (1-10):\n{c}\n\nRating:" 
            for c in candidates
        ]
        eval_tasks = [call_llm_async(ep) for ep in eval_prompts]
        evaluations = await asyncio.gather(*eval_tasks)
        
        state["candidates"] = candidates
        state["rounds"].append({
            "candidates": candidates,
            "feedback": evaluations
        })
    
    return state


# Usage
# result = asyncio.run(parallel_refinement_loop("Write a product description"))


# --- ThreadPool Version (for synchronous APIs) ---

def parallel_call_llm(prompt, model="gpt-4o"):
    """Synchronous LLM call."""
    time.sleep(0.5)  # Simulate API call
    return f"Response to: {prompt[:50]}..."

def parallel_map(func, items, max_workers=5):
    """Apply func to all items in parallel using thread pool."""
    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        futures = {executor.submit(func, item): item for item in items}
        results = []
        for future in as_completed(futures):
            results.append(future.result())
        return results


# --- Batched Evaluation ---

def batch_evaluate(outputs, criteria):
    """
    Evaluate multiple outputs in a single LLM call for efficiency.
    """
    outputs_text = "\n---\n".join([f"Output {i+1}:\n{o}" for i, o in enumerate(outputs)])
    
    prompt = (
        f"Evaluate each output on these criteria: {criteria}\n\n"
        f"{outputs_text}\n\n"
        f"For each output, provide:\n"
        f"- Score (0-10)\n"
        f"- Key strengths\n"
        f"- Key weaknesses\n"
        f"- Pass/Fail"
    )
    
    response = call_llm(prompt)
    
    # Parse results for each output
    results = []
    sections = response.split("Output ")[1:]
    for section in sections:
        # Extract score, pass/fail, etc.
        results.append({
            "raw": section.strip(),
            "score": extract_score(section),
            "passed": "PASS" in section.upper()
        })
    
    return results


def extract_score(text):
    """Extract numeric score from text."""
    import re
    match = re.search(r"Score[:\s]*(\d+)", text)
    return int(match.group(1)) if match else 0
```

---

## 9. Safe Loop with All Safety Checks

```python
import time
import logging

logger = logging.getLogger(__name__)

class SafeLoop:
    """
    A loop wrapper with comprehensive safety mechanisms:
    - Max iterations
    - Timeout
    - Cost budget
    - Degeneration detection
    - Cycle detection
    - Human-in-the-loop interrupt
    """
    
    def __init__(
        self,
        max_iterations=10,
        timeout_seconds=300,
        max_cost=1.0,
        degeneration_threshold=0.5,
        similarity_threshold=0.95,
        model_pricing=None
    ):
        self.max_iterations = max_iterations
        self.timeout = timeout_seconds
        self.max_cost = max_cost
        self.degeneration_threshold = degeneration_threshold
        self.similarity_threshold = similarity_threshold
        
        self.model_pricing = model_pricing or {
            "gpt-4o": {"input": 0.01, "output": 0.03},
            "gpt-4o-mini": {"input": 0.001, "output": 0.002},
        }
        
        # State
        self.start_time = None
        self.total_cost = 0.0
        self.best_output = None
        self.best_score = float("-inf")
        self.all_outputs = []
        self.all_scores = []
        self.iteration = 0
        self.interrupted = False
    
    def interrupt(self):
        """Signal the loop to stop (can be called from another thread)."""
        self.interrupted = True
        logger.info("Loop interrupted by external signal")
    
    def track_cost(self, tokens_input, tokens_output, model="gpt-4o"):
        """Track and accumulate costs."""
        pricing = self.model_pricing.get(model, {"input": 0.01, "output": 0.03})
        cost = (tokens_input * pricing["input"] + tokens_output * pricing["output"]) / 1000
        self.total_cost += cost
        return cost
    
    def check_convergence(self, output):
        """Check if output has converged (stopped changing meaningfully)."""
        if len(self.all_outputs) < 3:
            return False
        
        # Compare with previous outputs
        recent = self.all_outputs[-3:]
        similarities = [
            self.string_similarity(recent[i], recent[i+1])
            for i in range(len(recent) - 1)
        ]
        
        return all(s >= self.similarity_threshold for s in similarities)
    
    def check_cycle(self, output):
        """Check if we're in a cycle (same output appearing before)."""
        if len(self.all_outputs) < 4:
            return False
        
        for prev_output in self.all_outputs[:-1]:
            if self.string_similarity(output, prev_output) > 0.98:
                return True
        
        return False
    
    def string_similarity(self, a, b):
        """Simple text overlap similarity."""
        if not a or not b:
            return 0.0
        set_a = set(a.lower().split())
        set_b = set(b.lower().split())
        intersection = set_a & set_b
        union = set_a | set_b
        return len(intersection) / len(union) if union else 0.0
    
    def should_continue(self, output, score):
        """Check all stopping conditions."""
        reasons = []
        
        # 1. External interrupt
        if self.interrupted:
            return False, "interrupted"
        
        # 2. Max iterations
        if self.iteration >= self.max_iterations:
            return False, "max_iterations"
        
        # 3. Timeout
        if time.time() - self.start_time > self.timeout:
            return False, "timeout"
        
        # 4. Cost budget
        if self.total_cost >= self.max_cost:
            return False, "cost_exceeded"
        
        # 5. Degeneration
        if score < self.best_score * self.degeneration_threshold and self.best_score > 0:
            # Quality dropped significantly
            return False, "degeneration"
        
        # 6. Convergence
        if self.check_convergence(output):
            return False, "convergence"
        
        # 7. Cycle detection
        if self.check_cycle(output):
            return False, "cycle_detected"
        
        # Update best
        if score > self.best_score:
            self.best_output = output
            self.best_score = score
        
        return True, "continue"
    
    def run(self, generator, evaluator, task):
        """
        Run the safe loop.
        
        Args:
            generator: Function that takes task + previous output + feedback and returns output
            evaluator: Function that takes output and task and returns a score
            task: The task to work on
        """
        self.start_time = time.time()
        output = None
        feedback = None
        
        while True:
            # Generate
            output = generator(task, output, feedback)
            self.all_outputs.append(output)
            
            # Evaluate
            score = evaluator(output, task)
            self.all_scores.append(score)
            
            # Check stopping conditions
            should_continue, reason = self.should_continue(output, score)
            
            logger.info(
                f"Iteration {self.iteration + 1}: score={score:.2f}, "
                f"cost={self.total_cost:.4f}, reason={reason}"
            )
            
            if not should_continue:
                # Rollback if degenerated
                if reason == "degeneration" and self.best_output:
                    return {
                        "output": self.best_output,
                        "score": self.best_score,
                        "iterations": self.iteration,
                        "total_cost": self.total_cost,
                        "stop_reason": reason,
                        "all_outputs": self.all_outputs,
                        "all_scores": self.all_scores,
                        "rolled_back": True
                    }
                
                return {
                    "output": output,
                    "score": score,
                    "iterations": self.iteration + 1,
                    "total_cost": self.total_cost,
                    "stop_reason": reason,
                    "all_outputs": self.all_outputs,
                    "all_scores": self.all_scores,
                    "rolled_back": False
                }
            
            self.iteration += 1


# Usage
def generator(task, previous_output, feedback):
    prompt = f"Task: {task}"
    if previous_output:
        prompt += f"\n\nPrevious output: {previous_output}"
    if feedback:
        prompt += f"\n\nFeedback to address: {feedback}"
    prompt += "\n\nGenerate the best possible output:"
    return call_llm(prompt)

def evaluator(output, task):
    prompt = f"Task: {task}\n\nOutput:\n{output}\n\nRate this output 0-10:"
    response = call_llm(prompt)
    import re
    match = re.search(r"(\d+)", response)
    return float(match.group(1)) / 10.0 if match else 0.5

safe_loop = SafeLoop(max_iterations=10, timeout_seconds=60, max_cost=0.50)
result = safe_loop.run(generator, evaluator, "Write a compelling product description")
print(f"Stop reason: {result['stop_reason']}")
print(f"Final score: {result['score']:.2f}")
```
