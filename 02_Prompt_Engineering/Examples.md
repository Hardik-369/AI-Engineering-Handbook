# Code Examples: Prompt Engineering

> Working, production-ready code examples for every major prompt engineering technique. All examples use real API calls with the official SDKs.

---

## Setup

```python
import os
from openai import OpenAI
from anthropic import Anthropic
import google.generativeai as genai

# Configure clients
openai_client = OpenAI(api_key=os.environ["OPENAI_API_KEY"])
anthropic_client = Anthropic(api_key=os.environ["ANTHROPIC_API_KEY"])
genai.configure(api_key=os.environ["GEMINI_API_KEY"])
```

---

## 1. Zero-Shot Prompting

### OpenAI

```python
def zero_shot_classify(text, categories):
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system",
                "content": f"Classify the text into one of: {', '.join(categories)}. Output only the category name."
            },
            {
                "role": "user",
                "content": text
            }
        ],
        temperature=0
    )
    return response.choices[0].message.content.strip()

# Usage
print(zero_shot_classify(
    "The battery life on this phone is incredible, lasts two days!",
    ["positive", "negative", "neutral"]
))
# Output: positive
```

### Anthropic

```python
def zero_shot_translate(text, target_language):
    response = anthropic_client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=100,
        system="You are a professional translator. Translate accurately and naturally.",
        messages=[{
            "role": "user",
            "content": f"Translate this to {target_language}: {text}"
        }]
    )
    return response.content[0].text

# Usage
print(zero_shot_translate("Hello, how are you?", "French"))
# Output: Bonjour, comment allez-vous ?
```

---

## 2. Few-Shot Prompting

```python
def few_shot_extract(text, examples, schema):
    formatted_examples = "\n\n".join([
        f"Input: {ex['input']}\nOutput: {ex['output']}"
        for ex in examples
    ])

    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system",
                "content": (
                    f"Extract {', '.join(schema.keys())} from the input text. "
                    f"Output as JSON with these exact keys: {', '.join(schema.keys())}. "
                    f"Type hints: {schema}. Use null for missing fields."
                )
            },
            {
                "role": "user",
                "content": f"Examples:\n{formatted_examples}\n\nInput: {text}\nOutput:"
            }
        ],
        response_format={"type": "json_object"},
        temperature=0
    )
    return response.choices[0].message.content

# Example usage
EXAMPLES = [
    {"input": "Call me at 555-1234 or email john@test.com", "output": '{"email": "john@test.com", "phone": "555-1234", "date": null}'},
    {"input": "Meeting on Dec 25, 2024 at 3pm", "output": '{"email": null, "phone": null, "date": "2024-12-25T15:00:00"}'},
    {"input": "No contact info available", "output": '{"email": null, "phone": null, "date": null}'}
]

result = few_shot_extract(
    "Contact Sarah at sarah@company.com or +44 20 7946 0958 before Jan 15th 2025",
    EXAMPLES,
    {"email": "string", "phone": "string", "date": "string (ISO format)"}
)
print(result)
# {"email": "sarah@company.com", "phone": "+44 20 7946 0958", "date": "2025-01-15"}
```

---

## 3. Chain of Thought Reasoning

### Zero-Shot CoT

```python
def zero_shot_cot(question):
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {
                "role": "system",
                "content": "You are a reasoning assistant. Always think step by step."
            },
            {
                "role": "user",
                "content": f"{question}\n\nLet's think step by step."
            }
        ],
        temperature=0
    )
    return response.choices[0].message.content

# Usage
print(zero_shot_cot(
    "A store sells apples at $0.50 each and oranges at $0.75 each. "
    "If John buys 3 apples and 2 oranges with a $5 bill, how much change does he get?"
))
# Output:
# Step 1: Calculate cost of apples: 3 × $0.50 = $1.50
# Step 2: Calculate cost of oranges: 2 × $0.75 = $1.50
# Step 3: Total cost = $1.50 + $1.50 = $3.00
# Step 4: Change = $5.00 - $3.00 = $2.00
# Final answer: John gets $2.00 in change.
```

### Manual Few-Shot CoT

