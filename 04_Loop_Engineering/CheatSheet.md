# Cheat Sheet: Loop Engineering

---

## Loop Types Comparison

| Loop Type | Pattern | Evaluation Source | Best For | Cost | Complexity |
|---|---|---|---|---|---|
| **Retry** | Try → Fail → Retry | API/execution success | Handling transient failures | Low | Low |
| **Refinement** | Generate → Evaluate → Refine → Repeat | Any evaluator | Iterative improvement | Medium | Low |
| **Reflection** | Generate → Self-review → Correct | Self (same LLM) | Simple self-correction | Low | Low |
| **Critique** | Generate → Critic evaluates → Refine | Separate LLM | Quality-critical outputs | High | Medium |
| **Validation** | Generate → Validate → Regenerate | Schema/rules/constraints | Structured outputs | Low | Low |
| **Planning** | Plan → Execute → Monitor → Replan | Step completion | Multi-step tasks | High | High |
| **Agent (ReAct)** | Reason → Act → Observe → Repeat | Tool/environment | Interactive tasks | High | High |
| **Recursive** | Decompose → Solve → Compose | Completeness | Large/complex tasks | Variable | High |

---

## When to Use Each Loop Type

```
Task type → Recommended loop

┌─ API call → Retry (exponential backoff)
│
├─ Simple generation → Single call (no loop needed)
│
├─ Code generation → Refinement + Validation (test feedback)
│
├─ Content creation → Refinement (evaluate quality)
│
├─ Critical output → Critique (separate evaluator)
│
├─ Structured output → Validation (schema check)
│
├─ Multi-step task → Planning
│
├─ Research/analysis → Agent (ReAct with tools)
│
├─ Very long document → Recursive (summarization)
│
└─ Math/factual QA → Self-consistency (parallel voting)
```

---

## Six Loop Components

```
1. INPUT/STATE ─── Current context, iteration count, feedback history
2. LLM CALL ────── Generate output with current state
3. OUTPUT ──────── Raw LLM response
4. EVALUATION ──── Assess quality (LLM, rules, external signals)
5. DECISION ────── Continue or stop based on evaluation
6. STATE UPDATE ── Update state with feedback if continuing
```

---

## Stopping Conditions Quick Reference

| Condition | Check | Return |
|---|---|---|
| Max iterations | `iter >= max_iter` | Best effort |
| Convergence | Output stable for N iterations | Current output |
| Success | All constraints satisfied | Current output |
| Timeout | Wall clock > timeout | Best effort |
| Cost exceeded | Total cost > budget | Best effort |
| Degeneration | Quality < best * threshold | Previous best |
| Cycle | Output matches previous | Previous best |
| Human interrupt | External signal | Best effort |

---

## Cost Estimation Formulas

**Single call:**
```
cost = (input_tokens * input_price + output_tokens * output_price) / 1000
```

**Refinement loop (same model):**
```
total = iterations * cost_per_call
```

**Critique loop (different models):**
```
total = iterations * (cost_generator + cost_critic)
```

**Code gen loop (with free tests):**
```
total = iterations * cost_generation
```

**Planning loop:**
```
total = cost_plan + Σ(execution_costs) + replan_costs
```

### Typical Costs (USD per 1K tokens, approximate)

| Model | Input | Output |
|---|---|---|
| GPT-4o | $0.0025 | $0.01 |
| GPT-4o-mini | $0.00015 | $0.0006 |
| Claude 3.5 Sonnet | $0.003 | $0.015 |
| Claude 3 Haiku | $0.00025 | $0.00125 |
| Gemini 1.5 Pro | $0.00125 | $0.005 |
| Gemini 1.5 Flash | $0.000075 | $0.0003 |

### Budget Estimation Example (3-iteration critique loop)

```
Generator: GPT-4o, 500 input + 1000 output tokens = $0.01125/iteration
Critic: GPT-4o-mini, 1500 input + 200 output tokens = $0.00035/iteration
Total per iteration: $0.0116
3 iterations: $0.0348
Budget recommendation: $0.10 (3x safety margin)
```

---

## Latency Optimization

| Technique | Speedup | Complexity |
|---|---|---|
| Parallel evaluation | 2-5x | Low |
| Async execution | 1.5-3x | Medium |
| Streaming + early eval | 1.2-2x | Medium |
| Batched evaluations | 1.5-3x | Low |
| Cache identical prompts | Variable | Low |
| Speculative execution | 1.5-2x | High |

---

## Safety Checks Checklist

```
□ Max iterations set (always)
□ Cost budget set (always)
□ Timeout set (always)
□ Degeneration detection enabled
□ Cycle detection enabled
□ Best-so-far tracking enabled
□ Context window protection
□ Human-in-the-loop for critical paths
□ Fallback model configured
```

---

## Prompt Templates

### Self-Reflection Prompt
```
Review your response for specific errors:

1. Factual errors (list with corrections):
2. Reasoning errors (identify flawed step):
3. Omissions (missing important info):

Severity of each: CRITICAL / MODERATE / MINOR

Then provide a corrected version.
```

### Critique Prompt (Evaluator)
```
Evaluate this response on:
1. Accuracy (0-10):
2. Completeness (0-10):
3. Clarity (0-10):
4. Specific issues:
5. Improvement suggestions:

Final verdict: APPROVE or REVISE
```

### Code Generation Fix Prompt
```
The following errors occurred:

{errors}

Fix the code below to resolve ALL errors.
Return ONLY the corrected code, no explanations.

{code}
```

### Planning Prompt
```
Task: {task}

Create a step-by-step plan. Each step must include:
- Action: What to do
- Criteria: How to verify success

Format as JSON array of step objects.
```

---

## Key Papers Reference

| Paper | Year | Key Idea |
|---|---|---|
| ReAct (Yao et al.) | 2022 | Reasoning + Acting loop for agents |
| Reflexion (Shinn et al.) | 2023 | Self-reflection with episodic memory |
| Self-Refine (Madaan et al.) | 2023 | Generate → Self-feedback → Refine |
| Tree of Thoughts (Yao et al.) | 2023 | Tree search over reasoning paths |
| Self-Consistency (Wang et al.) | 2022 | Sample multiple paths, vote |
| LLM Debate (Du et al.) | 2023 | Multiple LLMs debate answers |
| Chain-of-Thought (Wei et al.) | 2022 | Intermediate reasoning steps |
| RAG (Lewis et al.) | 2020 | Retrieve-then-generate |

---

## Quick Decision Flowchart

```
Need a loop?
│
├─ API call fails sometimes?
│   └─ Retry with backoff
│
├─ Output needs to meet quality bar?
│   ├─ Self-review enough?
│   │   └─ Reflection loop
│   └─ Need independent review?
│       └─ Critique loop
│
├─ Output must follow strict format?
│   └─ Validation loop
│
├─ Multiple steps to complete?
│   └─ Planning loop
│
├─ Need to use tools?
│   └─ Agent (ReAct) loop
│
├─ Very long input?
│   └─ Recursive summarization
│
└─ None of the above?
    └─ Single LLM call (no loop)
```

---

## Common Pitfalls

| Pitfall | Symptom | Fix |
|---|---|---|
| Infinite loop | Never terminates | Set max_iterations |
| Cost explosion | Bill shock | Set cost budget + early stopping |
| Degenerate output | Quality decreases | Track best, rollback |
| Confirmation loop | Same errors persist | Use independent evaluator |
| Context overflow | Truncated history | Sliding window / summarization |
| Oscillation | A→B→A→B pattern | Cycle detection |
| Vague feedback | No improvement | Structured feedback template |
| Over-correction | Correct→incorrect | Validate before accepting |
