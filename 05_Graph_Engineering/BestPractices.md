# Best Practices — Production Graph Engineering

> Battle-tested patterns for building reliable, scalable, and maintainable graph-based AI systems.

---

## 1. Schema Design

### DO: Design Schema Before Extraction

Define your entity types, relationship types, and properties before running extraction. A schema-first approach prevents:

- Inconsistent entity types ("Person" vs "person" vs "People")
- Missing critical relationships
- Property type mismatches (strings vs integers vs dates)

```python
SCHEMA = {
    "entities": {
        "Person": {"properties": ["name", "birth_date", "nationality"], "required": ["name"]},
        "Organization": {"properties": ["name", "founded", "industry"], "required": ["name"]},
    },
    "relationships": {
        "WORKS_AT": {"source": "Person", "target": "Organization", "properties": ["since", "role"]},
        "DEVELOPED": {"source": ["Person", "Organization"], "target": "Technology"},
    }
}
```

### DON'T: Over-Engineer the Schema

Start with 5-10 entity types and 5-10 relationship types. Too many types cause:
- LLM confusion during extraction
- Sparse graphs with disconnected components
- Complex query logic

**Rule of thumb:** If a relationship type would be used in fewer than 5% of queries, merge it into a generic type with a property discriminator.

### DO: Use Meaningful Labels

- Node labels: `Person`, `Organization`, `Paper`, `Model` (PascalCase, singular)
- Relationship types: `WORKS_AT`, `DEVELOPED`, `CITES` (UPPER_SNAKE_CASE)
- Property names: `name`, `birthDate`, `foundedYear` (camelCase)

### DO: Add Constraints and Indexes

Always create unique constraints and indexes during schema initialization:

```cypher
// Constraints ensure data integrity
CREATE CONSTRAINT IF NOT EXISTS FOR (p:Person) REQUIRE p.name IS UNIQUE;
CREATE CONSTRAINT IF NOT EXISTS FOR (o:Organization) REQUIRE o.name IS UNIQUE;

// Indexes speed up lookups
CREATE INDEX IF NOT EXISTS FOR (p:Person) ON (p.nationality);
CREATE INDEX IF NOT EXISTS FOR (o:Organization) ON (o.industry);

// Full-text index for hybrid search
CREATE FULLTEXT INDEX IF NOT EXISTS entity_fulltext
FOR (n:Person|Organization|Technology) ON EACH [n.name];
```

---

## 2. Entity Extraction

### DO: Use Two-Pass Extraction for Complex Documents

Pass 1 extracts entities. Pass 2 extracts relationships between discovered entities.

```python
# Pass 1: Entities
entities = extract_entities(text)
# Pass 2: Relationships between discovered entities
relationships = extract_relationships(text, entities)
```

Two-pass reduces relationship extraction errors by giving the LLM the exact entity list.

### DO: Validate Extraction Output

Always validate that the LLM output conforms to your schema:

```python
def validate_extraction(extraction, schema):
    errors = []
    for entity in extraction.get("entities", []):
        if entity["type"] not in schema["entities"]:
            errors.append(f"Unknown entity type: {entity['type']}")
        for prop in schema["entities"].get(entity["type"], {}).get("required", []):
            if prop not in entity.get("properties", {}):
                errors.append(f"Missing required property {prop} for {entity['name']}")
    return errors
```

### DON'T: Extract Everything

Not all text needs to be in the graph. Be selective:

- Extract **domain-relevant** entities (not every noun)
- Extract **verifiable** facts (not opinions or hypotheticals)
- Set a confidence threshold — discard low-confidence extractions
- Skip chunks with no domain-relevant content

### DO: Handle Coreference Resolution

"OpenAI released GPT-4. It is their most capable model."

Resolve "It" → "GPT-4" and "their" → "OpenAI". Without coreference, you lose the relationship between OpenAI and GPT-4 in the second sentence.

### DO: Implement Deduplication at Insert Time

```python
def merge_entity(graph, entity):
    """Merge entity into graph, handling duplicates."""
    # Normalize name
    name = normalize_name(entity["name"])

    # Check for existing entity with similar name
    existing = graph.find_similar(name, threshold=0.85)

    if existing:
        # Merge properties: new values override old, but keep both
        merged = {**existing.properties, **entity.get("properties", {})}
        graph.update(existing.id, merged)
        graph.increment_mention_count(existing.id)
        return existing.id
    else:
        return graph.create_entity(name, entity["type"], entity.get("properties", {}))
```

---

## 3. Graph Database Operations

### DO: Use Parameterized Queries

Never concatenate values into query strings. Use parameters:

```cypher
// GOOD
MATCH (p:Person {name: $name}) RETURN p;

// BAD — SQL injection risk
MATCH (p:Person {name: '" + name + "'}) RETURN p;
```

