# Interview Questions: Agent Engineering

## Fundamentals

1. **What is an AI agent? How does it differ from a simple LLM call?**

2. **Explain the perceive-think-act loop in agent systems.**

3. **What are the six core components of an agent architecture?**

4. **Describe the difference between a stateless and a stateful agent.**

5. **What is the ReAct pattern? Walk through a complete ReAct cycle.**

6. **How does tool use extend the capabilities of an LLM-based agent?**

7. **Explain the difference between short-term, long-term, episodic, and semantic memory in agents.**

## Planning & Reasoning

8. **Compare and contrast ReAct, Plan-ahead, and Hierarchical planning. When would you use each?**

9. **How do you handle plan failures? Describe re-planning strategies.**

10. **What is chain-of-thought prompting, and how does it relate to agent planning?**

11. **How would you implement an agent that can dynamically adjust its plan based on intermediate results?**

12. **Explain the concept of "tool selection" — how does an agent decide which tool to use?**

## Memory

13. **How would you implement long-term memory for an agent that operates over multiple sessions?**

14. **Describe a strategy for managing the LLM context window when dealing with long agent trajectories.**

15. **What is episodic memory in the context of agents? How is it retrieved and used?**

16. **How would you implement a summarization strategy to compress agent conversation history?**

17. **Explain how vector databases are used in agent memory systems.**

## Tools & Functions

18. **How do you define a tool for an LLM? What makes a good tool description?**

19. **What is parallel tool calling? When would you use it?**

20. **How do you handle tool failures? Describe error recovery strategies.**

21. **Explain how you would implement forced tool calling.**

22. **How would you design a tool that returns large amounts of data? How do you prevent context overflow?**

## Multi-Agent Systems

23. **When would you use a single agent vs. a multi-agent system? What are the trade-offs?**

24. **Describe the supervisor agent pattern. How does delegation work?**

25. **Explain how agents can collaborate through shared memory.**

26. **What is the debate pattern for multi-agent systems? When is it effective?**

27. **How do you handle conflicts between agents in a multi-agent system?**

## Safety & Reliability

28. **What safety mechanisms would you implement for an agent that can execute code?**

29. **Explain human-in-the-loop patterns for agent systems.**

30. **How do you implement budget controls for cost management in production agents?**

31. **Describe how you would implement approval gates for high-risk actions.**

32. **What is sandboxed execution? Why is it important for agent safety?**

33. **How do you handle rate limiting in an agent that makes many API calls?**

## Production & Evaluation

34. **What metrics would you track for a production agent system?**

35. **How do you evaluate an agent? What's different from evaluating an LLM?**

36. **Explain how you would set up observability for an agent system.**

37. **What circuit breaker patterns are useful for agent systems?**

38. **How do you measure and optimize cost per task for an agent?**

39. **Describe a failure analysis process for agent systems.**

40. **What strategies would you use to reduce latency in an agent loop?**

## Design & Architecture

41. **Design an agent system for automated customer support. What components would you include?**

42. **How would you architect an agent that needs to be both fast (for simple queries) and thorough (for complex ones)?**

43. **Design a research agent that can search, read, synthesize, and generate reports. How would you ensure quality?**

44. **How would you build an agent that learns from user feedback over time?**

45. **Design a multi-agent code review system. What specialists would you include?**

## Advanced Topics

46. **Explain the Reflexion pattern (Shinn et al., 2023). How does it enable self-improvement in agents?**

47. **How would you implement agent memory that persists across thousands of conversations?**

48. **Describe how you would implement a hierarchical planning system with 3+ levels of abstraction.**

49. **How would you build an agent benchmark? What tasks would you include?**

50. **What are the failure modes of AI agents? How do you detect and recover from each?**

51. **Explain how you would implement distributed tracing across multiple agents in a multi-agent system.**

52. **How would you design an agent that can create and register new tools dynamically at runtime?**

53. **Describe a system for automated agent-to-agent negotiation. When would this be useful?**

54. **How would you implement a caching layer for an agent system to reduce costs?**

55. **What approaches exist for detecting and preventing prompt injection in agent systems?**

## System Design Questions

56. **Design a production agent system that handles 10,000 concurrent users. Cover: scaling, cost, latency, reliability.**

57. **Design an agent evaluation pipeline. How would you automate regression testing?**

58. **How would you build a system that uses different LLMs for different types of tasks (cheap vs. expensive models)?**

59. **Design an agent system that can recover from mid-task failures without losing progress.**

60. **How would you implement A/B testing for agent strategies (e.g., ReAct vs. Plan-ahead) in production?**

## Behavioral Questions

61. **Tell me about a time you debugged a complex agent failure. What was the root cause?**

62. **Describe a situation where you had to balance agent capability with safety. How did you decide?**

63. **How do you stay updated on the rapidly evolving field of AI agents?**

64. **Tell me about a time you optimized an agent system. What improvements did you make?**

65. **Describe a challenging multi-agent coordination problem you solved.**
