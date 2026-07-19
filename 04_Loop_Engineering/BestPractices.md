# Best Practices: Production Loop Engineering

---

## 1. Loop Budget Management

### Always Set Multiple Budgets

A loop must have at least three independent budgets:

```python
loop_config = {
    "max_iterations": 10,      # Hard iteration limit
    "max_cost_usd": 0.50,      # Cost limit
    "max_time_seconds": 120,   # Wall-clock timeout
}
```

**Why multiple?** Each budget catches different failure modes:
- Max iterations: Catches infinite loops
- Cost limit: Catches expensive loops that terminate legitimately but cost too much
- Timeout: Catches loops where each iteration is slow

### Budget Hierarchy

```
Task-level budget (highest priority)
  ├── User-level daily budget
  ├── Loop-type budget (per loop type)
  └── Per-execution budget (single loop run)
```

### Dynamic Budget Adjustment

Adjust budgets based on task complexity:

```python
def get_budget_for_task(task):
    complexity = estimate_task_complexity(task)
    
    if complexity == "simple":
        return {"max_iterations": 3, "max_cost": 0.10, "max_time": 30}
    elif complexity == "medium":
        return {"max_iterations": 5, "max_cost": 0.25, "max_time": 60}
    else:  # complex
        return {"max_iterations": 10, "max_cost": 0.50, "max_time": 120}
```

### Budget Tracking Dashboard

Monitor in real-time:
- Remaining iterations
- Remaining cost budget
- Remaining time
- Cost per iteration
- Cost per quality point

---

## 2. State Management

### Keep State Explicit and Serializable

```python
@dataclass
class LoopState:
    task: str
    iteration: int
    current_output: Optional[str]
    best_output: Optional[str]
    best_score: float
    feedback_history: List[Dict]
    cost_tracked: float
    start_time: float
    metadata: Dict
    
    def to_dict(self) -> Dict:
        return asdict(self)
    
    @classmethod
    def from_dict(cls, data: Dict) -> 'LoopState':
        return cls(**data)
```

### State Size Management

- **Maximum state size:** State should never exceed 20% of context window
- **Structured state:** Use JSON/structured formats instead of raw text
- **Summarization:** Summarize old feedback before appending new
- **Sliding window:** Keep only last N iterations of detailed history

### State Versioning

Version your state schema. If you change how state is structured, old logs can still be parsed:

```python
state = {
    "version": 2,
    "schema": "loop_state_v2",
    "data": { ... }
}
```

---

## 3. Evaluation Design

### Separate Evaluation from Generation

Never use the same prompt/function for generation and evaluation. They serve different purposes and need different designs.

| Aspect | Generation Prompt | Evaluation Prompt |
|---|---|---|
| Temperature | 0.3-0.7 | 0.0-0.1 |
| Model | Capable (GPT-4o, Claude Opus) | Fast/cheap (GPT-4o-mini, Claude Haiku) |
| Output format | Creative/expansive | Structured/scored |
| Focus | Completing the task | Finding errors |

### Evaluation Best Practices

1. **Be specific:** "Rate accuracy 0-10" is better than "Is this good?"
2. **Provide criteria:** List exactly what constitutes a pass/fail
3. **Use structured output:** Request JSON with specific fields
4. **Calibrate:** Test evaluator against human judgments
5. **Fail-fast:** Stop evaluation as soon as a critical error is found

### Multi-Stage Evaluation

```python
def evaluate(output, task):
    # Stage 1: Quick structural checks (free)
    if not has_required_sections(output):
        return {"passed": False, "reason": "missing_sections", "cost": 0}
    
    # Stage 2: Heuristic checks (free)
    if not meets_length_requirement(output):
        return {"passed": False, "reason": "too_short", "cost": 0}
    
    # Stage 3: LLM evaluation (costly)
    eval_result = llm_evaluate(output, task)
    return eval_result
```

---

## 4. Feedback Quality

### Characteristics of Good Feedback

| Good Feedback | Bad Feedback |
|---|---|
| "Line 23: IndexError — items list is empty" | "Your code has errors" |
| "Missing citation for claim about GDP growth" | "Needs more detail" |
| "The conclusion contradicts the introduction" | "Could be better" |
| "Add error handling for null inputs" | "Fix this" |

### Feedback Template

```python
FEEDBACK_TEMPLATE = """
Issue #{number}:
- Location: {location}
- Type: {error_type}
- Description: {description}
- Suggested fix: {suggestion}
- Severity: {severity}
"""
```

### Validate Feedback Before Using It

