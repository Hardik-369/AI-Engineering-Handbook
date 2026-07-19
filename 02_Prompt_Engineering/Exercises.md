# Exercises: Prompt Engineering

> 20+ exercises ranging from beginner to advanced. Each exercise specifies the technique, expected outcome, and evaluation criteria.

---

## Beginner Exercises

### Exercise 1: Zero-Shot Classification
**Write a zero-shot prompt** that classifies movie reviews as Positive, Negative, or Mixed.

**Input:** `"The cinematography was stunning but the plot was confusing."`

**Requirements:**
- No examples in the prompt
- Output must be exactly one word
- Test with 5 different reviews on 2 different models

**Evaluation:** Does the model consistently output one of the three labels? Compare results across models.

---

### Exercise 2: Zero-Shot Translation
**Write a zero-shot prompt** that translates text from English to Spanish. The prompt must handle both formal and informal register.

**Requirements:**
- Single prompt, no examples
- User can specify `[formal]` or `[informal]` before the text
- Test with: "Hello, how are you?", "Please send the documents.", "What's up?"

**Evaluation:** Does the model correctly switch between usted/tú based on the register marker?

---

### Exercise 3: Few-Shot Classification
**Write a few-shot prompt** that classifies customer support tickets into: Billing, Technical, Account, or Other.

**Requirements:**
- Include exactly 4 examples (one per category)
- Use `Input:` / `Output:` format
- Test on: "I can't log in", "You charged me twice", "How do I reset my password?"

**Evaluation:** All 3 test cases classified correctly. Try removing one example — does accuracy drop?

---

### Exercise 4: Few-Shot Extraction with Edge Cases
**Write a few-shot prompt** that extracts email addresses, phone numbers, and dates from text.

**Requirements:**
- Include 5 diverse examples covering:
  - Standard email and phone
  - International phone numbers
  - Dates in different formats (MM/DD/YYYY, DD Month YYYY)
  - Missing fields (use null)
- Test on: "Contact John at john@email.com or +44 20 7946 0958 by March 15th 2025"

**Evaluation:** Correctly extracts all three fields with proper formatting.

---

### Exercise 5: Role Prompting Comparison
**Write three versions** of a prompt that explains a complex technical concept (e.g., quantum computing):

1. "You are a PhD physicist explaining to a colleague"
2. "You are a high school science teacher"
3. No role specified

**Evaluation:** Compare the outputs for:
- Vocabulary level (Flesch-Kincaid grade level)
- Length
- Accuracy
- Use of analogies

---

### Exercise 6: Constraint Prompting — Length Control
**Create a prompt** that summarizes news articles in exactly 2 sentences.

**Requirements:**
- Use explicit length constraint
- Test on 3 different news articles
- Measure: how many outputs are exactly 2 sentences?
- Try variations: "exactly 2 sentences" vs "in 2 sentences" vs "2 sentences only"

**Evaluation:** Which phrasing produces the most reliable 2-sentence output?

---

## Intermediate Exercises

### Exercise 7: Chain of Thought — Math Reasoning
**Implement zero-shot CoT** to solve multi-step math word problems.

**Test problems:**
1. "A farmer has 15 chickens. Each chicken lays 2 eggs per day. How many eggs in a week?"
2. "A store sells apples at $0.50 each and oranges at $0.75 each. If John buys 3 apples and 2 oranges with a $5 bill, how much change does he get?"
3. "Train A leaves at 9 AM going 60 mph. Train B leaves at 10 AM going 80 mph. When will Train B catch Train A?"

**Requirements:**
- Must include "Let's think step by step"
- Output must show intermediate calculations
- Test with and without CoT — measure accuracy difference

**Evaluation:** CoT should achieve 100% accuracy; non-CoT may fail on 1-2 problems.

---

### Exercise 8: Manual CoT — Multi-Step Reasoning
**Write a few-shot CoT prompt** that solves logic puzzles:

```
Five people (Alice, Bob, Charlie, Diana, Eve) sit in a row.
Alice sits at one end.
Bob sits next to Charlie.
Diana sits between Alice and Eve.
Who sits in the middle?
```

