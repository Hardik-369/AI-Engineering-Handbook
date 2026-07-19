# Interview Questions: Building, Deploying, and Scaling LLM Projects

## Architecture & Design

1. **You are designing a customer support chatbot. Should it use RAG, fine-tuning, or both? Walk through your decision process.**  
   *Tests understanding of when to retrieve vs. when to memorize.*

2. **Design a multi-agent system where one agent researches, another writes, and a third critiques the output. How do you handle coordination, state passing, and error recovery?**  
   *Tests agent orchestration patterns.*

3. **Your RAG system returns irrelevant documents 20% of the time. How do you diagnose and fix this? List every possible failure point in the retrieval pipeline.**  
   *Tests systematic debugging of RAG systems.*

4. **Compare and contrast: single-agent with many tools vs. multi-agent with specialized sub-agents. When would you choose each?**  
   *Tests architectural trade-off analysis.*

5. **Design a system that maintains conversation memory across sessions, devices, and channels. How do you handle memory consolidation, decay, and conflict resolution?**  
   *Tests memory system design.*

6. **You need to build an LLM application that processes sensitive financial data. What architectural considerations do you make for security, compliance, and auditability?**  
   *Tests production security awareness.*

7. **How would you design a prompt caching layer? What invalidation strategies would you use? How do you balance cache hit rate with freshness?**  
   *Tests systems design thinking for LLM-specific infrastructure.*

## Prompting & LLM Interaction

8. **Your application uses chain-of-thought prompting, but the model sometimes produces plausible-sounding but incorrect reasoning chains. How do you detect and mitigate this?**  
   *Tests understanding of hallucination and reasoning faithfulness.*

9. **You need structured JSON output from an LLM, but the model occasionally returns malformed JSON. Design a robust parsing pipeline that handles this gracefully.**  
   *Tests practical error handling for LLM outputs.*

10. **How do you design prompts that work consistently across different models (GPT-4o, Claude, Gemini, Llama)? What patterns generalize, and what needs model-specific tuning?**  
    *Tests cross-model prompt engineering expertise.*

11. **You're building a multi-step agent loop. The LLM keeps getting stuck in loops or repeating the same tool call. How do you detect and break these cycles?**  
    *Tests agent loop safety mechanisms.*

## Evaluation & Quality

12. **Design an evaluation framework for an AI code generation agent. What metrics matter? How do you build a test suite that catches regressions?**  
    *Tests understanding of LLM evaluation methodology.*

13. **Your summarization system produces good summaries 90% of the time, but the 10% that fail are critical. How do you identify the failing cases and improve recall?**  
    *Tests evaluation and iterative improvement.*

14. **How do you detect and measure hallucination in a deployed RAG system? What automated and manual approaches work at scale?**  
    *Tests hallucination detection strategies.*

15. **You have human evaluators disagreeing on output quality 30% of the time. How do you resolve inter-annotator disagreement and establish reliable ground truth?**  
    *Tests evaluation methodology maturity.*

## Production & Scaling

16. **Your LLM application needs to handle 1000 requests per second with P99 latency under 2 seconds. Design the infrastructure, including model serving, caching, and load balancing.**  
    *Tests production scaling architecture.*

17. **How do you implement gradual rollout for a new prompt template? Design an experiment that compares old vs. new prompts with statistical significance.**  
    *Tests experimentation and deployment strategy.*

18. **You need to reduce per-request cost by 50% without significantly degrading quality. What techniques do you try? (List at least 5.)**  
    *Tests cost optimization knowledge.*

19. **Design a monitoring and alerting system for a deployed LLM application. What metrics do you track? What thresholds trigger alerts?**  
    *Tests production observability.*

20. **Your deployed chatbot starts producing toxic outputs after a model update. How do you detect this automatically, roll back, and prevent recurrence?**  
    *Tests safety monitoring and incident response.*

## Advanced Topics

21. **Design a system that uses smaller, cheaper models for 80% of queries and routes complex queries to larger models. How do you determine the routing criteria and validate the cost-quality trade-off?**  
    *Tests model routing architecture.*

22. **How would you implement real-time learning for a deployed LLM system? That is, the system should improve from user feedback without explicit retraining.**  
    *Tests online learning approaches for LLMs.*

23. **You are building a system that requires the LLM to reason about time, causality, and counterfactuals. What prompting techniques and architectural patterns support this?**  
    *Tests advanced reasoning support.*

24. **Design a privacy-preserving RAG system where sensitive documents are indexed without exposing their contents to the LLM provider or the system operator.**  
    *Tests privacy engineering for LLM systems.*

25. **How do you build a test suite for an LLM-powered application that is robust to non-deterministic model outputs? What's the difference between testing for correctness vs. testing for safety?**  
    *Tests testing philosophy for non-deterministic systems.*

26. **Compare and contrast: fine-tuning a small model on task-specific data vs. using few-shot prompting with a large model. Under what conditions does each approach win?**  
    *Tests understanding of the fine-tuning vs. prompting trade-off.*

27. **Design a caching strategy for LLM responses that handles semantically similar but not identical queries (e.g., "What's the weather in Paris?" vs. "Is it raining in Paris?").**  
    *Tests semantic caching design.*

28. **You're tasked with building an LLM-based system that must meet enterprise SLAs. What specific SLAs would you define, and how would you measure them?**  
    *Tests production reliability engineering for LLMs.*

29. **How do you handle the cold start problem for a personalized LLM system — when a new user arrives with no history?**  
    *Tests personalization strategy.*

30. **You observe that your agent's performance degrades as the conversation gets longer (more turns). Is this a context window issue, a memory issue, or something else? How do you diagnose and fix it?**  
    *Tests long-context degradation diagnosis.*
