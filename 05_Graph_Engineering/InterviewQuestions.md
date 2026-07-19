# Interview Questions — Graph Engineering

> 20+ interview questions covering knowledge graphs, GraphRAG, graph databases, entity extraction, and graph-based AI systems. Each question includes the expected answer depth.

---

## Foundational Questions

### Q1: What is a knowledge graph, and how is it different from a relational database?

**Expected answer depth:** 3-5 minutes

A knowledge graph is a structured representation of facts where entities are nodes and relationships between them are edges. The key differences from a relational database:

- **Schema flexibility:** Knowledge graphs use a label-based schema that is easy to extend. Relational databases require rigid table schemas with migrations.
- **Relationship primacy:** In a KG, relationships are first-class citizens stored adjacent to nodes. In SQL, relationships are implied through foreign keys requiring JOIN operations.
- **Traversal performance:** KGs support O(1) neighbor lookups via index-free adjacency. SQL JOINs scale O(n log n) with table size.
- **Query style:** KGs use pattern matching (Cypher, SPARQL). RDBMS use set operations (SQL).

**When to use each:** Use a knowledge graph when relationship traversal and semantic queries are central. Use a relational database when you need strictly normalized data with complex aggregations.

### Q2: Explain the difference between RDF and Property Graph models.

**Expected answer depth:** 3-5 minutes

**RDF (Resource Description Framework):**
- Data model: Triples (subject-predicate-object), each statement is atomic
- Schema: Ontology-based (RDFS, OWL), supports logical inference
- Query: SPARQL
- Storage: Triple stores
- Best for: Semantic web, linked data, interoperability

**Property Graph:**
- Data model: Nodes with properties + labeled relationships with properties
- Schema: Label-based, optional constraints
- Query: Cypher, Gremlin, GQL
- Storage: Native graph (adjacency lists)
- Best for: Applications, analytics, GraphRAG

**Key difference:** RDF treats every statement as a triple that can be reasoned over. Property graphs treat nodes and relationships as entities that can have arbitrary properties. For AI engineering, property graphs are more practical for GraphRAG because they map naturally to entity-relationship structures.

### Q3: What is index-free adjacency, and why does it matter?

**Expected answer depth:** 2-3 minutes

Index-free adjacency means each node in a graph database stores direct pointers (physical addresses) to its adjacent nodes. When traversing from node A to node B, the database doesn't need to look up an index — it follows the pointer directly.

This matters because:
- Traversal time depends only on the number of relationships traversed, not the total graph size
- O(1) neighbor lookup regardless of whether the graph has 100 or 100 million nodes
- Enables real-time traversals for path finding, recommendation, and knowledge retrieval

Neo4j is the most prominent database using index-free adjacency. Kuzu uses columnar storage with vectorized execution instead.

### Q4: What are the key components of the GraphRAG pipeline?

**Expected answer depth:** 4-6 minutes

The GraphRAG pipeline has three main phases:

**1. Indexing Phase:**
- Document chunking (typically 300-600 tokens)
- Entity extraction using LLMs (person, organization, concept, etc.)
- Relationship extraction between discovered entities
- Entity resolution (merging duplicate references)
- Graph construction and storage

**2. Community Detection Phase:**
- Leiden algorithm for community detection
- Hierarchical community structure (multiple granularity levels)
- Community summarization using LLMs
- Map-reduce pattern for large graphs

**3. Retrieval & Generation Phase:**
- Entity extraction from user query
- Entity matching in the knowledge graph
- Subgraph extraction (1-3 hop traversal)
- Community context retrieval
- Context assembly and LLM generation

---

## Graph Database Questions

### Q5: Compare Neo4j, Kuzu, and Memgraph. When would you choose each?

**Expected answer depth:** 4-5 minutes

| Aspect | Neo4j | Kuzu | Memgraph |
|---|---|---|---|
| Storage | Native graph on disk | Columnar on disk | In-memory |
| Deployment | Server/Cloud | Embedded | Server |
| ACID | Full | Yes | Yes |
| Graph algorithms | GDS library | Built-in (limited) | MAGE library |
| Vector search | Yes (5.11+) | Not native | Yes |
| Best for | Production systems | Local/analytics | Real-time/streaming |

**Choose Neo4j when:** You need a production-grade system with ACID compliance, rich ecosystem, and mature tooling. Best for enterprise GraphRAG.