### DO: Batch Operations

For large inserts, use UNWIND to batch:

```cypher
// Batch insert 1000 entities
UNWIND $entities AS entity
MERGE (e:Entity {name: entity.name})
SET e += entity.properties
```

### DON'T: Use MATCH + CREATE for Large Graphs

```cypher
// SLOW — one query per relationship
MATCH (s:Entity {name: $source})
MATCH (t:Entity {name: $target})
CREATE (s)-[:RELATED]->(t)

// FAST — batch all relationships
UNWIND $relationships AS rel
MATCH (s:Entity {name: rel.source})
MATCH (t:Entity {name: rel.target})
CREATE (s)-[:RELATED]->(t)
```

### DO: Use EXPLAIN and PROFILE

Before running queries in production, profile them:

```cypher
EXPLAIN MATCH (p:Person)-[:WORKS_AT]->(o:Organization)
WHERE o.industry = $industry
RETURN p.name, o.name;

PROFILE MATCH (p:Person)-[:WORKS_AT]->(o:Organization)
WHERE o.industry = $industry
RETURN p.name, o.name;
```

Look for: full node scans, cartesian products, unbounded path patterns.

### DO: Set Timeouts

Always set query timeouts to prevent runaway traversals:

```python
with driver.session() as session:
    result = session.run(
        "MATCH (s:Entity)-[*1..5]-(t:Entity) RETURN s, t",
        timeout=10  # seconds
    )
```

---

## 4. GraphRAG Patterns

### DO: Use Hierarchical Communities

Don't just run community detection once. Build a hierarchy:

```
Level 0: Fine-grained communities (100-1000 nodes each)
Level 1: Mid-level communities (merging related Level 0)
Level 2: Coarse communities (entire topic areas)
```

Query the appropriate level based on question scope:
- **Specific question → Level 0** subgraph + community summary
- **Broad question → Level 2** community summaries
- **Intermediate → Level 1**

### DO: Cache Community Summaries

Community summaries change infrequently. Cache them aggressively:

```python
class CommunitySummaryCache:
    def __init__(self, redis_client):
        self.redis = redis_client
        self.ttl = 3600 * 24  # 24 hours

    def get_summary(self, community_id, level):
        key = f"community:{level}:{community_id}"
        cached = self.redis.get(key)
        if cached:
            return cached

        summary = self.generate_summary(community_id, level)
        self.redis.setex(key, self.ttl, summary)
        return summary
```

### DON'T: Traverse Too Deep

Limit graph traversal to 2-3 hops in most cases:

| Hops | Nodes (branch factor=10) | Context Quality |
|---|---|---|
| 1 | 10 | Very narrow |
| 2 | 100 | Good for most queries |
| 3 | 1,000 | Comprehensive (may have noise) |
| 4 | 10,000 | Mostly noise |
| 5 | 100,000 | Context explosion |

### DO: Format Context for LLM Consumption

Structure graph context so LLMs can easily consume it:

```python
def format_context(subgraph):
    """Format graph triples as structured text for LLM."""
    lines = ["--- Knowledge Graph Context ---"]

    for item in subgraph:
        if isinstance(item, dict) and "triples" in item:
            for triple in item["triples"]:
                lines.append(f"({triple['source']}) -[{triple['type']}]-> ({triple['target']})")

    lines.append(f"Total facts: {len(subgraph)}")
    lines.append("---")
    return "\n".join(lines)
```

### DO: Implement Fallback Chains

If GraphRAG returns empty results, fall back gracefully:

```python
async def query_with_fallback(question):
    # Try GraphRAG
    result = await graphrag_query(question)
    if result.confidence > 0.7:
        return result.answer

    # Fall back to vector RAG
    result = await vector_rag_query(question)
    if result.confidence > 0.7:
        return result.answer

    # Final fallback: LLM knowledge only
    return await llm_direct_query(question)
```

---

## 5. Performance Optimization

### DO: Pre-compute Embeddings

Compute entity embeddings during indexing, not at query time:

```python
class EntityIndexer:
    def index_entity(self, entity):
        text = f"{entity['name']} {entity['type']} {entity.get('properties', {})}"
        entity['embedding'] = self.embedder.encode(text)
        self.graph.create_entity(entity)
```

### DO: Use Approximate Nearest Neighbor

For vector search at scale, use HNSW indexes instead of exact kNN:

```cypher
CREATE VECTOR INDEX entity_embeddings FOR (n:Entity) ON (n.embedding)
OPTIONS {indexConfig: {
  `vector.dimensions`: 1536,
  `vector.similarity_function`: 'cosine',
  `vector.hnsw_m`: 32,
  `vector.hnsw_ef_construction`: 200
}};
```

### DO: Monitor Query Performance

Track these metrics for every query:

