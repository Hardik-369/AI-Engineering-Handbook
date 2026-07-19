# Cheat Sheet — Graph Engineering

> Quick reference for graph databases, Cypher queries, GraphRAG, and graph-based AI patterns.

---

## Graph Database Comparison

| Feature | Neo4j | Kuzu | Memgraph |
|---|---|---|---|
| **Deployment** | Server/Cloud | Embedded | Server |
| **Query Language** | Cypher | Cypher (subset) | Cypher |
| **Storage** | Native graph (disk) | Columnar (disk) | In-memory |
| **ACID** | Yes | Yes | Yes |
| **Vector Index** | Yes (5.11+) | No | Yes |
| **Streaming** | CDC only | No | Kafka/Pulsar |
| **License** | Community/Commercial | MIT | Community/Commercial |
| **Best For** | Production systems | Local/analytics | Real-time/streaming |

---

## Cypher Quick Reference

### Creating Data

```cypher
// Node with labels and properties
CREATE (p:Person {name: "Alice", age: 30});

// Relationship with properties
MATCH (a:Person {name: "Alice"}), (b:Person {name: "Bob"})
CREATE (a)-[:KNOWS {since: 2020}]->(b);

// Merge (create if not exists, update if exists)
MERGE (p:Person {name: "Alice"})
ON CREATE SET p.age = 30
ON MATCH SET p.last_seen = timestamp();

// Batch create
UNWIND $people AS person
CREATE (p:Person) SET p = person;
```

### Reading Data

```cypher
// Match all
MATCH (p:Person) RETURN p;

// Filter
MATCH (p:Person) WHERE p.age > 25 AND p.name STARTS WITH "A" RETURN p;

// Pattern match
MATCH (p:Person)-[:WORKS_AT]->(o:Organization) RETURN p.name, o.name;

// Optional match (left outer join)
MATCH (p:Person)
OPTIONAL MATCH (p)-[:WORKS_AT]->(o:Organization)
RETURN p.name, o.name;
```

### Traversal

```cypher
// Fixed depth
MATCH (p:Person {name: "Alice"})-[:KNOWS]->(friends) RETURN friends;

// Variable depth (1 to 3 hops)
MATCH (p:Person {name: "Alice"})-[:KNOWS*1..3]->(network) RETURN network;

// Shortest path
MATCH path = shortestPath((a:Person {name: "Alice"})-[*]-(b:Person {name: "Bob"}))
RETURN path;

// All paths up to 4 hops
MATCH path = (a:Entity {name: "AI"})-[*..4]-(b:Entity {name: "Healthcare"})
RETURN path;
```

### Aggregation

```cypher
// Count relationships per type
MATCH ()-[r]->() RETURN type(r) AS type, count(*) AS count;

// Average property
MATCH (p:Person) RETURN avg(p.age) AS avg_age;

// Group by
MATCH (p:Person)-[:WORKS_AT]->(o:Organization)
RETURN o.name, count(p) AS employee_count;
```

### Indexes and Constraints

```cypher
// Unique constraint
CREATE CONSTRAINT FOR (p:Person) REQUIRE p.name IS UNIQUE;

// Single-property index
CREATE INDEX FOR (p:Person) ON (p.age);

// Composite index
CREATE INDEX FOR (p:Person) ON (p.name, p.nationality);

// Full-text index
CREATE FULLTEXT INDEX entity_text FOR (n:Entity) ON EACH [n.name, n.description];

// Vector index (Neo4j 5.11+)
CREATE VECTOR INDEX entity_embeddings FOR (n:Entity) ON (n.embedding)
OPTIONS {indexConfig: {
  `vector.dimensions`: 1536,
  `vector.similarity_function`: 'cosine'
}};
```

### Performance

```cypher
// Profile a query
PROFILE MATCH (p:Person) WHERE p.name = "Alice" RETURN p;

// Explain query plan (no execution)
EXPLAIN MATCH (p:Person)-[:WORKS_AT]->(o:Organization) RETURN p, o;
```

---

## Entity Extraction Prompts

### Schema-First Extraction

```
Extract entities and relationships from the text.

Entity Types: Person, Organization, Technology, Location, Event
Relationship Types: works_at, developed, founded, located_in, acquired, partnered_with

Text: {text}

Return JSON:
{
  "entities": [{"name": str, "type": str, "properties": {}}],
  "relationships": [{"source": str, "target": str, "type": str}]
}
```

