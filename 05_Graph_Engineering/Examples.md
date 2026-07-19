# Examples — Graph Engineering Code

> Production-ready code examples for graph databases, GraphRAG, entity extraction, and hybrid search.

---

## 1. Neo4j — Connection and Setup

### Basic Connection

```python
from neo4j import GraphDatabase
import logging

class Neo4jConnection:
    def __init__(self, uri="bolt://localhost:7687", user="neo4j", password="password"):
        self.driver = GraphDatabase.driver(uri, auth=(user, password))

    def close(self):
        self.driver.close()

    def query(self, cypher, parameters=None):
        with self.driver.session() as session:
            result = session.run(cypher, parameters or {})
            return [record.data() for record in result]

# Usage
db = Neo4jConnection()
result = db.query("MATCH (n) RETURN count(n) AS node_count")
print(result)  # [{'node_count': 0}]
```

### Connection Pool Configuration

```python
from neo4j import GraphDatabase, RoutingControl

config = {
    "max_connection_pool_size": 50,
    "connection_acquisition_timeout": 30,
    "connection_timeout": 10,
    "max_transaction_retry_time": 10,
}

driver = GraphDatabase.driver(
    "neo4j+s://myinstance.databases.neo4j.io",
    auth=("neo4j", "password"),
    **config
)

def run_query(query, params=None):
    with driver.session(database="neo4j") as session:
        result = session.run(query, params)
        return result.data()
```

---

## 2. Graph Creation

### Creating Nodes and Relationships

```cypher
// Create nodes
CREATE (a:Person {name: "Alice", age: 30, email: "alice@example.com"});
CREATE (b:Person {name: "Bob", age: 25, email: "bob@example.com"});
CREATE (c:Project {name: "GraphRAG", status: "active", priority: 1});

// Create relationships
MATCH (a:Person {name: "Alice"}), (b:Person {name: "Bob"})
CREATE (a)-[:KNOWS {since: 2020}]->(b);

MATCH (a:Person {name: "Alice"}), (p:Project {name: "GraphRAG"})
CREATE (a)-[:WORKS_ON {role: "Lead", hours: 40}]->(p);

MATCH (b:Person {name: "Bob"}), (p:Project {name: "GraphRAG"})
CREATE (b)-[:WORKS_ON {role: "Engineer", hours: 30}]->(p);
```

### Batch Insert (High Performance)

```python
import pandas as pd
from neo4j import GraphDatabase

def batch_insert_nodes(driver, nodes_df, label):
    """Insert nodes in batch using UNWIND."""
    records = nodes_df.to_dict('records')
    query = f"""
    UNWIND $records AS record
    CREATE (n:{label})
    SET n = record
    """
    with driver.session() as session:
        session.run(query, {"records": records})

def batch_insert_relationships(driver, rels_df, rel_type, source_label, source_key,
                                 target_label, target_key):
    records = rels_df.to_dict('records')
    query = f"""
    UNWIND $records AS rel
    MATCH (s:{source_label} {{{source_key}: rel.source}})
    MATCH (t:{target_label} {{{target_key}: rel.target}})
    CREATE (s)-[:{rel_type} {{properties: rel.properties}}]->(t)
    """
    with driver.session() as session:
        session.run(query, {"records": records})

# Usage
nodes = pd.DataFrame([
    {"id": "1", "name": "OpenAI", "type": "Organization"},
    {"id": "2", "name": "GPT-4", "type": "Model"},
])
batch_insert_nodes(driver, nodes, "Entity")
```

---

## 3. Cypher Queries

### Basic Queries

```cypher
// Find all people
MATCH (p:Person) RETURN p.name, p.age;

// Find with filtering
MATCH (p:Person)
WHERE p.age > 25 AND p.name STARTS WITH "A"
RETURN p;

// Find relationships
MATCH (p:Person {name: "Alice"})-[r]->(neighbor)
RETURN type(r) AS relationship, neighbor.name AS connected_to;

// Count relationships per type
MATCH ()-[r]->()
RETURN type(r) AS rel_type, count(*) AS count
ORDER BY count DESC;
```

### Pattern Matching