**Choose Kuzu when:** You want an embeddable database that ships with your application. Excellent for data science workflows, prototyping, and CI/CD.

**Choose Memgraph when:** You need real-time graph analytics with streaming data (Kafka, Pulsar). Best for fraud detection and low-latency applications.

### Q6: How do you optimize Cypher query performance?

**Expected answer depth:** 3-4 minutes

1. **Use indexes and constraints:** Create indexes on properties used in WHERE clauses and constraints for unique properties.

2. **Limit initial match sets:** Start queries with the most selective pattern to reduce the working set as early as possible.

3. **Use directed relationships:** `(a)-[:KNOWS]->(b)` is faster than `(a)-[:KNOWS]-(b)` because it eliminates reverse traversal.

4. **Profile your queries:** Use `PROFILE` to see db hits and identify bottlenecks.

5. **Avoid cartesian products:** Ensure MATCH clauses are connected. Use `OPTIONAL MATCH` only when needed.

6. **Use `LIMIT` early:** Push limits as early as possible in the query.

7. **Parameterize everything:** Never concatenate values into query strings. Use `$param`.

### Q7: How do you model time-varying data in a graph database?

**Expected answer depth:** 3-4 minutes

Four common patterns:

**1. Time-stamped properties:**
```cypher
(p:Person)-[:WORKS_AT {from: date("2020-01"), to: date("2023-12")}]->(o:Organization)
```

**2. Version nodes:**
```cypher
(p:Person)-[:HAS_VERSION]->(v:Version {date: "2024-01", role: "CEO"})
```

**3. Snapshot graphs:** Maintain multiple graph snapshots at different timestamps.

**4. Temporal property values:**
```cypher
CREATE (e:Entity {name: "GPT-4"})
CREATE (e)-[:HAS_PROPERTY_AT {
    property: "parameters", value: "1.7T", valid_from: "2023-03", valid_to: "2023-11"
}]->(:Snapshot)
```

---

## GraphRAG Questions

### Q8: How does GraphRAG improve over traditional RAG?

**Expected answer depth:** 4-6 minutes

Traditional RAG retrieves top-K text chunks by vector similarity. GraphRAG retrieves structured subgraphs.

**Advantages of GraphRAG:**

1. **Multi-hop reasoning:** RAG with text chunks requires finding Chunk A → Chunk B → Chunk C. GraphRAG traverses entity relationships directly.

2. **Entity resolution:** GraphRAG resolves "OpenAI" and "OpenAI Inc." to the same entity. RAG treats them as different text tokens.

3. **Structured context:** GraphRAG provides triples (Einstein -developed-> Relativity). RAG provides raw text chunks that may or may not contain relevant facts.

4. **Community awareness:** GraphRAG detects entity communities and provides global summaries. RAG has no concept of entity groupings.

5. **Reduced hallucination:** Structured facts from a knowledge graph are more verifiable than text chunks.

**Limitations:**
- Higher indexing cost (LLM extraction for every chunk)
- More complex infrastructure
- Entity extraction quality depends on LLM performance
- Schema design requires domain expertise

### Q9: Explain the role of community detection in GraphRAG.

**Expected answer depth:** 3-5 minutes

Community detection (typically using the Leiden algorithm) partitions the knowledge graph into densely connected groups of entities.

**Role in GraphRAG:**

1. **Global understanding:** Communities represent coherent topics or domains. Summarizing each community gives a global view of the knowledge base.

2. **Hierarchical context:** Communities at different granularity levels (fine to coarse) enable query-focused summarization at the right abstraction level.

3. **Efficient retrieval:** Instead of traversing the entire graph, you first identify the relevant community, then drill down.

4. **Map-reduce generation:** For broad questions, you can summarize each community independently, then summarize the summaries.

The community summaries act as a "table of contents" for the knowledge graph, enabling the LLM to understand what information exists before retrieving specific facts.

### Q10: How do you evaluate a GraphRAG system?

**Expected answer depth:** 4-5 minutes

**Retrieval Metrics:**
- **Recall@K:** Are the relevant subgraph entities in the top-K?
- **MRR (Mean Reciprocal Rank):** How high are relevant entities ranked?
- **Subgraph completeness:** Does the retrieved subgraph contain all necessary facts?