```python
def manual_cot(question):
    examples = """Q: Roger has 5 tennis balls. He buys 2 more cans of tennis balls. Each can has 3 balls. How many tennis balls does he have?
A: Roger starts with 5 balls.
He buys 2 cans, each with 3 balls → 2 × 3 = 6 balls.
Total = 5 + 6 = 11.
The answer is 11.

Q: The cafeteria had 23 apples. They used 20 to make lunch and bought 6 more. How many apples do they have?
A: The cafeteria starts with 23 apples.
They use 20 → 23 - 20 = 3 remaining.
They buy 6 → 3 + 6 = 9.
The answer is 9."""

    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "Solve math problems step by step."},
            {"role": "user", "content": f"{examples}\n\nQ: {question}\nA:"}
        ],
        temperature=0
    )
    return response.choices[0].message.content

print(manual_cot(
    "A farmer has 15 chickens. Each chicken lays 2 eggs per day. "
    "How many eggs in a week? If she sells eggs at $0.25 each, how much does she earn?"
))
```

### Self-Consistency with CoT

```python
import statistics
import json

def self_consistency_cot(question, num_runs=5):
    answers = []
    for _ in range(num_runs):
        response = openai_client.chat.completions.create(
            model="gpt-4o",
            messages=[
                {"role": "system", "content": "Solve step by step. End with: Therefore, the answer is [number]."},
                {"role": "user", "content": f"{question}\n\nLet's think step by step."}
            ],
            temperature=0.7,  # High temp for diverse reasoning paths
        )
        text = response.choices[0].message.content
        # Extract final numeric answer
        import re
        match = re.search(r'Therefore, the answer is\s*(-?\d+\.?\d*)', text)
        if match:
            answers.append(float(match.group(1)))

    if not answers:
        return None, []

    # Majority vote
    from collections import Counter
    counts = Counter(answers)
    most_common = counts.most_common(1)[0][0]
    return most_common, answers

answer, all_answers = self_consistency_cot(
    "A bat and a ball cost $1.10. The bat costs $1.00 more than the ball. "
    "How much does the ball cost?",
    num_runs=5
)
print(f"Voted answer: {answer}")
print(f"All answers: {all_answers}")
# All answers: [0.05, 0.05, 0.10, 0.05, 0.05]
# Voted answer: 0.05
```

---

## 4. Structured Outputs — JSON Mode

### OpenAI response_format

```python
def extract_invoice(text):
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        response_format={
            "type": "json_schema",
            "json_schema": {
                "name": "invoice",
                "schema": {
                    "type": "object",
                    "properties": {
                        "invoice_number": {"type": "string"},
                        "date": {"type": "string", "description": "ISO format date"},
                        "vendor": {
                            "type": "object",
                            "properties": {
                                "name": {"type": "string"},
                                "address": {"type": ["string", "null"]}
                            },
                            "required": ["name", "address"]
                        },
                        "items": {
                            "type": "array",
                            "items": {
                                "type": "object",
                                "properties": {
                                    "name": {"type": "string"},
                                    "quantity": {"type": "integer"},
                                    "unit_price": {"type": "number"},
                                    "total": {"type": "number"}
                                },
                                "required": ["name", "quantity", "unit_price", "total"]
                            }
                        },
                        "subtotal": {"type": "number"},
                        "tax": {
                            "type": "object",
                            "properties": {
                                "rate": {"type": "number"},
                                "amount": {"type": "number"}
                            },
                            "required": ["rate", "amount"]
                        },
                        "total": {"type": "number"}
                    },
                    "required": ["invoice_number", "date", "vendor", "items", "subtotal", "tax", "total"]
                }
            }
        },
        messages=[
            {"role": "system", "content": "Extract invoice data as structured JSON."},
            {"role": "user", "content": text}
        ]
    )
    return json.loads(response.choices[0].message.content)

invoice_text = """Invoice #INV-2025-0042
Date: March 15, 2025
Vendor: Acme Corp, 123 Main St
Items:
  - Widget A (x3) @ $10.99 = $32.97
  - Widget B (x1) @ $24.99 = $24.99
Subtotal: $57.96
Tax (8%): $4.64
Total: $62.60"""

result = extract_invoice(invoice_text)
print(json.dumps(result, indent=2))
```

### Anthropic Structured Output via Tools