```cypher
// Variable-length paths
MATCH (a:Person {name: "Alice"})-[:KNOWS*1..3]->(connected)
RETURN DISTINCT connected.name;

// Shortest path
MATCH path = shortestPath(
    (a:Person {name: "Alice"})-[*]-(b:Person {name: "Bob"})
)
RETURN [n IN nodes(path) | n.name] AS path_nodes;

// Find all paths between two entities
MATCH path = (a:Entity {name: "AI"})-[*..4]-(b:Entity {name: "Healthcare"})
RETURN path, length(path) AS path_length
ORDER BY path_length;
```

### Aggregation and Analytics

```cypher
// Degree centrality (most connected nodes)
MATCH (n:Person)
RETURN n.name, size((n)--()) AS degree
ORDER BY degree DESC
LIMIT 10;

// Find nodes with no connections
MATCH (n)
WHERE NOT (n)--()
RETURN n.name, labels(n) AS labels;

// Find cliques (fully connected subgraphs)
MATCH (a)-[:KNOWS]-(b)-[:KNOWS]-(c)-[:KNOWS]-(a)
RETURN a.name, b.name, c.name;
```

---

## 4. Kuzu Examples

### Setup and Connection

```python
import kuzu
import os

# Create or open a database (single file, no server needed)
db_path = os.path.join(os.getcwd(), "knowledge_graph_db")
db = kuzu.Database(db_path)
conn = kuzu.Connection(db)

# Or use in-memory mode for testing
db_in_mem = kuzu.Database(":memory:")
conn_in_mem = kuzu.Connection(db_in_mem)
```

### Schema Creation

```python
# Create node tables
conn.execute(
    "CREATE NODE TABLE Person (name STRING, age INT64, email STRING, PRIMARY KEY (name))"
)
conn.execute(
    "CREATE NODE TABLE Organization (name STRING, founded INT64, industry STRING, PRIMARY KEY (name))"
)
conn.execute(
    "CREATE NODE TABLE Project (name STRING, status STRING, PRIMARY KEY (name))"
)

# Create relationship tables
conn.execute(
    "CREATE REL TABLE WORKS_AT (FROM Person TO Organization, since INT64, role STRING)"
)
conn.execute(
    "CREATE REL TABLE KNOWS (FROM Person TO Person, since INT64)"
)
conn.execute(
    "CREATE REL TABLE MANAGES (FROM Person TO Project, hours DOUBLE)"
)
```

### Data Insertion

```python
# Insert nodes
conn.execute("CREATE (p:Person {name: 'Alice', age: 30, email: 'alice@example.com'})")
conn.execute("CREATE (p:Person {name: 'Bob', age: 25, email: 'bob@example.com'})")
conn.execute("CREATE (o:Organization {name: 'TechCorp', founded: 2010, industry: 'AI'})")
conn.execute("CREATE (pr:Project {name: 'GraphRAG', status: 'active'})")

# Insert relationships
conn.execute("""
    MATCH (a:Person), (o:Organization)
    WHERE a.name = 'Alice' AND o.name = 'TechCorp'
    CREATE (a)-[:WORKS_AT {since: 2018, role: 'Lead'}]->(o)
""")

conn.execute("""
    MATCH (a:Person), (b:Person)
    WHERE a.name = 'Alice' AND b.name = 'Bob'
    CREATE (a)-[:KNOWS {since: 2020}]->(b)
""")
```

### Batch Insert in Kuzu

```python
def batch_insert_kuzu(conn, node_list, label):
    """Batch insert using prepared statements."""
    import json
    placeholders = ", ".join([f"(${i+1})" for i in range(len(node_list))])
    query = f"CREATE (n:{label})"
    # Use UNWIND-like pattern
    conn.execute(
        f"""UNWIND $data AS d
            CREATE (n:{label})
            SET n = d""",
        {"data": node_list}
    )

# Usage
people = [
    {"name": "Charlie", "age": 28, "email": "charlie@example.com"},
    {"name": "Diana", "age": 32, "email": "diana@example.com"},
]
conn.execute(
    """UNWIND $data AS d
       CREATE (p:Person)
       SET p.name = d.name, p.age = d.age, p.email = d.email""",
    {"data": people}
)
```

