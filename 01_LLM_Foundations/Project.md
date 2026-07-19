# Project: Build a Smart Tokenizer CLI

Build a command-line tool that takes text input and provides detailed tokenization analysis. This project applies everything from Chapter 01: tokenization, BPE algorithms, context window management, cost estimation, and API integration.

---

## Learning Objectives

- Understand how BPE tokenizers work in practice
- Learn to estimate and manage token budgets
- Build a production-quality CLI tool
- Integrate with multiple AI providers' tokenizers
- Visualize token-level information

---

## Specification

### Smart Tokenizer CLI (`tokenize-cli`)

**Usage:**

```bash
# Basic token counting
python tokenize.py "Hello, world!"

# Token analysis with visualization
python tokenize.py "Hello, world!" --verbose

# Read from file
python tokenize.py --file input.txt

# Interactive mode
python tokenize.py --interactive

# Specific model tokenizer
python tokenize.py "Hello, world!" --model gpt-4o

# Cost estimation
python tokenize.py "Hello, world!" --cost

# JSON output
python tokenize.py "Hello, world!" --format json

# Compare across models
python tokenize.py "Hello, world!" --compare
```

### Features

#### 1. Token Counting
- Count tokens using the specified model's tokenizer
- Show character count, token count, and chars-per-token ratio
- Estimate tokens for different models

#### 2. Token Visualization
- Show how text is split into individual tokens
- Display each token in brackets with its token ID
- Highlight special tokens and whitespace
- Show byte-level representation for non-printable tokens

#### 3. Cost Estimation
- Estimate API cost based on current model pricing
- Show input cost and estimated output cost
- Support batch cost calculations
- Monthly projection at given query volume

#### 4. Multi-Model Support
- OpenAI models (GPT-4o, GPT-4o-mini, o3, etc.)
- Anthropic models (Claude 3.5, Claude 4)
- Open-source models via HuggingFace tokenizers
- Custom tokenizer files

#### 5. Context Window Analysis
- Show how much of the context window is used
- Warn if approaching the context limit
- Suggest truncation strategies

---

## Implementation Plan

### Phase 1: Core Tokenization (Required)

**File: `tokenize.py`**