```python
def extract_invoice_anthropic(text):
    response = anthropic_client.messages.create(
        model="claude-3-5-sonnet-20241022",
        max_tokens=1000,
        tools=[{
            "name": "extract_invoice",
            "description": "Extract structured invoice data",
            "input_schema": {
                "type": "object",
                "properties": {
                    "invoice_number": {"type": "string"},
                    "date": {"type": "string"},
                    "vendor": {
                        "type": "object",
                        "properties": {
                            "name": {"type": "string"},
                            "address": {"type": ["string", "null"]}
                        },
                        "required": ["name", "address"]
                    },
                    "items": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "name": {"type": "string"},
                                "quantity": {"type": "integer"},
                                "unit_price": {"type": "number"},
                                "total": {"type": "number"}
                            },
                            "required": ["name", "quantity", "unit_price", "total"]
                        }
                    },
                    "subtotal": {"type": "number"},
                    "tax": {
                        "type": "object",
                        "properties": {
                            "rate": {"type": "number"},
                            "amount": {"type": "number"}
                        },
                        "required": ["rate", "amount"]
                    },
                    "total": {"type": "number"}
                },
                "required": ["invoice_number", "date", "vendor", "items", "subtotal", "tax", "total"]
            }
        }],
        messages=[{"role": "user", "content": text}]
    )

    for content in response.content:
        if content.type == "tool_use":
            return content.input
    return None

result = extract_invoice_anthropic(invoice_text)
print(json.dumps(result, indent=2))
```

### Gemini Structured Output

```python
def extract_invoice_gemini(text):
    model = genai.GenerativeModel("gemini-1.5-pro")
    response = model.generate_content(
        f"Extract invoice data from:\n\n{text}",
        generation_config={
            "response_mime_type": "application/json",
            "response_schema": {
                "type": "OBJECT",
                "properties": {
                    "invoice_number": {"type": "STRING"},
                    "date": {"type": "STRING"},
                    "vendor": {
                        "type": "OBJECT",
                        "properties": {
                            "name": {"type": "STRING"},
                            "address": {"type": "STRING"}
                        }
                    },
                    "items": {
                        "type": "ARRAY",
                        "items": {
                            "type": "OBJECT",
                            "properties": {
                                "name": {"type": "STRING"},
                                "quantity": {"type": "INTEGER"},
                                "unit_price": {"type": "NUMBER"},
                                "total": {"type": "NUMBER"}
                            }
                        }
                    },
                    "subtotal": {"type": "NUMBER"},
                    "tax": {"type": "NUMBER"},
                    "total": {"type": "NUMBER"}
                }
            }
        }
    )
    return json.loads(response.text)

result = extract_invoice_gemini(invoice_text)
print(json.dumps(result, indent=2))
```

---

## 5. Function Calling

### Defining Tools and Executing Calls

```python
import json

def get_weather(city, units="celsius"):
    """Simulated weather function."""
    data = {
        "Tokyo": {"celsius": 22, "fahrenheit": 72},
        "Paris": {"celsius": 18, "fahrenheit": 64},
        "New York": {"celsius": 25, "fahrenheit": 77}
    }
    temp = data.get(city, {}).get(units, "unknown")
    return json.dumps({"city": city, "temperature": temp, "units": units})

def get_time(city):
    """Simulated time function."""
    timezones = {"Tokyo": "14:30", "Paris": "07:30", "New York": "01:30"}
    return json.dumps({"city": city, "local_time": timezones.get(city, "unknown")})

TOOLS = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get current weather for a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"},
                    "units": {"type": "string", "enum": ["celsius", "fahrenheit"]}
                },
                "required": ["city"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "get_time",
            "description": "Get current time in a city",
            "parameters": {
                "type": "object",
                "properties": {
                    "city": {"type": "string", "description": "City name"}
                },
                "required": ["city"]
            }
        }
    }
]

AVAILABLE_FUNCTIONS = {
    "get_weather": get_weather,
    "get_time": get_time,
}

def function_calling_loop(user_query):
    messages = [{"role": "user", "content": user_query}]

    while True:
        response = openai_client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=TOOLS,
            tool_choice="auto"
        )

        msg = response.choices[0].message
        messages.append(msg)

        if not msg.tool_calls:
            return msg.content

        for tool_call in msg.tool_calls:
            func = AVAILABLE_FUNCTIONS[tool_call.function.name]
            args = json.loads(tool_call.function.arguments)
            result = func(**args)

            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": result
            })

# Usage
print(function_calling_loop("What's the weather in Tokyo and the time in Paris?"))
```