Check that feedback is:
- **Specific:** Points to a concrete issue
- **Actionable:** Suggests what to change
- **Correct:** The issue actually exists
- **Relevant:** Pertains to the task requirements

---

## 5. Monitoring in Production

### Key Metrics

```python
METRICS = {
    "loop_executions": "counter",       # Total loops run
    "iterations_per_loop": "histogram",  # Distribution of iteration counts
    "stop_reasons": "counter",          # Distribution of stop reasons
    "cost_per_loop": "histogram",       # Cost distribution
    "latency_per_loop": "histogram",    # Latency distribution
    "quality_scores": "histogram",      # Final quality score distribution
    "quality_delta": "histogram",       # Quality improvement per iteration
    "degeneration_rate": "gauge",       # Rate of degeneration events
    "cycle_rate": "gauge",              # Rate of cycle detection
    "human_escalation_rate": "gauge",   # Rate of human-in-the-loop escalations
}
```

### Logging Format

```json
{
    "event": "loop_iteration",
    "task_id": "task_abc123",
    "loop_type": "code_generation",
    "iteration": 2,
    "state": {
        "compile_errors": 0,
        "test_failures": 2,
        "lint_warnings": 5,
        "quality_score": 0.65
    },
    "cost": {
        "generation": 0.012,
        "evaluation": 0.001,
        "cumulative": 0.031
    },
    "timing": {
        "generation_ms": 1200,
        "evaluation_ms": 300,
        "iteration_ms": 1650
    }
}
```

### Alerting Rules

```yaml
alerts:
  - name: high_iteration_count
    condition: iterations_per_loop.p95 > 8
    severity: warning
    
  - name: cost_explosion
    condition: cost_per_loop.max > 1.0
    severity: critical
    
  - name: high_degeneration
    condition: degeneration_rate > 0.1
    severity: critical
    
  - name: loop_timeout
    condition: stop_reasons.timeout > 0.05
    severity: warning
```

---

## 6. Testing Loops

### Unit Tests for Components

Test each component in isolation:

```python
def test_evaluator():
    assert evaluate("correct answer", "test task") == {"passed": True}
    assert evaluate("wrong answer", "test task")["passed"] == False

def test_stopping_conditions():
    loop = SafeLoop(max_iterations=3)
    assert loop.should_continue(0, "output", 0.5) == (True, "continue")
    assert loop.should_continue(3, "output", 0.5) == (False, "max_iterations")

def test_state_management():
    state = LoopState(task="test", iteration=0)
    state.update("new output", {"score": 0.8})
    assert state.iteration == 1
    assert state.best_score == 0.8
```

### Integration Tests

Test the full loop with mock LLM:

```python
def test_loop_convergence():
    """Test that loop converges to correct answer."""
    mock_llm = MockLLM(responses=[
        "wrong answer",
        "better answer",
        "correct answer",
    ])
    
    result = refinement_loop("test task", llm=mock_llm, max_iterations=5)
    assert result["output"] == "correct answer"
    assert result["iterations"] == 3

def test_loop_respects_budget():
    """Test that loop stops when budget exhausted."""
    mock_llm = MockLLM(responses=["bad"] * 100)
    
    result = refinement_loop("test task", llm=mock_llm, max_iterations=3)
    assert result["iterations"] == 3

def test_loop_handles_errors():
    """Test that loop handles LLM failures gracefully."""
    mock_llm = MockLLM(errors=[APIError(), APIError(), "good response"])
    
    result = refinement_loop("test task", llm=mock_llm, retry_count=2)
    assert result["output"] == "good response"
```

### Regression Test Suite

Maintain a fixed set of test tasks with known correct outputs:

```python
REGRESSION_TESTS = [
    {
        "task": "Write a Python function to check if a string is a palindrome",
        "expected_behavior": "Function passes all test cases",
        "min_quality": 0.8
    },
    {
        "task": "Explain the water cycle in 3 sentences",
        "expected_behavior": "Covers evaporation, condensation, precipitation",
        "min_quality": 0.7
    },
]
```

---

## 7. Graceful Degradation

### Best-Effort Return Pattern

```python
def run_loop_with_degradation(task):
    best_result = None
    best_quality = -1
    
    for iteration in range(max_iterations):
        try:
            output = generator(task, previous_output=best_result)
            quality = evaluator(output)
            
            if quality > best_quality:
                best_result = output
                best_quality = quality
            
            if quality >= ACCEPTABLE_THRESHOLD:
                return {"output": output, "quality": quality, "degraded": False}
                
        except Exception as e:
            log_error(e)
            continue
    
    # Degraded return
    if best_result:
        return {"output": best_result, "quality": best_quality, "degraded": True}
    
    # Complete failure
    return {"output": None, "quality": 0, "degraded": True, "error": "No output generated"}
```

