# Chapter 09: Production AI — Exercises

## Monitoring & Observability

### Exercise 1: Set Up LLM Monitoring
Install LangFuse or LangSmith and trace 50 LLM requests. Create a dashboard showing:
- Average latency (p50, p95, p99)
- Token usage distribution
- Error rate over time
- Cost per request

### Exercise 2: Build a Cost Dashboard
Write a script that reads LLM logs and produces a cost breakdown:
- Cost per user/customer
- Cost per model
- Cost per time of day (identify peak hours)
- Projected monthly cost based on current usage

### Exercise 3: Anomaly Detection in Metrics
Implement a simple anomaly detection system for latency metrics:
- Track rolling average and standard deviation
- Flag any request where latency > mean + 3*std
- Create an alert when anomaly rate exceeds 1%

## LLM Evaluation

### Exercise 4: Build an LLM-as-Judge Evaluator
Create an evaluation function that uses GPT-4 to rate outputs on:
- Correctness (1-5)
- Relevance (1-5)
- Tone alignment (1-5)
- Format compliance (1-5)

Test it against 20 known-good and 20 known-bad responses. Report precision and recall.

### Exercise 5: Golden Test Set Creation
Create a golden test set for a customer support chatbot with at least 30 test cases covering:
- 10 easy (simple policy questions)
- 10 medium (multi-part questions)
- 10 hard (edge cases, ambiguity, complaints)

Include expected responses and evaluation criteria for each.

### Exercise 6: Automatic Metric Comparison
Compare BLEU, ROUGE, METEOR, and BERTScore on 50 generated responses:
- Create a correlation matrix between metrics
- Identify which metric best matches human judgment
- Report cases where metrics disagree with humans

### Exercise 7: A/B Testing Framework
Build a framework to A/B test two prompt versions:
- Split traffic 50/50
- Collect latency, cost, and quality metrics
- Implement statistical significance testing
- Create a dashboard showing results

## Prompt Testing & Regression

### Exercise 8: Prompt Regression Test Suite
Write a pytest-based regression test suite for three prompts:
- Test factual accuracy against golden set
- Test safety constraints (no harmful outputs)
- Test format compliance (valid JSON, correct structure)
- Test for prompt injection robustness

### Exercise 9: Prompt Version Control System
Design and implement a prompt version control system:
- Store prompts in a versioned directory structure
- Implement rollback capability
- Create a CLI tool for promoting prompts across environments
- Add a diff tool showing what changed between versions

## Safety & Guardrails

### Exercise 10: Input Guardrail Implementation
Implement input guardrails that detect:
- Toxic language (profanity, hate speech)
- PII (credit cards, SSNs, emails)
- Prompt injection attempts (10+ attack patterns)
- Excessively long inputs (>4000 tokens)

Test with 50 malicious inputs and 50 benign inputs. Report false positive and false negative rates.

### Exercise 11: Output Guardrail Implementation
Implement output guardrails that:
- Validate JSON structure
- Detect and redact PII
- Check for toxic/unsafe content
- Enforce length limits (100-2000 tokens)
- Check brand voice compliance (against a style guide)

### Exercise 12: Prompt Injection Tournament
Organize a red-teaming exercise:
- 5 participants try to jailbreak your system
- Each gets 20 attempts
- Document successful attack vectors
- Implement fixes for top 5 attack types
- Re-test and report improvement

## Security

### Exercise 13: Security Audit
Perform a security audit of an LLM application:
- Check for hardcoded API keys
- Review authentication mechanisms
- Test rate limiting effectiveness
- Review audit logging completeness
- Write a security report with findings and recommendations

### Exercise 14: Build a Prompt Injection Detector
Train or implement a prompt injection classifier:
- Use a dataset of known prompt injections (e.g., from JailbreakBench)
- Extract features: presence of "ignore previous instructions", encoded text, etc.
- Achieve >95% detection rate with <1% false positive rate
- Implement it as a middleware service

## Rate Limiting & Cost Management

### Exercise 15: Token Budget System
Implement a complete token budget management system:
- Per-user monthly budgets with configurable limits
- Soft limit warnings at 80% usage
- Hard limit blocking at 100%
- Real-time usage tracking with Redis
- Admin dashboard for budget management

