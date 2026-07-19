# Chapter 09: Production AI — Interview Questions

## Fundamental Concepts

### Q1: What's the difference between a prototype AI system and a production AI system?
**Answer**: A prototype works once on curated examples; a production system works every time for every user within budget. Key differences: reliability (99.9%+ vs "it worked in the notebook"), monitoring (dashboards vs print statements), cost consciousness (P&L item vs irrelevant), security (SOC2-compliant vs nonexistent), and scalability (1000s of users vs 1 user).

### Q2: What metrics should you monitor in a production LLM system?
**Answer**: Latency (p50, p95, p99), token usage (input/output), cost per request, error rates (HTTP + LLM-specific), cache hit rate, user satisfaction, model drift, throughput (requests per second), and concurrency. Each maps to a specific concern: latency = UX, tokens/tokens = cost, errors = reliability, satisfaction = product-market fit.

### Q3: Explain the p50/p95/p99 latency metrics and why each matters.
**Answer**: p50 (median) tells you the typical user experience. p95 tells you what 1 in 20 users experience — the "slow but acceptable" threshold. p99 tells you about the worst 1 in 100 — the "something is wrong" threshold. Most monitoring focuses on p99 because that's where user frustration and churn happen, but optimizing p99 without watching p50 can mean hurting the average user for edge cases.

### Q4: What is LLM evaluation and why is it harder than traditional ML evaluation?
**Answer**: LLM evaluation assesses output quality, safety, and alignment rather than a single accuracy metric. It's harder because: (1) there's no single ground truth for open-ended generation, (2) quality is multidimensional (fluency, relevance, correctness, tone), (3) the same input can have many valid outputs, and (4) human judgment is often the only reliable evaluator for nuanced tasks.

## Monitoring & Observability

### Q5: What tools would you use for LLM observability and how do they differ?
**Answer**: LangSmith (full-stack: tracing + evaluation + playground), LangFuse (open-source: cost tracking + prompt management), Helicone (proxy-based: request logging + caching + rate limiting), and OpenTelemetry (standardized: vendor-neutral traces/metrics/logs). LangSmith/LangFuse are LLM-specific, Helicone works at the API proxy level, OpenTelemetry is general-purpose infrastructure monitoring.

### Q6: How would you detect model drift in a production LLM system?
**Answer**: Compare current outputs against a golden test set over time using automated metrics (BERTScore, LLM-as-judge). Track distributions of output length, sentiment, topic, and refusal rates. Use statistical process control — if metrics deviate by >2 standard deviations from the rolling mean, trigger an alert. Also monitor for silent model updates from API providers.

### Q7: How do you set up effective alerting for LLM systems without alert fatigue?
**Answer**: Use multi-level thresholds: warning (p99 > 3s), critical (p99 > 5s). Use rate-based alerts (error rate > 1% over 5 minutes) rather than absolute counts. Implement alert deduplication and grouping. Use time-based escalation (15 min for critical, 30 min for high). Establish SLOs and only alert when SLO burn rate is high. Create runbooks for each alert type.

## Evaluation

### Q8: Compare LLM-as-judge evaluation vs automated metrics (BLEU, ROUGE, BERTScore).
**Answer**: BLEU/ROUGE measure n-gram overlap with reference — fast but don't capture semantics. BERTScore uses embeddings for semantic similarity — better but still reference-dependent. LLM-as-judge can evaluate open-ended generation against arbitrary criteria — most flexible but expensive and potentially biased toward its own outputs. Best practice: combine all three — automated metrics for regression detection, LLM-as-judge for nuanced quality assessment.

### Q9: How do you build and maintain a golden test set?
**Answer**: Curate real production queries (sampled across categories and difficulty levels), write expected responses manually, define evaluation criteria per test case, version control the set alongside code, review and update quarterly, and balance for coverage (easy/medium/hard, different intents, edge cases). Size: start with 30-50, grow to 200-500 for mature systems.

### Q10: Design an A/B testing framework for LLM prompts.
**Answer**: Split traffic deterministically by user ID or session ID (not random per request). Both variants serve the same query; log responses, latency, cost, and user feedback. Use statistical tests (t-test or Mann-Whitney) on quality scores, compare cost per request, and monitor guardrail trigger rates. Run for minimum 7 days or 1000 samples per variant. Include a "no significant difference" outcome — don't force a winner.

## Safety & Guardrails

### Q11: What types of guardrails do you need in a production LLM system?
**Answer**: Input guardrails (content filtering, prompt injection detection, PII redaction, rate limiting) and output guardrails (format validation, content safety, PII scanning, factuality checking, brand voice compliance). Both are needed because attacks can come through input manipulation and model failures can produce unsafe outputs.