```python
#!/usr/bin/env python3
"""
Smart Tokenizer CLI — Analyze how LLMs tokenize your text.

Usage:
    python tokenize.py "Your text here"
    python tokenize.py --file input.txt
    python tokenize.py --interactive
"""

import argparse
import sys
import json
from pathlib import Path

# Try importing optional dependencies
try:
    import tiktoken
except ImportError:
    tiktoken = None

try:
    from transformers import AutoTokenizer
except ImportError:
    AutoTokenizer = None


# ─── Tokenizers ────────────────────────────────────────────────────────────

class TokenizerBase:
    """Base class for tokenizer wrappers."""
    
    def encode(self, text: str) -> list[int]:
        raise NotImplementedError
    
    def decode(self, token_ids: list[int]) -> str:
        raise NotImplementedError
    
    def token_count(self, text: str) -> int:
        return len(self.encode(text))
    
    def get_token_info(self, text: str) -> list[dict]:
        """Return list of {id, token_str, bytes, is_special} for each token."""
        token_ids = self.encode(text)
        tokens = []
        for tid in token_ids:
            token_str = self.decode([tid])
            tokens.append({
                "id": tid,
                "token_str": token_str,
                "bytes": list(token_str.encode("utf-8")),
                "is_special": tid >= 100_000,  # heuristic
            })
        return tokens


class OpenAITokenizer(TokenizerBase):
    def __init__(self, model="gpt-4o"):
        if tiktoken is None:
            raise ImportError("tiktoken is required for OpenAI tokenizers")
        self.model = model
        self.encoding = tiktoken.encoding_for_model(model)
    
    def encode(self, text):
        return self.encoding.encode(text)
    
    def decode(self, token_ids):
        return self.encoding.decode(token_ids)


class HuggingFaceTokenizer(TokenizerBase):
    def __init__(self, model_name):
        if AutoTokenizer is None:
            raise ImportError("transformers is required for HuggingFace tokenizers")
        self.model_name = model_name
        self.tokenizer = AutoTokenizer.from_pretrained(model_name)
    
    def encode(self, text):
        return self.tokenizer.encode(text)
    
    def decode(self, token_ids):
        return self.tokenizer.decode(token_ids)


# ─── Analysis ──────────────────────────────────────────────────────────────

MODEL_CONFIGS = {
    "gpt-4o": {
        "tokenizer_class": OpenAITokenizer,
        "context_window": 128_000,
        "pricing": {"input": 2.50, "output": 10.00},
    },
    "gpt-4o-mini": {
        "tokenizer_class": OpenAITokenizer,
        "context_window": 128_000,
        "pricing": {"input": 0.15, "output": 0.60},
    },
    "o3": {
        "tokenizer_class": OpenAITokenizer,
        "context_window": 200_000,
        "pricing": {"input": 10.00, "output": 40.00},
    },
    "claude-3-5-sonnet": {
        "tokenizer_class": OpenAITokenizer,  # Uses cl100k_base
        "context_window": 200_000,
        "pricing": {"input": 3.00, "output": 15.00},
    },
}


class TokenAnalyzer:
    def __init__(self, model="gpt-4o"):
        self.model = model
        config = MODEL_CONFIGS.get(model)
        if not config:
            raise ValueError(f"Unknown model: {model}")
        
        self.tokenizer = config["tokenizer_class"](model)
        self.context_window = config["context_window"]
        self.pricing = config["pricing"]
    
    def analyze(self, text):
        """Full token analysis of input text."""
        tokens = self.tokenizer.encode(text)
        token_info = self.tokenizer.get_token_info(text)
        
        return {
            "text": text,
            "char_count": len(text),
            "token_count": len(tokens),
            "chars_per_token": round(len(text) / max(len(tokens), 1), 2),
            "context_used_pct": round(len(tokens) / self.context_window * 100, 2),
            "tokens": token_info[:20],  # Show first 20
            "token_preview": "...",  # Truncation indicator
        }
    
    def estimate_cost(self, text, output_tokens=500):
        """Estimate API call cost."""
        input_tokens = len(self.tokenizer.encode(text))
        cost = (input_tokens / 1_000_000 * self.pricing["input"] +
                output_tokens / 1_000_000 * self.pricing["output"])
        
        return {
            "model": self.model,
            "input_tokens": input_tokens,
            "estimated_output_tokens": output_tokens,
            "total_tokens": input_tokens + output_tokens,
            "input_cost": round(input_tokens / 1_000_000 * self.pricing["input"], 6),
            "output_cost": round(output_tokens / 1_000_000 * self.pricing["output"], 6),
            "total_cost": round(cost, 6),
        }


# ─── Display ───────────────────────────────────────────────────────────────

def format_analysis(analysis):
    """Pretty-print token analysis."""
    lines = []
    lines.append("─" * 50)
    lines.append("Token Analysis")
    lines.append("─" * 50)
    lines.append(f"  Characters:    {analysis['char_count']}")
    lines.append(f"  Tokens:        {analysis['token_count']}")
    lines.append(f"  Chars/Token:   {analysis['chars_per_token']}")
    lines.append(f"  Context Used:  {analysis['context_used_pct']}%")
    lines.append("")
    
    if analysis['token_count'] > 0:
        lines.append("  Token Breakdown:")
        for t in analysis['tokens']:
            display_str = repr(t['token_str'])[1:-1]  # Remove quotes
            if not display_str:
                display_str = "␣"  # Space placeholder
            lines.append(f"    ID {t['id']:6d} → '{display_str}'")
    
    if analysis['token_count'] > 20:
        lines.append(f"    ... and {analysis['token_count'] - 20} more tokens")
    
    lines.append("─" * 50)
    return "\n".join(lines)


def format_cost(cost_estimate):
    """Pretty-print cost estimate."""
    lines = []
    lines.append("Cost Estimate")
    lines.append(f"  Model:              {cost_estimate['model']}")
    lines.append(f"  Input tokens:       {cost_estimate['input_tokens']}")
    lines.append(f"  Output tokens:      {cost_estimate['estimated_output_tokens']} (est.)")
    lines.append(f"  Input cost:         ${cost_estimate['input_cost']:.6f}")
    lines.append(f"  Output cost:        ${cost_estimate['output_cost']:.6f}")
    lines.append(f"  Total cost:         ${cost_estimate['total_cost']:.6f}")
    
    # Per-1M token reference
    lines.append("")
    lines.append(f"  (Per 1M tokens: ${cost_estimate['total_cost'] / cost_estimate['total_tokens'] * 1_000_000:.2f})")
    
    return "\n".join(lines)


# ─── CLI ───────────────────────────────────────────────────────────────────

def parse_args():
    parser = argparse.ArgumentParser(
        description="Smart Tokenizer CLI — Analyze how LLMs tokenize your text.",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  python tokenize.py "Hello, world!"
  python tokenize.py --file document.txt --model gpt-4o-mini
  python tokenize.py "Your text" --cost --verbose
  python tokenize.py --interactive --compare
  python tokenize.py "Your text" --format json
        """,
    )
    
    input_group = parser.add_mutually_exclusive_group(required=True)
    input_group.add_argument("text", nargs="?", help="Text to tokenize")
    input_group.add_argument("--file", "-f", type=str, help="Read text from file")
    input_group.add_argument("--interactive", "-i", action="store_true",
                             help="Interactive mode")
    
    parser.add_argument("--model", "-m", default="gpt-4o",
                        choices=list(MODEL_CONFIGS.keys()),
                        help="Model to use for tokenization (default: gpt-4o)")
    parser.add_argument("--cost", "-c", action="store_true",
                        help="Show cost estimate")
    parser.add_argument("--output-tokens", type=int, default=500,
                        help="Estimated output tokens for cost (default: 500)")
    parser.add_argument("--verbose", "-v", action="store_true",
                        help="Show detailed token breakdown")
    parser.add_argument("--format", "-F", choices=["text", "json"], default="text",
                        help="Output format (default: text)")
    parser.add_argument("--compare", action="store_true",
                        help="Compare tokenization across all models")
    parser.add_argument("--output", "-o", type=str,
                        help="Write output to file")
    
    return parser.parse_args()


def get_input_text(args):
    """Get input text from args, file, or interactive mode."""
    if args.text:
        return args.text
    elif args.file:
        path = Path(args.file)
        if not path.exists():
            print(f"Error: File not found: {args.file}", file=sys.stderr)
            sys.exit(1)
        return path.read_text(encoding="utf-8")
    elif args.interactive:
        print("Smart Tokenizer CLI — Interactive Mode")
        print("Type your text (Ctrl+D or Ctrl+Z to exit):")
        return sys.stdin.read().strip()


def interactive_mode():
    """Run interactive tokenization session."""
    print("\nSmart Tokenizer CLI — Interactive Mode")
    print("=" * 50)
    print("Commands: !model NAME, !cost, !compare, !help, !quit")
    print("=" * 50)
    
    current_model = "gpt-4o"
    
    while True:
        try:
            text = input("\n> ").strip()
        except (EOFError, KeyboardInterrupt):
            print("\nGoodbye!")
            break
        
        if not text:
            continue
        
        if text.startswith("!"):
            cmd = text[1:].split()
            if cmd[0] == "quit":
                break
            elif cmd[0] == "model" and len(cmd) > 1:
                current_model = cmd[1] if cmd[1] in MODEL_CONFIGS else "gpt-4o"
                print(f"  Model set to: {current_model}")
            elif cmd[0] == "cost":
                text_to_analyze = " ".join(cmd[1:]) if len(cmd) > 1 else input("  Text: ")
                analyzer = TokenAnalyzer(current_model)
                cost = analyzer.estimate_cost(text_to_analyze)
                print(format_cost(cost))
            elif cmd[0] == "compare":
                for model in MODEL_CONFIGS:
                    try:
                        analyzer = TokenAnalyzer(model)
                        analysis = analyzer.analyze(text)
                        print(f"\n  [{model}] {analysis['token_count']} tokens "
                              f"({analysis['chars_per_token']:.2f} cpt, "
                              f"{analysis['context_used_pct']:.1f}% context)")
                    except Exception as e:
                        print(f"  [{model}] Error: {e}")
            elif cmd[0] == "help":
                print("  Commands: !model NAME, !cost, !compare, !help, !quit")
            else:
                print(f"  Unknown command: {cmd[0]}")
            continue
        
        # Analyze the input
        analyzer = TokenAnalyzer(current_model)
        analysis = analyzer.analyze(text)
        
        print()
        print(format_analysis(analysis))


def main():
    args = parse_args()
    
    # If interactive, run interactive mode
    if args.interactive:
        interactive_mode()
        return
    
    # Get input text
    text = get_input_text(args)
    if not text:
        print("Error: No input text provided.", file=sys.stderr)
        sys.exit(1)
    
    # Compare across multiple models
    if args.compare:
        models_to_check = list(MODEL_CONFIGS.keys())
        results = {}
        for model in models_to_check:
            try:
                analyzer = TokenAnalyzer(model)
                analysis = analyzer.analyze(text)
                results[model] = analysis
                if args.format == "json":
                    continue
                print(f"\n[{model}]")
                print(f"  Tokens: {analysis['token_count']} | "
                      f"Chars/Token: {analysis['chars_per_token']:.2f} | "
                      f"Context: {analysis['context_used_pct']:.1f}%")
            except Exception as e:
                results[model] = {"error": str(e)}
                print(f"[{model}] Error: {e}")
        
        if args.format == "json":
            print(json.dumps(results, indent=2))
        return
    
    # Single model analysis
    analyzer = TokenAnalyzer(args.model)
    analysis = analyzer.analyze(text)
    
    if args.format == "json":
        output = json.dumps(analysis, indent=2)
        print(output)
    else:
        print(format_analysis(analysis))
    
    # Cost estimate
    if args.cost:
        cost = analyzer.estimate_cost(text, args.output_tokens)
        print()
        print(format_cost(cost))
    
    # Write to output file
    if args.output:
        output_content = json.dumps(analysis, indent=2) if args.format == "json" else format_analysis(analysis)
        Path(args.output).write_text(output_content, encoding="utf-8")
        print(f"\nOutput written to: {args.output}")


if __name__ == "__main__":
    main()
```

