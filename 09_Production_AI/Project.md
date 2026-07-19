# Chapter 09: Production AI — Project: Build a Production Monitoring & Evaluation System

## Overview

Build a complete production monitoring and evaluation system for an LLM-powered customer support chatbot. This system will track every aspect of the LLM's behavior in production — latency, cost, quality, safety, and user satisfaction — and provide dashboards, alerting, and automated evaluation pipelines.

This project simulates a real production deployment. You will build the monitoring infrastructure, evaluation framework, guardrails, and incident detection that a real AI engineering team operates daily.

## Prerequisites

- Python 3.10+
- Redis (can use Docker: `docker run -p 6379:6379 redis`)
- Basic familiarity with FastAPI or Flask
- OpenAI API key (or any LLM API)
- Docker and Kubernetes (for optional deployment parts)

## Project Structure

```
production-ai-system/
├── src/
│   ├── api.py                    # FastAPI server for chatbot
│   ├── monitoring.py             # Metrics collection and export
│   ├── evaluation.py             # Evaluation framework
│   ├── guardrails.py             # Input/output guardrails
│   ├── cost_manager.py           # Token budget and cost tracking
│   ├── incident_responder.py     # Alerting and incident detection
│   ├── model_router.py           # Model routing with fallbacks
│   ├── logger.py                 # Structured logging
│   └── config.py                 # Configuration
├── tests/
│   ├── test_monitoring.py
│   ├── test_evaluation.py
│   ├── test_guardrails.py
│   └── test_cost_manager.py
├── data/
│   ├── golden_test_set.json      # Golden test set
│   └── attack_vectors.json       # Known attack patterns
├── dashboards/
│   ├── grafana_dashboard.json    # Grafana dashboard config
│   └── prometheus.yml            # Prometheus config
├── k8s/
│   ├── deployment.yaml
│   └── hpa.yaml
└── scripts/
    ├── run_evaluation.py
    ├── deploy_prompts.py
    └── backup_logs.py
```

## Part 1: Monitoring & Observability (Weight: 20%)

### Goal
Set up comprehensive monitoring that captures every LLM request's latency, tokens, cost, and errors, exposing them as Prometheus metrics with a Grafana dashboard.

### Requirements

1. **Metrics Collection**: Expose Prometheus metrics for:
   - `llm_requests_total` — counter with labels: model, status, user_tier
   - `llm_latency_seconds` — histogram with configurable buckets
   - `llm_tokens_total` — counter with labels: model, type (input/output)
   - `llm_cost_dollars` — counter with labels: model
   - `llm_concurrent_requests` — gauge
   - `llm_guardrail_triggers_total` — counter with labels: guardrail_type

2. **Structured Logging**: Every request log should include request_id, user_id, model, prompt_version, latency_ms, token_counts, cost, cache_hit, and error status.

3. **Distributed Tracing** (bonus): Implement OpenTelemetry tracing for a full request lifecycle:
   - Guardrail check span
   - LLM call span
   - Output validation span

4. **Dashboard**: Create a Grafana dashboard with panels for:
   - Real-time latency (p50/p95/p99 line chart)
   - Request rate (requests/sec)
   - Error rate percentage
   - Cost per hour (stacked by model)
   - Token usage breakdown
   - Cache hit rate gauge
   - Guardrail trigger rate
   - Top 5 users by cost

### Implementation

```python
# src/monitoring.py
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time
from typing import Optional

class MetricsCollector:
    def __init__(self):
        self.requests = Counter(
            'llm_requests_total', 'Total requests',
            ['model', 'status', 'user_tier']
        )
        self.latency = Histogram(
            'llm_latency_seconds', 'Request latency',
            ['model'],
            buckets=(0.1, 0.5, 1.0, 2.0, 5.0, 10.0, 30.0, 60.0)
        )
        self.tokens = Counter(
            'llm_tokens_total', 'Total tokens',
            ['model', 'type']
        )
        self.cost = Counter(
            'llm_cost_dollars', 'Total cost',
            ['model']
        )
        self.concurrency = Gauge(
            'llm_concurrent_requests', 'Current concurrent requests'
        )
        self.guardrails = Counter(
            'llm_guardrail_triggers_total', 'Guardrail triggers',
            ['guardrail_type']
        )

    def observe_request(self, model: str, status: str, user_tier: str,
                        latency_ms: float, input_tokens: int,
                        output_tokens: int, cost: float):
        self.requests.labels(model=model, status=status, user_tier=user_tier).inc()
        self.latency.labels(model=model).observe(latency_ms / 1000)
        self.tokens.labels(model=model, type='input').inc(input_tokens)
        self.tokens.labels(model=model, type='output').inc(output_tokens)
        self.cost.labels(model=model).inc(cost)

# Start metrics server
def start_metrics_server(port: int = 8000):
    start_http_server(port)
```