**Requirements:**
- Include 3 reasoning examples in the prompt
- Each example must show step-by-step reasoning
- Use consistent format: `Reasoning: ... Therefore: ...`
- Test on 3 different logic puzzles

**Evaluation:** All 3 puzzles solved correctly. Compare against zero-shot CoT.

---

### Exercise 9: Self-Consistency with CoT
**Implement self-consistency** for the math problems from Exercise 7.

**Requirements:**
- Run the same CoT prompt 5 times with temperature = 0.7
- Aggregate answers by majority vote
- Compare accuracy of: single CoT, self-consistency (5 runs), self-consistency (10 runs)
- Track: how often does the majority answer differ from single-run answer?

**Evaluation:** Self-consistency should beat single CoT on harder problems.

---

### Exercise 10: Structured Output — JSON Schema
**Create a prompt** that extracts invoice data as structured JSON.

**Input:**
```
Invoice #INV-2025-0042
Date: March 15, 2025
Vendor: Acme Corp
Items:
  - Widget A (x3) @ $10.99 = $32.97
  - Widget B (x1) @ $24.99 = $24.99
Subtotal: $57.96
Tax (8%): $4.64
Total: $62.60
```

**Requirements:**
- Output must match this JSON schema:
```json
{
  "invoice_number": "string",
  "date": "string (ISO format)",
  "vendor": {"name": "string", "address": "string|null"},
  "items": [{"name": "string", "quantity": "number", "unit_price": "number", "total": "number"}],
  "subtotal": "number",
  "tax": {"rate": "number", "amount": "number"},
  "total": "number"
}
```
- Test with OpenAI `response_format`, Anthropic tools, and plain text + instruction
- Measure: parsing success rate across 10 invoices

**Evaluation:** Which method produces the most reliably parsable JSON?

---

### Exercise 11: Function Calling — Weather Agent
**Implement function calling** for a weather information agent.

**Tools to define:**
- `get_forecast(location: string, days: number)`
- `get_current_weather(location: string, units: string)`
- `get_air_quality(location: string)`

**Test queries:**
1. "What's the weather in Tokyo right now?"
2. "Will it rain in London this week?"
3. "What's the air quality in Beijing and the weather in Shanghai?"

**Requirements:**
- Implement both forced tool calls and auto tool calls
- Handle parallel tool calls for query 3
- Handle errors (invalid location, missing data)

**Evaluation:** Correct tool(s) called with correct arguments. Parallel calls for query 3.

---

### Exercise 12: Function Calling — E-Commerce Agent
**Build a function-calling system** for an e-commerce assistant.

**Tools:**
- `search_products(query: string, category: string, max_price: number)`
- `get_product_details(product_id: string)`
- `check_inventory(product_id: string, quantity: number)`
- `calculate_shipping(product_id: string, zip_code: string)`
- `place_order(product_id: string, quantity: number, zip_code: string, payment_method: string)`

**Test queries:**
1. "Find me wireless headphones under $100"
2. "Are the Sony WH-1000XM5 in stock?"
3. "I want to buy 2 pairs of AirPods Pro, ship to 10001"

**Requirements:**
- Multi-turn conversation support
- The agent should ask for missing information before calling `place_order`
- Track conversation state across turns

**Evaluation:** Correct multi-step tool calling with state management.

---

### Exercise 13: Structured Output — XML Extraction
**Create an XML-prompting system** that extracts medical information from clinical notes.

**Input:** "Patient presents with headache for 3 days, fever of 101.2°F, and mild cough. No chest pain. BP 130/85. Prescribed ibuprofen 400mg tid."

**Requirements:**
- Output must be well-formed XML
- Include: symptoms, vital signs, medications, diagnoses
- Handle missing data with empty elements

**Evaluation:** Valid XML output. All present data captured correctly.

---

### Exercise 14: ReAct-Pattern Tool Use
**Implement a ReAct agent** that can answer questions by calling calculator and search tools.

**Available tools:**
- `search(query)` — returns text results
- `calculate(expression)` — returns numeric result
- `current_date()` — returns today's date

**Test queries:**
1. "What was the population of New York City in 2020 divided by 1000?"
2. "If someone was born on January 15, 1990, how old are they today?"
3. "What is 15% of the GDP of France?"

