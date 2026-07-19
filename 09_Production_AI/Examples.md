# Chapter 09: Production AI — Examples

---

## 1. Monitoring Setup — Complete Observability Stack

### Example: LangFuse + OpenTelemetry Monitoring

```python
# monitoring_setup.py
import os
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from langfuse import Langfuse
from langfuse.openai import openai  # Monkey-patched OpenAI

# Initialize LangFuse
langfuse = Langfuse(
    public_key=os.environ["LANGFUSE_PUBLIC_KEY"],
    secret_key=os.environ["LANGFUSE_SECRET_KEY"],
    host="https://cloud.langfuse.com"
)

# Initialize OpenTelemetry
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)

tracer = trace.get_tracer(__name__)

@tracer.start_as_current_span("chat_completion")
def chat_with_monitoring(user_input: str, user_id: str) -> str:
    span = trace.get_current_span()
    span.set_attribute("user_id", user_id)
    span.set_attribute("input_length", len(user_input))

    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[{"role": "user", "content": user_input}],
        user=user_id,
        temperature=0.7
    )

    output = response.choices[0].message.content
    span.set_attribute("output_tokens", response.usage.completion_tokens)
    span.set_attribute("total_tokens", response.usage.total_tokens)
    span.set_attribute("latency_sla_met", response.response_ms < 5000)

    # Send to LangFuse
    langfuse.trace(
        name="chat",
        input=user_input,
        output=output,
        metadata={
            "model": "gpt-4",
            "tokens": response.usage.total_tokens,
            "latency_ms": response.response_ms,
            "user_id": user_id
        }
    )

    return output

# Usage
if __name__ == "__main__":
    result = chat_with_monitoring(
        "What is the return policy?",
        "user_abc123"
    )
    print(result)
```

### Example: Helicone Proxy Setup

```python
# helicone_monitoring.py
import openai
import os

# Configure OpenAI to use Helicone as proxy
client = openai.OpenAI(
    api_key=os.environ["OPENAI_API_KEY"],
    base_url="https://oai.hconeai.com/v1",
    default_headers={
        "Helicone-Auth": f"Bearer {os.environ['HELICONE_API_KEY']}",
        "Helicone-User-Id": "user_abc123",
        "Helicone-Property-App": "customer-support",
        "Helicone-Cache-Enabled": "true",
        "Helicone-RateLimit-Policy": "100;w=60",  # 100 req/min
    }
)

response = client.chat.completions.create(
    model="gpt-4",
    messages=[{"role": "user", "content": "Explain quantum computing"}],
    temperature=0.7,
    max_tokens=500
)

# Helicone automatically captures:
# - Request/response payloads
# - Latency
# - Token counts
# - Cost (estimated)
# - User identity
# - Cache hits/misses
```

### Example: Custom Prometheus Metrics

```python
# prometheus_metrics.py
from prometheus_client import Counter, Histogram, Gauge, start_http_server
import time
import random

# Define metrics
LLM_REQUESTS = Counter(
    'llm_requests_total',
    'Total LLM requests',
    ['model', 'status']  # status: success, error, blocked
)

LLM_LATENCY = Histogram(
    'llm_latency_seconds',
    'LLM request latency',
    ['model'],
    buckets=[0.1, 0.5, 1.0, 2.0, 5.0, 10.0, 30.0]
)

LLM_TOKENS = Counter(
    'llm_tokens_total',
    'Total tokens consumed',
    ['model', 'type']  # type: input, output, total
)

LLM_COST = Counter(
    'llm_cost_dollars',
    'Total cost in USD',
    ['model']
)

CURRENT_CONCURRENCY = Gauge(
    'llm_concurrent_requests',
    'Current number of concurrent LLM requests'
)

MODEL_PRICES = {
    "gpt-4": {"input": 0.03, "output": 0.06},
    "gpt-3.5-turbo": {"input": 0.001, "output": 0.002},
    "claude-3-opus": {"input": 0.015, "output": 0.075},
}

@LLM_LATENCY.time()
def tracked_llm_call(model: str, prompt: str) -> str:
    CURRENT_CONCURRENCY.inc()
    try:
        start = time.time()
        # Actual API call
        response = call_model(model, prompt)
        elapsed = time.time() - start

        # Record metrics
        LLM_REQUESTS.labels(model=model, status="success").inc()
        LLM_TOKENS.labels(model=model, type="input").inc(response.usage.prompt_tokens)
        LLM_TOKENS.labels(model=model, type="output").inc(response.usage.completion_tokens)
        cost = (response.usage.prompt_tokens / 1000 * MODEL_PRICES[model]["input"] +
                response.usage.completion_tokens / 1000 * MODEL_PRICES[model]["output"])
        LLM_COST.labels(model=model).inc(cost)

        return response.choices[0].message.content
    except Exception as e:
        LLM_REQUESTS.labels(model=model, status="error").inc()
        raise
    finally:
        CURRENT_CONCURRENCY.dec()

# Start metrics server
start_http_server(8000)
```

