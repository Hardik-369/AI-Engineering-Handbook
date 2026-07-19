# Interview Questions: Loop Engineering

---

## Fundamentals

### Q1: What is loop engineering in the context of LLMs?

Loop engineering is the discipline of designing iterative feedback architectures where an LLM's output is evaluated, refined, or used as input for subsequent LLM calls. It transforms LLMs from single-shot predictors into self-correcting, iterative reasoning systems. The core insight is that just as loops are essential for non-trivial computation in programming, loops are essential for non-trivial LLM applications in production.

### Q2: What are the six components of every loop?

1. **Input/State** — Current context and accumulated information
2. **LLM Call** — The actual API call with the current prompt
3. **Output** — The LLM's generated response
4. **Evaluation/Feedback** — Assessment of output quality
5. **Decision** — Continue or stop based on evaluation
6. **State Update** — Update state with feedback and new context if continuing

### Q3: Compare a single LLM call with a loop architecture. When would you use each?

| Dimension | Single Call | Loop |
|---|---|---|
| Quality | First attempt only | Iteratively improving |
| Reliability | Brittle | Robust (retry, refine) |
| Cost | Fixed, predictable | Variable, potentially higher |
| Latency | Fast, predictable | Slower, variable |
| Complexity | Simple | Moderate |

Use single calls for: simple Q&A, translation, summarization of short texts, tasks where first-attempt quality is sufficient.

Use loops for: code generation, complex reasoning, multi-step tasks, tasks requiring external feedback, any production system where reliability matters.

### Q4: What's the difference between a feedback loop and a reflection loop?

A **feedback loop** uses external signals from the environment (compiler errors, test results, API responses, user feedback) to drive improvement. The evaluation comes from outside the LLM.

A **reflection loop** has the same LLM evaluate its own output through self-critique. The evaluation is internal — the model reviews its own work.

The key difference is the source of evaluation: external vs. internal. Feedback loops are generally more reliable because external signals are objective, while reflection loops suffer from confirmation bias.

### Q5: Compare reflection loops and critique loops. When would you use each?

**Reflection loop:** One model generates and evaluates itself. Cheaper, simpler, but suffers from confirmation bias.

**Critique loop:** Separate models for generation and evaluation. More objective, can use cheaper model for evaluation, but more complex and expensive.

Use **reflection** when: cost is a concern, the task benefits from structured self-review, the model has high capability.

Use **critique** when: quality is critical, you need independent evaluation, you want to use different models optimized for different roles.

### Q6: What are the stopping conditions for a loop?

- **Max iterations** — Hard limit on iterations
- **Output convergence** — Output stops changing significantly
- **Constraint satisfaction** — All validation checks pass
- **Timeout** — Wall-clock time exceeded
- **Cost budget** — Total API cost exceeded
- **Degeneration** — Output quality decreased from previous iteration
- **Human interrupt** — External stop signal
- **Cycle detection** — Same output appearing multiple times

---

## Design & Architecture

### Q7: How would you design a code generation loop that handles compile errors, test failures, and lint warnings?

The architecture has three stages in increasing order of strictness:

1. **Compile stage:** Generate code → attempt to compile/parse → capture syntax errors → feed errors back → regenerate. Loop until syntax is valid.

2. **Test stage:** Run unit tests → capture failures → feed test output back → regenerate. Loop until all tests pass.

3. **Lint stage:** Run linter → capture warnings → feed warnings back → regenerate. Loop until acceptable lint score.

Each stage feeds into the next. The loop progresses through stages: once compile errors are fixed, move to running tests; once tests pass, move to linting. If new compile errors are introduced during test fixing, fall back to the compile stage.

Key design decisions:
- Max iterations per stage (e.g., 3 for compile, 5 for tests, 2 for lint)
- Whether to start fresh or build on previous code when regenerating
- How to handle non-deterministic test failures (flaky tests)

### Q8: What is the ReAct loop pattern? Describe its components.