### Deliverables
- Working Prometheus metrics endpoint
- Grafana dashboard JSON
- Script that sends 100 mock requests and verifies metrics appear

---

## Part 2: Evaluation Framework (Weight: 20%)

### Goal
Build an automated evaluation system that tests every prompt/model change against a golden test set, generating quality scores and detecting regressions.

### Requirements

1. **Golden Test Set**: Create a JSON file with at least 30 test cases covering:
   - Policy questions (return policy, shipping, warranty)
   - Complaint handling
   - Complex multi-part queries
   - Edge cases (empty input, very long input, contradictory requests)
   - Each case has: query, expected_response, category, difficulty, criteria

2. **Automated Evaluators**: Implement at least three evaluators:
   - **BERTScore** evaluator (semantic similarity with expected response)
   - **LLM-as-judge** evaluator that rates responses on correctness, relevance, tone, and completeness
   - **Keyword coverage** evaluator that checks required keywords are present

3. **Evaluation Runner**: Function that:
   - Loads the golden test set
   - Sends each query to the system under test
   - Runs all evaluators on each response
   - Aggregates scores
   - Reports pass/fail based on configurable thresholds
   - Generates an HTML/JSON report

4. **Regression Detection**: Track scores over time and alert if:
   - Any individual score drops > 0.1 below baseline
   - Average score drops > 0.05 below baseline
   - More than 5% of test cases fail

### Implementation

```python
# src/evaluation.py
import json
import numpy as np
from datetime import datetime

class EvalResult:
    def __init__(self):
        self.results = []
        self.baseline = {}

    def evaluate(self, test_cases: list[dict], system_under_test) -> dict:
        for case in test_cases:
            response = system_under_test(case["query"])
            scores = self.compute_scores(response, case)
            self.results.append({
                "query_id": case.get("id"),
                "category": case.get("category"),
                "difficulty": case.get("difficulty"),
                "query": case["query"],
                "response": response,
                "expected": case["expected_response"],
                "scores": scores,
                "passed": all(v >= case.get("threshold", 0.7) for v in scores.values())
            })

        return self.generate_report()

    def compute_scores(self, response: str, expected: dict) -> dict:
        return {
            "bert_score": self.bert_score(response, expected["expected_response"]),
            "judge_score": self.llm_as_judge(expected["query"], response, expected.get("criteria", [])),
            "keyword_coverage": self.keyword_check(response, expected.get("required_keywords", []))
        }

    def generate_report(self) -> dict:
        passed = sum(1 for r in self.results if r["passed"])
        return {
            "timestamp": datetime.utcnow().isoformat(),
            "total": len(self.results),
            "passed": passed,
            "failed": len(self.results) - passed,
            "pass_rate": passed / len(self.results) if self.results else 0,
            "avg_scores": {
                metric: np.mean([r["scores"][metric] for r in self.results])
                for metric in self.results[0]["scores"] if self.results
            },
            "details": self.results,
            "regression_detected": self.check_regression(len(self.results) - passed)
        }

    def check_regression(self, failed_count: int) -> bool:
        fail_rate = failed_count / len(self.results) if self.results else 0
        return fail_rate > 0.05  # More than 5% failure = regression
```

### Deliverables
- Golden test set JSON with 30+ cases
- Working evaluation pipeline
- Report generator producing structured output
- Regression detection with alerts

---

## Part 3: Guardrails & Safety (Weight: 15%)

### Goal
Implement input and output guardrails that protect both the system and the user, with proper logging and alerting on triggers.

### Requirements

1. **Input Guardrails**:
   - Content filter: block toxic/hateful/harassing language using a classification model or API
   - Prompt injection detection: detect at least 10 attack patterns with configurable sensitivity
   - PII redaction: detect and redact emails, phone numbers, credit card numbers, SSNs
   - Length limiting: reject inputs over N tokens (configurable per user tier)
   - Rate limiting: per-user, per-IP, per-endpoint limits