### Q12: How would you detect and prevent prompt injection attacks?
**Answer**: Multi-layer approach: (1) pattern matching for known attack patterns ("ignore previous instructions", "DAN"), (2) ML-based classifier trained on injection datasets, (3) instruction separation using delimiters and structured prompts, (4) input length limits, (5) output validation that checks if the model ignored instructions. Use tools like Lakera Guard, Rebuff, or NeMo Guardrails.

### Q13: How do you handle false positives in guardrails (blocking legitimate requests)?
**Answer**: Log all blocked requests with metadata. Maintain a "false positive" review queue. Use probabilistic thresholds (not binary) — flag high-risk but block only extreme-risk. Implement a human review workflow for edge cases. Regularly tune thresholds based on FP/FN analysis. Allow users to appeal blocks with a clear process.

### Q14: Explain the challenge of PII leakage in LLM systems and how to prevent it.
**Answer**: Models can memorize and regurgitate training data (including PII) and can inadvertently repeat user-provided PII. Prevention: input-side PII redaction before sending to the LLM, output-side PII scanning with regex + NER models, never logging raw inputs/outputs in audit trails, using differential privacy during fine-tuning, and maintaining a blocklist of known sensitive patterns.

## Security

### Q15: What are the most common security threats to production LLM systems?
**Answer**: Prompt injection (user hijacks model behavior), jailbreaking (bypassing safety), data leakage (model memorizes sensitive data), model inversion (recovering training data), supply chain attacks (compromised models/dependencies), API abuse (excessive usage), and indirect prompt injection (attacker-controlled content in RAG context).

### Q16: How would you implement least privilege for an LLM's permissions?
**Answer**: The LLM should only have access to what it needs for its specific task. A customer support LLM should not have database write access. Use scoped API keys, restrict tool/function calling to whitelisted operations, implement human-in-the-loop for destructive actions (deletions, refunds > $X), and time-box all external access. Never give the LLM direct access to internal systems — always use a middleware layer.

### Q17: Design an API key management system for an LLM service.
**Answer**: Keys generated with cryptographic randomness (e.g., `secrets.token_urlsafe(32)`). Store hashed (bcrypt), not plaintext. Support multiple keys per user with rotation policies. Implement key scopes (read-only, write, admin) and expiry. Track last-used timestamp to identify unused keys. Alert on keys older than 90 days. Provide a revocation endpoint with < 5 minute propagation. Never log full keys — log only last 4 characters.

## Rate Limiting & Cost Management

### Q18: How would you implement per-user token budgets in a multi-tenant LLM system?
**Answer**: Track usage in Redis with daily and monthly counters per user. Compare against configurable budgets: soft limit (warn at 80%), hard limit (block at 100%). Use a token bucket or sliding window algorithm for rate limiting. Support different tiers (free: 100K tokens/month, pro: 10M, enterprise: custom). Alert customers before they hit limits. For overage, allow auto-top-up or block gracefully.

### Q19: Describe cost optimization strategies for LLM systems.
**Answer**: (1) Semantic caching — cache embeddings of queries and return cached responses for similar queries. (2) Prompt compression — trim unnecessary context. (3) Model routing — simple queries go to cheap models, complex to expensive. (4) Batch processing — combine requests for async workloads. (5) Token-efficient prompts — shorter system prompts, fewer examples. (6) Fallback to smaller models under load. Typical savings: caching 30-60%, routing 40-60%, compression 20-40%.

## Deployment & Scaling

### Q20: Compare self-hosted vs managed API for LLM deployment.
**Answer**: Self-hosted (vLLM, TGI): lower latency (50-200ms), lower cost at high volume, full data control, but high operational complexity (GPU management, scaling, updates). Managed API (OpenAI, Anthropic): higher latency (200-500ms), pay per token (expensive at scale), no data control, but zero ops overhead. Hybrid approach: use managed API for variable loads, self-hosted for predictable high-volume workloads.

### Q21: How would you implement a circuit breaker for LLM API calls?
**Answer**: Track consecutive failures. After N failures (e.g., 5), open the circuit and fail fast instead of calling the API. After a timeout (e.g., 60s), half-open the circuit and allow one test request. If it succeeds, close the circuit; if it fails, stay open. Track failure types separately (timeout vs rate-limit vs auth error) — don't trip for auth errors that require manual fix. Log all state transitions for debugging.

