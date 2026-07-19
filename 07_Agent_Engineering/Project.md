# Project: Building a Research Agent

## Overview

Build a fully functional research agent that can:
1. Accept a research question or topic
2. Search the web for relevant information
3. Read and analyze the found content
4. Synthesize findings into a structured report
5. Cite sources properly
6. Handle follow-up questions about the research

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Research Agent                        │
├───────────┬───────────┬───────────┬─────────────────────┤
│  Search   │   Read    │ Synthesize│   Follow-up         │
│  Module   │  Module   │  Module   │   Module            │
├───────────┴───────────┴───────────┴─────────────────────┤
│                    Memory Systems                        │
├───────────┬───────────┬─────────────────────────────────┤
│ Episodic  │  Cache    │   Source Store                  │
│ (past      │ (avoid     │   (URLs, titles,               │
│  searches) │  repeats)  │    excerpts)                   │
└───────────┴───────────┴─────────────────────────────────┘
```

## Requirements

### Core Features

1. **Search module**: Accept queries, return ranked results
2. **Read module**: Fetch and extract content from URLs
3. **Synthesize module**: Generate coherent reports from multiple sources
4. **Citation system**: Track and cite all sources properly
5. **Follow-up system**: Handle multi-turn research conversations
6. **Memory**: Remember past searches and avoid redundancy
7. **Quality control**: Fact-check claims against sources

### Technical Requirements

- Use OpenAI/Anthropic API for the LLM
- Use at least 3 external search/web tools
- Implement a ReAct-style agent loop
- Include proper error handling and retries
- Add streaming progress updates
- Implement a simple caching system
- Write unit tests for each module

## Milestones

### Milestone 1: Basic Search Agent (Week 1)
- Implement a ReAct loop with web search and URL reading tools
- Agent can answer simple factual questions by searching
- Include tool descriptions and parameter schemas
- Test with: "What is the capital of Finland?"

**Deliverables:**
- Working ReAct agent that can search and read
- At least 2 tools: `search_web`, `read_url`
- 5 test queries with recorded trajectories

### Milestone 2: Report Generation (Week 2)
- Add synthesis capability
- Generate structured reports with sections
- Implement citation tracking
- Add source quality scoring

**Deliverables:**
- Agent produces markdown reports with:
  - Executive Summary
  - Key Findings (with citations)
  - Detailed Analysis
  - Conclusion
  - References
- Test with: "What are the latest developments in LLM fine-tuning?"

### Milestone 3: Memory & Caching (Week 3)
- Implement episodic memory for past searches
- Add result caching to avoid redundant searches
- Implement context management for long research sessions
- Add deduplication of sources

**Deliverables:**
- Vector store for past research trajectories
- Cache hits should be 10x faster than fresh searches
- Agent can reference past findings across sessions
- Test with: "Based on your last research, expand on the findings about RLHF."

### Milestone 4: Quality & Safety (Week 4)
- Add fact-checking against sources
- Implement source credibility scoring
- Add confidence scores to claims
- Handle contradictory information
- Add safety guardrails

**Deliverables:**
- Fact-checking module that verifies claims
- Contradiction detection and resolution
- Safety filter for harmful queries
- Test with queries containing conflicting information

### Milestone 5: Multi-Turn & Follow-ups (Week 5)
- Support follow-up questions
- Maintain context across turns
- Allow users to drill down on specific topics
- Support comparative research (multiple topics)

**Deliverables:**
- Multi-turn conversation support
- Contextual follow-up understanding
- Test with a 5-turn research conversation

### Milestone 6: Production Readiness (Week 6)
- Add streaming progress updates
- Implement cost tracking
- Add performance metrics
- Optimize for latency
- Write comprehensive tests
- Create documentation

**Deliverables:**
- Streaming API with progress events
- Cost dashboard
- Performance benchmark results
- User documentation
- Deployment guide

## Starter Code

```python
# research_agent.py
import asyncio
import json
import time
from dataclasses import dataclass, field
from typing import Optional

@dataclass
class Source:
    url: str
    title: str
    content: str
    relevance_score: float = 0.0
    credibility_score: float = 0.5