**Generation Metrics:**
- **Factuality:** What fraction of generated facts are present in the knowledge graph?
- **Completeness:** What fraction of relevant facts in the graph appear in the answer?
- **Hallucination rate:** Percentage of claims not supported by the graph.

**Graph Quality Metrics:**
- **Entity extraction accuracy:** Precision/recall of extracted entities vs. gold standard
- **Relation extraction accuracy:** Correctness of extracted relationships
- **Entity resolution accuracy:** Were duplicates correctly merged?
- **Graph density:** Ratio of edges to possible edges (indicates connectedness)

**End-to-End:**
- Human evaluation of answer quality (relevance, accuracy, completeness)
- A/B comparison against traditional RAG
- Task-specific metrics (QA accuracy, summarization quality)

---

## Entity Extraction Questions

### Q11: Design an entity extraction pipeline for a document corpus.

**Expected answer depth:** 4-6 minutes

```
Input Documents
    ↓
1. Preprocessing (clean text, normalize encoding, detect language)
    ↓
2. Document Chunking (300-600 tokens, 10-20% overlap)
    ↓
3. Entity Extraction (LLM per chunk)
   ├── Named entities (Person, Organization, Location)
   ├── Domain-specific entities (Gene, Drug, Algorithm)
   └── Events and temporal expressions
    ↓
4. Entity Resolution
   ├── String normalization (lowercase, stem, expand abbreviations)
   ├── Embedding similarity (cosine distance between entity names)
   └── LLM-based disambiguation (are these the same entity?)
    ↓
5. Relationship Extraction
   ├── Pairwise (for each entity pair, does a relationship exist?)
   └── Text-based (extract relationships from sentences in context)
    ↓
6. Graph Insertion
```

**Design decisions:**
- **Schema-first vs schema-last:** Schema-first constrains extraction to defined types. Schema-last discovers types from data.
- **Single-pass vs two-pass:** Single-pass extracts entities and relationships together (simpler but lower quality for complex docs). Two-pass extracts entities first, then relationships (better quality, double cost).
- **LLM vs fine-tuned NER:** LLMs are better for complex, varied domains. Fine-tuned models (e.g., spaCy, GLiNER) are cheaper for fixed, narrow domains.

### Q12: How do you handle entity resolution at scale?

**Expected answer depth:** 3-4 minutes

**Blocking:** Group similar entities into blocks using a cheap function (e.g., first 2 characters, common tokens) to avoid O(n²) comparisons.

**Similarity Functions:**
- **String:** Levenshtein distance, Jaro-Winkler, cosine similarity of character n-grams
- **Embedding:** Cosine similarity of entity name embeddings
- **Neighborhood:** Overlap in graph neighbors (two entities that connect to the same things are likely the same)

**Resolution Strategies (in increasing order of cost):**
1. **Rule-based:** Exact match after normalization (fastest, lowest recall)
2. **Similarity threshold:** String similarity > 0.9 → merge
3. **Classifier:** Train a binary classifier (same/different) on labeled pairs
4. **LLM-based:** Ask the LLM if two entities refer to the same thing (best quality, highest cost)

**Active learning:** Start with rules, use LLM to label uncertain cases, retrain, iterate.

### Q13: How do you design a schema for entity extraction prompts?

**Expected answer depth:** 3-4 minutes

Principles for schema design:

1. **Mutually exclusive types:** Ensure entity types don't overlap (e.g., "Person" and "Employee" — pick one).

2. **Appropriate granularity:** Too fine-grained (100 types) → LLM struggles. Too coarse (3 types) → loses information. Aim for 5-15 types.

3. **Relationship symmetry:** For each relationship type, define valid source and target types. `CEO_of` connects a Person to an Organization, not an Organization to a Person.

4. **Required vs optional properties:** Specify which properties are required for each type.

5. **Provide examples in the prompt:** Few-shot examples improve extraction quality significantly.

**Example schema prompt:**
```
Entity Types:
- Person: {name, birth_year?, nationality?}
- Organization: {name, founded?, headquarters?, industry?}
- Technology: {name, version?, creator?}
- Event: {name, date, location?}

Relationship Types:
- works_at: Person → Organization {since?, role?}
- developed: Person|Organization → Technology
- acquired: Organization → Organization {year, amount?}
- partnered_with: Organization → Organization {since?}
```

---

## Reasoning and Traversal Questions