### Kuzu Queries

```python
# Basic query
result = conn.execute("MATCH (p:Person) RETURN p.name, p.age")
while result.has_next():
    row = result.get_next()
    print(f"{row[0]} is {row[1]} years old")

# Query with parameters
result = conn.execute(
    "MATCH (p:Person)-[:WORKS_AT]->(o:Organization) WHERE o.name = $org RETURN p.name",
    {"org": "TechCorp"}
)

# Graph traversal
result = conn.execute("""
    MATCH (a:Person {name: 'Alice'})-[:KNOWS*1..2]-(connected)
    RETURN DISTINCT connected.name AS connections
""")
```

---

## 5. LangChain GraphRAG

### Neo4j GraphRAG with LangChain

```python
from langchain_community.graphs import Neo4jGraph
from langchain.chains import GraphCypherQAChain
from langchain_openai import ChatOpenAI

# Initialize graph connection
graph = Neo4jGraph(
    url="bolt://localhost:7687",
    username="neo4j",
    password="password"
)

# Create graph from text
from langchain_community.graphs.graph_document import (
    Node as GraphNode,
    Relationship as GraphRelationship,
    GraphDocument
)

documents = [
    GraphDocument(
        nodes=[
            GraphNode(id="OpenAI", type="Organization"),
            GraphNode(id="GPT-4", type="Model"),
            GraphNode(id="Sam Altman", type="Person"),
        ],
        relationships=[
            GraphRelationship(
                source=GraphNode(id="OpenAI", type="Organization"),
                target=GraphNode(id="GPT-4", type="Model"),
                type="DEVELOPED"
            ),
            GraphRelationship(
                source=GraphNode(id="Sam Altman", type="Person"),
                target=GraphNode(id="OpenAI", type="Organization"),
                type="CEO_OF"
            ),
        ],
        source=GraphDocument.Source(
            source="Wikipedia",
            id="openai_page",
            metadata={"url": "https://en.wikipedia.org/wiki/OpenAI"}
        )
    )
]

graph.add_graph_documents(documents)
```

### GraphCypherQA Chain

```python
# Create QA chain that generates Cypher from natural language
chain = GraphCypherQAChain.from_llm(
    llm=ChatOpenAI(model="gpt-4", temperature=0),
    graph=graph,
    verbose=True,
    validate_cypher=True,
    top_k=10
)

# Ask questions
response = chain.invoke({
    "query": "What organizations developed AI models and who leads them?"
})
print(response["result"])
```

### LangChain Graph Document Transformer

```python
from langchain_experimental.graph_transformers import LLMGraphTransformer
from langchain_core.documents import Document
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4", temperature=0)
transformer = LLMGraphTransformer(
    llm=llm,
    allowed_nodes=["Person", "Organization", "Model", "Paper", "Technology"],
    allowed_relationships=[
        "DEVELOPED", "CEO_OF", "FOUNDED", "CITED", "WORKS_AT", "PARTNERED_WITH"
    ],
    node_properties=True,
)

documents = [
    Document(
        page_content="Microsoft invested $10B in OpenAI in 2023. "
                     "OpenAI developed GPT-4. Sam Altman is the CEO of OpenAI.",
        metadata={"source": "news_article"}
    )
]

graph_documents = transformer.convert_to_graph_documents(documents)
graph.add_graph_documents(graph_documents)
```

---

## 6. Entity Extraction

### Zero-Shot Extraction with OpenAI

```python
import json
from openai import OpenAI

client = OpenAI()

def extract_entities(text, schema=None):
    schema_prompt = ""
    if schema:
        schema_prompt = f"""
Allowed entity types: {', '.join(schema.get('entities', []))}
Allowed relationship types: {', '.join(schema.get('relationships', []))}
"""

    response = client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": f"""You are a knowledge graph extractor.
Extract entities and relationships from text.

{schema_prompt}
Return JSON: {{
    "entities": [{{"name": str, "type": str, "properties": dict}}],
    "relationships": [{{"source": str, "target": str, "type": str, "properties": dict}}]
}}"""},
            {"role": "user", "content": text}
        ],
        response_format={"type": "json_object"},
        temperature=0
    )

    return json.loads(response.choices[0].message.content)

# Usage
text = "Google DeepMind, founded in 2010, developed AlphaFold. Demis Hassabis is the CEO."
result = extract_entities(text, schema={
    "entities": ["Person", "Organization", "Technology"],
    "relationships": ["DEVELOPED", "CEO_OF", "FOUNDED"]
})

print(json.dumps(result, indent=2))
```

