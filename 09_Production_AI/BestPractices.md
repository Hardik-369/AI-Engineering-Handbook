# Chapter 09: Production AI — Best Practices

## Monitoring & Observability

- **Monitor everything from day one.** Don't wait until something breaks to add monitoring. Log every request, response, latency, token count, error, and guardrail trigger.
- **Track p50, p95, and p99 latency separately.** They tell different stories. p50 is the typical experience, p99 is the worst-case experience. Optimize for p99 without sacrificing p50.
- **Know your cost per request to the millicent.** AI costs scale linearly with volume. A $0.01 API call × 1M requests = $10,000. Track this from the start.
- **Use structured logging with a consistent schema.** Every service should emit logs in the same format with the same field names. This makes querying and alerting across services possible.
- **Set up alerts, not just dashboards.** A dashboard you check once a week is not monitoring. Alerts should page someone when things go wrong, with clear severity levels and runbooks.
- **Monitor model drift passively.** Track distributions of output length, sentiment, refusal rates, and topic over time without needing test queries. Sudden changes often indicate a model update or drift.
- **Log the prompt version with every request.** You cannot debug a production issue without knowing exactly which prompt was used. Include the version hash in every request log.
- **Implement sampling for high-volume endpoints.** Trace 100% of errors and guardrail triggers, but sample successful requests (e.g., 10%) to control storage costs.

## Evaluation

- **Automate evaluation, but validate with humans.** Automated metrics catch regressions quickly. Human evaluation catches subtle issues that metrics miss. Use both.
- **Maintain a golden test set.** Curate real production queries, write expected responses, version control them. Run this set against every prompt change. Update quarterly.
- **Test with real production data, not curated examples.** Your golden test set should sample from actual production traffic, not hand-picked examples that your prompts were designed for.
- **Use multiple evaluation metrics.** No single metric captures quality. Combine BERTScore (semantic similarity), LLM-as-judge (nuanced assessment), and task-specific metrics.
- **Track evaluation trends, not just point scores.** A single evaluation run tells you if you passed. A trend over weeks tells you if your system is degrading — often more important.
- **Test edge cases explicitly.** Empty inputs, very long inputs, ambiguous queries, contradictory instructions, non-English inputs, and adversarial inputs all need test cases.
- **Pin your judge model version.** If your LLM-as-judge evaluator changes, your evaluation results become incomparable. Version-lock the judge model just like you version-lock your production model.
- **Evaluate cost alongside quality.** The best prompt is not the one with the highest quality score — it's the one with the best quality-to-cost ratio.

## Prompt Management

- **Treat prompts as code.** Version control them. Review them in PRs. Test them before deploying. Roll them back when they break.
- **One prompt per file.** Don't cram multiple prompts into a single notebook or config file. Each prompt gets its own file with clear naming and documentation.
- **Pin model versions in prompts.** `model: gpt-4-0613` not `model: gpt-4`. Silent model updates are a common source of production regressions.
- **Parameterize prompts, don't hardcode.** Temperature, max tokens, top_p, and model selection should be configurable per environment (dev/staging/prod).
- **Keep system prompts short and focused.** Long system prompts cost more tokens and give the model more surface area to misinterpret. Be as concise as possible while being clear.
- **Test prompt changes against your full golden set.** A change that improves one type of query might break another. Regression testing catches this.

## Safety & Guardrails

- **Layer your guardrails.** No single guardrail is perfect. Use input guardrails AND output guardrails AND conversation-level guardrails. Defense in depth.
- **Log every guardrail trigger.** If you don't log false positives, you can't tune your thresholds. If you don't log hits, you can't measure effectiveness.
- **Err on the side of blocking for safety issues.** A false positive (blocking a legitimate request) is a user annoyance. A false negative (allowing harmful content) is a PR disaster and regulatory risk.
- **Test guardrails with adversarial inputs.** Maintain a test suite of known attack patterns. Run it every time you update guardrails. Track detection rate over time.
- **Implement a kill switch.** A mechanism to immediately block all outputs or route to a safe fallback when a safety incident is detected. Test it regularly.
- **Don't put all guardrails in the prompt.** Relying on "Don't output harmful content" in the system prompt is not guardrails. Implement actual validation layers outside the model.
- **Balance security and user experience.** Overly aggressive guardrails frustrate users. Monitor false positive rates and tune thresholds regularly.

## Security

- **Never hardcode API keys.** In code, in config files, in CI/CD variables that print in logs. Use a secrets manager (Vault, AWS Secrets Manager, environment variables from a secure source).
- **Rotate API keys regularly.** Every 90 days minimum. Immediately on compromise suspicion. Support multiple keys per user so rotation doesn't require downtime.
- **Implement rate limiting at multiple levels.** Per user, per IP, per API endpoint. Use token bucket or sliding window algorithms. Communicate limits clearly in API responses.
- **Validate all input before it reaches the LLM.** Strip control characters, limit length, validate encoding, check for injection patterns. A compromised input can compromise your system.
- **Audit everything.** Every request, every response, every guardrail trigger, every admin action. Immutable audit logs with 90-day retention minimum.
- **Practice least privilege for LLM access.** The LLM should only call the tools and access the data it needs. Never give an LLM direct database access or admin permissions.
- **Run regular red team exercises.** Attack your own system. Find weaknesses before someone else does. Fix them. Repeat.

## Cost Management

