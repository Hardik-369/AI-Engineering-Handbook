# Project Starter Templates

This file provides directory structures, configuration files, and setup scripts for every project. Use these as your starting point — they establish consistent patterns across all 15 projects.

---

## Template Conventions

All projects follow this structure unless otherwise specified:

```
project-name/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── setup.py (or pyproject.toml)
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   └── ...
├── tests/
│   ├── __init__.py
│   └── test_core.py
├── data/
│   └── .gitkeep
├── logs/
│   └── .gitkeep
└── scripts/
    └── setup.sh
```

---

## 01 — ChatGPT Clone

```
chatgpt-clone/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── app.py
│   ├── chat.py
│   ├── config.py
│   ├── history.py
│   └── streaming.py
├── tests/
│   ├── __init__.py
│   ├── test_chat.py
│   └── test_streaming.py
├── static/
│   └── index.html
└── scripts/
    └── setup.sh
```

**.env.example:**
```
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
DEFAULT_MODEL=gpt-4o-mini
MAX_TOKENS=2048
TEMPERATURE=0.7
PORT=8080
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
python-dotenv>=1.0.1
pydantic>=2.7.0
sse-starlette>=2.0.0
```

**scripts/setup.sh:**
```bash
#!/bin/bash
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
echo "Setup complete. Edit .env with your API keys."
```

---

## 02 — GraphRAG System

```
graphrag-system/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── extraction/
│   │   ├── __init__.py
│   │   ├── entities.py
│   │   └── relations.py
│   ├── graph/
│   │   ├── __init__.py
│   │   ├── builder.py
│   │   ├── store.py
│   │   └── query.py
│   ├── rag/
│   │   ├── __init__.py
│   │   ├── retriever.py
│   │   └── generator.py
│   └── ingestion/
│       ├── __init__.py
│       └── loader.py
├── tests/
│   ├── __init__.py
│   ├── test_extraction.py
│   └── test_queries.py
├── data/
│   ├── documents/
│   └── graph_export/
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
networkx>=3.3
neo4j>=5.22.0
numpy>=1.26.0
pandas>=2.2.0
chromadb>=0.5.0
tiktoken>=0.7.0
python-dotenv>=1.0.1
```

---

## 03 — Memory Agent

```
memory-agent/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── memory/
│   │   ├── __init__.py
│   │   ├── short_term.py
│   │   ├── long_term.py
│   │   ├── working.py
│   │   └── summarizer.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── core.py
│   │   └── context.py
│   └── storage/
│       ├── __init__.py
│       ├── vector_store.py
│       └── sql_store.py
├── tests/
│   ├── __init__.py
│   ├── test_memory.py
│   └── test_agent.py
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
chromadb>=0.5.0
sqlite-vec>=0.1.0
pydantic>=2.7.0
python-dotenv>=1.0.1
numpy>=1.26.0
```

---

## 04 — Research Agent

```
research-agent/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── planner.py
│   │   ├── executor.py
│   │   └── synthesizer.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── web_search.py
│   │   ├── web_scrape.py
│   │   └── summarizer.py
│   ├── memory/
│   │   ├── __init__.py
│   │   └── research_state.py
│   └── output/
│       ├── __init__.py
│       └── report.py
├── tests/
│   ├── __init__.py
│   ├── test_agent.py
│   └── test_tools.py
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
requests>=2.31.0
beautifulsoup4>=4.12.0
markdown>=3.6.0
weasyprint>=62.0
python-dotenv>=1.0.1
pydantic>=2.7.0
```

---

## 05 — AI Coding Agent

```
ai-coding-agent/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── planner.py
│   │   ├── coder.py
│   │   └── debugger.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── file_ops.py
│   │   ├── shell.py
│   │   └── search.py
│   └── sandbox/
│       ├── __init__.py
│       └── executor.py
├── tests/
│   ├── __init__.py
│   ├── test_coding.py
│   └── test_sandbox.py
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
docker>=7.0.0
gitpython>=3.1.0
python-dotenv>=1.0.1
pydantic>=2.7.0
pygments>=2.18.0
```

---

## 06 — PDF Chat