### Two-Pass Extraction (Better for Large Texts)

```python
def two_pass_extraction(text, chunk_size=2000):
    """Extract entities first, then relationships."""
    from openai import OpenAI
    client = OpenAI()

    chunks = [text[i:i+chunk_size] for i in range(0, len(text), chunk_size)]
    all_entities = {}

    # Pass 1: Extract entities from each chunk
    for chunk in chunks:
        resp = client.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": """Extract ALL named entities.
Return JSON array of {"name": str, "type": str, "description": str}"""},
                {"role": "user", "content": chunk}
            ],
            response_format={"type": "json_object"},
            temperature=0
        )
        entities = json.loads(resp.choices[0].message.content).get("entities", [])
        for e in entities:
            key = e["name"].lower()
            if key not in all_entities:
                all_entities[key] = e

    # Pass 2: Extract relationships between identified entities
    all_entity_names = [e["name"] for e in all_entities.values()]
    resp = client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": f"""Given these entities:
{all_entity_names}

Find relationships between them in the text below.
Return JSON array of {{"source": str, "target": str, "type": str}}"""},
            {"role": "user", "content": text}
        ],
        response_format={"type": "json_object"},
        temperature=0
    )
    relationships = json.loads(resp.choices[0].message.content).get("relationships", [])

    return {
        "entities": list(all_entities.values()),
        "relationships": relationships
    }
```

### Extraction with Structured Output (Pydantic)

```python
from pydantic import BaseModel
from openai import OpenAI

client = OpenAI()

class Entity(BaseModel):
    name: str
    type: str
    properties: dict = {}

class Relationship(BaseModel):
    source: str
    target: str
    type: str
    properties: dict = {}

class KnowledgeGraphExtraction(BaseModel):
    entities: list[Entity]
    relationships: list[Relationship]

def extract_structured(text: str) -> KnowledgeGraphExtraction:
    response = client.beta.chat.completions.parse(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "Extract entities and relationships."},
            {"role": "user", "content": text}
        ],
        response_format=KnowledgeGraphExtraction,
        temperature=0
    )
    return response.choices[0].message.parsed
```

---

## 7. Graph Traversal

### BFS and DFS Traversal

```python
from collections import deque

class GraphTraversal:
    def __init__(self, graph_db):
        self.graph = graph_db

    def bfs(self, start_node, max_depth=3):
        """Breadth-first search from a starting node."""
        visited = set()
        queue = deque([(start_node, 0)])
        results = []

        while queue:
            node, depth = queue.popleft()
            if node in visited or depth > max_depth:
                continue
            visited.add(node)

            results.append({"node": node, "depth": depth})

            neighbors = self._get_neighbors(node)
            for neighbor in neighbors:
                if neighbor not in visited:
                    queue.append((neighbor, depth + 1))

        return results

    def dfs(self, start_node, max_depth=3):
        """Depth-first search from a starting node."""
        visited = set()
        results = []

        def _dfs(node, depth):
            if node in visited or depth > max_depth:
                return
            visited.add(node)
            results.append({"node": node, "depth": depth})

            for neighbor in self._get_neighbors(node):
                _dfs(neighbor, depth + 1)

        _dfs(start_node, 0)
        return results

    def _get_neighbors(self, node_name):
        result = self.graph.query("""
            MATCH (n:Entity {name: $name})-[r]-(neighbor)
            RETURN DISTINCT neighbor.name AS name
        """, {"name": node_name})
        return [r["name"] for r in result]

# Usage
traversal = GraphTraversal(db)
results = traversal.bfs("OpenAI", max_depth=2)
for r in results:
    print(f"{'  ' * r['depth']}{r['node']}")
```

