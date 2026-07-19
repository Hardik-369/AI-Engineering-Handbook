# Interview Questions: Context Engineering

## Foundational

**Q1:** What is context engineering and why is it important as models get larger context windows?

**Q2:** Explain the relationship between context windows and attention mechanisms. Why does performance degrade with extremely long contexts?

**Q3:** What is the "lost in the middle" problem? Who published the original paper and what were the key findings?

**Q4:** How do context windows differ across major model families (OpenAI, Anthropic, Google, open-source)? What are the practical implications?

**Q5:** Explain the difference between a model's context window limit and its "effective" context window. Why aren't they the same?

## Retrieval & Chunking

**Q6:** Compare sparse retrieval (BM25) with dense retrieval (embeddings). When would you use each? What are the failure modes?

**Q7:** What is hybrid retrieval and how does Reciprocal Rank Fusion (RRF) work? When is it worth the complexity?

**Q8:** Explain the role of a reranker in a retrieval pipeline. Why is it more important than the initial retriever in many cases?

**Q9:** What chunking strategies have you used? Compare fixed-size, recursive, and semantic chunking. How do you choose chunk size?

**Q10:** What is the optimal chunk size for a RAG system? How does it depend on the use case?

**Q11:** How does chunk overlap affect retrieval performance? What's the tradeoff between recall and token waste?

## Context Packing & Ordering

**Q12:** Given 5 retrieved documents with varying relevance scores, how would you order them in context? Defend your strategy with reference to the lost in the middle problem.

**Q13:** What structured formats (XML, JSON, markdown) work best for packing context? Does structure affect model performance?

**Q14:** If you have only 4,000 tokens of budget for retrieved documents but have 10 relevant documents totaling 8,000 tokens, what strategies can you use?

## Memory Systems

**Q15:** Describe the three-tier memory hierarchy (working, episodic, semantic). How do they differ in access patterns and storage format?

**Q16:** How would you design a memory system for a customer support agent that handles multi-session conversations?

**Q17:** What are the tradeoffs between FIFO sliding windows, summary-buffer windows, and importance-scored windows?

**Q18:** When should you rewrite a memory summary vs. append to it? What factors influence this decision?

## Compression & Efficiency

**Q19:** Compare extractive vs. abstractive context compression. What are the quality vs. cost tradeoffs?

**Q20:** How does LLMLingua or AutoCompressor work? What type of compression ratios can they achieve?

**Q21:** You have a 200K context window model but only 32K of truly useful information. How do you design a system that doesn't waste the remaining capacity?

## Production Systems

**Q22:** Design a context management system for a coding assistant that works on a large codebase across hundreds of conversation turns. How do you manage system prompts, retrieved code, and conversation history?

**Q23:** How would you measure the effectiveness of your context management system? What metrics matter?

**Q24:** A user reports that your RAG system was performing well but degraded after 10+ turns. What could be wrong and how would you diagnose it?

**Q25:** How would you implement adaptive context management — where the system dynamically adjusts its context budget allocation based on the query type?

**Q26:** What is the "core context" pattern? How does it structure agent working memory?

## Advanced

**Q27:** How do positional encodings (absolute vs. relative vs. rotary) affect the lost in the middle problem?

**Q28:** If a model uses RoPE (Rotary Position Embedding), how does that change context engineering strategies compared to models with absolute positional encodings?

**Q29:** Design a context management system for a multi-agent system where agents share a common memory. How do you prevent context pollution between agents?

**Q30:** How would you implement a system that dynamically decides what to retrieve based on the current context utilization? For example, retrieving more aggressively when context is sparsely used.

**Q31:** What is the relationship between context engineering and prompt engineering? Where do they overlap and where do they diverge?

**Q32:** How would you evaluate whether your context compression is losing too much information? Design an evaluation protocol.

**Q33:** If context windows grow to 10M tokens, does context engineering become obsolete? Argue both sides.