ReAct (Reasoning + Acting, Yao et al., 2022) combines reasoning traces with action execution in a loop:

1. **Thought:** The model reasons about the current state and what to do next
2. **Action:** The model calls a tool or API (search, calculator, database, etc.)
3. **Observation:** The tool's result is fed back into the context
4. **Repeat:** The model uses the observation in its next reasoning step

The loop terminates when the model produces a "Final" answer instead of an action.

Key: The reasoning trace is preserved in context, allowing the model to maintain a chain of thought across multiple tool interactions.

### Q9: How would you implement recursive summarization for documents exceeding the context window?

Recursive summarization uses a divide-and-conquer approach:

1. **Base case:** If the text fits within the context window, summarize it directly.
2. **Split:** Otherwise, split into chunks of ~3000 tokens with overlap.
3. **Recurse:** Recursively summarize each chunk.
4. **Merge:** Combine chunk summaries and summarize the combination.

Key considerations:
- **Overlap between chunks** (100-200 tokens) prevents information loss at boundaries
- **Depth limit** (5-7 levels) prevents infinite recursion and cost explosion
- **Fact preservation:** Use prompts that emphasize preserving key facts, not just abstracting
- **Progressive summarization:** Each level should preserve more detail than conventional summarization

Optimization: Use cheaper models for intermediate summarization levels, reserving expensive models for the final synthesis.

### Q10: Describe the RAG loop pattern. How does it differ from simple RAG?

Simple RAG: Retrieve once, generate once.
RAG loop: Retrieve → Generate → Evaluate → Retrieve more if needed.

In a RAG loop:
1. Initial retrieval based on the query
2. Generate an answer from retrieved documents
3. Evaluate the answer for:
   - **Completeness:** Does the answer fully address the query?
   - **Confidence:** Is the model confident in the answer?
   - **Citation quality:** Are claims properly supported?
4. If evaluation reveals gaps, reformulate the query and retrieve additional documents
5. Generate an improved answer with the expanded context
6. Repeat until evaluation passes or max iterations reached

The RAG loop is more robust than simple RAG because it handles cases where the initial retrieval misses relevant documents or the generated answer has gaps.

### Q11: How do you handle the trade-off between exploration and exploitation in loops?

**Exploration:** Trying different approaches, generating diverse candidates.
**Exploitation:** Refining the best candidate found so far.

Strategies:
- **Parallel exploration + serial exploitation:** Generate N candidates in parallel, evaluate all, then refine the top K.
- **Temperature annealing:** Start with high temperature (exploration), decrease over iterations (exploitation).
- **Beam search:** Maintain top-K candidates, generate refinements for each, re-rank.
- **Epsilon-greedy:** Most of the time refine the best candidate, occasionally try something random.

The right balance depends on the task. For creative tasks (writing, design), favor exploration. For well-defined tasks (code, math), favor exploitation.

### Q12: How would you implement a planning loop with replanning capability?

1. **Plan generation:** LLM creates a step-by-step plan with success criteria for each step.
2. **Step execution:** Execute each step using LLM calls or tool calls.
3. **Verification:** After each step, verify the result against success criteria.
4. **Replanning:** If a step fails after max retries, call the planner to revise remaining steps based on what was learned.
5. **Completion:** All steps completed successfully, or max plan attempts reached.

The planner needs access to: original task, completed steps and their results, the failed step, and the context of what went wrong. The replan should avoid repeating the same failure pattern.

---

## Safety & Quality

### Q13: What are the main safety concerns when building LLM loops?

1. **Infinite loops:** Loop never terminates. Prevention: always set max_iterations, timeout, and cost budget.

2. **Cost explosions:** Loop consumes excessive API costs. Prevention: cost tracking, budget limits, early stopping, cheap evaluators.

3. **Degenerate outputs:** Quality decreases each iteration. Prevention: track quality metrics, compare against best-so-far, roll back on degradation.