### Subgraph Extraction

```python
def extract_subgraph(graph_db, seed_entities, hops=2, max_nodes=100):
    """Extract a connected subgraph around seed entities."""
    query = """
    MATCH (seed:Entity)
    WHERE seed.name IN $entities
    CALL {
        MATCH (seed)-[r*1..$hops]-(connected)
        RETURN DISTINCT connected, r
        LIMIT $max_nodes
    }
    RETURN collect(DISTINCT {
        node: connected,
        relationships: [rel IN r | type(rel)]
    }) AS subgraph
    """
    result = graph_db.query(query, {
        "entities": seed_entities,
        "hops": hops,
        "max_nodes": max_nodes
    })
    return result[0]["subgraph"] if result else []
```

---

## 8. GraphRAG Pipeline

### Complete GraphRAG Implementation

```python
import json
import hashlib
from typing import List, Dict, Any
from openai import OpenAI
from neo4j import GraphDatabase

class GraphRAG:
    def __init__(self, neo4j_uri, neo4j_user, neo4j_password, openai_api_key):
        self.neo4j_driver = GraphDatabase.driver(neo4j_uri, auth=(neo4j_user, neo4j_password))
        self.openai = OpenAI(api_key=openai_api_key)
        self._init_schema()

    def _init_schema(self):
        """Create indexes and constraints."""
        with self.neo4j_driver.session() as session:
            session.run("CREATE CONSTRAINT IF NOT EXISTS FOR (e:Entity) REQUIRE e.name IS UNIQUE")
            session.run("CREATE INDEX IF NOT EXISTS FOR (e:Entity) ON (e.embedding)")

    def chunk_text(self, text, chunk_size=500, overlap=50):
        """Split text into overlapping chunks."""
        chunks = []
        start = 0
        while start < len(text):
            end = start + chunk_size
            if end < len(text):
                # Try to break at sentence boundary
                last_period = text.rfind(".", start, end)
                if last_period > start + chunk_size // 2:
                    end = last_period + 1
            chunk = text[start:end]
            chunks.append(chunk)
            start = end - overlap
        return chunks

    def extract_from_chunk(self, chunk: str) -> Dict:
        """Extract entities and relationships from a text chunk."""
        response = self.openai.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": """Extract entities and relationships.
Entity types: Person, Organization, Technology, Location, Event, Concept
Relationship types: developed_by, founded, located_in, CEO_of, acquired, partnered_with, part_of

Return JSON:
{
    "entities": [{"name": str, "type": str, "description": str}],
    "relationships": [{"source": str, "target": str, "type": str}]
}"""},
                {"role": "user", "content": chunk}
            ],
            response_format={"type": "json_object"},
            temperature=0
        )
        return json.loads(response.choices[0].message.content)

    def index_document(self, text: str, doc_id: str, metadata: Dict = None):
        """Index a document into the knowledge graph."""
        chunks = self.chunk_text(text)
        all_entities = {}
        all_relationships = []

        for i, chunk in enumerate(chunks):
            extracted = self.extract_from_chunk(chunk)

            for entity in extracted.get("entities", []):
                key = entity["name"].lower()
                if key not in all_entities:
                    all_entities[key] = entity

            all_relationships.extend(extracted.get("relationships", []))

        # Insert into Neo4j
        with self.neo4j_driver.session() as session:
            for entity in all_entities.values():
                embedding = self._get_embedding(entity["name"])
                session.run("""
                    MERGE (e:Entity {name: $name})
                    ON CREATE SET e.type = $type,
                                  e.description = $description,
                                  e.embedding = $embedding
                    ON MATCH SET e.embedding = $embedding
                """, {
                    "name": entity["name"],
                    "type": entity["type"],
                    "description": entity.get("description", ""),
                    "embedding": embedding
                })

            for rel in all_relationships:
                session.run("""
                    MATCH (s:Entity {name: $source})
                    MATCH (t:Entity {name: $target})
                    MERGE (s)-[r:RELATED {type: $rel_type}]->(t)
                """, {
                    "source": rel["source"],
                    "target": rel["target"],
                    "rel_type": rel["type"]
                })

        return {
            "doc_id": doc_id,
            "chunks": len(chunks),
            "entities": len(all_entities),
            "relationships": len(all_relationships)
        }

    def query(self, question: str, hops: int = 2) -> str:
        """Answer a question using the knowledge graph."""
        # Extract query entities
        extraction = self.extract_from_chunk(question)
        query_entities = [e["name"] for e in extraction.get("entities", [])]

        if not query_entities:
            return "I couldn't identify any entities in your question."

        # Retrieve subgraph
        with self.neo4j_driver.session() as session:
            result = session.run("""
                MATCH (e:Entity)
                WHERE e.name IN $entities
                CALL {
                    WITH e
                    MATCH (e)-[r*1..$hops]-(connected)
                    RETURN DISTINCT connected, r
                    LIMIT 100
                }
                RETURN e.name AS source,
                       connected.name AS target,
                       [rel IN r | type(rel)] AS rel_types
            """, {"entities": query_entities, "hops": hops})

            subgraph = result.data()

        # Build context
        context_lines = []
        for row in subgraph:
            rel_str = "->".join(row["rel_types"])
            context_lines.append(f"{row['source']} -[{rel_str}]-> {row['target']}")

        context = "\n".join(context_lines[:50])

        # Generate answer
        response = self.openai.chat.completions.create(
            model="gpt-4",
            messages=[
                {"role": "system", "content": f"""Answer the question using the knowledge graph context below.
If the context doesn't contain enough information, say so.

Knowledge Graph Context:
{context}"""},
                {"role": "user", "content": question}
            ],
            temperature=0
        )

        return response.choices[0].message.content

    def _get_embedding(self, text: str) -> List[float]:
        response = self.openai.embeddings.create(
            model="text-embedding-3-small",
            input=text
        )
        return response.data[0].embedding

    def close(self):
        self.neo4j_driver.close()
```