**Requirements:**
- Implement the Thought → Action → Observation → Thought loop
- Handle tool errors gracefully
- Maximum 5 steps before forced answer

**Evaluation:** Correct final answers. Correct intermediate tool calls.

---

## Advanced Exercises

### Exercise 15: Tree of Thoughts Implementation
**Implement a ToT solver** for the Game of 24 (use 4 numbers with arithmetic to make 24).

**Requirements:**
- Use BFS search with branching factor of 3
- Evaluate each intermediate state as: likely/impossible
- Prune impossible branches
- Track: number of branches explored vs CoT baseline

**Evaluation:** Solve 10 Game of 24 puzzles. Compare success rate and steps with CoT.

---

### Exercise 16: Prompt Optimizer
**Build a system** that automatically optimizes prompts using iterative refinement.

**Requirements:**
- Input: initial prompt, test cases with expected outputs
- Generate 5 prompt variants using meta-prompting
- Test variants against test cases
- Select best variant
- Repeat for 3 iterations
- Track: accuracy per iteration, token usage

**Evaluation:** Demonstrate improvement from baseline across 3 iterations.

---

### Exercise 17: DSPy Optimization Pipeline
**Use DSPy** to optimize a classification prompt.

**Requirements:**
- Define a DSPy module for sentiment classification
- Create a training set of 50 examples
- Use BootstrapFewShot optimizer
- Evaluate on 50 test examples
- Compare: hand-written prompt vs DSPy-optimized prompt

**Evaluation:** DSPy-optimized prompt should match or exceed hand-written prompt performance.

---

### Exercise 18: ReAct Agent with Memory
**Build a ReAct agent** that maintains conversation memory across multiple questions.

**Requirements:**
- Must remember facts from earlier in conversation
- Must know when to search vs when to use memory
- Tools: search, memory_store(key, value), memory_recall(key)
- Test: ask 3 sequential questions where later questions depend on earlier answers

**Evaluation:** Correctly recalls facts without re-searching.

---

### Exercise 19: Prompt Compression Benchmark
**Build a benchmark** comparing prompt compression techniques.

**Techniques to compare:**
- LLMLingua compression (50% ratio)
- Manual summarization
- Selective context (retrieve only relevant sections)
- Stopword removal

**Test on:** 10 long prompts (1000+ tokens each)

**Metrics:**
- Compression ratio
- Output quality (compared to uncompressed)
- Latency savings
- Cost savings

**Evaluation:** Report a table comparing all techniques across all metrics.

---

### Exercise 20: Cross-Model Prompt Adaptation
**Write a meta-prompt** that adapts prompts between different LLM providers.

**Requirements:**
- Input: a working OpenAI prompt
- Output: equivalent Anthropic and Gemini prompts
- Must handle differences in: system message, tool calling, structured output
- Test: adapt a function-calling prompt and verify on all three providers

**Evaluation:** Adapted prompts produce equivalent results on each provider.

---

### Exercise 21: Anti-Pattern Detection System
**Build a system** that detects prompt anti-patterns automatically.

**Anti-patterns to detect:**
- Contradictory instructions
- Ambiguous pronouns
- Over-constraining (more than 5 constraints)
- Ordering bias risks

**Requirements:**
- Rule-based detection and LLM-based detection
- Score 0-10 for each anti-pattern
- Generate improvement suggestions

**Evaluation:** Test on 20 prompts (10 with known issues, 10 clean). Precision and recall.

---

### Exercise 22: Production Prompt Testing Framework
**This is the main project.** See [Project.md](./Project.md) for full details.

Build a complete prompt testing framework that:
- Loads prompt templates from files
- Runs prompts against multiple models
- Tests with multiple parameter combinations
- Evaluates outputs against expected results
- Generates comparison reports

**Evaluation:** The framework should identify which prompt variant performs best across models.

---

## Submission Guidelines

For each exercise, submit:
1. The prompt(s) you wrote
2. The model outputs
3. Your analysis (what worked, what didn't, why)
4. Any code used for evaluation

**Bonus:** For exercises 15-22, include visualizations of results (accuracy curves, latency comparisons, etc.).