@dataclass
class ResearchState:
    topic: str
    queries: list[str] = field(default_factory=list)
    sources: list[Source] = field(default_factory=list)
    report: Optional[str] = None
    follow_ups: list[str] = field(default_factory=list)
    status: str = "initialized"

class ResearchAgent:
    def __init__(self, llm_client, search_tool, read_tool):
        self.llm = llm_client
        self.search_tool = search_tool
        self.read_tool = read_tool
        self.memory = {}  # Simple dict for now
        self.cache = {}
        self.cost_tracker = {"llm_calls": 0, "search_calls": 0, "total_tokens": 0}
        self.state = ResearchState(topic="")

    async def research(self, topic: str) -> str:
        """Main research method — orchestrates the full pipeline."""
        self.state = ResearchState(topic=topic)

        await self._update_status("generating_queries")
        queries = await self._generate_queries(topic)

        await self._update_status("searching")
        all_sources = []
        for query in queries:
            sources = await self._search(query)
            all_sources.extend(sources)

        await self._update_status("reading")
        sources = await self._read_sources(all_sources[:5])

        await self._update_status("analyzing")
        analysis = await self._analyze_findings(topic, sources)

        await self._update_status("generating_report")
        report = await self._generate_report(topic, sources, analysis)

        await self._update_status("completed")
        self.state.report = report

        return report

    async def _generate_queries(self, topic: str) -> list[str]:
        """Generate optimal search queries for the topic."""
        prompt = f"""I need to research: {topic}

Generate 3-5 specific search queries that will find the most relevant and 
up-to-date information. Consider different aspects of the topic.

Return as a JSON list of strings."""
        response = await self.llm.chat(prompt)
        self.cost_tracker["llm_calls"] += 1
        try:
            queries = json.loads(response)
            self.state.queries = queries
            return queries
        except json.JSONDecodeError:
            return [topic]  # Fallback

    async def _search(self, query: str) -> list[Source]:
        """Search the web for a query, checking cache first."""
        # Check cache
        cache_key = f"search:{query}"
        if cache_key in self.cache:
            return self.cache[cache_key]

        results = await self.search_tool.search(query, num_results=5)
        self.cost_tracker["search_calls"] += 1

        sources = [
            Source(
                url=r.get("url", ""),
                title=r.get("title", ""),
                content=r.get("snippet", ""),
                relevance_score=r.get("score", 0.5)
            )
            for r in results
        ]

        # Cache results (expire after 1 hour)
        self.cache[cache_key] = sources
        self.cache[f"cache_time:{cache_key}"] = time.time()

        self.state.sources.extend(sources)
        return sources

    async def _read_sources(self, sources: list[Source]) -> list[Source]:
        """Fetch full content from each source URL."""
        for source in sources:
            try:
                content = await self.read_tool.read(source.url)
                source.content = content[:5000]  # Truncate
            except Exception as e:
                source.content = f"Failed to read: {e}"
        return sources

    async def _analyze_findings(self, topic: str, sources: list[Source]) -> dict:
        """Analyze collected sources for key themes and contradictions."""
        sources_text = "\n\n".join(
            f"Source {i+1} ({s.url}):\n{s.content[:1500]}"
            for i, s in enumerate(sources)
        )
        prompt = f"""Analyze these sources about "{topic}":

{sources_text}

Identify:
1. Key themes and findings
2. Areas of agreement across sources
3. Contradictions or disagreements
4. Gaps in the information
5. Credibility assessment of sources

Return as JSON."""
        response = await self.llm.chat(prompt)
        self.cost_tracker["llm_calls"] += 1
        try:
            return json.loads(response)
        except json.JSONDecodeError:
            return {"analysis": response}

    async def _generate_report(self, topic: str, sources: list[Source], 
                               analysis: dict) -> str:
        """Generate a structured research report."""
        sources_text = "\n\n".join(
            f"Source {i+1} ({s.url}): {s.title}\n{s.content[:2000]}"
            for i, s in enumerate(sources)
        )
        prompt = f"""Generate a comprehensive research report about "{topic}".

Sources:
{sources_text}

Analysis:
{json.dumps(analysis, indent=2)}

Report structure:
- Executive Summary (2-3 paragraphs)
- Key Findings (bullet points with source citations)
- Detailed Analysis (sections as needed)
- Areas of Agreement and Disagreement
- Gaps and Limitations
- Conclusion
- References

Use markdown formatting. Cite sources as [1], [2], etc."""
        report = await self.llm.chat(prompt)
        self.cost_tracker["llm_calls"] += 1

        # Add references section
        references = "\n\n## References\n"
        for i, source in enumerate(sources, 1):
            references += f"[{i}] [{source.title}]({source.url})\n"

        return report + references

    async def _update_status(self, status: str):
        self.state.status = status
        # In a real app, emit this as an event
        print(f"[{status}]")

    async def follow_up(self, question: str) -> str:
        """Handle a follow-up question based on current research."""
        if not self.state.report:
            return "Please run a research first."

        prompt = f"""Previous research topic: {self.state.topic}
Previous report: {self.state.report[:3000]}

Follow-up question: {question}

Answer the question based on the previous research.
If you need more information, suggest what to search for."""
        response = await self.llm.chat(prompt)
        self.cost_tracker["llm_calls"] += 1
        self.state.follow_ups.append((question, response))
        return response

    def get_cost_summary(self) -> dict:
        """Return cost tracking summary."""
        return {
            "llm_calls": self.cost_tracker["llm_calls"],
            "search_calls": self.cost_tracker["search_calls"],
            "total_tokens": self.cost_tracker["total_tokens"],
            "estimated_cost_usd": (
                self.cost_tracker["llm_calls"] * 0.01 +  # $0.01 per LLM call
                self.cost_tracker["search_calls"] * 0.002  # $0.002 per search
            )
        }


