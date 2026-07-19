# Cheat Sheet: Prompt Engineering Quick Reference

> One-page-per-technique quick reference for daily use.

---

## Prompt Techniques Overview

| Technique | Syntax/Pattern | Best For | Cost | Reliability |
|-----------|---------------|----------|------|-------------|
| Zero-shot | `Translate: {text}` | Simple, well-known tasks | Low | Medium |
| Few-shot | `Input: {ex1}\nOutput: {ans1}\nInput: {target}` | New formats, domain tasks | Medium | High |
| Chain of Thought | `Let's think step by step.` | Multi-step reasoning | Medium | High |
| Tree of Thoughts | Branch + evaluate + prune | Complex exploration | Very High | Very High |
| Structured Output | `Output JSON: {...}` | Programmatic consumption | Low | High (with API) |
| Function Calling | Tool definitions + API | Tool use, APIs | Low | High |
| Role Prompting | `You are an expert...` | Domain tasks | Low | Medium |
| Constraint Prompting | `Respond in exactly N sentences` | Output boundaries | Low | Medium |
| ReAct | `Thought: ... Action: ... Observation: ...` | Agent loops | High | Medium |
| Meta Prompting | `Generate a prompt that...` | Prompt development | Medium | Medium |

---

## Zero-Shot Prompting

```
Template:
  System: "You are a {role}. {instruction}"
  User: "{input}"

When it works:
  ✓ Translation, summarization, simple classification
  ✓ General knowledge questions
  ✓ High-resource languages

When it fails:
  ✗ Novel formats or taxonomies
  ✗ Domain-specific terminology
  ✗ Tasks requiring precise output structure

Parameters:
  temperature: 0 (deterministic)
  max_tokens: proportional to expected output
```

---

## Few-Shot Prompting

```
Template:
  System: "Follow the pattern shown in the examples."

  User: "
  Input: {example_1_input}
  Output: {example_1_output}

  Input: {example_2_input}
  Output: {example_2_output}

  Input: {target_input}
  Output: "

Best practices:
  - 3-5 examples for classification
  - 5-10 examples for complex reasoning
  - Put most relevant example last (recency bias)
  - Consistent format across all examples
  - Cover edge cases and diverse inputs

Common mistakes:
  - Random examples without diversity
  - Inconsistent formatting
  - Too many examples (diminishing returns after 10)
  - Examples that don't match task distribution
```

---

## Chain of Thought (CoT)

```
Zero-shot CoT:
  System: "Reason step by step."
  User: "{question}\n\nLet's think step by step."

Manual CoT:
  System: "Solve problems step by step."
  User: "
  Question: {example_1_q}
  Reasoning: {example_1_steps}
  Answer: {example_1_answer}

  Question: {target_q}
  Reasoning: "

Self-consistency:
  - Run CoT N times (temperature: 0.5-0.7)
  - Aggregate answers by majority vote
  - N=5 is good baseline; N=10 for critical tasks
  - Improves accuracy 10-20% on reasoning tasks

When to use:
  ✓ Math word problems
  ✓ Logic puzzles
  ✓ Multi-step reasoning
  ✓ Planning tasks
```

---

## Tree of Thoughts (ToT)

```
Template:
  "You are a problem solver. For each step, generate {k} possible
  next thoughts. Evaluate each as likely/impossible. Prune impossible
  branches. Continue until solution is found."

Parameters:
  branching_factor: 3-5
  max_depth: 5-10
  evaluation: "Evaluate this intermediate state as SURE/LIKELY/IMPOSSIBLE"
  search: BFS (breadth) or DFS (depth)

vs CoT:
  - CoT: 1 path, evaluate at end, $ cost
  - ToT: k^d paths, evaluate at each step, $$$ cost
  - Use ToT when CoT consistently fails on complex problems
```

---

## Structured Outputs