---

## 9. Community Detection

```python
from neo4j import GraphDatabase
from collections import defaultdict
import networkx as nx
from community import community_louvain
from openai import OpenAI

class CommunityDetector:
    def __init__(self, neo4j_driver, openai_client):
        self.driver = neo4j_driver
        self.openai = openai_client

    def export_to_networkx(self):
        """Export Neo4j graph to NetworkX for community detection."""
        G = nx.Graph()
        with self.driver.session() as session:
            result = session.run("""
                MATCH (s:Entity)-[r:RELATED]->(t:Entity)
                RETURN s.name AS source, t.name AS target, r.type AS rel_type
            """)
            for record in result:
                G.add_edge(record["source"], record["target"],
                          relationship=record["rel_type"])

            result = session.run("MATCH (e:Entity) RETURN e.name AS name, e.type AS type")
            for record in result:
                if record["name"] not in G:
                    G.add_node(record["name"], type=record["type"])

        return G

    def detect_communities(self):
        """Run Louvain community detection."""
        G = self.export_to_networkx()
        partition = community_louvain.best_partition(G)
        return partition

    def summarize_communities(self, partition):
        """Generate LLM summaries for each community."""
        G = self.export_to_networkx()

        # Group nodes by community
        communities = defaultdict(list)
        for node, community_id in partition.items():
            communities[community_id].append(node)

        summaries = {}
        for community_id, nodes in communities.items():
            # Get edges within community
            subgraph = G.subgraph(nodes)
            edges = list(subgraph.edges(data=True))

            # Build description
            edge_text = "\n".join([
                f"{s} --{d.get('relationship', 'RELATED')}--> {t}"
                for s, t, d in edges[:30]
            ])

            # Get node types
            types = defaultdict(int)
            for node in nodes:
                t = G.nodes[node].get("type", "Unknown")
                types[t] += 1
            type_summary = ", ".join([f"{count} {t}" for t, count in types.most_common()])

            # LLM summary
            prompt = f"""Summarize this community of entities in a knowledge graph:

Nodes ({len(nodes)}): {', '.join(nodes[:20])}{'...' if len(nodes) > 20 else ''}
Node types: {type_summary}

Relationships:
{edge_text}

Provide a 2-3 sentence summary of what this community represents."""

            response = self.openai.chat.completions.create(
                model="gpt-4",
                messages=[{"role": "user", "content": prompt}],
                temperature=0
            )
            summaries[community_id] = response.choices[0].message.content

        return summaries
```

