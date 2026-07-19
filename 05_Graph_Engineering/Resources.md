# Resources — Graph Engineering

> Curated list of papers, tools, tutorials, and communities for graph engineering in AI systems.

---

## Core Papers

### GraphRAG

| Paper | Year | Summary |
|---|---|---|
| [From Local to Global: A Graph RAG Approach to Query-Focused Summarization](https://arxiv.org/abs/2404.16130) | 2024 | Microsoft's GraphRAG paper. Introduces community detection + hierarchical summarization for global queries. |
| [GraphRAG: A Modular Graph-Based Retrieval System](https://arxiv.org/abs/2405.12345) | 2024 | Extended analysis of the GraphRAG architecture with ablation studies. |
| [Knowledge Graph-Augmented Language Models via Retrieval](https://arxiv.org/abs/2310.09236) | 2023 | Early work on combining KGs with LLMs through retrieval augmentation. |
| [StructGPT: A General Framework for Large Language Model to Reason over Structured Data](https://arxiv.org/abs/2305.09645) | 2023 | Framework for LLM reasoning over structured data including graphs. |
| [Think-on-Graph: Deep and Responsible Reasoning with Large Language Model on Knowledge Graph](https://arxiv.org/abs/2307.07697) | 2023 | Iterative reasoning over knowledge graphs with LLM guidance. |

### Knowledge Graph Construction

| Paper | Year | Summary |
|---|---|---|
| [Knowledge Graph Construction: A Survey](https://arxiv.org/abs/2304.02730) | 2023 | Comprehensive survey of KG construction techniques. |
| [REBEL: Relation Extraction By End-to-end Language generation](https://arxiv.org/abs/2104.07708) | 2021 | End-to-end relation extraction using seq2seq models. |
| [Large Language Models for Knowledge Graph Construction](https://arxiv.org/abs/2305.12053) | 2023 | How LLMs are changing the landscape of KG construction. |

### Graph Neural Networks

| Paper | Year | Summary |
|---|---|---|
| [Graph Neural Networks: A Review of Methods and Applications](https://arxiv.org/abs/1812.08434) | 2018 | Foundational survey of GNN architectures. |
| [Inductive Representation Learning on Large Graphs (GraphSAGE)](https://arxiv.org/abs/1706.02216) | 2017 | Scalable node embedding for inductive settings. |
| [Graph Attention Networks](https://arxiv.org/abs/1710.10903) | 2017 | Attention mechanisms for graph neural networks. |

### Community Detection

| Paper | Year | Summary |
|---|---|---|
| [From Louvain to Leiden: Guaranteeing Well-Connected Communities](https://arxiv.org/abs/1810.08473) | 2019 | The Leiden algorithm — standard for community detection in GraphRAG. |
| [Fast unfolding of communities in large networks (Louvain)](https://arxiv.org/abs/0803.0476) | 2008 | Original Louvain method for community detection. |

### Agent Graphs

| Paper | Year | Summary |
|---|---|---|
| [LangGraph: Multi-Agent Workflows](https://blog.langchain.dev/langgraph/) | 2024 | LangChain's graph-based agent orchestration framework. |
| [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629) | 2022 | Reasoning + acting pattern used in many agent graphs. |
| [Plan-and-Solve: Prompting for Zero-Shot Planning](https://arxiv.org/abs/2305.04091) | 2023 | Decomposition-based planning for LLM agents. |

---

## Books

| Title | Author(s) | Year | Coverage |
|---|---|---|---|
| [Graph Databases: New Opportunities for Connected Data](https://graphdatabases.com/) | Ian Robinson, Jim Webber, Emil Eifrem | 2015 | Comprehensive introduction to graph databases with Neo4j focus. |
| [Knowledge Graphs: Fundamentals, Techniques, and Applications](https://mitpress.mit.edu/9780262045097/) | Aidan Hogan et al. | 2021 | Complete reference on knowledge graphs. Available free online: [kgbook.org](https://kgbook.org/) |
| [Designing Data-Intensive Applications](https://dataintensive.net/) | Martin Kleppmann | 2017 | Chapters on graph databases and their use cases. |
| [Graph-Powered Machine Learning](https://www.manning.com/books/graph-powered-machine-learning) | Alessandro Negro | 2021 | Practical guide to using graph algorithms in ML. |

---

## Tools & Frameworks

### Graph Databases

| Tool | Description | License |
|---|---|---|
| [Neo4j](https://neo4j.com/) | Leading graph database. Cypher, ACID, cloud and self-hosted. | Community (GPL) / Commercial |
| [Neo4j AuraDB](https://neo4j.com/cloud/aura/) | Fully managed Neo4j cloud service. | Commercial |
| [Kuzu](https://kuzudb.com/) | Embeddable columnar property graph database. | MIT |
| [Memgraph](https://memgraph.com/) | In-memory graph database with streaming. | Community / Commercial |
| [ArangoDB](https://arangodb.com/) | Multi-model (graph + document + key-value). | Community / Commercial |

### Graph Libraries (In-Memory)

| Tool | Description | Language |
|---|---|---|
| [NetworkX](https://networkx.org/) | Standard Python graph library. | Python |
| [igraph](https://igraph.org/) | High-performance graph library. | Python, R, C |
| [Networkit](https://networkit.github.io/) | Large-scale network analysis. | Python, C++ |
| [CuGraph](https://rapids.ai/cugraph/) | GPU-accelerated graph analytics. | Python |

### GraphRAG Frameworks

| Tool | Description |
|---|---|
| [Microsoft GraphRAG](https://github.com/microsoft/graphrag) | Official Microsoft implementation of the GraphRAG paper. |
| [LangChain Graph Integration](https://python.langchain.com/docs/integrations/graphs/) | LangChain graph integrations (Neo4j, Kuzu, Memgraph). |
| [LlamaIndex Knowledge Graph](https://docs.llamaindex.ai/en/stable/examples/structured_data/knowledge_graph/) | LlamaIndex KG integration. |
| [Neo4j GraphRAG Python](https://github.com/neo4j/neo4j-graphrag-python) | Neo4j's official GraphRAG Python library. |

### Agent Graph Frameworks

| Tool | Description |
|---|---|
| [LangGraph](https://langchain-ai.github.io/langgraph/) | Graph-based agent orchestration by LangChain. |
| [CrewAI](https://docs.crewai.com/) | Multi-agent orchestration (graph-based workflows). |
| [AutoGen](https://microsoft.github.io/autogen/) | Microsoft's multi-agent framework. |
| [Dify](https://dify.ai/) | Open-source LLM app platform with graph-based workflows. |

### Entity Extraction

| Tool | Description |
|---|---|
| [GLiNER](https://github.com/urchade/GLiNER) | Lightweight NER model, good for entity extraction. |
| [spaCy](https://spacy.io/) | Industrial-strength NLP with NER pipeline. |
| [Stanza](https://stanfordnlp.github.io/stanza/) | Stanford NLP library with NER. |
| [OpenAI Structured Outputs](https://platform.openai.com/docs/guides/structured-outputs) | JSON-structured output for LLM extraction. |

### RDF / Semantic Web

| Tool | Description |
|---|---|
| [RDFLib](https://rdflib.readthedocs.io/) | Python library for RDF and SPARQL. |
| [Apache Jena](https://jena.apache.org/) | Java framework for RDF and SPARQL. |
| [Protégé](https://protege.stanford.edu/) | Ontology editor for OWL/RDF. |
| [Stardog](https://www.stardog.com/) | Enterprise knowledge graph platform. |

---

## Tutorials & Courses

### Interactive

| Resource | Platform | Description |
|---|---|---|
| [Neo4j GraphAcademy](https://graphacademy.neo4j.com/) | Neo4j | Free courses from Cypher basics to GraphRAG. |
| [GraphRAG with Neo4j](https://graphacademy.neo4j.com/courses/graphrag/) | Neo4j | Official GraphRAG tutorial using Neo4j. |
| [Kuzu Getting Started](https://kuzudb.com/docs/get-started/) | Kuzu | Quick start guide for Kuzu. |

### Video Courses

| Course | Creator | Platform |
|---|---|---|
| [GraphRAG Explained](https://www.youtube.com/watch?v=kn8k1wI7gL0) | Alice Y. Liang | YouTube |
| [Knowledge Graphs Course](https://www.youtube.com/playlist?list=PLoROMvodv4rPLKxIpqhjhPgdQy7imNkDn) | Stanford (CS520) | YouTube |
| [Neo4j GraphRAG Deep Dive](https://www.youtube.com/watch?v=TIegjMHYMIM) | Neo4j | YouTube |
| [LangGraph Crash Course](https://www.youtube.com/watch?v=MV7FsR1tWQs) | LangChain | YouTube |

### Written Tutorials

| Tutorial | Source |
|---|---|
| [Building a Knowledge Graph with LLMs](https://medium.com/neo4j/building-a-knowledge-graph-with-llms-7a9c0d3e4b1a) | Neo4j Blog |
| [GraphRAG Tutorial](https://www.microsoft.com/en-us/research/project/graphrag/) | Microsoft Research |
| [Building GraphRAG with LangChain](https://python.langchain.com/docs/tutorials/graph/) | LangChain Docs |

---

## Python Packages

```bash
# Core graph databases
pip install neo4j          # Neo4j Python driver
pip install kuzu           # Kuzu embedded database
pip install pymemgraph     # Memgraph Python client

# Graph algorithms (in-memory)
pip install networkx       # Standard graph library
pip install python-igraph   # High-performance graphs
pip install community       # Louvain community detection

# NER / Entity Extraction
pip install spacy          # NLP with NER
pip install gliner         # Lightweight NER
pip install stanza         # Stanford NLP

# RDF / Semantic Web
pip install rdflib         # RDF and SPARQL

# LangChain / LLM integration
pip install langchain      # LLM application framework
pip install langchain-community  # Community integrations
pip install langchain-openai     # OpenAI integration

# GraphRAG
pip install graphrag       # Microsoft GraphRAG

# Utilities
pip install pyvis          # Graph visualization
pip install matplotlib     # Graph plotting
```

---

## Communities

| Community | Platform | Focus |
|---|---|---|
| [Neo4j Community](https://community.neo4j.com/) | Forum | Neo4j support and discussion |
| [r/KnowledgeGraph](https://www.reddit.com/r/KnowledgeGraph/) | Reddit | Knowledge graph discussions |
| [r/GraphRAG](https://www.reddit.com/r/GraphRAG/) | Reddit | GraphRAG-specific discussions |
| [LangChain Discord](https://discord.gg/langchain) | Discord | LangChain including graph integrations |
| [Kuzu Discord](https://discord.gg/kuzu) | Discord | Kuzu community |
| [Memgraph Discord](https://discord.gg/memgraph) | Discord | Memgraph community |
| [Graph AI Summit](https://www.graphaisummit.com/) | Conference | Annual graph AI conference |
| [Neo4j NODES Conference](https://neo4j.com/nodes/) | Conference | Annual Neo4j conference |

---

## Visualization Tools

| Tool | Description |
|---|---|
| [Neo4j Browser](https://neo4j.com/developer/neo4j-browser/) | Built-in graph visualization for Neo4j. |
| [Kuzu Explorer](https://kuzudb.com/explorer) | Web-based graph explorer for Kuzu. |
| [yFiles](https://www.yworks.com/yfiles/) | Professional graph visualization library. |
| [D3.js Force-Directed Graph](https://d3js.org/d3-force) | Custom graph visualization with D3.js. |
| [Cytoscape.js](https://js.cytoscape.org/) | Graph visualization for the web. |
| [Graphviz](https://graphviz.org/) | Classic graph visualization (dot language). |
| [PyVis](https://pyvis.readthedocs.io/) | Python graph visualization (wraps vis.js). |
| [Gephi](https://gephi.org/) | Desktop graph visualization and analysis. |

---

## Open Datasets

| Dataset | Description | Source |
|---|---|---|
| [Wikidata](https://www.wikidata.org/) | Largest open knowledge graph. 100M+ entities. | Wikimedia |
| [DBpedia](https://www.dbpedia.org/) | Structured data from Wikipedia. | University of Leipzig |
| [Freebase](https://developers.google.com/freebase) | Large KG of general knowledge. | Google (deprecated) |
| [ConceptNet](https://conceptnet.io/) | Semantic network of general knowledge. | MIT |
| [WordNet](https://wordnet.princeton.edu/) | Lexical database of English. | Princeton |
| [NELL](http://rtw.ml.cmu.edu/rtw/) | Never-Ending Language Learning system. | CMU |
| [YAGO](https://yago-knowledge.org/) | High-quality knowledge base. | Max Planck Institute |
| [Bio2RDF](https://bio2rdf.org/) | Linked data for life sciences. | Open source |

---

## Production Infrastructure

| Component | Options |
|---|---|
| **Graph Database** | Neo4j (primary), Kuzu (embedded), Memgraph (streaming) |
| **Vector Search** | Neo4j vector index, Pinecone, Weaviate, Qdrant |
| **Embedding Model** | OpenAI text-embedding-3-small, text-embedding-3-large, Voyage, Cohere |
| **LLM** | GPT-4, Claude 3, Gemini, Llama 3, Mistral |
| **Monitoring** | LangFuse, LangSmith, Weights & Biases, Datadog |
| **Caching** | Redis, Memcached |
| **Queue** | Kafka, RabbitMQ, Redis Streams |
| **Orchestration** | LangGraph, Prefect, Airflow, Temporal |
| **Storage** | S3, GCS, Azure Blob (for raw docs + extracted JSON) |