2. **Output Guardrails**:
   - Format validation: ensure JSON responses are valid JSON
   - Content safety: scan for harmful content in outputs
   - PII scanning: check that model didn't leak PII
   - Length enforcement: truncate or reject overly long responses
   - Brand voice check (bonus): check against tone guidelines

3. **Guardrail Manager**: A unified interface that:
   - Runs all guardrails in sequence
   - Short-circuits on critical failures
   - Logs every trigger with request context
   - Reports metrics to monitoring system
   - Supports different policies per user tier

### Implementation

```python
# src/guardrails.py
import re
import hashlib
from typing import Optional

class GuardrailResult:
    def __init__(self, passed: bool, reason: str = None, data: dict = None):
        self.passed = passed
        self.reason = reason
        self.data = data or {}

class GuardrailManager:
    def __init__(self, metrics_collector=None):
        self.metrics = metrics_collector
        self.input_guardrails = []
        self.output_guardrails = []

    def check_input(self, text: str, user_id: str, ip: str,
                    user_tier: str = "standard") -> GuardrailResult:
        # 1. Content filter
        if self.is_toxic(text):
            self.log_trigger("input:content_filter", user_id)
            return GuardrailResult(False, "Content blocked: inappropriate language")

        # 2. Prompt injection detection
        injection_score = self.detect_injection(text)
        if injection_score > 0.7:
            self.log_trigger("input:prompt_injection", user_id)
            return GuardrailResult(False, "Input blocked: potential prompt injection")

        # 3. PII redaction
        redacted, pii_found = self.redact_pii(text)
        if pii_found:
            self.log_trigger("input:pii_redacted", user_id)
            # Continue with redacted text

        # 4. Length limit
        if len(text.split()) > self.get_token_limit(user_tier):
            self.log_trigger("input:too_long", user_id)
            return GuardrailResult(False, f"Input too long (max: {self.get_token_limit(user_tier)} tokens)")

        return GuardrailResult(True, data={"redacted_text": redacted})

    def check_output(self, text: str, user_id: str) -> GuardrailResult:
        # 1. Content safety
        if self.is_toxic(text):
            self.log_trigger("output:unsafe_content", user_id)
            return GuardrailResult(False, "Output blocked: unsafe content")

        # 2. PII leak check
        _, pii_found = self.detect_pii(text)
        if pii_found:
            self.log_trigger("output:pii_leak", user_id)
            return GuardrailResult(False, "Output blocked: PII detected")

        return GuardrailResult(True)

    def detect_injection(self, text: str) -> float:
        patterns = [
            (r"(?i)ignore\s+.*(previous|above|all)\s+(instructions|prompts)", 0.9),
            (r"(?i)you\s+are\s+(free|released|DAN|jailbroken|unrestricted)", 0.95),
            (r"(?i)do\s+not\s+(follow|obey|adhere)", 0.8),
            (r"(?i)pretend.*(no restrictions|unrestricted)", 0.85),
            (r"(?i)system\s+prompt\s*:", 0.7),
            (r"(?i)<\|im_end\|>", 0.6),
            (r"(?:[A-Za-z0-9+/]{4}){10,}={0,2}", 0.5),  # Base64
        ]
        max_score = max((score for _pattern, score in patterns if re.search(_pattern, text)), default=0)
        return min(max_score, 1.0)

    def log_trigger(self, guardrail_type: str, user_id: str):
        if self.metrics:
            self.metrics.guardrails.labels(guardrail_type=guardrail_type).inc()
```

### Deliverables
- Working input guardrails (content filter, injection detection, PII, length)
- Working output guardrails (safety, PII, format)
- Guardrail manager with logging and metrics
- Test suite with 20+ attack vectors

---

## Part 4: Cost Management (Weight: 15%)

### Goal
Build a token budget and cost management system that tracks usage per user, enforces limits, and generates billing reports.

### Requirements

1. **Usage Tracking**: Real-time per-user counters for:
   - Daily and monthly token counts (input/output)
   - Daily and monthly cost
   - Request counts

2. **Budget Enforcement**:
   - Soft limit: warn at 80% of monthly budget
   - Hard limit: block requests at 100% of monthly budget
   - Configurable per user tier
   - Grace period for enterprise customers (3-day overage before block)