### Q14: How do you implement multi-hop reasoning over a knowledge graph?

**Expected answer depth:** 4-5 minutes

Multi-hop reasoning means answering questions that require traversing multiple relationships. Example:

*"What papers did researchers at companies founded by Stanford alumni publish?"*

Hop 1: Stanford → alumni → Person
Hop 2: Person → founded → Company
Hop 3: Company → employs → Researcher
Hop 4: Researcher → authored → Paper

**Implementation approaches:**

1. **Cypher pattern matching:**
```cypher
MATCH (stanford:Organization {name: "Stanford"})
MATCH (stanford)<-[:GRADUATED_FROM]-(alumnus:Person)
MATCH (alumnus)-[:FOUNDED]->(company:Organization)
MATCH (company)<-[:WORKS_AT]-(researcher:Person)
MATCH (researcher)-[:AUTHORED]->(paper:Paper)
RETURN paper.title, researcher.name
```

2. **Graph traversal (BFS/DFS):** Walk the graph from seed entities up to N hops, collecting paths.

3. **LLM-guided traversal:** Use the LLM to decide which path to traverse at each step based on the question.

**Key challenge:** Path explosion — the number of paths grows exponentially with hop count. Mitigations: limit hops, prune low-relevance paths, use PageRank to prioritize important nodes.

### Q15: What graph algorithms are most useful for AI systems?

**Expected answer depth:** 3-4 minutes

1. **PageRank** — Find the most important/relevant entities in the graph. Used for prioritizing context.

2. **Community Detection (Leiden/Louvain)** — Group related entities. Used for GraphRAG community summarization.

3. **Shortest Path** — Find the minimal connection between two entities. Used for explaining relationships.

4. **Betweenness Centrality** — Find bridge entities that connect different parts of the graph. Used for identifying key connectors.

5. **BFS/DFS** — Extract subgraphs. Used for context building around seed entities.

6. **Personalized PageRank** — PageRank biased toward seed entities. Used for personalized recommendations.

7. **Triangle Counting / Clustering Coefficient** — Measure graph density. Used for assessing graph connectivity and potential for community discovery.

---

## Agent Graphs Questions

### Q16: What is an agent graph, and how does it differ from a knowledge graph?

**Expected answer depth:** 3-4 minutes

An agent graph is a directed graph that defines the control flow of an AI agent's execution. Nodes are steps in the workflow (LLM calls, tool executions, decision points) and edges are transitions.

**Differences:**

| Aspect | Knowledge Graph | Agent Graph |
|---|---|---|
| **Purpose** | Store facts and relationships | Define execution flow |
| **Nodes** | Entities (people, concepts, etc.) | Steps (LLM calls, tools, decisions) |
| **Edges** | Semantic relationships | Control flow transitions |
| **Properties** | Entity attributes | State, configuration |
| **Traversal** | Semantic (find connected info) | Execution (determine next step) |
| **Mutation** | Insert/update facts | Execute and update state |

**Analogy:** A knowledge graph is a database of facts. An agent graph is a program that processes those facts.

LangGraph is the most popular framework for building agent graphs.

### Q17: Design a state machine for a multi-agent system.

**Expected answer depth:** 4-5 minutes

**Requirements:** A customer support system with:
- Classification node
- Specialized agents (billing, technical, general)
- Escalation path
- Human handoff

**State machine design:**

```
Nodes:
1. classify — Determine issue category
2. billing_agent — Handle billing
3. technical_agent — Handle technical
4. general_agent — Handle general
5. escalate — Escalate to human
6. resolve — Mark as resolved

Edges:
classify → billing_agent (if category = billing)
classify → technical_agent (if category = technical)
classify → general_agent (if category = general)
billing_agent → resolve (if resolved)
billing_agent → escalate (if needs human)
technical_agent → resolve (if resolved)
technical_agent → escalate (if needs human)
general_agent → resolve (if resolved)
escalate → resolve (handled by human)

State:
- messages: conversation history
- category: issue classification
- attempts: retry counter
- resolved: boolean
- human_needed: boolean
```

**Implementation considerations:**
- Max iterations to prevent infinite loops
- State validation at each transition
- Logging for debugging
- Parallel execution for independent sub-tasks

---

## Production Questions

### Q18: How do you scale a GraphRAG system to millions of documents?

**Expected answer depth:** 4-6 minutes