---

### Phase 2: Enhanced Visualization (Optional but Recommended)

Add an ASCII visualization of token boundaries:

```python
def visualize_tokens(text, tokenizer, width=60):
    """Show token boundaries with visual markers."""
    token_info = tokenizer.get_token_info(text)
    
    # Build a visual line showing token boundaries
    visual = ""
    for t in token_info:
        token_len = len(t['token_str'])
        if token_len == 0:
            continue
        # Token boundary marker
        visual += "|"
        # Token content (truncated for display)
        display = t['token_str'].replace("\n", "\\n").replace("\t", "\\t")
        visual += display
    
    visual += "|"
    
    # Color-coded (using ANSI for terminal)
    colors = ["\033[31m", "\033[32m", "\033[33m", "\033[34m", "\033[35m", "\033[36m"]
    reset = "\033[0m"
    
    print("\nToken Visualization:")
    print("-" * width)
    
    colored = ""
    for i, t in enumerate(token_info):
        token_len = len(t['token_str'])
        if token_len == 0:
            continue
        display = t['token_str'].replace("\n", "\\n").replace("\t", "\\t")
        if not display:
            display = "·"
        color = colors[i % len(colors)]
        colored += f"{color}{display}{reset}|"
    
    print(colored)
    print("-" * width)
    
    # Legend showing token-to-ID mapping
    print("\nToken Legend:")
    for i, t in enumerate(token_info[:15]):
        color = colors[i % len(colors)]
        display = t['token_str'].replace("\n", "\\n").replace("\t", "\\t")
        if not display:
            display = "␣"
        print(f"  {color}[{t['id']:5d}]{reset} → '{display}'")
```