# Usage example
async def main():
    agent = ResearchAgent(
        llm_client=your_llm_client,
        search_tool=your_search_tool,
        read_tool=your_read_tool
    )

    report = await agent.research(
        "What are the environmental impacts of large language models?"
    )
    print(report)

    # Follow-up
    answer = await agent.follow_up(
        "How does this compare to traditional data center energy use?"
    )
    print(answer)

    print(json.dumps(agent.get_cost_summary(), indent=2))


if __name__ == "__main__":
    asyncio.run(main())
```

## Extension Ideas

### Advanced Features

1. **Multi-language support**: Research in languages other than English
2. **Image analysis**: Include and analyze images from sources
3. **Comparative research**: Compare multiple topics side by side
4. **Trend analysis**: Track how findings change over time
5. **Personalized reports**: Adapt report style to user preferences
6. **Collaborative research**: Multiple research agents working together
7. **Automated alerts**: Monitor topics and generate reports on schedule
8. **API integration**: Serve research capabilities via REST API

### Evaluation Framework

Create a test suite:
```python
test_cases = [
    {
        "topic": "Current state of quantum computing",
        "expected_sections": ["Overview", "Key Players", "Challenges"],
        "min_sources": 3,
        "max_cost": 0.50
    },
    {
        "topic": "Health benefits of the Mediterranean diet",
        "expected_sections": ["Nutrition", "Studies", "Recommendations"],
        "min_sources": 4,
        "max_cost": 0.50
    },
]
```

## Grading Rubric

| Criterion | Excellent (90-100%) | Good (70-89%) | Fair (50-69%) | Poor (<50%) |
|-----------|-------------------|---------------|---------------|-------------|
| **Search** | Optimal queries, diverse sources, cached | Good queries, some diversity | Basic search, few sources | Poor search |
| **Reading** | Full content extraction, error handling | Content extraction works | Partial extraction | Fails |
| **Synthesis** | Excellent report, clear citations | Good report, some citations | Basic summary | Poor quality |
| **Follow-up** | Contextual answers, suggests searches | Understands context | Basic follow-up | Doesn't work |
| **Memory** | Uses past research, avoids redundancy | Basic caching | Some memory | None |
| **Quality** | Fact-checks, handles contradictions | Fact-checks claims | Basic quality | No checking |
| **Errors** | Graceful handling of all failures | Handles common errors | Some error handling | Crashes |
| **Performance** | <10s per search, streaming updates | <30s per search | <60s per search | Slow |
| **Testing** | Comprehensive tests | Good test coverage | Basic tests | No tests |
| **Code** | Clean, modular, documented | Well-organized | Works but messy | Poor quality |