### Parallel Tool Calls

```python
def parallel_tool_call(user_query):
    messages = [{"role": "user", "content": user_query}]

    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        tools=TOOLS,
        parallel_tool_calls=True
    )

    msg = response.choices[0].message
    print(f"Tool calls received: {len(msg.tool_calls)}")

    results = []
    for tool_call in msg.tool_calls:
        func = AVAILABLE_FUNCTIONS[tool_call.function.name]
        args = json.loads(tool_call.function.arguments)
        result = func(**args)
        results.append({
            "tool_call_id": tool_call.id,
            "function": tool_call.function.name,
            "args": args,
            "result": result
        })
        messages.append({
            "role": "tool",
            "tool_call_id": tool_call.id,
            "content": result
        })

    final = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=messages,
        tools=TOOLS
    )
    return final.choices[0].message.content

print(parallel_tool_call("What's the weather in Tokyo and Paris?"))
```

---

## 6. ReAct Loop Implementation

```python
import json
import re

# Simulated tools
def search(query):
    database = {
        "population of France": "67.97 million",
        "GDP of France": "3.05 trillion USD",
        "capital of France": "Paris",
        "population of Paris": "2.16 million",
        "population of New York City 2020": "8.8 million",
    }
    for key, value in database.items():
        if query.lower() in key.lower():
            return value
    return f"No results found for '{query}'"

def calculate(expression):
    try:
        return str(eval(expression))
    except:
        return f"Error evaluating: {expression}"

TOOLS_DESCRIPTION = """You are an agent that answers questions using tools.

Available tools:
- search(query: str) — Search for information. Returns text results.
- calculate(expression: str) — Evaluate a math expression. Returns numeric result.

For each step, output exactly:
Thought: <your reasoning>
Action: <tool_name>(<args_json>)

When you have enough information, output:
Final Answer: <answer>"""

def react_agent(question, max_steps=5):
    prompt = f"{TOOLS_DESCRIPTION}\n\nQuestion: {question}\n"
    messages = [{"role": "user", "content": prompt}]

    for step in range(max_steps):
        response = openai_client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            temperature=0
        )

        output = response.choices[0].message.content
        print(f"\nStep {step + 1}:\n{output}")
        messages.append({"role": "assistant", "content": output})

        # Check for final answer
        if "Final Answer:" in output:
            return output.split("Final Answer:")[-1].strip()

        # Parse action
        action_match = re.search(r'Action:\s*(\w+)\(([^)]*)\)', output)
        if not action_match:
            messages.append({
                "role": "user",
                "content": "Error: No valid action found. Use format: Action: tool_name(args)"
            })
            continue

        tool_name, args_str = action_match.groups()
        try:
            # Parse args — could be quoted string or JSON
            args_str = args_str.strip()
            if args_str.startswith('"') or args_str.startswith("'"):
                args = args_str[1:-1]  # Remove quotes
            else:
                args = json.loads(f"[{args_str}]")
                args = args[0] if len(args) == 1 else args
        except:
            args = args_str

        # Execute tool
        if tool_name == "search":
            observation = search(args)
        elif tool_name == "calculate":
            observation = calculate(args)
        else:
            observation = f"Unknown tool: {tool_name}"

        print(f"  → Observation: {observation}")
        messages.append({
            "role": "user",
            "content": f"Observation: {observation}\n\nContinue with Thought:"
        })

    return "Max steps reached without final answer."

# Usage
result = react_agent("What is 15% of the population of France?")
print(f"\nFinal: {result}")
```

---

## 7. Prompt Compression with LLMLingua