4. **Confirmation loops:** Flawed evaluation reinforces incorrect outputs. Prevention: use independent evaluators, validate evaluation quality, human-in-the-loop for critical decisions.

5. **Context window overflow:** Accumulated history exceeds context limits. Prevention: context management, summarization of history, sliding window.

6. **Oscillation:** Output oscillates between two states. Prevention: cycle detection, similarity checking, convergence detection.

### Q14: How do you detect and handle degenerate outputs in a loop?

Degeneration means output quality decreases with iteration instead of improving.

**Detection:**
- Track quality score per iteration (using LLM evaluation or heuristics)
- Compare current score against a running best score
- Flag if score drops below `best_score * threshold` (e.g., 0.5 = 50% drop)

**Handling:**
- Roll back to the best-so-far output
- Log the degeneration for analysis
- Optionally, try a fresh start instead of continuing to refine the degenerating path

```python
if current_score < best_score * 0.5:
    return best_output  # Rollback
```

### Q15: When does self-correction fail? Provide examples.

Self-correction fails in several documented scenarios:

1. **Lack of knowledge:** The model doesn't know the correct answer. Reflection can't create knowledge.

2. **Confirmation bias:** The model tends to confirm its original output rather than genuinely critique it. It may say "no errors found" when errors exist.

3. **Degenerate self-correction:** The model "corrects" correct answers to incorrect ones. Example: A correct code solution gets "fixed" to introduce bugs.

4. **Over-correction:** The model becomes excessively conservative, hedging statements, or removing confident claims.

5. **Subtle errors:** The model misses subtle reasoning errors that require deep analysis.

6. **Overconfidence:** The model rates its original answer highly and refuses to meaningfully critique it.

Research (Huang et al., 2023) shows that self-correction helps most on factual errors and obvious mistakes, but can hurt performance on reasoning tasks where the model is already near its capability ceiling.

---

## Cost & Latency

### Q16: How would you optimize the cost of a loop system?

1. **Early stopping:** Stop when quality plateaus (diminishing returns set in after 2-3 iterations).

2. **Increasingly strict evaluation:** Cheap checks first, expensive LLM evaluation only for near-passing outputs.

3. **Cheaper models for evaluation:** Use GPT-4o-mini to evaluate GPT-4o output (90% cost reduction).

4. **Caching:** Cache exact or near-exact LLM calls (same prompt → same response).

5. **Batch processing:** Batch multiple evaluation calls into one LLM call.

6. **Adaptive iterations:** Simple tasks get fewer iterations, complex tasks get more.

7. **Cost budgets:** Hard stop when cumulative cost exceeds threshold.

### Q17: How would you optimize the latency of a loop system?

1. **Parallel LLM calls:** Run independent evaluations or generations in parallel using async or threads.

2. **Streaming:** Stream LLM output and start evaluation before generation completes.

3. **Async execution:** Overlap computation and I/O using async/await patterns.

4. **Batched evaluations:** Evaluate multiple outputs in a single LLM call.

5. **Model selection:** Use faster models for evaluation and iteration, slower models only for final generation.

6. **Early termination:** Stop evaluation as soon as a fatal error is found (fail-fast).

7. **Pre-generation:** Start generating refinements before evaluation completes (speculative execution).

### Q18: How do you calculate the total cost of a loop system?

```python
total_cost = 0
for iteration in range(iterations):
    # Generation call
    gen_cost = gen_input_tokens * gen_input_price + gen_output_tokens * gen_output_price
    
    # Evaluation call(s)
    eval_cost = eval_input_tokens * eval_input_price + eval_output_tokens * eval_output_price
    
    # Any additional calls (critique, verification, etc.)
    extra_cost = ...
    
    total_cost += gen_cost + eval_cost + extra_cost
```

Typical cost breakdown for a 3-iteration critique loop:
- 3 generator calls (expensive model)
- 3 critic calls (cheap model)
- Total: ~3x the cost of a single call (if critic is 10x cheaper)

### Q19: What cost estimation formulas do you use when designing loops?