```
- Total query time (P50, P95, P99)
- Graph traversal time
- Vector search time
- LLM generation time
- Number of tokens in context
- Number of nodes/edges retrieved
- Cache hit rate
```

### DON'T: Return Full Graphs to the Client

Always truncate, format, and limit graph results before sending to the LLM or client:

```python
def truncate_subgraph(subgraph, max_triples=100):
    """Limit subgraph size to prevent context overflow."""
    if len(subgraph) > max_triples:
        # Keep highest-PageRank triples
        sorted_triples = sorted(
            subgraph,
            key=lambda t: t.get("pagerank", 0),
            reverse=True
        )
        return sorted_triples[:max_triples]
    return subgraph
```

---

## 6. Graph Memory for Agents

### DO: Implement Forgetting

Graphs grow unboundedly without a forgetting mechanism:

```python
def consolidate_memory(graph, max_age_days=30, max_entities=10000):
    """Prune old or low-importance entities."""
    graph.execute("""
        MATCH (e:Entity)
        WHERE e.last_seen < datetime() - duration({days: $max_age})
        AND e.mention_count < 3
        DETACH DELETE e
    """, {"max_age": max_age_days})
```

### DO: Track Source Provenance

Always track where facts came from:

```cypher
CREATE (e:Entity {name: "GPT-4"})
CREATE (e)-[:HAS_FACT {
    fact: "developed_by",
    value: "OpenAI",
    source: "doc_123_chunk_5",
    extracted_at: datetime("2024-03-15T10:30:00"),
    confidence: 0.95
}]->(f:Fact)
```

This enables:
- Fact verification
- Source attribution
- Debugging extraction errors
- Confidence-weighted retrieval

### DO: Handle Contradictions

When new information contradicts existing facts:
1. Keep both with timestamps
2. Let the LLM decide which is current when queried
3. Track contradiction count as a signal for entity resolution issues

```cypher
MATCH (e:Entity {name: "Company X"})
MATCH (e)-[r:ACQUIRED_BY]->(acquirer)
WHERE r.valid_to IS NULL
// New fact: acquired by different company
CREATE (e)-[:ACQUIRED_BY {
    acquirer: "Company Y",
    valid_from: datetime("2024-06"),
    valid_to: null,
    contradicts: id(r)
}]->(new)
```

---

## 7. GraphRAG Production Deployment

### DO: Separate Indexing and Serving

| Phase | Infrastructure | Scheduling |
|---|---|---|
| Entity Extraction | GPU workers (LLM inference) | Event-driven / batch |
| Graph Insertion | Database writer pool | Batch (every N docs / every M minutes) |
| Community Detection | CPU-heavy compute | Periodic (hourly / daily) |
| Query Serving | Load-balanced read replicas | Always on |

### DO: Implement Idempotent Indexing

If a document is indexed twice, the result should be the same:

```python
def index_document_idempotent(doc_id, text, graph):
    """Idempotent document indexing."""
    # Check if already indexed
    existing = graph.execute(
        "MATCH (d:Document {id: $id}) RETURN d",
        {"id": doc_id}
    )
    if existing and existing[0]["d"]["hash"] == hash(text):
        return {"status": "skipped", "reason": "unchanged"}

    # Index (will overwrite/merge)
    # ... extraction, insertion ...
    graph.execute("""
        MERGE (d:Document {id: $id})
        SET d.hash = $hash, d.indexed_at = timestamp()
    """, {"id": doc_id, "hash": hash(text)})
```

### DO: Set Up Monitoring Dashboards

Track these key metrics:

```
GraphRAG Dashboard:
├── Ingestion Pipeline
│   ├── Documents indexed (rate)
│   ├── Entities extracted (rate)
│   ├── Extraction error rate
│   └── Graph size (nodes + edges)
├── Query Performance
│   ├── P50/P95/P99 latency
│   ├── Throughput (QPS)
│   ├── Cache hit rate
│   └── Empty result rate
├── Graph Health
│   ├── Entity resolution conflicts
│   ├── Orphan nodes (no relationships)
│   ├── Average degree
│   └── Community detection coherence
└── Generation Quality
    ├── Factuality score
    ├── User feedback score
    └── Hallucination rate
```

---

## 8. Testing

### DO: Test Extraction with Golden Datasets

Create a labeled test set of documents with expected entities and relationships:

```python
# Golden test set
TEST_SET = [
    {
        "text": "Google DeepMind developed AlphaFold.",
        "expected_entities": [
            {"name": "Google DeepMind", "type": "Organization"},
            {"name": "AlphaFold", "type": "Technology"}
        ],
        "expected_relationships": [
            {"source": "Google DeepMind", "target": "AlphaFold", "type": "developed"}
        ]
    }
]

def test_extraction_pipeline(extraction_fn):
    for test in TEST_SET:
        result = extraction_fn(test["text"])
        assert result["entities"] == test["expected_entities"]
        assert result["relationships"] == test["expected_relationships"]
```