```
pdf-chat/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── app.py
│   ├── config.py
│   ├── ingestion/
│   │   ├── __init__.py
│   │   ├── loader.py
│   │   └── chunker.py
│   ├── rag/
│   │   ├── __init__.py
│   │   ├── embeddings.py
│   │   ├── retriever.py
│   │   └── generator.py
│   └── storage/
│       ├── __init__.py
│       └── vector_store.py
├── tests/
│   ├── __init__.py
│   ├── test_ingestion.py
│   └── test_rag.py
├── data/
│   └── pdfs/
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
pypdf2>=3.0.0
pypdfium2>=4.30.0
chromadb>=0.5.0
langchain-text-splitters>=0.2.0
tiktoken>=0.7.0
python-dotenv>=1.0.1
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
```

---

## 07 — Meeting Assistant

```
meeting-assistant/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── audio/
│   │   ├── __init__.py
│   │   ├── recorder.py
│   │   └── transcriber.py
│   ├── processing/
│   │   ├── __init__.py
│   │   ├── summarizer.py
│   │   ├── action_items.py
│   │   └── topics.py
│   └── memory/
│       ├── __init__.py
│       └── meeting_store.py
├── tests/
│   ├── __init__.py
│   ├── test_transcription.py
│   └── test_summary.py
├── data/
│   └── recordings/
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
pydub>=0.25.1
whisper>=1.1.10
chromadb>=0.5.0
python-dotenv>=1.0.1
pydantic>=2.7.0
sounddevice>=0.5.0
```

---

## 08 — Personal AI

```
personal-ai/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── personality.py
│   │   └── conversations.py
│   ├── memory/
│   │   ├── __init__.py
│   │   ├── episodic.py
│   │   ├── semantic.py
│   │   └── user_profile.py
│   └── services/
│       ├── __init__.py
│       ├── calendar.py
│       └── reminders.py
├── tests/
│   ├── __init__.py
│   ├── test_memory.py
│   └── test_agent.py
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
chromadb>=0.5.0
sqlite-vec>=0.1.0
pydantic>=2.7.0
python-dotenv>=1.0.1
schedule>=1.2.0
```

---

## 09 — Knowledge Base

```
knowledge-base/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── app.py
│   ├── config.py
│   ├── ingestion/
│   │   ├── __init__.py
│   │   ├── loader.py
│   │   ├── chunker.py
│   │   └── embedder.py
│   ├── retrieval/
│   │   ├── __init__.py
│   │   ├── vector_search.py
│   │   ├── keyword_search.py
│   │   └── hybrid.py
│   ├── generation/
│   │   ├── __init__.py
│   │   └── qa.py
│   └── storage/
│       ├── __init__.py
│       ├── documents.py
│       └── embeddings.py
├── tests/
│   ├── __init__.py
│   ├── test_ingestion.py
│   └── test_retrieval.py
├── data/
│   └── documents/
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
chromadb>=0.5.0
qdrant-client>=1.9.0
pypdf2>=3.0.0
tiktoken>=0.7.0
sentence-transformers>=3.0.0
bm25s>=0.1.0
python-dotenv>=1.0.1
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
```

---

## 10 — Support Agent

```
support-agent/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── app.py
│   ├── config.py
│   ├── classification/
│   │   ├── __init__.py
│   │   ├── intent.py
│   │   ├── sentiment.py
│   │   └── routing.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── responder.py
│   │   └── escalation.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── order_lookup.py
│   │   ├── refund.py
│   │   └── knowledge_base.py
│   └── memory/
│       ├── __init__.py
│       └── conversation.py
├── tests/
│   ├── __init__.py
│   ├── test_classification.py
│   └── test_agent.py
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
pydantic>=2.7.0
sqlalchemy>=2.0.0
python-dotenv>=1.0.1
```

---

## 11 — SQL Agent

```
sql-agent/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── nl_to_sql.py
│   │   ├── validator.py
│   │   └── executor.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── schema.py
│   │   └── database.py
│   └── memory/
│       ├── __init__.py
│       └── query_history.py
├── tests/
│   ├── __init__.py
│   ├── test_sql_generation.py
│   └── test_execution.py
├── data/
│   └── sample.db
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
sqlalchemy>=2.0.0
psycopg2-binary>=2.9.0
sqlite-vec>=0.1.0
tabulate>=0.9.0
python-dotenv>=1.0.1
pydantic>=2.7.0
```