**Simple refinement loop (same model for all calls):**
```
total_cost = iterations * cost_per_call
```

**Critique loop (separate evaluator):**
```
total_cost = iterations * (cost_generator + cost_critic)
```

**Code generation with test feedback (tests are free but LLM eval may not be):**
```
total_cost = iterations * cost_generation
+ iterations * cost_test_evaluation
(if using LLM to analyze test output)
```

**Planning loop:**
```
total_cost = cost_planning
+ sum(execution_costs)
+ replanning_costs
```

**Rule of thumb:** Budget 3-5x the cost of a single call for robust loop systems.

---

## Production & Best Practices

### Q20: How would you monitor loops in production?

Key metrics to monitor:

- **Iteration count distribution:** Are most tasks completing in 1-2 iterations, or is there a long tail?
- **Stop reason distribution:** What percentage stop due to: success vs. max iterations vs. timeout vs. cost?
- **Cost per task:** Average, P50, P95, P99. Alert on outliers.
- **Latency per iteration:** Both generation and evaluation latency.
- **Quality over iterations:** Does quality improve, plateau, or degrade?
- **Cycle detection alerts:** Same output appearing multiple times.
- **Cost explosion alerts:** Individual task exceeding cost threshold.
- **Degeneration rate:** How often does quality decrease on iteration?

Logging format:
```json
{
  "task_id": "abc123",
  "loop_type": "refinement",
  "iterations": 4,
  "stop_reason": "convergence",
  "total_cost": 0.042,
  "latency_ms": 8500,
  "quality_scores": [0.6, 0.75, 0.82, 0.84],
  "model": "gpt-4o"
}
```

### Q21: How do you test a loop system?

Testing loops is harder than testing single calls because loops are stateful and non-deterministic. Approaches:

1. **Unit tests for components:**
   - Test the evaluation function in isolation
   - Test stopping condition logic
   - Test state management

2. **Integration tests with mock LLM:**
   - Mock LLM returns predictable outputs
   - Test that the loop terminates
   - Test that feedback flows correctly
   - Test edge cases (always-failing evaluation, cycling outputs)

3. **Convergence tests:**
   - Does the loop converge on known-correct answers?
   - How many iterations does it take?
   - Does it converge faster with better prompts?

4. **Stress tests:**
   - Test with max iterations = 0, 1, 10, 100
   - Test with very short timeout
   - Test with zero-cost budget

5. **Regression tests:**
   - Fixed set of test tasks
   - Track quality over time
   - Alert on regression

### Q22: What is graceful degradation in loop systems? How do you implement it?

Graceful degradation means the loop returns the best available result even when it can't achieve an ideal outcome, rather than failing entirely.

Implementation:
- **Best-so-far tracking:** Always keep the best output seen so far. Return it if the loop terminates early.
- **Fallback models:** If the primary model fails, try a cheaper/faster model.
- **Iteration budget:** When max iterations reached, return best output rather than nothing.
- **Partial completion:** In planning loops, if some steps completed, return partial results.
- **Quality bands:** Define acceptable quality levels. If "excellent" is unreachable, return "good" or "acceptable."

```python
QUALITY_BANDS = [
    ("excellent", 0.9),
    ("good", 0.7),
    ("acceptable", 0.5),
    ("best_effort", 0.0),
]
```

### Q23: How do you handle context window limits in loops?

Loops accumulate context (previous outputs, feedback, history). This can overflow context windows.

Strategies:
- **Sliding window:** Keep only the last N iterations of history.
- **Summarization:** Summarize old iterations into a condensed form.
- **Selective context:** Include only relevant feedback, not all history.
- **Structured state:** Store feedback in a structured format (JSON) rather than raw text.
- **External memory:** Store history in a vector database, retrieve relevant portions.
- **Fixed-size state:** Design the state to have a maximum size regardless of iterations.

### Q24: What's the difference between a retry loop and a refinement loop?