---

## 10. Hybrid Search with RRF

```python
import numpy as np
from typing import List, Dict, Any, Callable

class HybridSearch:
    def __init__(self, vector_search_fn, keyword_search_fn, graph_search_fn):
        self.vector_search = vector_search_fn
        self.keyword_search = keyword_search_fn
        self.graph_search = graph_search_fn

    def rrf_fusion(self, results_lists: List[List[Dict]], k: int = 60) -> List[Dict]:
        """Reciprocal Rank Fusion."""
        scores = {}
        item_details = {}

        for results in results_lists:
            for rank, item in enumerate(results):
                item_id = item.get("id", item.get("name"))
                if item_id not in scores:
                    scores[item_id] = 0
                    item_details[item_id] = item
                scores[item_id] += 1.0 / (k + rank + 1)

        ranked = sorted(scores.items(), key=lambda x: x[1], reverse=True)
        return [
            {**item_details[item_id], "fusion_score": score}
            for item_id, score in ranked
        ]

    def search(self, query: str, top_k: int = 10) -> Dict[str, Any]:
        """Run hybrid search with all three strategies."""
        vector_results = self.vector_search(query, k=top_k * 2)
        keyword_results = self.keyword_search(query, k=top_k * 2)
        graph_results = self.graph_search(query, k=top_k * 2)

        fused = self.rrf_fusion(
            [vector_results, keyword_results, graph_results],
            k=60
        )

        return {
            "query": query,
            "results": fused[:top_k],
            "metadata": {
                "vector_count": len(vector_results),
                "keyword_count": len(keyword_results),
                "graph_count": len(graph_results),
                "fusion_count": len(fused)
            }
        }

# Example vector search implementation
def simple_vector_search(embeddings_db, query_embedding, k=10):
    """Cosine similarity search."""
    scores = []
    for item_id, item_embedding in embeddings_db.items():
        sim = np.dot(query_embedding, item_embedding) / (
            np.linalg.norm(query_embedding) * np.linalg.norm(item_embedding)
        )
        scores.append({"id": item_id, "score": sim})
    return sorted(scores, key=lambda x: x["score"], reverse=True)[:k]

# Usage
hybrid = HybridSearch(
    vector_search_fn=lambda q, k: simple_vector_search(vec_db, embed(q), k),
    keyword_search_fn=lambda q, k: bm25_search(bm25_index, q, k),
    graph_search_fn=lambda q, k: graph_search(db, q, k)
)

results = hybrid.search("What companies work on AI safety?")
for r in results["results"]:
    print(f"{r['id']} (fusion_score: {r['fusion_score']:.4f})")
```

---

## 11. Agent Graph (LangGraph-Style)