### Quality Bands

```python
QUALITY_BANDS = [
    ("excellent", 0.90, "return immediately"),
    ("good", 0.75, "return immediately"),
    ("acceptable", 0.60, "continue but accept if budget near limit"),
    ("poor", 0.40, "continue refining"),
    ("unacceptable", 0.0, "always regenerate"),
]

def get_quality_band(score):
    for band_name, threshold, action in QUALITY_BANDS:
        if score >= threshold:
            return band_name, action
    return "unacceptable", "always regenerate"
```

---

## 8. Production Checklist

### Pre-Deployment

- [ ] All three budgets set (iterations, cost, time)
- [ ] State is serializable and size-managed
- [ ] Evaluation function tested against human judgments
- [ ] Feedback quality validated
- [ ] All LLM calls have retry logic
- [ ] Monitoring metrics defined and implemented
- [ ] Alerts configured for failure modes
- [ ] Graceful degradation implemented
- [ ] Regression test suite passes
- [ ] Human-in-the-loop integration for critical decisions
- [ ] Context window overflow protection
- [ ] Cycle detection enabled

### Post-Deployment Monitoring

- [ ] Track iteration count distribution daily
- [ ] Alert on cost outliers
- [ ] Monitor degeneration rate
- [ ] Review human escalation rate
- [ ] Update regression tests weekly
- [ ] A/B test loop configuration changes
- [ ] Review quality scores per loop type
- [ ] Audit feedback quality periodically

---

## 9. Common Anti-Patterns

### Anti-Pattern 1: Infinite Faith in Self-Correction

**Bad:**
```python
# No max iterations, no evaluation — just hope the model fixes itself
while True:
    output = llm("Review and improve your previous answer")
```

**Good:**
```python
for i in range(max_iterations):
    output = llm(prompt)
    if evaluator(output):
        return output
```

### Anti-Pattern 2: Ignoring Diminishing Returns

**Bad:**
```python
# Always runs 10 iterations regardless
for i in range(10):
    output = refine(output)
```

**Good:**
```python
for i in range(max_iterations):
    new_output = refine(output)
    if similarity(output, new_output) > 0.95:
        break  # Converged
    output = new_output
```

### Anti-Pattern 3: Unstructured Feedback

**Bad:**
```
"Please fix your response."
```

**Good:**
```
"Your response has 2 errors:
1. Factual error: The capital of France is Paris, not Lyon (severity: CRITICAL)
2. Omission: Missing the year of the French Revolution (severity: MODERATE)"
```

### Anti-Pattern 4: Cost Blindness

**Bad:**
```python
# No cost tracking — potential bill shock
output = llm(prompt)  # $0.03
output = llm(f"Refine: {output}")  # $0.03
output = llm(f"Refine more: {output}")  # $0.03
# 100 iterations later: $3.00
```

**Good:**
```python
budget = LoopBudget(max_cost=0.50)
while budget.can_continue():
    cost = track_cost(llm(prompt))
    budget.record(cost)
```

### Anti-Pattern 5: Feedback Loops Without Escape

Never nest loops without escape hatches:

**Bad:**
```python
# Outer loop has max_iterations, but inner loop doesn't
for task in tasks:
    while True:  # No limit!
        output = llm(f"Fix: {output}")
```

**Good:**
```python
for task in tasks:
    for fix_attempt in range(3):  # Inner limit
        output = llm(f"Fix: {output}")
```

---

## 10. Configuration Reference

### Loop Configuration Template

```yaml
loop:
  type: refinement  # refinement | critique | validation | planning | agent
  
  # Models
  generator:
    model: gpt-4o
    temperature: 0.3
    max_tokens: 2048
  
  evaluator:
    model: gpt-4o-mini
    temperature: 0.0
    max_tokens: 512
  
  # Budgets
  budgets:
    max_iterations: 5
    max_cost_usd: 0.25
    max_time_seconds: 60
    max_tokens: 50000
  
  # Stopping
  stopping:
    convergence_threshold: 0.95
    convergence_window: 3
    min_quality_score: 0.7
  
  # Safety
  safety:
    degeneration_threshold: 0.5
    cycle_detection: true
    human_escalation_threshold: 0.3
    context_window_safety_margin: 0.8
  
  # Cost optimization
  optimization:
    early_stopping: true
    cheap_evaluator: true
    caching: true
    adaptive_iterations: true
  
  # Monitoring
  monitoring:
    log_level: info
    trace_full_state: false
    metrics_interval_seconds: 10
```
