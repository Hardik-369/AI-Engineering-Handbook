# Interview Questions: Prompt Engineering

> 25+ questions covering fundamental concepts, practical techniques, and advanced topics. Questions are grouped by difficulty with answer guidance.

---

## Beginner Questions

### Q1: What is prompt engineering, and why is it important?

**Expected answer:** Prompt engineering is the systematic design and optimization of inputs to LLMs to produce desired outputs. It's important because LLMs are sensitive to input phrasing, formatting, and structure — small changes can dramatically affect output quality. Without prompt engineering, LLM outputs are unreliable for production use.

**Key points:** Mental model of prompting as programming, prompt as program, LLM as runtime.

---

### Q2: Explain the difference between zero-shot and few-shot prompting.

**Expected answer:** Zero-shot provides no examples — the model relies entirely on its pre-training. Few-shot provides input-output examples in the prompt, leveraging in-context learning. Zero-shot is faster and cheaper but less reliable for novel formats. Few-shot is more accurate for domain-specific tasks but costs more in tokens.

**Example:** Zero-shot: "Translate to French: Hello" → "Bonjour". Few-shot includes examples like `Input: Hello\nOutput: Bonjour` before the target input.

---

### Q3: What is in-context learning?

**Expected answer:** In-context learning is the ability of LLMs to infer patterns from examples provided in the prompt, without any weight updates or fine-tuning. The model generalizes from the input-output pairs to perform similar transformations on new inputs. It's not true learning — it's conditioned generation guided by the provided pattern.

---

### Q4: What is Chain of Thought prompting?

**Expected answer:** CoT prompting instructs the model to produce intermediate reasoning steps before the final answer. This improves accuracy on multi-step reasoning by creating a "scratchpad" that maintains state and breaks complex problems into manageable steps. Two variants: zero-shot CoT ("Let's think step by step") and manual CoT (providing reasoning examples).

**Attribution:** Wei et al. (2022).

---

### Q5: What are system messages vs user messages?

**Expected answer:** System messages set the model's behavior, role, and constraints — they have the highest priority in determining response style. User messages contain the actual task input. Best practice: put identity, format rules, and constraints in system messages; put the specific task and data in user messages. This separation improves reliability and makes prompts easier to maintain.

---

### Q6: What is temperature in LLM parameters?

**Expected answer:** Temperature controls the randomness of token selection. Low temperature (0-0.2) produces deterministic, focused outputs. High temperature (0.7-1.0) produces more creative, diverse outputs. For factual tasks, use low temperature. For creative tasks, use higher temperature. Temperature interacts with prompt engineering — deterministic tasks need strict prompts, creative tasks need looser structure.

---

## Intermediate Questions

### Q7: When would you choose few-shot prompting over fine-tuning?

**Expected answer:** Choose few-shot when:
- You have limited labeled data (under 100 examples)
- You need to quickly iterate on different tasks
- The task definition changes frequently
- You can't afford fine-tuning infrastructure

Choose fine-tuning when:
- You have thousands of labeled examples
- Latency and cost per token must be minimized
- The task is stable and well-defined
- You need consistent behavior across diverse inputs

Few-shot is a low-cost starting point; fine-tuning is for production at scale.

---

### Q8: How do you select examples for few-shot prompting? What are common mistakes?

**Expected answer:** Select examples that are:
- Diverse (cover different categories, formats, edge cases)
- Representative (reflect real-world distribution)
- Clear (unambiguous input-output mapping)
- Consistent (same format across all examples)

**Common mistakes:**
- Random sampling without diversity
- All examples from the same distribution (fails on edge cases)
- Too many examples (diminishing returns after 5-10)
- Inconsistent formatting (confuses the model)
- Examples that don't match the actual task distribution

---

### Q9: Explain self-consistency with CoT. How does it improve accuracy?

**Expected answer:** Self-consistency runs the same CoT prompt multiple times with temperature > 0.0, generating diverse reasoning paths. Final answers are aggregated by majority vote. This improves accuracy because:
1. Different reasoning paths may reach different conclusions
2. The correct answer tends to appear more consistently across paths
3. Random errors cancel out while correct reasoning converges

Reported improvements: 10-20% on math reasoning benchmarks.

---

### Q10: What is Tree of Thoughts, and how does it differ from Chain of Thought?

**Expected answer:** ToT extends CoT by exploring multiple reasoning branches at each step, evaluating intermediate states, and pruning unpromising paths. Key differences:
- CoT: single linear path; ToT: branching tree
- CoT: evaluates only final answer; ToT: evaluates intermediate states
- CoT: lower cost; ToT: 5-10x more expensive
- CoT: sufficient for straightforward reasoning; ToT: better for exploratory problems

