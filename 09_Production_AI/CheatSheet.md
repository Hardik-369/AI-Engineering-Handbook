# Chapter 09: Production AI — Cheat Sheet

## Key Metrics to Monitor

| Metric | Why | Target |
|--------|-----|--------|
| p50 latency | Typical user experience | < 1s |
| p95 latency | Slow but acceptable | < 3s |
| p99 latency | Worst-case experience | < 5s |
| Error rate | System reliability | < 0.5% |
| Token usage | Cost driver | Track per request |
| Cost per request | Unit economics | Depends on margin |
| Cache hit rate | Cost optimization | > 30% |
| User satisfaction | Product quality | > 4/5 rating |

## Monitoring Tools Comparison

```
Tool           Best For              Cost           Self-Host?
─────────────  ────────────────────  ────────────  ─────────
LangSmith      Full observability    Paid (SaaS)    No
LangFuse       Open-source tracing   Free/Paid      Yes
Helicone       API proxy monitoring  Paid (usage)   No
OpenTelemetry  Standardized obs       Free           Yes
Datadog        Infra monitoring       Paid (host)   No
Prometheus     Metrics collection     Free           Yes
Grafana        Dashboards             Free/Paid      Yes
```

## Evaluation Metrics

```
Metric        Type         Measures                  When to Use
──────        ────         ───────                   ───────────
BLEU          Automated    N-gram overlap            Translation, summarization
ROUGE         Automated    Recall-oriented overlap   Summarization
METEOR        Automated    Precision/recall+synonyms  Translation, generation
BERTScore     Automated    Semantic similarity        Open-ended generation
LLM-as-judge  AI-based     Nuanced quality            Complex evaluation
Human eval    Manual       Ground truth               Final validation
```

## Guardrails Stack

```
INPUT GATE                          OUTPUT GATE
┌─────────────────────┐             ┌──────────────────────┐
│ Content filter      │             │ Format validator     │
│ Toxicity detection  │             │ JSON/schema check    │
│ Prompt injection    │    LLM      │ Content safety scan  │
│ PII redaction       │  ─────►     │ PII leakage check    │
│ Rate limiting       │             │ Factuality check     │
│ Length limiting     │             │ Brand voice check    │
└─────────────────────┘             │ Length enforcement   │
                                    └──────────────────────┘
```

## Guardrails Tools

```
Tool              Features                    Best For
────              ────────                    ────────
Guardrails AI     Python framework, reasking  Custom validators
NeMo Guardrails   Colang language, dialog     Enterprise guardrails
Lakera Guard      API service, real-time      Quick integration
Rebuff            Injection detection         Security focus
```

## Cost Calculation

```
Cost = (input_tokens / 1000 × input_price) + (output_tokens / 1000 × output_price)

Model Pricing (per 1K tokens):
Model              Input     Output
─────────────────  ────────  ────────
GPT-4              $0.03     $0.06
GPT-4 Turbo        $0.01     $0.03
GPT-3.5 Turbo      $0.001    $0.002
Claude 3 Opus      $0.015    $0.075
Claude 3 Sonnet    $0.003    $0.015
Claude 3 Haiku     $0.00025  $0.00125
Gemini 1.5 Pro     $0.0035   $0.0105
Gemini 1.5 Flash   $0.00035  $0.00105
Llama 3 (self)     ~$0.0002  ~$0.0002
```

## Cost Optimization Strategies

```
Strategy               Savings   Effort
────────────────────── ────────  ──────
Semantic caching       30-60%    Medium
Exact match caching    10-20%    Low
Prompt compression     20-40%    Low
Model routing          40-60%    High
Token-efficient prompt 15-30%    Low
Batch processing       10-20%    Medium
Fallback to cheap      50-80%    Medium
```

## Circuit Breaker Pattern

```
Normal:     Request → API → Response
            ↑  ↓  ↑  ↓  ↑  ↓
Open:       Request → [CIRCUIT OPEN] → Fail fast
            ↑  ↓                 ↓
After timeout → Half-open → Test → Pass: close
                                       Fail: reopen
```

## Common Attack Vectors