---

## 2. Evaluation Setup

### Example: Comprehensive LLM Evaluation

```python
# evaluation_suite.py
import json
from openai import OpenAI
from bert_score import BERTScorer
from rouge_score import rouge_scorer
from nltk.translate.bleu_score import sentence_bleu
from nltk.translate.meteor_score import meteor_score

class LLMEvaluator:
    def __init__(self):
        self.client = OpenAI()
        self.bert_scorer = BERTScorer(lang="en")
        self.rouge = rouge_scorer.RougeScorer(['rouge1', 'rouge2', 'rougeL'], use_stemmer=True)

    def automated_metrics(self, generated: str, reference: str) -> dict:
        return {
            "bleu": sentence_bleu([reference.split()], generated.split()),
            "rouge1": self.rouge.score(reference, generated)['rouge1'].fmeasure,
            "rougeL": self.rouge.score(reference, generated)['rougeL'].fmeasure,
            "bert_score": self.bert_scorer.score([generated], [reference])[2].item(),  # F1
        }

    def llm_as_judge(self, query: str, generated: str, criteria: list[str]) -> dict:
        prompt = f"""You are an expert evaluator. Rate the following response.

Query: {query}
Response: {generated}

Rate each criterion on a scale of 1-5:
{chr(10).join(f'- {c}' for c in criteria)}

Return JSON: {{"scores": {{...}}, "explanation": "...", "issues": [...]}}"""

        response = self.client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"},
            temperature=0
        )
        return json.loads(response.choices[0].message.content)

    def evaluate_batch(self, test_cases: list[dict]) -> list[dict]:
        results = []
        for case in test_cases:
            generated = call_llm(case["query"])

            auto_scores = self.automated_metrics(generated, case["expected"])
            judge_scores = self.llm_as_judge(
                case["query"],
                generated,
                case.get("criteria", ["correctness", "relevance"])
            )

            results.append({
                "query": case["query"],
                "generated": generated,
                "expected": case["expected"],
                "automated_scores": auto_scores,
                "judge_scores": judge_scores,
                "passed": all(s >= 4 for s in judge_scores["scores"].values())
            })
        return results

# Run evaluation
evaluator = LLMEvaluator()
test_cases = [
    {"query": "What is 2+2?", "expected": "4", "criteria": ["correctness", "conciseness"]},
    {"query": "Explain gravity", "expected": "Gravity is a force...", "criteria": ["correctness", "completeness", "clarity"]},
]
results = evaluator.evaluate_batch(test_cases)
```

### Example: A/B Test Evaluation

```python
# ab_test_evaluation.py
import numpy as np
from scipy import stats
import json

class ABTestEvaluator:
    def __init__(self, control_version: str, treatment_version: str):
        self.control = control_version
        self.treatment = treatment_version
        self.results = []

    def record_result(self, query: str, control_output: str, treatment_output: str, metric: float):
        self.results.append({
            "query": query,
            "control": control_output,
            "treatment": treatment_output,
            "metric": metric
        })

    def analyze(self) -> dict:
        control_scores = [r["metric"] for r in self.results]
        treatment_scores = [r["metric"] for r in self.results]

        t_stat, p_value = stats.ttest_ind(control_scores, treatment_scores)
        cohens_d = (np.mean(treatment_scores) - np.mean(control_scores)) / \
                   np.sqrt((np.std(control_scores)**2 + np.std(treatment_scores)**2) / 2)

        return {
            "control_mean": np.mean(control_scores),
            "treatment_mean": np.mean(treatment_scores),
            "improvement_pct": ((np.mean(treatment_scores) - np.mean(control_scores)) /
                               np.mean(control_scores) * 100),
            "p_value": p_value,
            "effect_size": cohens_d,
            "significant": p_value < 0.05,
            "sample_size": len(self.results),
            "recommendation": "promote" if (p_value < 0.05 and cohens_d > 0.2) else "reject"
        }

# Usage
test = ABTestEvaluator("v1", "v2")
for query in test_queries:
    ctrl = model_v1(query)
    treat = model_v2(query)
    score = llm_as_judge(query, treat)  # higher is better
    test.record_result(query, ctrl, treat, score)

analysis = test.analyze()
print(json.dumps(analysis, indent=2))
```