**Analogy:** CoT is like depth-first search with a single path; ToT is like BFS/DFS with backtracking.

---

### Q11: How does function calling work in LLMs? Describe the flow.

**Expected answer:** 
1. Application sends user query + tool definitions to LLM
2. LLM decides which tools to call and generates structured arguments
3. Application receives tool call request(s), executes the function
4. Application sends tool results back to LLM
5. LLM incorporates results into final response

**Key concepts:** `tool_choice` (auto/required/force), parallel tool calls, streaming tool calls.

---

### Q12: What is the ReAct framework? What problem does it solve?

**Expected answer:** ReAct (Reasoning + Acting) interleaves reasoning steps (Thought) with actions (Act/tool use) in a single prompt-driven loop. Each cycle: Thought → Action → Observation → repeat. It solves the problem of LLMs lacking access to external information — the agent can search, query databases, or call APIs mid-reasoning.

**Attribution:** Yao et al. (2022).

---

### Q13: How do you force an LLM to output structured data (JSON, XML)?

**Expected answer:** Multiple approaches:
1. **API-level enforcement** — OpenAI `response_format: {"type": "json_object"}` or JSON schema; Anthropic tools with input schema; Gemini `response_mime_type: "application/json"`
2. **Prompt-level** — Explicit format instructions, output contracts, XML templates
3. **Post-processing** — Parse output with regex, validate schema, retry on failure
4. **Constrained decoding** — Limit token selection to valid JSON tokens (e.g., `jsonformer`, `outlines`)

**Best practice:** Combine API-level enforcement (most reliable) with prompt-level instructions as backup.

---

### Q14: What is prompt injection? How do you defend against it?

**Expected answer:** Prompt injection occurs when user input contains instructions that override or manipulate the system prompt. Types:
- Direct injection: User says "Ignore previous instructions..."
- Indirect injection: Attacker-controlled text in retrieved documents contains instructions