```python
from llmlingua import PromptCompressor

compressor = PromptCompressor(
    model_name="microsoft/llmlingua-2-xlm-roberta-large-meetingbank",
    use_llm_lingua2=True
)

def compress_and_run(original_prompt, compression_rate=0.5):
    # Compress
    compressed = compressor.compress_prompt(
        original_prompt,
        rate=compression_rate,
        force_tokens=["JSON", "extract", "fields", "null"],
        drop_consecutive=True,
        verbose=True
    )

    print(f"Original: {compressed['origin_tokens']} tokens")
    print(f"Compressed: {compressed['compressed_tokens']} tokens")
    print(f"Ratio: {compressed['rate']:.2%}")

    # Run both
    original_result = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": original_prompt}]
    )

    compressed_result = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": compressed['compressed_prompt']}]
    )

    return {
        "original": original_result.choices[0].message.content,
        "compressed": compressed_result.choices[0].message.content,
        "stats": {
            "original_tokens": compressed['origin_tokens'],
            "compressed_tokens": compressed['compressed_tokens'],
            "ratio": compressed['rate']
        }
    }

# Usage
long_prompt = """You are an expert data extraction assistant. Your task is to extract
structured information from text. Extract the following fields from the provided text:
name, age, email, phone number, address, and company. For each field, provide the value
if present. If a field is not present, set its value to null. Return the result as a JSON
object with exactly these keys: name, age, email, phone, address, company. Only extract
information that is explicitly stated in the text. Do not make assumptions or infer
missing information. Do not fabricate data. Be precise and accurate in your extraction."""

results = compress_and_run(long_prompt, compression_rate=0.4)
print(f"Original quality: {results['original'][:100]}...")
print(f"Compressed quality: {results['compressed'][:100]}...")
```

---

## 8. A/B Prompt Testing

```python
import time
from collections import defaultdict

def ab_test_prompts(prompt_variants, test_cases, model="gpt-4o", num_runs=3):
    results = defaultdict(list)

    for variant_name, prompt_template in prompt_variants.items():
        for test_input, expected in test_cases:
            for run in range(num_runs):
                start = time.time()
                response = openai_client.chat.completions.create(
                    model=model,
                    messages=[
                        {"role": "system", "content": prompt_template},
                        {"role": "user", "content": test_input}
                    ],
                    temperature=0
                )
                latency = time.time() - start
                output = response.choices[0].message.content
                tokens = response.usage.total_tokens

                results[variant_name].append({
                    "input": test_input,
                    "expected": expected,
                    "output": output,
                    "latency": latency,
                    "tokens": tokens,
                    "correct": expected.lower() in output.lower()
                })

    # Summary report
    for variant, runs in results.items():
        accuracy = sum(r["correct"] for r in runs) / len(runs)
        avg_latency = sum(r["latency"] for r in runs) / len(runs)
        avg_tokens = sum(r["tokens"] for r in runs) / len(runs)
        print(f"\n=== {variant} ===")
        print(f"Accuracy: {accuracy:.1%}")
        print(f"Avg latency: {avg_latency:.2f}s")
        print(f"Avg tokens: {avg_tokens:.0f}")

    return results

# Example usage
prompts = {
    "basic": "Classify the sentiment of this text.",
    "role": "You are a sentiment analysis expert. Classify the sentiment.",
    "constrained": "Classify as positive, negative, or neutral. Output one word only."
}

test_cases = [
    ("This product is amazing!", "positive"),
    ("Terrible customer service.", "negative"),
    ("The package arrived on time.", "neutral"),
]

# ab_test_prompts(prompts, test_cases)
```

---

## 9. DSPy Optimization

```python
import dspy
from dspy.teleprompt import BootstrapFewShot
from dspy.evaluate import answer_exact_match

# Configure DSPy
lm = dspy.OpenAI(model="gpt-4o", max_tokens=100)
dspy.settings.configure(lm=lm)

# Define a DSPy module
class SentimentClassifier(dspy.Module):
    def __init__(self):
        self.classify = dspy.ChainOfThought("text -> label")

    def forward(self, text):
        return self.classify(text=text)

# Create training data
trainset = [
    dspy.Example(text="Absolutely love this product!", label="positive").with_inputs("text"),
    dspy.Example(text="Worst purchase ever, completely disappointed", label="negative").with_inputs("text"),
    dspy.Example(text="It works as expected, nothing special", label="neutral").with_inputs("text"),
    dspy.Example(text="The quality exceeded my expectations", label="positive").with_inputs("text"),
    dspy.Example(text="Customer support was unhelpful and rude", label="negative").with_inputs("text"),
]

# Create test data
testset = [
    dspy.Example(text="Best decision I ever made!", label="positive").with_inputs("text"),
    dspy.Example(text="Broken on arrival, waste of money", label="negative").with_inputs("text"),
    dspy.Example(text="It's okay, does the job", label="neutral").with_inputs("text"),
]

# Baseline
baseline = SentimentClassifier()
baseline_score = sum(
    1 for ex in testset if baseline(text=ex.text).label == ex.label
) / len(testset)
print(f"Baseline accuracy: {baseline_score:.1%}")

# Optimize
optimizer = BootstrapFewShot(metric=answer_exact_match, max_bootstrapped_demos=4)
optimized = optimizer.compile(SentimentClassifier(), trainset=trainset)

optimized_score = sum(
    1 for ex in testset if optimized(text=ex.text).label == ex.label
) / len(testset)
print(f"Optimized accuracy: {optimized_score:.1%}")

# Inspect optimized prompt
lm.inspect_history(n=1)
```