3. **Cost Allocation**:
   - Track cost per user, per model, per project
   - Generate billing report at month end
   - Support different pricing tiers

4. **Alerting**:
   - Alert admin when any user exceeds 80% budget
   - Alert on cost anomaly (>2x daily average)
   - Daily cost summary email

### Implementation

```python
# src/cost_manager.py
import redis
import json
from datetime import datetime

class CostManager:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.pricing = {
            "gpt-4": (0.03, 0.06),        # per 1K tokens (input, output)
            "gpt-3.5-turbo": (0.001, 0.002),
            "claude-3-sonnet": (0.003, 0.015),
        }
        self.budgets = {
            "free": {"monthly": 100000, "daily": 10000},       # tokens
            "standard": {"monthly": 1_000_000, "daily": 100000},
            "enterprise": {"monthly": 50_000_000, "daily": 5_000_000},
        }

    def track_usage(self, user_id: str, model: str, input_tokens: int,
                    output_tokens: int, tier: str = "standard"):
        cost = self.calculate_cost(model, input_tokens, output_tokens)
        today = datetime.utcnow().strftime("%Y-%m-%d")
        month = datetime.utcnow().strftime("%Y-%m")
        pipe = self.redis.pipeline()

        # Increment counters
        pipe.hincrby(f"usage:daily:{today}:tokens", user_id, input_tokens + output_tokens)
        pipe.hincrbyfloat(f"usage:daily:{today}:cost", user_id, cost)
        pipe.hincrby(f"usage:monthly:{month}:tokens", user_id, input_tokens + output_tokens)
        pipe.hincrbyfloat(f"usage:monthly:{month}:cost", user_id, cost)
        pipe.hincrby(f"requests:daily:{today}", user_id, 1)
        pipe.execute()

        # Check budget
        total = int(self.redis.hget(f"usage:monthly:{month}:tokens", user_id) or 0)
        budget = self.budgets.get(tier, self.budgets["standard"])["monthly"]
        usage_pct = total / budget

        if usage_pct >= 1.0:
            return "BLOCKED"
        elif usage_pct >= 0.8:
            return "WARN"
        return "OK"

    def calculate_cost(self, model: str, input_tokens: int, output_tokens: int) -> float:
        input_price, output_price = self.pricing.get(model, self.pricing["gpt-3.5-turbo"])
        return (input_tokens / 1000 * input_price) + (output_tokens / 1000 * output_price)

    def generate_billing_report(self, month: str = None) -> dict:
        if not month:
            month = datetime.utcnow().strftime("%Y-%m")

        report = {
            "month": month,
            "users": {},
            "total_cost": 0,
            "total_requests": 0,
            "total_tokens": 0,
        }

        costs = self.redis.hgetall(f"usage:monthly:{month}:cost")
        tokens = self.redis.hgetall(f"usage:monthly:{month}:tokens")
        requests = self.redis.hgetall(f"requests:daily:{month.split('-')[0]}*")

        for user_id in costs:
            user_cost = float(costs[user_id])
            user_tokens = int(tokens.get(user_id, 0))
            report["users"][user_id] = {
                "cost": user_cost,
                "tokens": user_tokens,
            }
            report["total_cost"] += user_cost
            report["total_tokens"] += user_tokens

        return report
```

### Deliverables
- Working usage tracking with Redis
- Budget enforcement (soft + hard limits)
- Billing report generation
- Cost anomaly detection

---

## Part 5: Model Router with Resilience (Weight: 15%)

### Goal
Build a model router that intelligently routes requests, handles failures with circuit breakers, fallback chains, and retry logic.

### Requirements

1. **Model Router**:
   - Route requests to the appropriate model based on query complexity
   - Support configurable routing rules per user tier
   - Measure and track routing decisions

2. **Circuit Breaker**: For each model endpoint:
   - Track consecutive failures
   - Open circuit after N failures
   - Half-open after timeout
   - Track different failure types separately

3. **Fallback Chain**: When primary model fails:
   - Try fallback_1 (cheaper API)
   - Try fallback_2 (different provider)
   - Try emergency (self-hosted, always available)

4. **Retry Logic**:
   - Exponential backoff with jitter
   - Max 3 retries
   - Don't retry on 4xx errors (only 5xx and network errors)

### Implementation