**Indexing Pipeline:**

1. **Parallel chunk processing:** Use message queues (Kafka, RabbitMQ) to distribute chunks across workers. Each worker extracts entities independently.

2. **Batch graph insertion:** Use UNWIND in Cypher for batch inserts (1000+ records per transaction). Avoid per-row transactions.

3. **Entity resolution at scale:**
   - Use blocking to reduce comparison pairs
   - Distributed entity resolution (e.g., Apache Spark)
   - Approximate nearest neighbor for embedding-based resolution

4. **Community detection on large graphs:**
   - Use the Leiden algorithm (O(n log n) complexity)
   - Hierarchical clustering for incremental updates
   - Recompute communities periodically, not on every insert

**Storage:**

5. **Graph partitioning:** Shard by domain or community for parallel queries.

6. **Read replicas:** For Neo4j, use causal clustering with read replicas.

7. **Caching:** Cache frequent queries, community summaries, and popular subgraphs.

**Retrieval:**

8. **Pre-compute embeddings:** Store entity embeddings at insert time, not query time.

9. **Approximate nearest neighbor:** Use HNSW indexes for vector search rather than exact kNN.

10. **Query planning:** For complex multi-hop queries, use the query optimizer to choose the best traversal order.

### Q19: How do you handle knowledge graph updates and versioning?

**Expected answer depth:** 3-4 minutes

**Update patterns:**

1. **Incremental updates:** Add new nodes and edges without rebuilding the entire graph. Use MERGE to avoid duplicates.

2. **Soft deletes:** Add a `valid_to` timestamp instead of deleting. Query with `WHERE r.valid_to IS NULL` for current state.

3. **Versioned graph snapshots:** Periodically create a complete snapshot. Use for rollback and temporal queries.

4. **Change data capture (CDC):** Log all mutations with timestamps. Replay for debugging or analysis.

**Consistency considerations:**
- **Eventual consistency:** Accept that different replicas may have different versions temporarily.
- **Read-your-writes:** Ensure a query after a write sees the write.
- **Conflict resolution:** When two updates conflict (same entity, different properties), use last-writer-wins or merge strategies.

### Q20: What monitoring and evaluation would you set up for a production GraphRAG system?

**Expected answer depth:** 4-5 minutes

**Pipeline Monitoring:**

1. **Extraction quality:**
   - Entities per document (mean, std, min, max)
   - Relationships per document
   - Extraction failure rate (LLM errors, JSON parsing errors)
   - Entity type distribution

2. **Graph growth:**
   - Total nodes and edges over time
   - Graph density
   - Average degree
   - New entities per hour/day

3. **Retrieval performance:**
   - P50/P95/P99 latency for vector search
   - P50/P95/P99 latency for graph traversal
   - Cache hit rate
   - Empty result rate

4. **Generation quality:**
   - Factuality (compare generated facts against graph)
   - User feedback (thumbs up/down)
   - Answer completeness

**Alerting thresholds:**
- Extraction failure rate > 5%
- P99 retrieval latency > 2s
- Empty result rate > 10%
- Entity resolution merging errors > 1%

**A/B Testing:**
- Compare different extraction prompts
- Compare different hop counts
- Compare community summarization vs. raw traversal
- Compare GraphRAG vs. traditional RAG on the same queries

---

## Scenario-Based Questions

### Q21: Your GraphRAG system is returning incorrect facts. How do you debug it?

**Expected answer depth:** 4-5 minutes

**Step 1: Identify where the error occurs**
- Is the fact in the knowledge graph? → Query the graph directly
- Is the fact correctly extracted? → Check the extraction output
- Was the correct subgraph retrieved? → Log the retrieved context
- Is the LLM ignoring the context? → Check if the context is in the prompt

**Step 2: Common failure modes and fixes**

| Symptom | Likely Cause | Fix |
|---|---|---|
| Entity not in graph | Poor extraction | Improve extraction prompt, add entity-specific patterns |
| Wrong entity matched | Poor entity resolution | Tune similarity thresholds, add domain aliases |
| Wrong subgraph | Too many/few hops | Adjust hop count, add relationship type filtering |
| LLM ignores context | Context too long | Improve context formatting, prioritize important triples |
| LLM hallucinates | LLM overconfident | Add "only use facts from context" instruction |