---

## 12 — GitHub Agent

```
github-agent/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── planner.py
│   │   └── executor.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── github_api.py
│   │   ├── file_ops.py
│   │   └── search.py
│   └── memory/
│       ├── __init__.py
│       └── session.py
├── tests/
│   ├── __init__.py
│   ├── test_github_tools.py
│   └── test_agent.py
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
PyGithub>=2.3.0
gitpython>=3.1.0
pydantic>=2.7.0
python-dotenv>=1.0.1
httpx>=0.27.0
```

---

## 13 — Writing Assistant

```
writing-assistant/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── app.py
│   ├── config.py
│   ├── editor/
│   │   ├── __init__.py
│   │   ├── generator.py
│   │   ├── polisher.py
│   │   └── expander.py
│   ├── feedback/
│   │   ├── __init__.py
│   │   ├── critique.py
│   │   └── style.py
│   └── output/
│       ├── __init__.py
│       └── export.py
├── tests/
│   ├── __init__.py
│   ├── test_generation.py
│   └── test_feedback.py
├── data/
│   └── templates/
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
pydantic>=2.7.0
python-dotenv>=1.0.1
fastapi>=0.110.0
uvicorn[standard]>=0.29.0
markdown>=3.6.0
```

---

## 14 — AI Tutor

```
ai-tutor/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── tutor/
│   │   ├── __init__.py
│   │   ├── instructor.py
│   │   ├── quiz.py
│   │   └── feedback.py
│   ├── memory/
│   │   ├── __init__.py
│   │   ├── student_model.py
│   │   └── progress.py
│   ├── rag/
│   │   ├── __init__.py
│   │   ├── curriculum.py
│   │   └── retriever.py
│   └── storage/
│       ├── __init__.py
│       └── vector_store.py
├── tests/
│   ├── __init__.py
│   ├── test_tutor.py
│   └── test_quiz.py
├── data/
│   ├── curriculum/
│   └── exercises/
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
chromadb>=0.5.0
pydantic>=2.7.0
python-dotenv>=1.0.1
sqlalchemy>=2.0.0
```

---

## 15 — Financial Assistant

```
financial-assistant/
├── .env.example
├── .gitignore
├── README.md
├── requirements.txt
├── src/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── agent/
│   │   ├── __init__.py
│   │   ├── advisor.py
│   │   ├── planner.py
│   │   └── analyzer.py
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── market_data.py
│   │   ├── portfolio.py
│   │   └── calculator.py
│   ├── memory/
│   │   ├── __init__.py
│   │   └── user_prefs.py
│   └── analysis/
│       ├── __init__.py
│       ├── risk.py
│       └── reports.py
├── tests/
│   ├── __init__.py
│   ├── test_analysis.py
│   └── test_agent.py
├── data/
│   └── market_data/
└── scripts/
    └── setup.sh
```

**requirements.txt:**
```
openai>=1.30.0
anthropic>=0.35.0
yfinance>=0.2.40
pandas>=2.2.0
numpy>=1.26.0
matplotlib>=3.9.0
pydantic>=2.7.0
python-dotenv>=1.0.1
httpx>=0.27.0
```

---

## Common .gitignore Template

```gitignore
# Environments
venv/
.env
__pycache__/
*.pyc

# Data
data/*.db
data/vector_store/
logs/*.log

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Secrets
*.key
*.pem
credentials.json
```

---

## Common setup.py Template

```python
from setuptools import setup, find_packages

setup(
    name="project-name",
    version="0.1.0",
    packages=find_packages("src"),
    package_dir={"": "src"},
    python_requires=">=3.11",
    install_requires=[
        line.strip()
        for line in open("requirements.txt")
        if line.strip() and not line.startswith("#")
    ],
)
```

---

## Using the Templates

1. Copy the project folder structure.
2. Run `scripts/setup.sh` (or manually create venv and install requirements).
3. Copy `.env.example` to `.env` and fill in your API keys.
4. Start implementing from `src/main.py`.
5. Run tests with `pytest tests/`.

Each template is designed to be minimal — just enough structure to get started, not so much that it gets in the way.