### Two-Pass Extraction

```
Pass 1: "Extract all named entities of types {types}. Return JSON list."

Pass 2: "Given these entities: {entities}, find relationships between them
         of types {types}. Return JSON list of {source, target, type}."
```

### Few-Shot Extraction

```
Extract entities and relationships. Follow the examples.

Example:
Text: "OpenAI released GPT-4."
Entities: [{"name": "OpenAI", "type": "Organization"}, {"name": "GPT-4", "type": "Model"}]
Relationships: [{"source": "OpenAI", "target": "GPT-4", "type": "developed"}]

Text: {text}
Entities:
Relationships:
```

---

## GraphRAG Pipeline

### Indexing

```
Documents → Chunk (300-600 tokens, 10-20% overlap)
         → Extract entities (LLM per chunk)
         → Extract relationships (LLM)
         → Resolve entities (dedup)
         → Insert into graph DB
         → Compute embeddings (for entity nodes)
```

### Community Detection

```
Graph → Leiden Algorithm
     → Level 0: fine-grained communities (100-1000 nodes)
     → Level 1: mid-level communities
     → Level 2: coarse communities
     → LLM summary per community
     → Cache summaries
```

### Retrieval

```
Query → Extract query entities
     → Match entities in graph
     → Traverse 1-3 hops for subgraph
     → Retrieve community summaries (appropriate level)
     → Format context: triples + summaries
     → LLM generation
     → Fallback: Vector RAG → Direct LLM
```

---

## GraphRAG vs RAG

| Capability | RAG | GraphRAG |
|---|---|---|
| **Retrieval unit** | Text chunks | Entities + relationships |
| **Multi-hop queries** | Difficult | Natural via edge hops |
| **Summarization** | Per-chunk | Hierarchical (communities) |
| **Entity resolution** | Not built-in | Built-in |
| **Query-focused** | Top-K similarity | Subgraph + community |
| **Infrastructure** | Vector DB + LLM | Graph DB + Vector DB + LLM |
| **Indexing cost** | Low | High (LLM extraction) |
| **Context quality** | Variable | Structured, verifiable |

---

## Graph Algorithms

| Algorithm | Purpose | Cypher/Usage |
|---|---|---|
| **PageRank** | Node importance | `CALL gds.pageRank.stream('graph')` |
| **Leiden/Louvain** | Community detection | `CALL gds.leiden.stream('graph')` |
| **Shortest Path** | Minimal connection | `shortestPath((a)-[*]-(b))` |
| **BFS** | Level-order traversal | `(a)-[*..n]-(b)` (breadth-first by default) |
| **Betweenness** | Bridge entities | `CALL gds.betweenness.stream('graph')` |
| **Degree Centrality** | Connectedness | `size((n)--())` |

---

## Hybrid Search Fusion

### Reciprocal Rank Fusion (RRF)

```
Score(item) = Σ(1 / (k + rank_strategy(item)))
```

Default k = 60. Smaller k gives more weight to top ranks.

### Fusion Methods Compared

```python
# RRF — best overall
def rrf(lists, k=60):
    scores = {}
    for results in lists:
        for rank, item in enumerate(results):
            scores[item["id"]] = scores.get(item["id"], 0) + 1 / (k + rank + 1)
    return sorted(scores, key=lambda x: x[1], reverse=True)

# Simple — rank sum
def rank_sum(lists):
    scores = {}
    for results in lists:
        for rank, item in enumerate(results):
            scores[item["id"]] = scores.get(item["id"], 0) + rank
    return sorted(scores, key=lambda x: x[1])
```

---

## Python Library Quick Reference

### Neo4j

```python
from neo4j import GraphDatabase
driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))
with driver.session() as session:
    result = session.run("MATCH (n) RETURN count(n) AS c")
    print(result.data())
```

### Kuzu

```python
import kuzu
db = kuzu.Database("my_db")
conn = kuzu.Connection(db)
conn.execute("CREATE NODE TABLE Person (name STRING, PRIMARY KEY (name))")
conn.execute("CREATE (p:Person {name: 'Alice'})")
result = conn.execute("MATCH (p:Person) RETURN p.name")
```

### NetworkX

```python
import networkx as nx
G = nx.Graph()
G.add_edge("Alice", "Bob", relationship="knows")
nx.pagerank(G)
nx.shortest_path(G, "Alice", "Charlie")
```