**Step 3: Add observability**
- Log every extraction with chunk_id
- Log every query with the retrieved subgraph
- Log the final prompt sent to the LLM
- Enable tracing (e.g., LangSmith, LangFuse)

### Q22: Compare GraphRAG with vector RAG for a question-answering system.

**Expected answer depth:** 4-5 minutes

**Choose GraphRAG when:**
- Questions require multi-hop reasoning (3+ connections)
- Entity relationships are central to the domain (biomedical, legal, finance)
- You need global understanding (summarize the whole corpus)
- Entity disambiguation is important
- Facts change over time and you need versioning

**Choose Vector RAG when:**
- Questions are factoid (single entity, single fact)
- Documents are semi-structured or unstructured with implicit relationships
- Latency is critical (vector search is faster than graph traversal)
- Simplicity matters (GraphRAG has more components to manage)
- You have limited budget for LLM extraction calls

**Choose Hybrid when:**
- You need the best of both worlds
- Your query workload is diverse (some simple, some multi-hop)
- You have the infrastructure budget

### Q23: How would you build a graph-based memory system for an AI agent?

**Expected answer depth:** 4-5 minutes

**Architecture:**

```python
class GraphMemory:
    def remember(self, observation):
        """Extract facts and store in graph."""
        entities = extract_entities(observation)
        relations = extract_relations(observation, entities)
        for e in entities: merge_entity(e)
        for r in relations: merge_relation(r)
        add_temporal_metadata(observation)

    def recall(self, query, k=5):
        """Retrieve relevant memories."""
        entities = extract_query_entities(query)
        subgraph = traverse_graph(entities, hops=2)
        ranked = rank_by_relevance(subgraph, query)
        return format_context(ranked[:k])

    def consolidate(self):
        """Periodic maintenance."""
        merge_duplicate_entities()
        prune_stale_facts(older_than=30_days)
        update_embeddings()
```

**Key design decisions:**
1. **Forgetting mechanism:** Remove or down-weight entities that haven't been mentioned in N days.
2. **Recency weighting:** Recent facts should be more accessible. Use temporal decay in scoring.
3. **Importance scoring:** Track entity mention frequency. Important entities survive pruning.
4. **Conflict resolution:** When new information contradicts old, keep both with timestamps.
5. **Context window management:** Only the most relevant K entities are included in each query.

---

## Advanced Questions

### Q24: Explain the Leiden algorithm and why it's used in GraphRAG.

**Expected answer depth:** 3-4 minutes

The Leiden algorithm is a community detection algorithm that improves on Louvain by guaranteeing well-connected communities.

**How it works:**
1. **Local moving:** Move nodes between communities to optimize modularity.
2. **Refinement:** Split poorly-connected communities into well-connected sub-communities.
3. **Aggregation:** Merge each community into a single node and repeat.

**Why for GraphRAG:**
- Produces hierarchical community structure (fine → coarse)
- Guarantees communities are internally connected (no disconnected subsets)
- Scalable to millions of nodes (O(n log n))
- Deterministic with fixed seed (reproducible results)

The hierarchical output is crucial for GraphRAG: fine-grained communities give precise local context, coarse-grained communities give global summaries.

### Q25: How would you implement real-time graph updates with streaming data?

**Expected answer depth:** 4-5 minutes

**Architecture:**

```
Kafka Stream → Stream Processor → Graph Database
     ↓                                ↓
   Events                          Nodes/Relationships
```

**Implementation approaches:**

1. **Memgraph + Kafka:** Memgraph natively supports streaming ingestion from Kafka topics. Each message is processed as a Cypher query.

2. **Custom stream processor:** Use Apache Flink or Kafka Streams to process events and batch-insert into Neo4j.

3. **Event Sourcing + CQRS:** Write events to an event store. Materialize the graph view separately. Query the materialized view.

**Example with Memgraph:**
```cypher
CREATE KAFKA STREAM transactions_stream
TOPICS 'financial-transactions'
TRANSFORM 'transaction.process'
BATCH_INTERVAL 100;
```

**Real-time considerations:**
- **Micro-batching:** Batch inserts every 100ms rather than per-event to reduce overhead.
- **Idempotency:** Use MERGE with composite keys to handle duplicate events.
- **Late data:** Use event time (not processing time) for temporal properties.
- **Windowing:** For streaming analytics, use sliding windows for time-bound queries.