**Defenses:**
- Input sanitization (strip instruction-like patterns)
- Delimiter-based separation of instructions from data
- Least-privilege prompts (don't give model unnecessary capabilities)
- Output validation layer (check output against allowed patterns)
- Separate classifier for harmful inputs

Covered in detail in Chapter 07: Safety.

---

### Q15: How does output token count affect prompt design?

**Expected answer:** Output token count affects:
- **Cost** — Output tokens are often billed at higher rates than input
- **Latency** — More output tokens = longer generation time (autoregressive)
- **Quality** — Very long outputs may lose coherence; very short may lack detail
- **Prompt strategy** — CoT requires more output tokens but improves accuracy

**Design implications:**
- Set `max_tokens` appropriately (don't over-allocate)
- Use `stop` sequences to truncate at desired length
- For structured output, minimize output by using concise formats

---

## Advanced Questions

### Q16: Design a prompt testing framework. What metrics would you track?

**Expected answer:** A prompt testing framework should include:

**Infrastructure:**
- Prompt template registry (versioned)
- Test case database (inputs + expected outputs)
- Model router (test across multiple models/providers)
- Evaluation pipeline (automated scoring)
- Report generator (comparison dashboards)

**Metrics:**
- Accuracy / F1 / Exact match
- Latency (P50, P95, P99)
- Token usage (input, output, total)
- Cost per call
- Parse success rate (for structured outputs)
- Failure modes (hallucination, refusal, off-topic)
- Consistency across runs (for temperature > 0)

---

### Q17: How would you optimize a prompt for cost without sacrificing quality?

**Expected answer:** 

1. **Compress instructions** — Use concise language, remove redundant phrases
2. **Minimize examples** — Find minimum viable few-shot count (diminishing returns after 3-5)
3. **Reduce output tokens** — Use shorter formats, `max_tokens`, stop sequences
4. **Selective context** — Only include relevant information, not full documents
5. **Prompt compression** — Use LLMLingua or similar for 2-5x compression
6. **Cache repeated prefixes** — System prompts are often identical across calls
7. **Route to cheaper models** — Use GPT-4o-mini for simple tasks, GPT-4o for complex

**Trade-off curve:** Plot quality vs cost for each optimization to find Pareto frontier.

---

### Q18: Describe a situation where few-shot prompting performs worse than zero-shot. Why?

**Expected answer:** Few-shot can underperform when:
1. **Examples are misleading** — If examples don't match the target distribution, the model learns wrong patterns
2. **Recency bias dominates** — The model over-fits to the last example(s)
3. **Label bias** — Example labels create unintended associations
4. **Format distraction** — Complex example formats distract from the actual task
5. **Contradictory examples** — Inconsistent mapping in examples confuses the model

**Solution:** Use diverse, representative examples. Validate that examples actually help. Test zero-shot as baseline.

---

### Q19: How would you handle multi-turn conversations in a function-calling agent?

**Expected answer:**

1. **Conversation state** — Maintain message history with alternating user/assistant/tool messages
2. **Context management** — Summarize or truncate old messages when approaching context limits
3. **Tool call persistence** — Track which tools have been called and their results
4. **Missing information** — Agent should ask clarifying questions when required parameters are missing
5. **Confirmation flow** — For destructive actions (place_order, delete), ask user confirmation before executing
6. **Error recovery** — Handle failed tool calls, retry with different args, or ask user

**Pattern:** Append each turn to message list. Check if all required info for the target action is collected before calling the final tool.

---

### Q20: Explain how you would build a ReAct agent that can browse the web.

**Expected answer:**

**Tools:**
- `search_web(query)` — Returns search results
- `visit_page(url)` — Returns page content (truncated)
- `extract_text(html)` — Strips HTML, returns clean text
- `scroll_down()` — Gets more content from current page

**Prompt design:**
```
You are a web research agent. You have browsing tools available.
Your goal is to answer the user's question using information found on the web.

For each step:
- Thought: What information do I need? Where should I look?
- Action: Which tool to call and with what arguments?

When you find the answer:
- Final Answer: <answer with citation>

Rules:
- Only visit links from reliable sources
- Cross-reference information from multiple sources
- If information is contradictory, note the discrepancy
- Never fabricate information — if not found, say so
```

**State management:** Track visited URLs to avoid loops. Summarize page content when context is full.

---

### Q21: What are prompt anti-patterns? Give 5 examples.

**Expected answer:** Prompt anti-patterns are common mistakes that degrade output quality:

1. **Over-constraining** — Too many rules the model can't satisfy simultaneously
2. **Contradictory instructions** — "Be concise" + "Explain in detail"
3. **Ambiguous pronouns** — "Process it and then summarize it" (what's "it"?)
4. **Ordering bias** — Important instruction buried in the middle of a long prompt
5. **Information leakage** — Exposing sensitive instructions in system prompt
6. **Label bias** — Using biased category names that skew classification
7. **Missing delimiters** — Unclear boundaries between instructions and data

---

### Q22: How does DSPy optimize prompts? What are its limitations?

**Expected answer:** DSPy treats LLM programs as differentiable functions and optimizes prompts algorithmically:

**How it works:**
1. Defines a "module" (e.g., ChainOfThought with input/output specification)
2. Bootstraps demonstrations from training examples
3. Uses a teleprompter (optimizer) to select optimal examples and instruction phrasing
4. Evaluates candidate prompts against a metric
5. Iterates to find best-performing configuration

**Limitations:**
- Requires labeled training data (10-100+ examples)
- Computationally expensive (many LLM calls during optimization)
- Metric design is critical — bad metric = bad optimization
- Overfitting to training set if not enough validation data
- No support for multi-turn or tool-using agents (yet)
- Black-box optimization — hard to interpret why one prompt outperforms

---

### Q23: Design a prompt that can reliably extract information from unstructured text. Walk through your design decisions.

**Expected answer:**

```
System: You are a data extraction engine. Extract fields according to the
provided schema. Output ONLY valid JSON. Use null for missing values.

Schema: {schema: definition}

Rules:
1. Extract only explicitly stated information
2. Normalize dates to ISO 8601 format
3. Normalize currencies to decimal numbers (no symbols)
4. If multiple values exist for a field, use the most recent
5. For uncertain extractions, include confidence: 0.0-1.0
6. If the text is empty or irrelevant, return: {"error": "no_data"}

Examples:
{examples}

Input:
{text}
```

**Design decisions:**
- **System message** sets role and primary constraint (JSON only)
- **Schema** provides explicit structure → increases parse rate
- **Rules** handle edge cases (missing data, normalization, uncertainty)
- **Examples** demonstrate schema usage for diverse scenarios
- **Error case** prevents hallucination on irrelevant input
- **JSON enforcement** enables programmatic consumption

---

### Q24: Compare and contrast prompt engineering for code generation vs text generation.

**Expected answer:**

| Aspect | Code Generation | Text Generation |
|--------|----------------|-----------------|
| Output structure | Strict syntax required | Flexible format acceptable |
| Error tolerance | Zero — invalid code breaks | Higher — typos are tolerable |
| Context sensitivity | High — variable names, types, imports | Lower — general knowledge |
| Testing | Compilation + unit tests | Human evaluation |
| Few-shot value | Very high — shows pattern | Moderate |
| Common issues | Off-by-one, wrong API, imports | Hallucination, tone mismatch |
| Best model | Specialized (GitHub Copilot, Codex) | General (GPT-4o, Claude) |

**Key insight:** Code generation benefits more from constrained decoding and syntax validation. Text generation benefits more from role and constraint prompting.

---

### Q25: You have a prompt that works well for GPT-4 but poorly for Claude. How do you adapt it?

**Expected answer:**

1. **System message format** — OpenAI uses system role; Anthropic uses `system` parameter (different handling)
2. **Tool/function calling** — Different API formats need translation
3. **XML vs Markdown** — Claude generally prefers XML delimiters; GPT handles both
4. **Few-shot presentation** — May need different example formatting
5. **Role phrasing** — Claude responds differently to role prompts

**Adaptation checklist:**
- [ ] Convert system message to Anthropic `system` parameter
- [ ] Wrap instructions in `<instruction>` tags (Claude)
- [ ] Convert OpenAI function definitions to Anthropic tool format
- [ ] Adjust `response_format` to Anthropic tool-based structured output
- [ ] Test temperature scaling (Claude often uses higher default temp)
- [ ] Add explicit "Only respond with the tool call" for function calling

**Meta-approach:** Use a meta-prompt that adapts prompts between providers.

---

### Q26: What is the relationship between prompt engineering and fine-tuning? When would you use each?

**Expected answer:** They exist on a spectrum:

| Approach | Data Needed | Cost | Flexibility | Performance |
|----------|-------------|------|-------------|-------------|
| Zero-shot prompt | 0 | Lowest | Highest | Baseline |
| Few-shot prompt | 5-20 examples | Low | High | Good |
| Many-shot prompt | 100-1000 examples | Medium | Medium | Better |
| Fine-tuning | 1000+ examples | High | Low | Best |
| RLHF/DPO | 10k+ preferences | Highest | Lowest | Best aligned |

**When to use each:**
- **Prompting** — Rapid prototyping, task exploration, frequent changes
- **Fine-tuning** — Stable task, large dataset, need to reduce cost/tokens
- **Hybrid** — Fine-tune base model, use few-shot for per-instance customization

---

### Q27: How would you evaluate the quality of a prompt systematically?

**Expected answer:**

**Quantitative metrics:**
- Accuracy / F1 / Precision / Recall / Exact match
- BLEU / ROUGE / METEOR (for generation tasks)
- Parse rate (for structured outputs)
- Token efficiency (output tokens / input tokens)
- Latency percentiles
- Cost per successful output

**Qualitative evaluation:**
- Human rating on 1-5 scale (clarity, completeness, tone)
- A/B preference tests (blind comparison)
- Error analysis (categorize failure modes)
- Edge case coverage

**Systematic approach:**
1. Define task and metric
2. Create test set (100+ examples with labeled ground truth)
3. Run baseline prompt
4. Run candidate prompt
5. Statistical significance test (bootstrap, McNemar's)
6. Report: metric, variance, failure modes, cost

---

### Q28: Describe a production system for prompt management.

**Expected answer:**

```yaml
components:
  - Prompt Registry:
      - Versioned prompt templates (git-based)
      - Environment separation (dev/staging/prod)
      - Template variables for dynamic content
  
  - Experiment Platform:
      - A/B testing infrastructure
      - Parameter sweep (temperature, model, prompt variant)
      - Result tracking and dashboards
  
  - Evaluation Pipeline:
      - Automated test suite (unit tests for prompts)
      - Regression testing against historical outputs
      - Online evaluation (user feedback, click-through)
  
  - Monitoring:
      - Output quality metrics (real-time)
      - Cost tracking per prompt variant
      - Error rate alerts
      - Drift detection (output distribution changes)

  - Deployment:
      - Gradual rollout (5% → 25% → 50% → 100%)
      - Rollback capability
      - Canary testing
```

Covered in depth in [BestPractices.md](./BestPractices.md).

---

## Questions by Category

| Category | Questions |
|----------|-----------|
| Fundamentals | Q1, Q2, Q3, Q4, Q5, Q6 |
| Few-shot & In-context Learning | Q7, Q8, Q18 |
| Reasoning (CoT, ToT, Self-consistency) | Q9, Q10 |
| Tools & Function Calling | Q11, Q19 |
| Agents (ReAct) | Q12, Q20 |
| Structured Output | Q13, Q23 |
| Safety & Anti-patterns | Q14, Q21 |
| Optimization & Cost | Q16, Q17, Q22, Q27 |
| Production & Management | Q15, Q24, Q25, Q26, Q28 |