**Retry loop:** Same call, different parameters (or same). Used for transient failures (network, rate limits) or non-deterministic quality. No learning between attempts — each retry is independent.

**Refinement loop:** Each iteration builds on the previous one. Feedback from the evaluation is used to improve the output. The state carries forward.

Hybrid: Retry with refinement — if output quality is low, retry with feedback from the failed attempt.

### Q25: How would you design a loop budget management system?

```python
class LoopBudget:
    def __init__(self, max_iterations=10, max_cost=1.0, max_time=300, max_tokens=100000):
        self.max_iterations = max_iterations
        self.max_cost = max_cost
        self.max_time = max_time
        self.max_tokens = max_tokens
        
        self.iteration_count = 0
        self.total_cost = 0.0
        self.total_tokens = 0
        self.start_time = None
    
    def can_continue(self):
        if self.iteration_count >= self.max_iterations:
            return False, "max_iterations"
        if self.total_cost >= self.max_cost:
            return False, "max_cost"
        if self.total_tokens >= self.max_tokens:
            return False, "max_tokens"
        if time.time() - self.start_time >= self.max_time:
            return False, "max_time"
        return True, "ok"
    
    def record_iteration(self, cost, tokens):
        self.iteration_count += 1
        self.total_cost += cost
        self.total_tokens += tokens
```

### Q26: How would you implement human-in-the-loop for critical decisions?

```python
class HumanInTheLoop:
    def __init__(self, approval_threshold=0.95, max_wait_seconds=3600):
        self.approval_threshold = approval_threshold
        self.max_wait = max_wait_seconds
    
    def request_approval(self, output, context):
        """Request human approval for a loop output."""
        if self.estimated_quality(output) >= self.approval_threshold:
            return True  # Auto-approve for high confidence
        
        # Send to human for review
        notification = {
            "output": output,
            "context": context,
            "estimated_quality": self.estimated_quality(output),
            "timestamp": time.time()
        }
        
        # Wait for human response (async in production)
        response = self.send_for_review(notification)
        return response["approved"]
```

### Q27: How would you design a system to detect and break cycles in loops?

Cycle detection can be done with:

1. **Exact match:** Hash of output appears before → cycle detected.
2. **Similarity threshold:** Cosine similarity or Jaccard similarity above 0.98 with previous output.
3. **Pattern detection:** Same sequence of outputs repeating (A → B → A → B).
4. **Semantic similarity:** Using embeddings to detect semantically identical outputs.

Breaking the cycle:
- Return best output seen before the cycle started
- Introduce randomness (higher temperature, different model)
- Add new information (search, additional context)
- Escalate to human

### Q28: How do loops relate to agents? (Cross-reference to Chapter 07)

**Chapter 07 (Agents):** Agents are loop engineering applied to tool use and environment interaction. The agent loop is a specific instance of a general loop pattern:

```
General loop:        Generate → Evaluate → Update State → Repeat
Agent loop (ReAct):  Reason → Act (use tool) → Observe → Repeat
```

All agent architectures are loops at their core:
- ReAct agents use observation → thought → action loops
- Tool-use agents use task → tool selection → execution → result processing loops
- Multi-agent systems use communication → response → synthesis loops

The concepts from this chapter (stopping conditions, safety, cost optimization, state management) apply directly to agent design.

### Q29: What is a "self-consistency" loop? How does it differ from other loop types?

Self-consistency (Wang et al., 2022) generates multiple independent answers and selects the most consistent one. Unlike refinement loops that iterate sequentially, self-consistency runs in parallel:

```
Generate N answers independently → Cluster answers → Select most common answer
```

Key differences:
- **Parallel vs. sequential:** Self-consistency generates all answers at once; refinement loops are sequential.
- **No feedback:** Self-consistency doesn't use feedback between generations.
- **Voting mechanism:** The "evaluation" is majority voting or consistency checking.
- **Best for:** Tasks with a single correct answer (math, facts) where models sometimes make random errors.

### Q30: How would you determine the optimal number of iterations for a loop?