```python
# src/model_router.py
import time
import random
from enum import Enum
from datetime import datetime, timedelta

class CircuitState(Enum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"

class CircuitBreaker:
    def __init__(self, name: str, fail_threshold: int = 5, reset_timeout: int = 60):
        self.name = name
        self.fail_threshold = fail_threshold
        self.reset_timeout = reset_timeout
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.last_failure_time = None

    def call(self, func, *args, **kwargs):
        if self.state == CircuitState.OPEN:
            if datetime.utcnow() - self.last_failure_time > timedelta(seconds=self.reset_timeout):
                self.state = CircuitState.HALF_OPEN
            else:
                raise Exception(f"Circuit {self.name} is OPEN")

        try:
            result = func(*args, **kwargs)
            if self.state == CircuitState.HALF_OPEN:
                self.state = CircuitState.CLOSED
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            self.last_failure_time = datetime.utcnow()
            if self.failure_count >= self.fail_threshold:
                self.state = CircuitState.OPEN
            raise

class ModelRouter:
    def __init__(self):
        self.models = {
            "gpt-4": CircuitBreaker("gpt-4"),
            "gpt-3.5-turbo": CircuitBreaker("gpt-3.5-turbo"),
            "claude-3-haiku": CircuitBreaker("claude-3-haiku"),
        }
        self.fallback_chain = ["gpt-4", "gpt-3.5-turbo", "claude-3-haiku"]

    def call_with_fallback(self, prompt: str, user_tier: str = "standard") -> tuple[str, str]:
        errors = []
        for model in self._get_chain(user_tier):
            try:
                response = self.models[model].call(self._make_api_call, model, prompt)
                return response, model
            except Exception as e:
                errors.append(f"{model}: {str(e)}")
                continue

        raise Exception(f"All models failed: {'; '.join(errors)}")

    def _get_chain(self, user_tier: str) -> list:
        if user_tier == "enterprise":
            return self.fallback_chain
        elif user_tier == "standard":
            return ["gpt-3.5-turbo", "claude-3-haiku"]
        return ["claude-3-haiku"]

    def _make_api_call(self, model: str, prompt: str) -> str:
        # Simulated API call with potential failures
        if random.random() < 0.1:  # 10% failure rate
            raise TimeoutError(f"Model {model} timed out")
        return f"Response from {model}"
```

### Deliverables
- Working model router with configurable chains
- Circuit breaker implementation
- Retry with exponential backoff
- Metrics on routing decisions and failures

---

## Part 6: CI/CD Pipeline (Weight: 10%)

### Goal
Create a script that simulates a CI/CD pipeline: runs evaluation, safety checks, and cost estimation before "deploying" a prompt change.

### Requirements

1. **Pipeline Stages**:
   - Validate golden test set format
   - Run full evaluation suite
   - Run safety scan on all prompts
   - Check latency SLO
   - Estimate cost impact
   - Generate deployment report

2. **Quality Gate**: Block deployment if:
   - Evaluation pass rate < 90%
   - Any safety test fails
   - Latency SLO violation
   - Cost increase > 20%

### Implementation

```python
# scripts/run_evaluation.py
import json
import sys

class CICDPipeline:
    def __init__(self):
        self.stages = []
        self.results = {}

    def run(self) -> bool:
        print("=" * 60)
        print("AI CI/CD Pipeline")
        print("=" * 60)

        # Stage 1: Validate golden set
        print("\n[1/5] Validating golden test set...")
        valid, errors = self.validate_golden_set()
        self.results["golden_set_valid"] = valid
        if not valid:
            print(f"  FAILED: {errors}")
            return False
        print("  PASSED")

        # Stage 2: Run evaluation
        print("\n[2/5] Running evaluation suite...")
        eval_result = self.run_evaluation()
        self.results["evaluation"] = eval_result
        if eval_result["pass_rate"] < 0.9:
            print(f"  FAILED: Pass rate {eval_result['pass_rate']:.0%} < 90%")
            return False
        print(f"  PASSED: {eval_result['pass_rate']:.0%} pass rate")

        # Stage 3: Safety scan
        print("\n[3/5] Running safety scan...")
        safe = self.run_safety_scan()
        self.results["safety_passed"] = safe
        if not safe:
            print("  FAILED: Safety violations detected")
            return False
        print("  PASSED")

        # Stage 4: Latency check
        print("\n[4/5] Checking latency SLO...")
        latency_ok = self.check_latency_slo()
        self.results["latency_slo_ok"] = latency_ok
        if not latency_ok:
            print("  FAILED: Latency SLO violation")
            return False
        print("  PASSED")

        # Stage 5: Cost estimation
        print("\n[5/5] Estimating cost impact...")
        cost_ok = self.estimate_cost()
        self.results["cost_within_limit"] = cost_ok
        if not cost_ok:
            print("  FAILED: Cost increase > 20%")
            return False
        print("  PASSED")

        print("\n" + "=" * 60)
        print("ALL CHECKS PASSED — Ready for deployment")
        print("=" * 60)
        return True
```