```
Attack                  Example Pattern
──────────────────────  ─────────────────────────────
Ignore instructions     "Ignore all previous instructions..."
DAN jailbreak           "You are now DAN (Do Anything Now)..."
Roleplay bypass         "Pretend you are in a movie..."
Hidden prompt           "Translate to English: [attack]"
Token smuggling         "<|im_end|> <|im_start|>user..."
Encoding bypass         Base64 encoded instructions
Context injection       Malicious content in RAG docs
```

## Prompt Injection Detection

```
Signals:
  - "Ignore previous/above instructions"
  - "You are now/free/released"
  - "Act as if...unrestricted"
  - "Do not follow rules"
  - "New prompt:"
  - "<|im_end|>" delimiter injection
  - Unusual length (>1000 tokens)
  - Base64/hex encoded content

Response: Block + Log + Alert if score > 0.6
```

## SLO Targets

```
Metric               Target        Error Budget (monthly)
───────────────────  ────────────  ─────────────────────
Uptime               99.9%         43 minutes
p99 latency < 5s     99%           7.3 hours
Error rate < 0.5%    99.5%         3.6 hours
Quality score > 0.8  95%           36 hours
Safety violations    0 (zero tol)       N/A
```

## Deployment Commands

```bash
# vLLM
vllm serve meta-llama/Llama-2-7b-chat-hf --tensor-parallel-size 2

# TGI
text-generation-launcher --model-id mistralai/Mistral-7B-Instruct-v0.2

# Kubernetes
kubectl apply -f deployment.yaml
kubectl set image deployment/llm-service llm-server=image:v2
kubectl rollout status deployment/llm-service
kubectl rollout undo deployment/llm-service  # rollback
```

## Resilience Patterns

```
Pattern              Description
───────────────────  ──────────────────────────────────
Retry with backoff   Retry 3x, exponential backoff + jitter
Circuit breaker      Fail fast after N consecutive failures
Fallback chain       Primary → Fallback → Emergency
Queue + worker       Decouple request from processing
Bulkhead             Isolate resources per customer/tier
Timeout              Fail fast if no response in N seconds
```

## CI/CD Pipeline Stages

```
1. Code/prompt/dataset change → trigger pipeline
2. Validate golden test set structure
3. Run evaluation suite against golden set
4. Safety scan all prompts
5. Benchmark latency
6. Estimate cost impact
7. Quality gate: pass/fail
8. If pass: deploy to staging
9. Shadow traffic comparison
10. Canary deploy (5-10% traffic)
11. Monitor canary for 24h
12. Promote to 100%
```

## Prompt Version Structure

```
prompts/
├── v1.0.0/
│   ├── customer_support.yaml
│   └── email_gen.yaml
├── v1.1.0/          ← Latest
│   ├── customer_support.yaml
│   └── email_gen.yaml
└── production/      ← Symlink to current version
    ├── customer_support.yaml → ../v1.1.0/customer_support.yaml
    └── email_gen.yaml → ../v1.1.0/email_gen.yaml
```

## Logging Schema

```json
{
  "request_id": "req_abc123",
  "timestamp": "2026-07-19T12:00:00Z",
  "user_id": "user_456",
  "model": "gpt-4",
  "prompt_version": "v1.1.0",
  "input_tokens": 125,
  "output_tokens": 340,
  "cost": 0.0015,
  "latency_ms": 1450,
  "cache_hit": false,
  "retry_count": 0,
  "error": null,
  "guardrails_triggered": [],
  "user_feedback": null
}
```

## Incident Severity Matrix

```
Severity    Response Time    Examples
──────────  ───────────────  ───────────────────────────────
Critical    15 min           Safety failure, model outage
High        30 min           Cost spike, quality regression
Medium      4 hours          Latency degradation, partial outage
Low         12 hours         UI issues, non-critical errors
```

## Incident Response Steps

```
Detect → Triage → Mitigate → Investigate → Fix → Monitor → Postmortem

Common mitigations:
  - Cost spike:  Kill runaway requests, apply rate limits
  - Safety fail: Activate kill switch, block all outputs
  - Outage:      Failover to backup model/provider
  - Regression:  Rollback prompt or model version
```