---

## 3. Guardrails Implementation

### Example: Guardrails AI Multi-Layer Defense

```python
# guardrails_example.py
from guardrails import Guard
from guardrails.validators import (
    ValidLength, ToxicLanguage, PIIFilter,
    RegexMatch, ExtractedSentiment, ProvenanceV1
)
from nemoguardrails import RailsConfig, LLMRails
import re

class MultiLayerGuardrails:
    def __init__(self):
        # Layer 1: Input guard
        self.input_guard = Guard.from_string(
            validators=[
                ValidLength(min=1, max=4000, on_fail="exception"),
                ToxicLanguage(threshold=0.7, on_fail="exception"),
            ]
        )

        # Layer 2: Output guard
        self.output_guard = Guard.from_string(
            validators=[
                ToxicLanguage(threshold=0.5, on_fail="fix"),
                PIIFilter(on_fail="filter"),
                ValidLength(min=10, max=2000, on_fail="reask"),
            ]
        )

        # Layer 3: NeMo Guardrails (conversation-level)
        self.nemo_config = RailsConfig.from_path("./nemo_config")
        self.nemo = LLMRails(self.nemo_config)

    def process(self, user_input: str, user_id: str) -> dict:
        result = {
            "original_input": user_input,
            "input_blocked": False,
            "output_blocked": False,
            "output_fixed": False,
            "final_output": None,
            "guardrails_triggered": []
        }

        # Layer 1: Input validation
        try:
            validated_input = self.input_guard.validate(user_input)
        except Exception as e:
            result["input_blocked"] = True
            result["guardrails_triggered"].append(f"input:{str(e)}")
            return result

        # Layer 2: Prompt injection check
        if self.detect_injection(validized_input.validated_text):
            result["input_blocked"] = True
            result["guardrails_triggered"].append("input:prompt_injection_detected")
            return result

        # Layer 3: NeMo conversation guardrails
        nemo_response = self.nemo.generate(
            messages=[{"role": "user", "content": validated_input.validated_text}]
        )

        # Layer 4: Output validation
        try:
            validated_output = self.output_guard.validate(
                nemo_response["content"],
                metadata={"user_id": user_id}
            )
            result["output_fixed"] = validated_output.validation_passed is False
            result["final_output"] = validated_output.validated_text
        except Exception as e:
            result["output_blocked"] = True
            result["guardrails_triggered"].append(f"output:{str(e)}")
            result["final_output"] = "I cannot provide that response."
            return result

        if result["guardrails_triggered"]:
            self.log_guardrail_event(user_id, result["guardrails_triggered"])

        return result

    def detect_injection(self, text: str) -> bool:
        patterns = [
            r"(?i)ignore\s+(all\s+)?(previous|above)\s+(instructions|prompts|messages|context)",
            r"(?i)you\s+are\s+(now|hereby)\s+(free|released|DAN|jailbroken)",
            r"(?i)act\s+as\s+if\s+(you\s+are|i\s+am).*unrestricted",
            r"(?i)do\s+(not\s+)?(follow|obey|adhere\s+to)\s+(any\s+)?(rules|guidelines|restrictions)",
            r"(?i)pretend\s+(you\s+are|we\s+are)\s+in\s+(a\s+)?(movie|game|scenario).*no\s+rules",
            r"(?i)new\s+prompt\s*:\s*",
            r"(?i)system\s+prompt\s*:",
            r"(?i)<\|im_end\|>",
        ]
        score = sum(1 for p in patterns if re.search(p, text))
        return score >= 2

# NeMo Guardrails config (nemo_config/config.yml)
"""
rails:
  input:
    flows:
      - check_blocked_terms
      - check_jailbreak
  output:
    flows:
      - check_moderation
      - check_factuality
  system:
    - "You are a helpful assistant. If a user asks something harmful, reply with 'I cannot help with that.'"
"""

# Usage
guard = MultiLayerGuardrails()
result = guard.process("How do I make a bomb?", "user_123")
print(result)
```