### Deliverables
- Pipeline script that runs all stages
- Quality gate implementation
- Deployment report output

---

## Part 7: Incident Detection & Response (Weight: 5%)

### Goal
Implement automated incident detection and response for common production issues.

### Requirements

1. **Anomaly Detection**: Detect:
   - Cost spike (>2x rolling 7-day average)
   - Latency degradation (p99 > 5s for 5+ minutes)
   - Error rate spike (>1% for 5+ minutes)
   - Quality regression (evaluation score drop > 0.1)

2. **Automated Response**: For each incident type:
   - Log the incident
   - Send alert (print to console / webhook)
   - Take mitigation action (rate limit user, failover model, activate kill switch)

3. **Runbook Integration**: Each incident type has a runbook with steps.

### Deliverables
- Anomaly detection module
- Automated mitigation actions
- Incident logging and alerting

---

## Part 8: Integration & Demo (Weight: Bonus)

Connect all components into a working FastAPI server that:

1. Accepts a user query
2. Checks input guardrails
3. Checks budget
4. Routes to LLM (simulated)
5. Checks output guardrails
6. Logs everything
7. Exposes metrics
8. Collects user feedback
9. Runs periodic evaluation

```python
# src/api.py
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import uuid

app = FastAPI()

class QueryRequest(BaseModel):
    user_id: str
    query: str
    user_tier: str = "standard"

class QueryResponse(BaseModel):
    request_id: str
    response: str
    model_used: str
    latency_ms: float
    cost: float

@app.post("/chat", response_model=QueryResponse)
async def chat(request: QueryRequest):
    request_id = str(uuid.uuid4())

    # 1. Input guardrails
    guardrail_result = guardrails.check_input(request.query, request.user_id)
    if not guardrail_result.passed:
        raise HTTPException(status_code=403, detail=guardrail_result.reason)

    # 2. Budget check
    budget_status = cost_mgr.track_usage(request.user_id, ...)
    if budget_status == "BLOCKED":
        raise HTTPException(status_code=429, detail="Budget exceeded")

    # 3. Route to model
    response, model_used = router.call_with_fallback(request.query, request.user_tier)

    # 4. Output guardrails
    output_result = guardrails.check_output(response, request.user_id)
    if not output_result.passed:
        response = "I'm unable to provide that response."

    # 5. Record metrics
    metrics.observe_request(...)

    return QueryResponse(
        request_id=request_id,
        response=response,
        model_used=model_used,
        latency_ms=latency,
        cost=cost
    )
```

## Evaluation Criteria

Your project will be evaluated on:

| Criterion | Weight | Description |
|-----------|--------|-------------|
| Monitoring | 20% | Prometheus metrics, structured logging, dashboard |
| Evaluation | 20% | Golden test set, automated evaluators, regression detection |
| Guardrails | 15% | Input/output guardrails, injection detection |
| Cost Management | 15% | Tracking, budgets, billing reports |
| Model Router | 15% | Circuit breakers, fallbacks, retries |
| CI/CD Pipeline | 10% | Evaluation pipeline, quality gates |
| Incident Response | 5% | Anomaly detection, automated mitigation |
| Code Quality | Bonus | Clean code, tests, documentation |

## Submission

Submit a GitHub repository with:
- All source code
- Golden test set JSON
- Grafana dashboard JSON
- Tests in `tests/`
- `README.md` explaining architecture and how to run
- A short demo video or screenshots showing the monitoring dashboard

## Extension Ideas

- Add a web UI showing real-time metrics
- Implement semantic caching
- Add support for multiple LLM providers
- Implement shadow deployment mode
- Add user feedback visualization
- Create a mobile alert integration (Slack, PagerDuty)
- Implement automated A/B testing between prompt versions