### Phase 3: Budget Planning (Optional)

Add a budget planner for production cost estimation:

```python
def budget_planner():
    """Interactive budget planning tool."""
    print("\nToken Budget Planner")
    print("=" * 50)
    
    queries_per_day = int(input("Queries per day: "))
    avg_input_tokens = int(input("Average input tokens per query: "))
    avg_output_tokens = int(input("Average output tokens per query: "))
    
    print("\nModel Options:")
    models = list(MODEL_CONFIGS.keys())
    for i, m in enumerate(models):
        print(f"  [{i}] {m}")
    
    model_idx = int(input("Select model: "))
    model = models[model_idx]
    
    daily_input = queries_per_day * avg_input_tokens
    daily_output = queries_per_day * avg_output_tokens
    
    pricing = MODEL_CONFIGS[model]["pricing"]
    daily_cost = (daily_input / 1_000_000 * pricing["input"] +
                  daily_output / 1_000_000 * pricing["output"])
    
    print(f"\n{'─' * 50}")
    print(f"Budget Projection for {model}")
    print(f"{'─' * 50}")
    print(f"  Daily queries:     {queries_per_day}")
    print(f"  Daily tokens in:   {daily_input:>10,}  (${daily_input/1_000_000*pricing['input']:.2f})")
    print(f"  Daily tokens out:  {daily_output:>10,}  (${daily_output/1_000_000*pricing['output']:.2f})")
    print(f"  Daily cost:        ${daily_cost:.2f}")
    print(f"  Weekly cost:       ${daily_cost * 7:.2f}")
    print(f"  Monthly cost:      ${daily_cost * 30:.2f}")
    print(f"  Yearly cost:       ${daily_cost * 365:.2f}")
```