### LangChain GraphRAG

```python
from langchain_community.graphs import Neo4jGraph
from langchain.chains import GraphCypherQAChain
graph = Neo4jGraph(url="bolt://localhost:7687", username="neo4j", password="password")
chain = GraphCypherQAChain.from_llm(llm=llm, graph=graph, verbose=True)
chain.invoke({"query": "What entities are connected to OpenAI?"})
```

### RDFLib

```python
import rdflib
g = rdflib.Graph()
g.parse("data.ttl", format="turtle")
results = g.query("SELECT ?s ?p ?o WHERE {?s ?p ?o}")
```

---

## Common Cypher Patterns

### Find All Neighbors

```cypher
MATCH (e:Entity {name: "OpenAI"})-[r]-(neighbor)
RETURN neighbor.name, type(r), r.properties;
```

### Find Shared Connections

```cypher
MATCH (a:Entity {name: "Alice"})-[:KNOWS]-(mutual)-[:KNOWS]-(b:Entity {name: "Bob"})
RETURN mutual.name;
```

### Find Entities by Text Search

```cypher
CALL db.index.fulltext.queryNodes('entity_text', 'neural network')
YIELD node, score
RETURN node.name, score;
```

### Find by Vector Similarity

```cypher
CALL db.index.vector.queryNodes('entity_embeddings', 10, $query_embedding)
YIELD node, score
RETURN node.name, score;
```

### Temporal Query (Point-in-Time)

```cypher
MATCH (e:Entity {name: "Sam Altman"})-[r:EMPLOYED]->(o:Organization)
WHERE r.from <= $timestamp AND (r.to IS NULL OR r.to >= $timestamp)
RETURN o.name, r.role;
```

### Aggregate Graph Statistics

```cypher
// Graph density (approximate)
MATCH (n) WITH count(n) AS nodes
MATCH ()-[r]->() WITH nodes, count(r) AS edges
RETURN nodes, edges, (2.0 * edges) / (nodes * (nodes - 1)) AS density;
```

---

## JSON Structures

### Extraction Output

```json
{
  "entities": [
    {"name": "OpenAI", "type": "Organization", "properties": {"founded": 2015}},
    {"name": "GPT-4", "type": "Model", "properties": {"version": "2023"}}
  ],
  "relationships": [
    {"source": "OpenAI", "target": "GPT-4", "type": "developed", "properties": {}}
  ]
}
```

### Community Summary

```json
{
  "community_id": 42,
  "level": 0,
  "size": 150,
  "top_entities": ["AI", "Machine Learning", "Neural Networks", "Deep Learning"],
  "summary": "This community covers foundational AI/ML concepts..."
}
```

### GraphRAG Query Response

```json
{
  "query": "What did OpenAI develop?",
  "answer": "OpenAI developed GPT-4, a large language model...",
  "context": {
    "entities_used": ["OpenAI", "GPT-4"],
    "triples_retrieved": 15,
    "communities_used": [42],
    "confidence": 0.92
  },
  "latency_ms": {
    "entity_extraction": 350,
    "graph_traversal": 45,
    "context_building": 12,
    "llm_generation": 1200
  }
}
```

---

## Environment Variables

```bash
# Neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your_password

# Kuzu
KUZU_DATABASE_PATH=./knowledge_graph

# Memgraph
MEMGRAPH_URI=bolt://localhost:7687

# OpenAI
OPENAI_API_KEY=sk-...
OPENAI_EMBEDDING_MODEL=text-embedding-3-small
OPENAI_LLM_MODEL=gpt-4

# Vector DB (if separate)
VECTOR_DB_URL=localhost:8080
VECTOR_DB_API_KEY=...
```

---

## Quick Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| Slow queries | No index | Add `CREATE INDEX` on filtered properties |
| Empty results | Entity not in graph | Verify extraction, check entity resolution |
| LLM hallucinates | Context too long / weak instruction | Format context clearly, add "only use facts below" |
| Extraction errors | Poor prompt | Add few-shot examples, use response_format |
| Graph too large | No pruning | Implement `consolidate()` with age/importance thresholds |
| High latency | Deep traversal | Reduce max hops (2-3), use directed relationships |
| Duplicate entities | Weak resolution | Improve dedup: string norm + embedding + LLM vote |
| Community detection slow | Too many nodes | Sample or partition graph, run overnight |