- **Set budgets per user, per team, per model.** Don't let one customer's usage bankrupt your margins. Hard limits are non-negotiable — soft limits give warning.
- **Cache aggressively.** Exact match caching is simple. Semantic caching (cache responses for similar queries) saves 30-60% on costs. The investment in implementation pays for itself quickly.
- **Use prompt compression.** Remove unnecessary context, trim few-shot examples, use shorter system prompts. Every token costs money — don't waste it.
- **Route requests to the cheapest appropriate model.** Simple queries (summarization, simple Q&A) should not use GPT-4. Classify query complexity and route accordingly.
- **Monitor cost per request trend, not just absolute cost.** A rising cost per request indicates prompt bloat, model changes, or inefficient usage patterns.
- **Alert on cost anomalies.** If daily cost is 2x the rolling average, investigate. A runaway loop or misconfigured retry policy can burn thousands in minutes.
- **Negotiate volume discounts.** If you do >$10K/month with an API provider, you can negotiate custom pricing. Don't pay list price at scale.

## Deployment

- **Use Kubernetes for LLM services.** It provides auto-scaling, rolling updates, health checks, and resource management. The learning curve is worth it for production systems.
- **Configure health checks properly.** LLM models take time to load (30-120s). Use startup probes with long initial delays. Liveness probes should not depend on the model — check process health, not inference.
- **Implement canary deployments for model changes.** Never roll out a new model to all users at once. Start with 5-10% traffic, monitor for 24 hours, then gradually increase.
- **Use GPU inference servers.** vLLM (PagedAttention for high throughput), TGI (Hugging Face optimized), or Triton (enterprise, multi-framework). They handle batching, caching, and memory management.
- **Separate inference from application logic.** Deploy models on GPU nodes, your application on CPU nodes. They scale independently and failures in one don't affect the other.
- **Pin dependencies.** Lock model versions, inference server versions, and library versions. A minor update can change behavior silently.
- **Plan for cold starts.** Model loading takes minutes. Keep minimum replicas warm. Use pod priority and preemption to ensure model pods get GPU resources first.

## Scaling

- **Use queues, not synchronous calls.** For any workload that doesn't need a real-time response, use a queue. It decouples production from consumption, handles spikes gracefully, and enables retries.
- **Implement circuit breakers.** Don't keep calling an API that's failing. Trip after N failures, retry after a timeout, and fail fast in between.
- **Build fallback chains.** Have a hierarchy of models: primary → fallback → emergency. If GPT-4 is down, use GPT-3.5. If GPT-3.5 is down, use your self-hosted model.
- **Use concurrency limits.** Don't send 1000 simultaneous requests to an API. Implement a semaphore or thread pool with a configurable maximum concurrency.
- **Implement retries with exponential backoff.** And jitter. And a maximum retry count. Three retries with backoff handles most transient failures. More than that usually indicates a persistent problem.
- **Prefer horizontal scaling.** More instances of smaller models is usually better than one instance of a huge model. It's more resilient and easier to manage.

## CI/CD

- **Run evaluation on every PR that touches prompts, models, or datasets.** Quality regression is silent until users complain. Automated evaluation catches it before merge.
- **Gate deployments on evaluation scores.** If the golden test set score drops below a threshold, the PR should not be merged and the deployment should not proceed.
- **Version everything together.** Tag deployments with (code version, prompt version, model version, dataset version). Reproducibility depends on knowing all four.
- **Test in a staging environment first.** Staging should mirror production (same model, same guardrails, same data). Shadow traffic from production to staging for comparison.
- **Automate rollback.** If a deployment degrades quality, the pipeline should automatically rollback. Manual rollback is too slow for safety issues.
- **Include cost estimation in the pipeline.** A prompt change might double token usage. The CI pipeline should estimate the cost impact before deployment.

## Incident Response

- **Have runbooks for common incidents.** Cost spike, latency degradation, quality regression, safety failure, model outage, data leak. Write them down. Test them in drills.
- **Automate the first response.** A script that blocks a user, activates a kill switch, or fails over to a backup model should be one click or automatic.
- **Do postmortems without blame.** The question is "what can we change so this never happens again?" not "whose fault was this?".
- **Track incident metrics.** Time to detection, time to resolution, number of incidents per week, mean time between failures. Measure and improve.
- **Communicate during incidents.** Status page updates for users, internal Slack for the team. Silence makes users and stakeholders more anxious.
- **Invest in prevention after every incident.** Every incident should produce at least one action item that prevents recurrence. If the same incident happens twice, you didn't learn.
- **Test your incident response.** Run table-top exercises quarterly. Simulate a model failure, a cost spike, a safety breach. Practice makes real incidents less chaotic.

## Logging & Tracing

- **Use correlation IDs.** Every request gets a unique ID that flows through every service. You should be able to trace a single user request from API gateway → guardrails → LLM → output validation → response.
- **Log everything, but store smartly.** Hot storage (30 days) for recent logs. Warm storage (90 days) for recent history. Cold storage (1 year) for compliance and analysis.
- **Don't log PII in raw payloads.** Redact sensitive fields before logging. Log that a PII field was present, not the value itself.
- **Trace all errors to root cause.** Every error log should include enough context (request ID, model, prompt version, timestamp) to reproduce the issue without guessing.
- **Collect user feedback systematically.** Thumbs up/down, ratings, free text feedback. Link it to request IDs for analysis. Use it to trigger quality investigations.
- **Make logs queryable.** Store in Elasticsearch, Loki, or similar. Engineers should be able to find "all requests from user X with latency > 5s in the last hour" in seconds.
- **Audit all production changes.** Who deployed what, when, and what changed. Immutable audit logs for compliance and debugging.