---

## Testing

```python
# test_tokenize.py
import pytest
import json
from tokenize import TokenAnalyzer, format_analysis, format_cost

class TestTokenAnalyzer:
    def setup_method(self):
        self.analyzer = TokenAnalyzer("gpt-4o-mini")
    
    def test_empty_string(self):
        analysis = self.analyzer.analyze("")
        assert analysis["token_count"] == 0
        assert analysis["char_count"] == 0
    
    def test_simple_text(self):
        analysis = self.analyzer.analyze("Hello, world!")
        assert analysis["token_count"] > 0
        assert analysis["char_count"] == 13
    
    def test_long_text(self):
        analysis = self.analyzer.analyze("Hello " * 1000)
        assert analysis["token_count"] > 100
    
    def test_cost_estimation(self):
        cost = self.analyzer.estimate_cost("Test", output_tokens=100)
        assert cost["total_cost"] > 0
        assert cost["input_tokens"] > 0
        assert cost["model"] == "gpt-4o-mini"
    
    def test_token_info(self):
        text = "AI"
        info = self.analyzer.analyze(text)
        assert len(info["tokens"]) > 0
        for t in info["tokens"]:
            assert "id" in t
            assert "token_str" in t
    
    def test_model_switch(self):
        analyzer = TokenAnalyzer("gpt-4o")
        assert analyzer.model == "gpt-4o"
        assert analyzer.context_window == 128000
    
    def test_multiple_models_different_counts(self):
        text = "The quick brown fox jumps over the lazy dog."
        counts = {}
        for model in ["gpt-4o-mini", "gpt-4o"]:
            analyzer = TokenAnalyzer(model)
            counts[model] = analyzer.analyze(text)["token_count"]
        # Different models may have slightly different token counts
        print(f"Token counts: {counts}")


class TestOutputFormatting:
    def test_format_analysis(self):
        analyzer = TokenAnalyzer("gpt-4o-mini")
        analysis = analyzer.analyze("Hello")
        output = format_analysis(analysis)
        assert "Token Analysis" in output
        assert "Characters:" in output
        assert "Tokens:" in output
    
    def test_format_cost(self):
        analyzer = TokenAnalyzer("gpt-4o-mini")
        cost = analyzer.estimate_cost("Hello", 100)
        output = format_cost(cost)
        assert "Cost Estimate" in output
        assert "Total cost:" in output
```

---

## Extension Ideas

| Feature | Difficulty | Description |
|---------|------------|-------------|
| Token heatmap | Medium | Color-code tokens by frequency in the model's vocabulary |
| Cross-lingual comparison | Medium | Compare token efficiency across languages |
| Batch mode | Easy | Process multiple texts from a CSV file |
| API server | Hard | Serve tokenization as a REST API |
| Token-level cost optimization | Medium | Suggest rewrites to reduce token count |
| Prompt compression | Hard | Suggest token-efficient alternatives for verbose prompts |
| Vocabulary browser | Medium | Browse the model's full token vocabulary |
| Stream mode | Medium | Tokenize streaming input in real-time |

---

## Deliverables

1. **`tokenize.py`** — Working CLI tool implementing Phases 1-3
2. **`test_tokenize.py`** — Unit tests with >90% coverage
3. **`README.md`** (in project directory) — Installation, usage examples, and API documentation

### Evaluation Criteria

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Correctness | 30% | Accurate token counts and cost estimates |
| Code quality | 25% | Clean, modular, well-typed code |
| UX | 20% | Clear output, helpful error messages, good CLI ergonomics |
| Testing | 15% | Comprehensive test coverage |
| Extensions | 10% | Bonus features (visualization, compare mode, budget planner) |

---

## Submission

Zip the project directory and submit. Include a `requirements.txt`:

```
tiktoken>=0.7.0
transformers>=4.38.0  # optional, for HuggingFace support
```

---

## Hints

1. **Start with a single model** (GPT-4o-mini) and add others later
2. **Use `tiktoken`** — it's faster and simpler than HF tokenizers
3. **Handle edge cases**: empty strings, very long texts, non-ASCII, binary data
4. **Test with real API pricing** — the numbers matter in production
5. **Use `argparse`** for clean CLI argument handling
6. **For interactive mode**, prompt with `input()` and check for command prefixes