```
OpenAI JSON mode:
  response_format = {"type": "json_object"}
  # OR
  response_format = {
    "type": "json_schema",
    "json_schema": { "name": "...", "schema": {...} }
  }

Anthropic structured output (via tools):
  tools = [{"name": "extract", "input_schema": {...}}]
  tool_choice = {"type": "tool", "name": "extract"}

Gemini structured output:
  generation_config = {
    "response_mime_type": "application/json",
    "response_schema": {...}
  }

XML prompting:
  <instruction>...</instruction>
  <input>...</input>
  <output>...</output>

Markdown prompting:
  ## Task
  ## Input
  ## Output Format
  | Col1 | Col2 |

Output contract template:
  """
  Required fields:
    - field_name: type (constraint)
    - field_name: type (constraint)

  Optional fields:
    - field_name: type (default: null)

  Format: JSON
  Rules:
    1. Use null for missing values
    2. Normalize dates to ISO 8601
    3. Output ONLY the JSON object
  """
```

---

## Function Calling

```
Tool definition:
  {
    "type": "function",
    "function": {
      "name": "tool_name",
      "description": "What this tool does",
      "parameters": {
        "type": "object",
        "properties": {
          "param1": {"type": "string", "description": "..."},
          "param2": {"type": "integer"}
        },
        "required": ["param1"]
      }
    }
  }

tool_choice options:
  "auto"       → Model decides
  "required"   → Must call at least one tool
  {"type": "function", "function": {"name": "tool_name"}}
               → Force specific tool

Execution flow:
  1. Send query + tool definitions
  2. Parse tool_call from response
  3. Execute function with parsed args
  4. Send result back as tool message
  5. Get final response

Parallel calls: OpenAI supports multiple tool_calls in one response
```

---

## Role Prompting

```
System message patterns:

Expert:
  "You are a world-class {domain} expert with 20 years of experience."

Teacher:
  "You are a patient teacher explaining {topic} to a beginner."

Assistant:
  "You are a helpful assistant specialized in {task}."

Critic:
  "You are a critical reviewer. Identify flaws and suggest improvements."

Creative:
  "You are a creative {role} known for {style}."

Mix-and-match:
  "You are an expert {domain} specialist who always responds in {format}.
  You prioritize {value} and avoid {anti_value}."
```

---

## Constraint Prompting

```
Length constraints:
  "Respond in exactly {N} sentences."
  "Maximum {N} words."
  "One paragraph only."

Format constraints:
  "Output valid JSON only."
  "Use Markdown tables."
  "Start each line with '- '"

Content constraints:
  "Only use information from the provided text."
  "Never mention {topic}."
  "If unsure, respond with 'I don't know'."

Style constraints:
  "Use formal language."
  "Write at a 6th-grade reading level."
  "Be enthusiastic and encouraging."

Stacking order:
  1. System: identity + format rules
  2. Output contract: exact schema
  3. Constraints: length, tone, content
  4. Example: concrete demonstration
```

---

## ReAct (Reasoning + Acting)

```
Full template:

  "You are an agent that answers questions using tools.

  Available tools:
  - {tool_name}({param}): {description}

  Follow this format for each step:

  Thought: <reasoning about what to do next>
  Action: {tool_name}({args})

  After receiving the observation:
  Thought: <reasoning about the observation>
  Action: {tool_name}({args})

  When you have the answer:
  Final Answer: <answer>

  Question: {question}"

Notes:
  - Always start with Thought
  - Max {N} steps before force-answering
  - Handle tool errors gracefully
  - If stuck, ask user for clarification
```

---

## Meta Prompting

```
Prompt generation:
  "Generate a prompt for {task}. Include:
   1. System message
   2. Clear instructions
   3. {N} few-shot examples
   4. Output format specification
   5. Edge case handling"

Prompt evaluation:
  "Rate this prompt 1-10: {prompt}
   Criteria: clarity, completeness, efficiency, safety
   Suggest specific improvements."

Prompt adaptation:
  "Adapt this {source_model} prompt for {target_model}:
   {prompt}
   Consider: system message format, tool calling, structured output."
```

---

## Parameter Quick Reference