### Q22: Explain canary deployments for LLM models.
**Answer**: Route a small percentage of traffic (e.g., 5-10%) to a new model version while the rest uses the current version. Monitor key metrics (latency, quality scores, error rates, user feedback) for both versions. If canary metrics are within acceptable range (e.g., quality score >= current - 0.05, error rate <= current + 0.1%), gradually increase traffic. If metrics degrade, rollback immediately. Duration: minimum 4-24 hours depending on traffic volume.

## CI/CD

### Q23: What belongs in a CI/CD pipeline for AI systems that differs from traditional software CI/CD?
**Answer**: Prompt evaluation against golden test set, safety scanning of all prompt variants, latency benchmarking, cost estimation of new prompts/models, data drift detection, model version validation, and evaluation gates that block deployment if quality drops below threshold. Traditional CI/CD checks compile, unit tests, and integration tests — AI CI/CD additionally checks that the AI *behaves correctly and safely*.

### Q24: How do you version control prompts and datasets alongside code?
**Answer**: Store prompts as YAML/JSON files in the repository with semantic versioning. Use symlinks (`production -> v1.2.0`) for environment management. Use DVC (Data Version Control) for datasets — it creates pointer files in git and stores actual data in S3/GCS. Use MLflow for model registry with metrics, parameters, and artifacts. Tag every deployment with git commit hash + model version + dataset version for full reproducibility.

## Incident Response

### Q25: Walk through your response to a production incident where the LLM starts generating harmful content.
**Answer**: (1) IMMEDIATE: Activate kill switch — block all outputs via guardrails, route through a safe fallback. (2) TRIAGE: Identify affected users, time window, and root cause (prompt change? model update? bypassed guardrail?). (3) MITIGATE: Rollback to previous safe version, pin model version, add failing outputs to golden test set. (4) FIX: Strengthen output guardrails, add failing cases to evaluation suite, implement automated safety monitoring. (5) POSTMORTEM: Write incident report, implement action items, update runbook, schedule follow-up review within 30 days.

### Q26: How do you handle a situation where an underlying LLM API (e.g., GPT-4) silently changes behavior and degrades your system?
**Answer**: Pin model version explicitly (use dated versions like `gpt-4-0613` not `gpt-4`). Run daily golden test set evaluation — if scores drop, trigger regression alert. Maintain a shadow deployment that runs the new version in parallel without serving. Compare shadow vs production outputs. Have a fallback plan: rollback to previous pinned version or switch to an alternative provider.

### Q27: Design a logging and tracing system for a multi-model LLM service.
**Answer**: Structured JSON logs with consistent schema (request_id, timestamp, user_id, model, prompt_version, input_tokens, output_tokens, latency_ms, cost, error, guardrails_triggered). Use OpenTelemetry for distributed tracing across services. Store in Elasticsearch or Loki for queryability. Export to your monitoring stack (Grafana, Datadog) for dashboards. Implement sampling for high-volume endpoints (trace 100% of errors, 10% of successes). Retention: 30 days hot, 90 days warm, 1 year cold.

### Q28: How do you collect and use user feedback in a production LLM system?
**Answer**: Collect implicit feedback (did user copy output? edit query? abandon quickly?) and explicit feedback (thumbs up/down, star rating, free text). Link feedback to the specific request_id. Aggregate feedback per prompt version, model, and user segment. Use negative feedback to trigger re-evaluation and potential rollback. Create a feedback dashboard showing trends over time. Implement a feedback loop: collect → analyze → improve → redeploy → monitor.

### Q29: What SLOs would you set for a production LLM system?
**Answer**: Latency: p99 < 5s (target 99% within SLO). Error rate: < 0.5% (target 99.5% success). Uptime: 99.9%. Quality: evaluation score >= 0.8 (target 95% of checks). Cost: +/- 10% of budget. Safety: zero tolerance for harmful outputs. Service objectives: 99.9% of requests complete within 5 seconds with correct output. Error budget: 0.1% of total requests can fail before violating SLO.

### Q30: How would you architect a system to handle a sudden 10x spike in LLM traffic?
**Answer**: (1) Queue-based architecture — requests go to a queue (Redis/SQS), processed at max sustainable rate. (2) Auto-scaling — Kubernetes HPA scales pods based on queue depth or CPU/memory. (3) Rate limiting at the API gateway — prioritize paying customers, queue or drop free tier. (4) Caching — enable aggressive semantic caching (TTL 1 hour). (5) Fallback models — route to cheaper/faster models under load. (6) Graceful degradation — if queue exceeds threshold, return "try again later" instead of blocking. (7) Circuit breakers — if downstream is overwhelmed, fail fast with cached or fallback responses.