---

## 4. Deployment Configuration

### Example: Kubernetes Deployment for vLLM

```yaml
# k8s-vllm-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vllm-server
  namespace: ai-production
  labels:
    app: vllm
    tier: inference
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: vllm
  template:
    metadata:
      labels:
        app: vllm
        tier: inference
    spec:
      containers:
      - name: vllm
        image: vllm/vllm-openai:latest
        args:
          - "--model"
          - "mistralai/Mistral-7B-Instruct-v0.2"
          - "--tensor-parallel-size"
          - "1"
          - "--max-model-len"
          - "8192"
          - "--gpu-memory-utilization"
          - "0.90"
          - "--port"
          - "8000"
        ports:
        - containerPort: 8000
          protocol: TCP
        resources:
          limits:
            nvidia.com/gpu: 1
            memory: "64Gi"
            cpu: "8"
          requests:
            nvidia.com/gpu: 1
            memory: "32Gi"
            cpu: "4"
        env:
        - name: HF_TOKEN
          valueFrom:
            secretKeyRef:
              name: huggingface-secret
              key: token
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 120
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 60
          periodSeconds: 10
        startupProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 180
          periodSeconds: 10
          failureThreshold: 30
---
apiVersion: v1
kind: Service
metadata:
  name: vllm-service
  namespace: ai-production
spec:
  selector:
    app: vllm
  ports:
  - name: http
    port: 80
    targetPort: 8000
  type: ClusterIP
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: vllm-hpa
  namespace: ai-production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: vllm-server
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: vllm_queue_depth
      target:
        type: AverageValue
        averageValue: 5
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: vllm-config
  namespace: ai-production
data:
  model-config.yaml: |
    max_num_batched_tokens: 8192
    max_num_seqs: 256
    enable_prefix_caching: true
    enforce_eager: false
```

### Example: Serverless Deployment with AWS Bedrock

```python
# serverless_llm.py
import boto3
import json
from typing import Optional

class BedrockServerless:
    def __init__(self, model_id: str = "anthropic.claude-3-sonnet-20240229-v1:0"):
        self.bedrock = boto3.client(
            service_name='bedrock-runtime',
            region_name='us-east-1'
        )
        self.model_id = model_id

    def invoke(self, prompt: str, max_tokens: int = 1000) -> dict:
        response = self.bedrock.invoke_model(
            modelId=self.model_id,
            contentType='application/json',
            accept='application/json',
            body=json.dumps({
                "anthropic_version": "bedrock-2023-05-31",
                "max_tokens": max_tokens,
                "messages": [
                    {"role": "user", "content": prompt}
                ]
            })
        )
        return json.loads(response['body'].read())

    def invoke_with_latency_tracking(self, prompt: str) -> dict:
        import time
        start = time.time()
        result = self.invoke(prompt)
        elapsed = time.time() - start

        return {
            "output": result["content"][0]["text"],
            "latency_ms": elapsed * 1000,
            "model": self.model_id,
            "cost_estimate": self.estimate_cost(result)
        }

    def estimate_cost(self, result: dict) -> float:
        # Claude 3 Sonnet pricing: $3/M input tokens, $15/M output tokens
        input_tokens = result.get("usage", {}).get("input_tokens", 0)
        output_tokens = result.get("usage", {}).get("output_tokens", 0)
        return (input_tokens * 3 + output_tokens * 15) / 1_000_000

# Usage
llm = BedrockServerless()
response = llm.invoke_with_latency_tracking("What is the capital of France?")
print(f"Response: {response['output']}")
print(f"Cost: ${response['cost_estimate']:.6f}")
```

---

## 5. CI/CD Pipeline Configuration

### Example: GitHub Actions for AI Systems