| Parameter | Range | Default | Effect |
|-----------|-------|---------|--------|
| temperature | 0.0 - 2.0 | 1.0 | Randomness. 0 = deterministic |
| top_p | 0.0 - 1.0 | 1.0 | Nucleus sampling. 0.9 = restrict to top 90% |
| max_tokens | 1 - context | varies | Max output length |
| presence_penalty | -2.0 - 2.0 | 0.0 | Penalize repeated topics |
| frequency_penalty | -2.0 - 2.0 | 0.0 | Penalize repeated tokens |
| stop | list of strings | null | Stop generation at these strings |
| seed | int | null | Deterministic generation (if supported) |

**Recommended settings by task:**

| Task | temp | top_p | Notes |
|------|------|-------|-------|
| Classification | 0 | 1 | Deterministic |
| Extraction | 0 | 1 | Deterministic |
| Code generation | 0 - 0.2 | 0.9 | Low creativity |
| Summarization | 0.3 | 0.9 | Some variety |
| Creative writing | 0.7 - 0.9 | 0.95 | High creativity |
| Brainstorming | 0.9 - 1.0 | 1.0 | Maximum variety |
| CoT reasoning | 0 - 0.3 | 1 | Focused reasoning |
| Self-consistency | 0.5 - 0.7 | 1 | Diverse paths |

---

## Token Budget Estimation

```python
# Rough estimates
input_tokens ≈ len(text.split()) * 1.3
output_tokens ≈ len(text.split()) * 1.3
cost = (input_tokens / 1_000_000) * input_rate + (output_tokens / 1_000_000) * output_rate

# GPT-4o pricing (as of 2025):
#   Input: $2.50 / 1M tokens
#   Output: $10.00 / 1M tokens

# Example: 500 token prompt, 100 token output
# Cost = (500/1M * $2.50) + (100/1M * $10.00) = $0.00125 + $0.001 = $0.00225
# At 1000 calls/day: $2.25/day
```

---

## Prompt Anti-Pattern Quick Reference

| Anti-Pattern | Wrong | Right |
|-------------|-------|-------|
| Over-constraining | "In 50 words, JSON, with haiku, formal but accessible" | Limit to 3 constraints max |
| Contradictory | "Be concise. Explain in detail." | "First answer concisely, then explain" |
| Ambiguous pronouns | "Process it and summarize it" | "Process the file. Then summarize the output." |
| Ordering bias | Critical instruction in paragraph 5/10 | Critical instruction first or last |
| Information leakage | Secrets in system prompt | External validation layer |
| Label bias | "Bug" vs "Issue" (loaded labels) | Use neutral, descriptive labels |
| Missing delimiters | "Classify: Great product!" | "Text: Great product!\nLabel:" |

---

## Model-Specific Tips

### OpenAI (GPT-4o, GPT-4o-mini)
- Use `response_format: {"type": "json_object"}` for JSON
- Use `response_format: {"type": "json_schema", ...}` for typed JSON
- System message is reliable for role/constraints
- Supports parallel function calling natively

### Anthropic (Claude 3.5 Sonnet, Opus)
- Use `system` parameter for instructions
- Prefers XML delimiters `<tag>` over Markdown
- Structured output via `tools` with `tool_choice`
- Longer context window (200K tokens)
- Less sensitive to instruction ordering

### Google (Gemini 1.5 Pro, Flash)
- Use `response_mime_type: "application/json"`
- Use `response_schema` for typed output
- Very long context (1M+ tokens)
- System instruction in `system_instruction` parameter
- Lower cost per token

---

## Quick Decision Tree

```
What do you need?
├── Simple answer to known question
│   └── Zero-shot prompt
├── Classification/extraction in new format
│   └── Few-shot prompt (3-5 examples)
├── Multi-step reasoning
│   └── Chain of Thought
│       └── If CoT fails → Tree of Thoughts
├── Machine-readable output
│   └── Structured output (JSON/XML)
├── External tool/data access
│   └── Function calling or ReAct
├── Multi-step agent task
│   └── ReAct (Thought → Action → Observation)
├── Need to iterate quickly
│   └── Meta prompting
└── Production deployment
    └── Prompt templates + testing + monitoring
```