There's no universal answer, but here's a methodology:

1. **Run empirical tests:** Execute the loop on a representative test set with max_iterations=10.
2. **Plot quality vs. iterations:** Identify the "elbow" where marginal quality gain drops below a threshold.
3. **Calculate cost-benefit:** For each additional iteration, compute (quality_gain / cost). Stop when this ratio falls below acceptable.

Typical findings:
- **Code generation:** 2-4 iterations for most tasks
- **Creative writing:** 2-3 iterations before diminishing returns
- **Reflection:** 1-2 iterations (self-correction degrades after 2)
- **Planning:** 1-2 plan revisions sufficient
- **Retry:** 3-5 attempts before giving up

**Rule of thumb:** Start with 3 iterations, measure, adjust based on data.

### Q31: How do you handle non-deterministic LLM outputs in loops?

Non-determinism means the same prompt can produce different outputs. This affects loops because:
- The same feedback might produce different "fixes"
- Evaluation scores might vary
- Convergence might not happen

Strategies:
- **Temperature 0 for evaluation:** Use temperature 0 for evaluation calls to get consistent scoring.
- **Multiple samples:** Take 2-3 samples and use majority vote for evaluation decisions.
- **Stable sorting:** When comparing outputs, use a deterministic comparison function.
- **Convergence with tolerance:** Allow small variations — convergence doesn't mean identical, it means "good enough and stable."
- **Test with temperature:** Run the loop multiple times (3-5) with the same input to measure variance.

### Q32: What's the difference between a forward loop and a backward loop?

**Forward loop:** Each iteration adds new information or capabilities. Example: RAG loop where each iteration retrieves more documents to fill gaps.

**Backward loop:** Each iteration removes or corrects errors. Example: Code generation loop where each iteration fixes bugs identified by tests.

Most production loops are bidirectional: they both add new content and fix errors. The distinction is useful for analysis but most loops combine both directions.

### Q33: How would you implement a loop that handles multiple feedback sources simultaneously?

Use a weighted scoring system:

```python
def evaluate_multisource(output, sources):
    """
    sources: dict of {source_name: (score_function, weight)}
    """
    total_score = 0.0
    total_weight = 0.0
    all_feedback = []
    
    for name, (score_fn, weight) in sources.items():
        score, feedback = score_fn(output)
        total_score += score * weight
        total_weight += weight
        all_feedback.append({"source": name, "score": score, "feedback": feedback})
    
    normalized_score = total_score / total_weight if total_weight > 0 else 0
    
    return {
        "score": normalized_score,
        "feedback": all_feedback,
        "passed": normalized_score >= 0.8  # Configurable threshold
    }
```

### Q34: How do you test that a loop converges?

A convergence test verifies that the loop terminates within expected bounds across diverse inputs:

```python
test_cases = [
    "Simple math problem",
    "Complex reasoning question",
    "Creative writing task",
    "Code generation task",
    "Ambiguous question with no single answer",
]

for task in test_cases:
    result = run_loop(task)
    assert result["stop_reason"] in ["success", "convergence", "max_iterations"]
    assert result["iterations"] <= max_iterations
    assert result["total_cost"] <= max_cost
    assert result["quality_score"] >= minimum_acceptable_quality
```

### Q35: What is the relationship between loop engineering and chain-of-thought prompting?

Chain-of-thought (CoT) prompting is a single-pass technique where the model generates intermediate reasoning steps. Loop engineering extends this by:

1. **External verification:** CoT reasoning is internal and unverified. Loops can check each reasoning step against external sources.
2. **Iterative refinement:** CoT is one-shot. Loops can revisit and revise reasoning.
3. **Feedback integration:** CoT can't incorporate external feedback. Loops can use test results, search results, etc.
4. **Multi-model:** CoT uses one model. Loops can use different models for different reasoning steps.

CoT can be *part of* a loop — the LLM call within each iteration can use CoT prompting. The loop adds the iterative structure around the CoT reasoning.