### Exercise 16: Cost Optimization Analysis
Analyze 10,000 LLM requests and identify optimization opportunities:
- What percentage of requests hit cache? (simulate with exact match)
- How much could prompt compression save? (estimate token reduction)
- How many requests could use a cheaper model? (classify by complexity)
- Project monthly savings for each strategy

## Deployment & Scaling

### Exercise 17: Deploy an LLM with Kubernetes
Create Kubernetes manifests to deploy a self-hosted LLM:
- Deployment with GPU resource requests/limits
- Service for internal routing
- Horizontal Pod Autoscaler based on request queue depth
- Liveness and readiness probes
- ConfigMap for model configuration

### Exercise 18: Circuit Breaker Implementation
Implement a circuit breaker pattern for LLM API calls:
- Trip circuit after 5 consecutive failures
- Half-open after 60 seconds
- Track failure types (timeout vs rate-limit vs auth error)
- Fallback to a cheaper model when circuit is open
- Log all state transitions

## CI/CD for AI

### Exercise 19: AI CI Pipeline
Create a GitHub Actions workflow that:
- Runs on PRs that change prompts, datasets, or model config
- Executes golden test suite
- Runs safety evaluation
- Checks latency SLO
- Estimates cost impact
- Creates an evaluation report as PR comment

### Exercise 20: Automated Rollback System
Implement an automated rollback system:
- Monitor evaluation scores in production
- If score drops below threshold, automatically rollback
- Notify team via Slack/PagerDuty
- Create a rollback audit trail
- Implement a manual override mechanism

## Incident Response

### Exercise 21: Incident Response Drill
Create an incident response runbook and conduct a table-top exercise with:
- Scenario A: Cost spikes 10x due to infinite retry loop
- Scenario B: Model starts generating harmful content
- Scenario C: All LLM API calls returning 503 errors
- Scenario D: Latency p99 jumps from 2s to 15s

For each: document detection, triage, mitigation, fix, and postmortem steps.

### Exercise 22: Postmortem Analysis
Write a postmortem for a fictional production incident:
- Timeline of events
- Root cause analysis (5 whys)
- Action items with owners and deadlines
- Prevention mechanisms
- Follow-up review date

## Logging & Tracing

### Exercise 23: Full Request Trace
Implement OpenTelemetry tracing for a complete LLM request lifecycle:
- Input guardrail check span
- LLM API call span (with model, token count attributes)
- Output guardrail check span
- Connect spans in a trace
- Export traces to Jaeger or similar

### Exercise 24: Feedback Collection System
Build a user feedback collection system:
- Thumbs up/down on responses
- Optional text feedback
- Store feedback linked to request ID
- Dashboard showing feedback trends
- Auto-flag responses with 3+ consecutive downvotes

## Advanced Projects

### Exercise 25: Multi-Model Router
Build a system that routes requests to the optimal model based on:
- Query complexity (classified by a lightweight model)
- Current latency of each model
- Cost budget remaining
- User tier (free vs premium)
- Implement fallback chain and circuit breakers

### Exercise 26: Production AI Health Score
Design a health score system that combines:
- Latency SLO compliance (30%)
- Error rate (20%)
- Cost per request (20%)
- User satisfaction score (20%)
- Guardrail trigger rate (10%)

Implement alerts when health score drops below 0.8.

### Exercise 27: Shadow Deployment
Implement shadow deployment where a new model version:
- Receives a copy of all production traffic
- But doesn't serve responses to users
- Its responses are compared against the production model
- Quality differences are logged and alerted on
- After 24h of good performance, it's promoted

### Exercise 28: Multi-Tenant Cost Allocation
Build a multi-tenant cost allocation system:
- Per-customer token tracking
- Configurable pricing tiers (per 1K tokens)
- Monthly billing report generation
- Real-time usage dashboard per tenant
- Automated throttling when budget exceeded

### Exercise 29: LLM Cache Optimization
Design and implement a semantic cache for LLM responses:
- Use embedding similarity to detect similar queries
- Configurable similarity threshold
- Cache hit/miss logging
- Automatic cache invalidation after TTL
- Cache warmup for common queries

### Exercise 30: Full Production AI Stack
Build the complete production AI system:
- API gateway with authentication
- Input guardrails (content filter + injection detection)
- Prompt management with version control
- Model router with fallback chain
- Output guardrails (validation + safety)
- Monitoring dashboard (latency, cost, quality)
- User feedback collection
- Incident alerting system