---

## 10. Meta-Prompting for Prompt Generation

```python
def generate_prompt(task_description, examples=None):
    meta_prompt = f"""You are a prompt engineering expert. Generate a system prompt
for an LLM to perform the following task:

Task: {task_description}

The prompt should:
1. Set the model's role/expertise
2. Provide clear instructions
3. Specify output format
4. {f'Include these examples: {examples}' if examples else 'Include 2 examples in few-shot format'}
5. Handle edge cases

Return only the prompt text, no explanation."""

    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": meta_prompt}],
        temperature=0.7
    )
    return response.choices[0].message.content

# Usage
generated_prompt = generate_prompt(
    "Extract product names, prices, and ratings from e-commerce product descriptions. "
    "Output as JSON array.",
    examples=[
        "Product: 'Apple iPhone 15 Pro - $999 - 4.5 stars' → {{\"name\": \"iPhone 15 Pro\", \"price\": 999, \"rating\": 4.5}}"
    ]
)
print(generated_prompt)
```

---

## 11. Cross-Provider Structured Output

```python
def extract_entity_cross_provider(text, schema, providers=["openai", "anthropic", "gemini"]):
    results = {}

    if "openai" in providers:
        response = openai_client.chat.completions.create(
            model="gpt-4o",
            response_format={"type": "json_object"},
            messages=[
                {"role": "system", "content": f"Extract data as JSON matching: {json.dumps(schema)}"},
                {"role": "user", "content": text}
            ]
        )
        results["openai"] = json.loads(response.choices[0].message.content)

    if "anthropic" in providers:
        response = anthropic_client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=500,
            tools=[{
                "name": "extract",
                "description": "Extract structured data",
                "input_schema": schema
            }],
            messages=[{"role": "user", "content": text}]
        )
        for content in response.content:
            if content.type == "tool_use":
                results["anthropic"] = content.input

    if "gemini" in providers:
        model = genai.GenerativeModel("gemini-1.5-pro")
        response = model.generate_content(
            f"Extract data from: {text}",
            generation_config={
                "response_mime_type": "application/json",
                "response_schema": schema
            }
        )
        results["gemini"] = json.loads(response.text)

    return results

# Schema definition (works across providers)
schema = {
    "type": "object",
    "properties": {
        "person_name": {"type": "string"},
        "age": {"type": ["integer", "null"]},
        "email": {"type": ["string", "null"]},
        "phone": {"type": ["string", "null"]}
    },
    "required": ["person_name"]
}

# Test
results = extract_entity_cross_provider(
    "John Doe, 32 years old, can be reached at john@email.com or 555-1234",
    schema
)

for provider, data in results.items():
    print(f"{provider}: {json.dumps(data, indent=2)}")
```

---

## Running the Examples

1. **Set environment variables:**
   ```bash
   export OPENAI_API_KEY="sk-..."
   export ANTHROPIC_API_KEY="sk-ant-..."
   export GEMINI_API_KEY="..."
   ```

2. **Install dependencies:**
   ```bash
   pip install openai anthropic google-generativeai dspy-ai llmlingua
   ```

3. **Run any example directly:**
   ```bash
   python -c "from examples import zero_shot_classify; print(zero_shot_classify('Great product!', ['positive', 'negative']))"
   ```

All examples use production API patterns. Adapt the simulated tools (weather, search, calculator) to real APIs for production use.