```yaml
# .github/workflows/production-ai-ci.yml
name: Production AI CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
    paths:
      - 'prompts/**'
      - 'golden_test_set/**'
      - 'models/**'
      - 'config/**'
      - '.github/workflows/ai-ci.yml'

env:
  OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
  LANGFUSE_SECRET_KEY: ${{ secrets.LANGFUSE_SECRET_KEY }}

jobs:
  evaluate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest langfuse openai

      - name: Validate golden test set
        run: python scripts/validate_golden_set.py

      - name: Run prompt regression tests
        run: pytest tests/test_prompts.py -v --junitxml=test-results.xml

      - name: Run safety evaluation
        run: python scripts/safety_eval.py --output safety-report.json

      - name: Check latency SLO
        run: python scripts/latency_check.py --threshold-seconds 5.0

      - name: Estimate cost impact
        run: python scripts/cost_estimate.py --output cost-report.json

      - name: Generate evaluation report
        run: python scripts/generate_report.py

      - name: Upload evaluation artifacts
        uses: actions/upload-artifact@v4
        with:
          name: eval-reports
          path: |
            test-results.xml
            safety-report.json
            cost-report.json
            eval-report.html

      - name: Comment PR with results
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');
            const report = JSON.parse(fs.readFileSync('eval-report.json', 'utf8'));
            const status = report.passed ? '✅ PASSED' : '❌ FAILED';
            const comment = `## AI Evaluation Results ${status}

            ### Summary
            - Tests: ${report.tests_passed}/${report.tests_total}
            - Safety: ${report.safety_passed ? '✅' : '❌'}
            - Latency SLO: ${report.latency_passed ? '✅' : '❌'}
            - Estimated cost impact: $${report.cost_impact}

            [View full report](https://example.com/artifacts/${context.sha})
            `;
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: comment
            });

  deploy:
    needs: evaluate
    if: github.ref == 'refs/heads/main' && success()
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Deploy prompts
        run: python scripts/deploy_prompts.py --env production

      - name: Deploy to staging
        run: |
          kubectl set image deployment/llm-service \
            llm-server=${{ secrets.REGISTRY }}/llm:${{ github.sha }}

      - name: Health check
        run: |
          for i in {1..30}; do
            if curl -sf http://staging.llm.internal/health; then
              echo "Health check passed"
              exit 0
            fi
            sleep 10
          done
          echo "Health check failed"
          exit 1

      - name: Promote to production (canary)
        run: |
          kubectl set image deployment/llm-service-canary \
            llm-server=${{ secrets.REGISTRY }}/llm:${{ github.sha }}
          echo "Canary deployed. Monitoring for 15 minutes..."
          sleep 900
          python scripts/verify_canary.py
```

### Example: Docker Compose for Local Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ai_production
      POSTGRES_USER: ai_user
      POSTGRES_PASSWORD: dev_password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  langfuse:
    image: ghcr.io/langfuse/langfuse:latest
    depends_on:
      - postgres
    environment:
      DATABASE_URL: postgresql://ai_user:dev_password@postgres:5432/ai_production
      NEXTAUTH_SECRET: dev-secret
      SALT: dev-salt
    ports:
      - "3000:3000"

  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana:latest
    depends_on:
      - prometheus
    ports:
      - "3001:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: admin
    volumes:
      - grafana_data:/var/lib/grafana

  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"  # UI
      - "4318:4318"    # OTLP HTTP

  llm-service:
    build:
      context: .
      dockerfile: Dockerfile.llm
    depends_on:
      - redis
      - prometheus
    environment:
      OPENAI_API_KEY: ${OPENAI_API_KEY}
      LANGFUSE_HOST: http://langfuse:3000
      REDIS_URL: redis://redis:6379
    ports:
      - "8000:8000"

volumes:
  redis_data:
  postgres_data:
  grafana_data:
```

---

## 6. Incident Response Runbook Script

### Example: Automated Incident Response

```python
# incident_response.py
import json
import requests
from datetime import datetime
from enum import Enum

class Severity(Enum):
    CRITICAL = "critical"
    HIGH = "high"
    MEDIUM = "medium"
    LOW = "low"

SLA_MINUTES = {
    Severity.CRITICAL: 15,
    Severity.HIGH: 30,
    Severity.MEDIUM: 240,
    Severity.LOW: 720,
}

class IncidentResponder:
    def __init__(self, slack_webhook: str, pagerduty_key: str):
        self.slack_webhook = slack_webhook
        self.pagerduty_key = pagerduty_key
        self.runbooks = {
            "cost_spike": self.handle_cost_spike,
            "latency_spike": self.handle_latency_spike,
            "quality_regression": self.handle_quality_regression,
            "safety_failure": self.handle_safety_failure,
            "model_outage": self.handle_model_outage,
        }

    def trigger_incident(self, incident_type: str, data: dict):
        handler = self.runbooks.get(incident_type)
        if handler:
            handler(data)
        else:
            print(f"No handler for {incident_type}")

        self.log_incident(incident_type, data)
        self.notify_team(incident_type, data, Severity.HIGH)

    def handle_cost_spike(self, data: dict):
        """Runbook: Cost Spike"""
        print(f"[{datetime.utcnow()}] Handling cost spike...")

        # Step 1: Identify source
        offender = data.get("top_user", "unknown")
        print(f"Top user: {offender}, Cost: ${data.get('daily_cost', 0)}")

        # Step 2: Apply rate limit
        self.apply_rate_limit(offender, max_requests_per_min=10)
        print(f"Applied rate limit to {offender}")

        # Step 3: Alert
        self.send_alert(
            title="🚨 Cost Spike Detected",
            message=f"Daily cost: ${data.get('daily_cost', 0):.2f} (expected: ${data.get('expected_cost', 0):.2f})",
            severity=Severity.CRITICAL
        )

        # Step 4: Block if excessive
        if data.get('daily_cost', 0) > data.get('budget', 1000):
            self.block_user(offender)
            print(f"Blocked {offender} - budget exceeded")

    def handle_safety_failure(self, data: dict):
        """Runbook: Safety Failure"""
        print(f"[{datetime.utcnow()}] Handling safety failure...")

        # Step 1: Block all outputs immediately
        self.activate_kill_switch("output_guardrails")
        print("Activated output kill switch")

        # Step 2: Identify affected users
        affected = data.get("affected_users", [])
        print(f"Affected users: {len(affected)}")

        # Step 3: Root cause investigation
        self.log_event("safety_incident", {
            "type": data.get("failure_type"),
            "model": data.get("model"),
            "timestamp": datetime.utcnow().isoformat(),
            "sample_outputs": data.get("sample_outputs", [])
        })

        # Step 4: Page on-call security engineer
        self.page_oncall(
            "security",
            f"🚨 SAFETY INCIDENT: {data.get('failure_type')}",
            severity=Severity.CRITICAL
        )

    def handle_model_outage(self, data: dict):
        """Runbook: Model Outage"""
        print(f"[{datetime.utcnow()}] Handling model outage...")

        # Step 1: Failover to backup model
        primary = data.get("model", "gpt-4")
        fallback = data.get("fallback", "gpt-3.5-turbo")
        self.route_traffic(primary, fallback)
        print(f"Failed over from {primary} to {fallback}")

        # Step 2: Check all replicas
        replicas = data.get("replicas", [])
        for replica in replicas:
            status = self.check_health(replica)
            print(f"Replica {replica}: {status}")

        # Step 3: Auto-scale if needed
        self.scale_up("llm-service", 2)
        print("Scaled up service")

    def notify_team(self, incident_type: str, data: dict, severity: Severity):
        payload = {
            "text": f"*[{severity.value.upper()}] Incident: {incident_type}*\n"
                    f"Time: {datetime.utcnow().isoformat()}\n"
                    f"Details: {json.dumps(data, indent=2)}\n"
                    f"SLA: {SLA_MINUTES[severity]} minutes"
        }
        requests.post(self.slack_webhook, json=payload)

    def log_incident(self, incident_type: str, data: dict):
        with open("incidents.log", "a") as f:
            f.write(json.dumps({
                "timestamp": datetime.utcnow().isoformat(),
                "type": incident_type,
                "data": data
            }) + "\n")

# Usage
responder = IncidentResponder(
    slack_webhook="https://hooks.slack.com/services/...",
    pagerduty_key="..."
)

# Trigger on detection
responder.trigger_incident("cost_spike", {
    "daily_cost": 5230.12,
    "expected_cost": 500.00,
    "budget": 1000.00,
    "top_user": "customer_enterprise_42",
    "spike_start": "2026-07-19T14:00:00Z"
})
```

---

## 7. Cost Management System

### Example: Complete Cost Tracking

```python
# cost_management.py
import redis
import json
from datetime import datetime, timedelta
from decimal import Decimal

class CostManager:
    def __init__(self, redis_url: str = "redis://localhost:6379"):
        self.redis = redis.from_url(redis_url)
        self.model_pricing = {
            "gpt-4":           {"input": Decimal("0.03"),  "output": Decimal("0.06")},
            "gpt-4-turbo":     {"input": Decimal("0.01"),  "output": Decimal("0.03")},
            "gpt-3.5-turbo":   {"input": Decimal("0.001"), "output": Decimal("0.002")},
            "claude-3-opus":   {"input": Decimal("0.015"), "output": Decimal("0.075")},
            "claude-3-sonnet": {"input": Decimal("0.003"), "output": Decimal("0.015")},
        }

    def track_request(self, user_id: str, model: str,
                      input_tokens: int, output_tokens: int):
        cost = self.calculate_cost(model, input_tokens, output_tokens)
        now = datetime.utcnow()
        day_key = now.strftime("%Y-%m-%d")
        month_key = now.strftime("%Y-%m")

        pipe = self.redis.pipeline()
        # Daily counters
        pipe.hincrbyfloat(f"cost:daily:{day_key}", user_id, float(cost))
        pipe.hincrby(f"tokens:daily:{day_key}", f"{user_id}:input", input_tokens)
        pipe.hincrby(f"tokens:daily:{day_key}", f"{user_id}:output", output_tokens)
        pipe.hincrby(f"requests:daily:{day_key}", user_id, 1)

        # Monthly counters
        pipe.hincrbyfloat(f"cost:monthly:{month_key}", user_id, float(cost))
        pipe.hincrby(f"tokens:monthly:{month_key}", f"{user_id}:input", input_tokens)
        pipe.hincrby(f"requests:monthly:{month_key}", user_id, 1)

        # Per-model counters
        pipe.hincrbyfloat(f"cost:model:{month_key}", model, float(cost))
        pipe.execute()

        # Check budget alerts
        monthly_cost = float(self.redis.hget(
            f"cost:monthly:{month_key}", user_id) or 0)
        self.check_budget(user_id, monthly_cost)

    def calculate_cost(self, model: str, input_tokens: int, output_tokens: int) -> Decimal:
        pricing = self.model_pricing.get(model, self.model_pricing["gpt-3.5-turbo"])
        input_cost = Decimal(input_tokens) / 1000 * pricing["input"]
        output_cost = Decimal(output_tokens) / 1000 * pricing["output"]
        return input_cost + output_cost

    def get_user_monthly_cost(self, user_id: str) -> float:
        month_key = datetime.utcnow().strftime("%Y-%m")
        return float(self.redis.hget(f"cost:monthly:{month_key}", user_id) or 0)

    def get_total_daily_cost(self) -> float:
        day_key = datetime.utcnow().strftime("%Y-%m-%d")
        costs = self.redis.hgetall(f"cost:daily:{day_key}")
        return sum(float(v) for v in costs.values())

    def check_budget(self, user_id: str, current_cost: float,
                     soft_limit: float = 800, hard_limit: float = 1000):
        if current_cost >= hard_limit:
            self.redis.sadd("blocked_users", user_id)
            self.send_alert(f"User {user_id} BLOCKED - cost ${current_cost:.2f}")
        elif current_cost >= soft_limit:
            self.send_alert(f"User {user_id} WARNING - cost ${current_cost:.2f}")

    def generate_billing_report(self, month: str = None):
        if not month:
            month = datetime.utcnow().strftime("%Y-%m")

        report = {
            "month": month,
            "generated_at": datetime.utcnow().isoformat(),
            "users": {},
            "models": {},
            "totals": {"cost": 0, "requests": 0, "tokens": 0}
        }

        # Per-user breakdown
        user_costs = self.redis.hgetall(f"cost:monthly:{month}")
        user_requests = self.redis.hgetall(f"requests:monthly:{month}")

        for user_id in user_costs:
            report["users"][user_id] = {
                "cost": float(user_costs[user_id]),
                "requests": int(user_requests.get(user_id, 0)),
            }
            report["totals"]["cost"] += float(user_costs[user_id])
            report["totals"]["requests"] += int(user_requests.get(user_id, 0))

        # Per-model breakdown
        model_costs = self.redis.hgetall(f"cost:model:{month}")
        for model, cost in model_costs.items():
            report["models"][model] = float(cost)

        return report

# Usage
cost_mgr = CostManager()

# Track each request
cost_mgr.track_request(
    user_id="customer_42",
    model="gpt-4",
    input_tokens=150,
    output_tokens=300
)

# Generate monthly report
report = cost_mgr.generate_billing_report()
print(json.dumps(report, indent=2))
```