### DO: Test Query Coverage

Before deploying, verify that common queries return useful results:

```python
QUERY_TEST_SET = [
    "What companies work on AI?",
    "Who founded OpenAI?",
    "What models has Google developed?",
    "Which researchers work on both AI safety and alignment?"
]

def test_query_coverage(graphrag, test_set):
    results = []
    for query in test_set:
        answer = graphrag.query(query)
        results.append({
            "query": query,
            "has_answer": len(answer) > 50,
            "has_citations": "[" in answer,
            "tokens_used": count_tokens(answer)
        })
    return results
```

### DO: Load Test Your Database

Before production, benchmark your graph database:

```python
def load_test_graphdb(driver, queries, concurrency=10, iterations=100):
    import concurrent.futures
    import time

    def run_query(query):
        start = time.time()
        with driver.session() as session:
            session.run(query)
        return time.time() - start

    with concurrent.futures.ThreadPoolExecutor(max_workers=concurrency) as executor:
        futures = [executor.submit(run_query, q) for q in queries * iterations]
        latencies = [f.result() for f in futures]

    print(f"P50: {sorted(latencies)[len(latencies)//2]:.3f}s")
    print(f"P95: {sorted(latencies)[int(len(latencies)*0.95)]:.3f}s")
    print(f"P99: {sorted(latencies)[int(len(latencies)*0.99)]:.3f}s")
```

---

## 9. Security

### DO: Sanitize Graph Outputs

Never expose internal graph structures to end users:

```python
def sanitize_for_user(subgraph):
    """Remove internal metadata before returning to user."""
    return [
        {
            "source": item["source"],
            "target": item["target"],
            "relationship": item["type"]
        }
        for item in subgraph
        if item.get("public", True)
    ]
```

### DO: Use Read-Only Credentials for Queries

```python
# Write connection (indexing pipeline only)
write_driver = GraphDatabase.driver(uri, auth=("neo4j", "write_password"))

# Read connection (query serving)
read_driver = GraphDatabase.driver(uri, auth=("neo4j", "read_only_password"))
```

### DO: Rate Limit Graph Queries

```python
import time
from collections import defaultdict

class RateLimiter:
    def __init__(self, max_per_minute=60):
        self.max_per_minute = max_per_minute
        self.usage = defaultdict(list)

    def check(self, user_id):
        now = time.time()
        recent = [t for t in self.usage[user_id] if now - t < 60]
        if len(recent) >= self.max_per_minute:
            raise Exception("Rate limit exceeded")
        self.usage[user_id].append(now)
```

---

## 10. Development Workflow

### DO: Start Small, Iterate

1. **Week 1:** Build with NetworkX (in-memory). 100 documents. Validate the approach.
2. **Week 2:** Migrate to Kuzu (embeddable). 1000 documents. Profile performance.
3. **Week 3-4:** Migrate to Neo4j (production). 10,000+ documents. Add monitoring.
4. **Week 5+:** Scale. Community detection. Hybrid search. Production hardening.

### DO: Version Your Graphs

```python
class VersionedGraph:
    def __init__(self, driver):
        self.driver = driver

    def create_snapshot(self, version_tag):
        """Create a labeled snapshot of the current graph state."""
        self.driver.execute("""
            MATCH (n)
            SET n._snapshot_version = $version
        """, {"version": version_tag})

    def rollback_to(self, version_tag):
        """Restore graph to a previous snapshot."""
        self.driver.execute("""
            MATCH (n)
            WHERE n._snapshot_version IS NOT NULL
            AND n._snapshot_version <> $version
            DETACH DELETE n
        """, {"version": version_tag})
```

### DO: Maintain a Development Playbook

Document for your team:

1. How to add a new entity type
2. How to add a new relationship type
3. How to update extraction prompts
4. How to debug a bad extraction
5. How to rollback a bad graph update
6. How to optimize a slow query

---

## Summary Checklist

- [ ] Schema defined before extraction begins
- [ ] Constraints and indexes created at init
- [ ] Extraction outputs validated against schema
- [ ] Entity resolution implemented with fallbacks
- [ ] Batch inserts used for all database operations
- [ ] Queries use parameterized inputs
- [ ] Community detection hierarchy built
- [ ] Community summaries cached
- [ ] Traversal depth limited (2-3 hops)
- [ ] Embeddings pre-computed at index time
- [ ] Fallback chain implemented (GraphRAG → Vector → LLM)
- [ ] Monitoring dashboard configured
- [ ] Load testing completed
- [ ] Golden test set for extraction quality
- [ ] Read/write database credentials separated
- [ ] Rate limiting applied to query endpoints