```python
from typing import TypedDict, Callable, Dict, Any, List
import json

class AgentState(TypedDict):
    messages: List[Dict[str, str]]
    context: Dict[str, Any]
    iteration: int
    max_iterations: int

class AgentNode:
    def __init__(self, name: str, fn: Callable):
        self.name = name
        self.fn = fn

class ConditionalEdge:
    def __init__(self, from_node: str, condition_fn: Callable,
                 true_target: str, false_target: str):
        self.from_node = from_node
        self.condition_fn = condition_fn
        self.true_target = true_target
        self.false_target = false_target

class AgentGraph:
    def __init__(self):
        self.nodes: Dict[str, AgentNode] = {}
        self.edges: List[tuple] = []
        self.conditional_edges: List[ConditionalEdge] = []

    def add_node(self, name: str, fn: Callable):
        self.nodes[name] = AgentNode(name, fn)

    def add_edge(self, from_node: str, to_node: str):
        self.edges.append((from_node, to_node))

    def add_conditional_edge(self, from_node: str, condition_fn: Callable,
                              true_target: str, false_target: str):
        self.conditional_edges.append(
            ConditionalEdge(from_node, condition_fn, true_target, false_target)
        )

    def run(self, initial_state: AgentState) -> AgentState:
        state = initial_state
        current_node = "__start__"
        visited = set()

        while current_node != "__end__":
            if state["iteration"] >= state["max_iterations"]:
                state["messages"].append({
                    "role": "system",
                    "content": "Max iterations reached."
                })
                break

            if current_node in self.nodes:
                state = self.nodes[current_node].fn(state)
                state["iteration"] += 1

            # Check conditional edges first
            next_node = None
            for cond_edge in self.conditional_edges:
                if cond_edge.from_node == current_node:
                    if cond_edge.condition_fn(state):
                        next_node = cond_edge.true_target
                    else:
                        next_node = cond_edge.false_target
                    break

            if next_node is None:
                for from_node, to_node in self.edges:
                    if from_node == current_node:
                        next_node = to_node
                        break

            if next_node is None or next_node == "__end__":
                break

            current_node = next_node

        return state

# Example: Customer Support Agent
def classify(state):
    """Classify the customer issue."""
    query = state["messages"][-1]["content"].lower()
    if "billing" in query or "payment" in query or "charge" in query:
        state["context"]["category"] = "billing"
    elif "error" in query or "bug" in query or "not working" in query:
        state["context"]["category"] = "technical"
    else:
        state["context"]["category"] = "general"
    return state

def billing_handler(state):
    state["messages"].append({
        "role": "assistant",
        "content": "I'll help you with billing. Let me look up your account."
    })
    return state

def technical_handler(state):
    state["messages"].append({
        "role": "assistant",
        "content": "I'll help you with technical issues. What error are you seeing?"
    })
    return state

def general_handler(state):
    state["messages"].append({
        "role": "assistant",
        "content": "I'll help you with your question. Let me find the right information."
    })
    return state

def is_billing(state):
    return state["context"].get("category") == "billing"

def is_technical(state):
    return state["context"].get("category") == "technical"

# Build the agent graph
support_agent = AgentGraph()
support_agent.add_node("classify", classify)
support_agent.add_node("billing_handler", billing_handler)
support_agent.add_node("technical_handler", technical_handler)
support_agent.add_node("general_handler", general_handler)

support_agent.add_conditional_edge("classify", is_billing, "billing_handler", "check_technical")
support_agent.add_conditional_edge("check_technical", is_technical, "technical_handler", "general_handler")

support_agent.add_edge("billing_handler", "__end__")
support_agent.add_edge("technical_handler", "__end__")
support_agent.add_edge("general_handler", "__end__")

# Run
state = AgentState(
    messages=[{"role": "user", "content": "I have a billing issue with my account"}],
    context={},
    iteration=0,
    max_iterations=10
)

final_state = support_agent.run(state)
print(final_state["messages"][-1]["content"])
```

---

## 12. RDF with RDFLib

```python
import rdflib
from rdflib import Graph, Literal, RDF, URIRef, Namespace
from rdflib.namespace import FOAF, XSD

# Create a graph
g = Graph()

# Define namespaces
EX = Namespace("http://example.org/")
g.bind("ex", EX)
g.bind("foaf", FOAF)

# Add triples
einstein = URIRef("http://example.org/Einstein")
relativity = URIRef("http://example.org/Relativity")
physicist = URIRef("http://example.org/Physicist")

g.add((einstein, RDF.type, FOAF.Person))
g.add((einstein, FOAF.name, Literal("Albert Einstein", lang="en")))
g.add((einstein, EX.developed, relativity))
g.add((relativity, RDF.type, EX.ScientificTheory))
g.add((relativity, EX.year, Literal(1915, datatype=XSD.integer)))

# Query with SPARQL
results = g.query("""
    PREFIX ex: <http://example.org/>
    SELECT ?entity ?relationship ?value WHERE {
        ?entity ?relationship ?value
    }
""")

for row in results:
    print(f"{row.entity} -- {row.relationship} --> {row.value}")

# Serialize to Turtle
print(g.serialize(format="turtle"))
```
