# Code Examples — Chapter 08: AI System Design

All examples assume you have API keys set in your environment:

```bash
pip install openai anthropic tiktoken redis numpy pandas matplotlib fastapi uvicorn sse-starlette
```

---

## 1. Chatbot with Context Management

```python
import openai
import tiktoken
from dataclasses import dataclass
from typing import List

client = openai.OpenAI()
enc = tiktoken.encoding_for_model("gpt-4o")

@dataclass
class Message:
    role: str
    content: str
    timestamp: float

class ContextManager:
    def __init__(self, max_tokens=128_000, system_prompt=""):
        self.max_tokens = max_tokens
        self.system_prompt = system_prompt
        self.messages: List[Message] = []
    
    def add_message(self, role, content):
        self.messages.append(Message(role, content, __import__("time").time()))
    
    def get_context(self, extra_tokens=0):
        budget = self.max_tokens - extra_tokens - 500  # safety margin
        
        # Start with system prompt
        context = [{"role": "system", "content": self.system_prompt}] if self.system_prompt else []
        context_tokens = len(enc.encode(self.system_prompt)) if self.system_prompt else 0
        
        # Add messages from newest to oldest
        sorted_messages = sorted(self.messages, key=lambda m: m.timestamp)
        
        for msg in reversed(sorted_messages):
            msg_tokens = len(enc.encode(msg.content))
            if context_tokens + msg_tokens > budget:
                # Summarize remaining instead of adding
                remaining = [m for m in reversed(sorted_messages) if m.timestamp < msg.timestamp]
                if remaining:
                    summary = self.summarize(remaining)
                    context.insert(1, {"role": "system", "content": f"Earlier: {summary}"})
                break
            context.append({"role": msg.role, "content": msg.content})
            context_tokens += msg_tokens
        
        return context
    
    def summarize(self, messages):
        text = "\n".join(f"{m.role}: {m.content}" for m in messages)
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"Summarize this conversation in 200 tokens:\n{text}"
            }],
            max_tokens=300,
        )
        return resp.choices[0].message.content

# Usage
cm = ContextManager(system_prompt="You are a helpful assistant.")
cm.add_message("user", "What is machine learning?")
cm.add_message("assistant", "Machine learning is a subset of AI...")
cm.add_message("user", "Can you give me an example?")

context = cm.get_context()
resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=context,
)
print(resp.choices[0].message.content)
```

---

## 2. Research Assistant with Web Search

```python
import openai
import json
from urllib.parse import quote_plus
import httpx

client = openai.OpenAI()

class ResearchAssistant:
    def __init__(self, search_api_key):
        self.search_api_key = search_api_key
    
    async def research(self, query: str) -> dict:
        # Step 1: Query understanding
        analysis = self.understand_query(query)
        
        # Step 2: Parallel search across sources
        web_results = await self.search_web(analysis["search_queries"])
        # knowledge_results = await self.search_knowledge_base(query)
        
        # Step 3: Source ranking
        ranked = self.rank_sources(web_results)
        
        # Step 4: Content extraction
        extracted = await self.extract_content(ranked[:5])
        
        # Step 5: Synthesis with citations
        answer = self.synthesize(query, extracted)
        
        return answer
    
    def understand_query(self, query):
        return json.loads(client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"""Analyze this research query and return JSON:
Query: {query}

Return: {{"search_queries": ["query1", "query2"], "intent": "factual|comparative", "entities": []}}"""
            }],
            response_format={"type": "json_object"},
        ).choices[0].message.content)
    
    async def search_web(self, queries):
        results = []
        async with httpx.AsyncClient() as client:
            for q in queries:
                resp = await client.get(
                    f"https://api.serpapi.com/search",
                    params={"q": q, "api_key": self.search_api_key, "num": 5},
                )
                results.extend(resp.json().get("organic_results", []))
        return results
    
    def rank_sources(self, sources):
        DOMAIN_RANKINGS = {
            "wikipedia.org": 0.9, ".edu": 0.95,
            ".gov": 1.0, ".org": 0.7,
            "arxiv.org": 0.95, "github.com": 0.6,
        }
        
        for source in sources:
            domain_rank = 0.3
            for domain, rank in DOMAIN_RANKINGS.items():
                if domain in source.get("link", ""):
                    domain_rank = rank
                    break
            source["score"] = 0.6 * domain_rank + 0.4 * source.get("position", 5) / 10
        
        return sorted(sources, key=lambda s: s["score"], reverse=True)
    
    def synthesize(self, query, sources):
        context = "\n\n".join(
            f"[{i+1}] {s['title']}\n{s.get('snippet', '')}"
            for i, s in enumerate(sources)
        )
        
        return client.chat.completions.create(
            model="gpt-4o",
            messages=[{
                "role": "system",
                "content": f"Synthesize an answer using ONLY these sources. Cite [1], [2], etc."
            }, {
                "role": "user",
                "content": f"Query: {query}\n\nSources:\n{context}"
            }],
            temperature=0,
        ).choices[0].message.content
```

---

## 3. Coding Agent with Multi-File Edit

```python
import openai
import subprocess
import ast
from pathlib import Path
from typing import List

client = openai.OpenAI()

class CodingAgent:
    def __init__(self, repo_path: str):
        self.repo = Path(repo_path)
    
    def solve_task(self, task: str):
        # Step 1: Understand codebase
        context = self.get_repo_context()
        
        # Step 2: Plan changes
        plan = self.plan_changes(task, context)
        
        # Step 3: Apply changes
        for change in plan:
            self.apply_change(change)
        
        # Step 4: Test and fix
        for _ in range(3):
            test_results = self.run_tests()
            if test_results["success"]:
                return {"status": "success", "plan": plan}
            self.fix_failures(test_results)
        
        return {"status": "requires_review", "plan": plan, "test_errors": test_results}
    
    def get_repo_context(self):
        files = list(self.repo.rglob("*.py"))[:50]
        context = []
        for f in files[:10]:  # Read top 10 files
            rel = f.relative_to(self.repo)
            context.append(f"=== {rel} ===\n{f.read_text()[:2000]}")
        return "\n".join(context)
    
    def plan_changes(self, task, context):
        prompt = f"""Task: {task}

Repository structure:
{context}

Plan the changes needed. Return a JSON list of:
[{{"action": "create|modify|delete", "file": "path", "content": "full content (for create)", "changes": [{{"old": "...", "new": "..."}}]}}]"""
        
        resp = client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": prompt}],
            response_format={"type": "json_object"},
            temperature=0,
        )
        return json.loads(resp.choices[0].message.content)["changes"]
    
    def apply_change(self, change):
        file_path = self.repo / change["file"]
        
        if change["action"] == "create":
            file_path.parent.mkdir(parents=True, exist_ok=True)
            file_path.write_text(change["content"])
        
        elif change["action"] == "modify":
            content = file_path.read_text()
            for edit in change.get("changes", []):
                content = content.replace(edit["old"], edit["new"])
            file_path.write_text(content)
        
        # Validate syntax
        if file_path.suffix == ".py":
            try:
                ast.parse(file_path.read_text())
            except SyntaxError as e:
                print(f"Syntax error in {change['file']}: {e}")
    
    def run_tests(self):
        result = subprocess.run(
            ["python", "-m", "pytest", "-x", "--tb=short"],
            capture_output=True, text=True, cwd=self.repo,
        )
        return {
            "success": result.returncode == 0,
            "output": result.stdout + result.stderr,
        }
    
    def fix_failures(self, test_results):
        """Use LLM to fix failing tests."""
        files = list(self.repo.rglob("*.py"))[:10]
        code = "\n".join(f"=== {f.relative_to(self.repo)} ===\n{f.read_text()[:2000]}" for f in files)
        
        fix = client.chat.completions.create(
            model="gpt-4o",
            messages=[{
                "role": "user",
                "content": f"""Fix these test failures:
{test_results['output'][:3000]}

Current code:
{code}

Return the fixes as JSON changes."""
            }],
            response_format={"type": "json_object"},
        )
        
        changes = json.loads(fix.choices[0].message.content)
        for change in changes.get("changes", []):
            self.apply_change(change)
```

---

## 4. Customer Support Router

```python
import openai
import json
from enum import Enum

client = openai.OpenAI()

class Category(Enum):
    BILLING = "billing"
    TECHNICAL = "technical"
    ACCOUNT = "account"
    GENERAL = "general"

class SupportRouter:
    def __init__(self):
        self.knowledge_base = self.load_kb()
        self.handlers = {
            Category.BILLING: BillingHandler(),
            Category.TECHNICAL: TechnicalHandler(),
            Category.ACCOUNT: AccountHandler(),
            Category.GENERAL: GeneralHandler(),
        }
    
    def load_kb(self):
        return {
            "return_policy": "Items can be returned within 30 days...",
            "shipping": "Free shipping on orders over $50...",
            "refund": "Refunds are processed within 5-7 business days...",
        }
    
    async def handle_ticket(self, ticket: dict) -> dict:
        # Step 1: Classify
        classification = await self.classify(ticket["text"])
        
        # Step 2: Sentiment analysis
        sentiment = await self.analyze_sentiment(ticket["text"])
        
        # Step 3: Knowledge retrieval
        knowledge = self.retrieve_knowledge(classification["category"], ticket["text"])
        
        # Step 4: Route to handler
        handler = self.handlers[classification["category"]]
        response = await handler.generate(ticket, knowledge, sentiment)
        
        # Step 5: Confidence check
        if response["confidence"] < 0.7 or sentiment["requires_human"]:
            return {
                "action": "escalate",
                "response": response["text"],
                "reason": "low confidence" if response["confidence"] < 0.7 else "sentiment",
                "assigned_to": "human_agent",
            }
        
        return {"action": "auto_reply", "response": response["text"]}
    
    async def classify(self, text: str) -> dict:
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"Classify this support ticket:\n{text}\n\nCategory: billing, technical, account, general"
            }],
            response_format={"type": "json_object"},
        )
        return json.loads(resp.choices[0].message.content)
    
    async def analyze_sentiment(self, text: str) -> dict:
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"Analyze sentiment:\n{text}\n\nReturn JSON: {{sentiment: positive|neutral|negative|angry, urgency: 0-1, requires_human: bool}}"
            }],
            response_format={"type": "json_object"},
        )
        return json.loads(resp.choices[0].message.content)
    
    def retrieve_knowledge(self, category, query):
        if category == "billing":
            return [v for k, v in self.knowledge_base.items() if k in ["return_policy", "refund"]]
        return list(self.knowledge_base.values())


class BillingHandler:
    async def generate(self, ticket, knowledge, sentiment):
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "system",
                "content": f"You are billing support. Use this knowledge:\n{knowledge}"
            }, {
                "role": "user",
                "content": ticket["text"]
            }],
        )
        return {
            "text": resp.choices[0].message.content,
            "confidence": 0.85,
        }

# Usage
# router = SupportRouter()
# result = await router.handle_ticket({"id": "T123", "text": "I want a refund for order #456"})
```

---

## 5. Document Q&A System

```python
import openai
import tiktoken
import numpy as np
from typing import List
from pathlib import Path

client = openai.OpenAI()
enc = tiktoken.encoding_for_model("gpt-4o")

class DocumentQA:
    def __init__(self):
        self.chunks = []
        self.embeddings = None
    
    def ingest_pdf(self, pdf_path: str):
        """Simple text-based PDF ingestion."""
        # In production, use PyMuPDF, pdfplumber, or marker-pdf
        import PyPDF2
        text = ""
        with open(pdf_path, "rb") as f:
            reader = PyPDF2.PdfReader(f)
            for page in reader.pages:
                text += page.extract_text() + "\n\n"
        
        self.chunk_text(text)
        self.index_chunks()
    
    def chunk_text(self, text: str, chunk_size=512, overlap=64):
        paragraphs = text.split("\n\n")
        self.chunks = []
        current = ""
        
        for para in paragraphs:
            para_tokens = len(enc.encode(para))
            current_tokens = len(enc.encode(current))
            
            if current_tokens + para_tokens > chunk_size and current:
                self.chunks.append(current.strip())
                # Overlap: keep last `overlap` tokens
                overlap_text = enc.decode(enc.encode(current)[-overlap:])
                current = overlap_text + "\n" + para
            else:
                current += "\n" + para if current else para
        
        if current:
            self.chunks.append(current.strip())
    
    def index_chunks(self):
        self.embeddings = []
        # Batch embedding for efficiency
        batch_size = 20
        for i in range(0, len(self.chunks), batch_size):
            batch = self.chunks[i:i+batch_size]
            resp = client.embeddings.create(
                model="text-embedding-3-small",
                input=batch,
            )
            self.embeddings.extend([d.embedding for d in resp.data])
        self.embeddings = np.array(self.embeddings)
    
    def retrieve(self, query: str, top_k=5):
        query_emb = np.array(
            client.embeddings.create(
                model="text-embedding-3-small",
                input=query,
            ).data[0].embedding
        )
        
        similarities = np.dot(self.embeddings, query_emb) / (
            np.linalg.norm(self.embeddings, axis=1) * np.linalg.norm(query_emb)
        )
        
        top_indices = np.argsort(similarities)[-top_k:][::-1]
        return [self.chunks[i] for i in top_indices], similarities[top_indices]
    
    def answer(self, query: str):
        chunks, scores = self.retrieve(query)
        
        context = "\n\n---\n\n".join(
            f"[Source {i+1} (relevance: {s:.2f})]\n{c}"
            for i, (c, s) in enumerate(zip(chunks, scores))
        )
        
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "system",
                "content": "Answer using ONLY the provided sources. Cite sources as [Source 1], [Source 2], etc. Say 'I cannot find this in the document' if the answer isn't in the sources."
            }, {
                "role": "user",
                "content": f"Query: {query}\n\nSources:\n{context}"
            }],
            temperature=0,
        )
        
        return {
            "answer": resp.choices[0].message.content,
            "sources": chunks,
            "relevance_scores": scores.tolist(),
        }

# Usage
# dqa = DocumentQA()
# dqa.ingest_pdf("document.pdf")
# result = dqa.answer("What is the main conclusion?")
```

---

## 6. NL-to-SQL System

```python
import openai
import json
import sqlite3
import pandas as pd
import matplotlib.pyplot as plt
from io import BytesIO
import base64

client = openai.OpenAI()

class NLToSQL:
    def __init__(self, db_path: str):
        self.conn = sqlite3.connect(db_path)
        self.schema = self.get_schema()
        self.sample_data = self.get_sample_data()
    
    def get_schema(self):
        cursor = self.conn.cursor()
        cursor.execute("SELECT name FROM sqlite_master WHERE type='table';")
        tables = cursor.fetchall()
        
        schema = {}
        for (table_name,) in tables:
            cursor.execute(f"PRAGMA table_info({table_name});")
            columns = cursor.fetchall()
            schema[table_name] = [
                {"name": col[1], "type": col[2], "nullable": not col[3]}
                for col in columns
            ]
        return schema
    
    def get_sample_data(self, rows=3):
        samples = {}
        for table in self.schema:
            try:
                samples[table] = pd.read_sql(
                    f"SELECT * FROM {table} LIMIT {rows}", self.conn
                ).to_dict(orient="records")
            except Exception:
                samples[table] = []
        return samples
    
    def generate_sql(self, question: str) -> str:
        prompt = f"""Database schema:
{json.dumps(self.schema, indent=2)}

Sample data:
{json.dumps(self.sample_data, indent=2)}

Question: {question}

Generate a SQLite SQL query. Rules:
- SELECT only (no INSERT/UPDATE/DELETE/DROP)
- Add LIMIT 100
- Return ONLY the SQL query."""

        resp = client.chat.completions.create(
            model="gpt-4o",
            messages=[{"role": "user", "content": prompt}],
            temperature=0,
        )
        
        sql = resp.choices[0].message.content.strip()
        sql = sql.replace("```sql", "").replace("```", "").strip()
        
        # Security validation
        forbidden = ["INSERT", "UPDATE", "DELETE", "DROP", "ALTER", "CREATE", "EXEC"]
        for kw in forbidden:
            if kw in sql.upper().split():
                raise ValueError(f"Forbidden SQL keyword: {kw}")
        
        return sql
    
    def execute_sql(self, sql: str) -> pd.DataFrame:
        return pd.read_sql(sql, self.conn)
    
    def interpret_result(self, question: str, df: pd.DataFrame) -> str:
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"""Question: {question}

Query result (first 20 rows):
{df.head(20).to_string()}

Summarize the result in plain English:"""
            }],
            temperature=0,
        )
        return resp.choices[0].message.content
    
    def visualize(self, df: pd.DataFrame, question: str) -> str:
        """Generate a chart and return base64 encoded image."""
        numeric_cols = df.select_dtypes(include=["number"]).columns
        cat_cols = df.select_dtypes(include=["object"]).columns
        
        if len(numeric_cols) >= 1 and len(cat_cols) >= 1:
            fig, ax = plt.subplots(figsize=(8, 4))
            df.plot(kind="bar", x=cat_cols[0], y=numeric_cols[0], ax=ax)
            plt.title(question[:50])
            plt.tight_layout()
            
            buf = BytesIO()
            plt.savefig(buf, format="png")
            buf.seek(0)
            img_b64 = base64.b64encode(buf.read()).decode()
            plt.close()
            return img_b64
        
        return ""
    
    def ask(self, question: str) -> dict:
        try:
            sql = self.generate_sql(question)
            df = self.execute_sql(sql)
            interpretation = self.interpret_result(question, df)
            chart = self.visualize(df, question)
            
            return {
                "success": True,
                "sql": sql,
                "result": df.head(20).to_dict(orient="records"),
                "interpretation": interpretation,
                "chart_base64": chart if chart else None,
            }
        except Exception as e:
            return {"success": False, "error": str(e)}

# Usage
# nlsql = NLToSQL("chinook.db")
# result = nlsql.ask("What are the top 5 best-selling tracks?")
```

---

## 7. Content Generation Pipeline

```python
import openai
import json

client = openai.OpenAI()

class ContentPipeline:
    def __init__(self, brand_guide: dict = None):
        self.brand = brand_guide or {
            "voice": "Professional and approachable",
            "tone": "Confident but humble",
            "audience": "Technical professionals",
            "forbidden_terms": ["revolutionary", "game-changing", "disruptive"],
        }
    
    def generate(self, brief: str) -> dict:
        outline = self.create_outline(brief)
        approved = self.review_outline(outline, brief)
        
        if not approved["approved"]:
            outline = self.revise_outline(outline, approved["feedback"])
        
        draft = self.write_draft(outline)
        edited = self.edit_pass(draft, pass_number=1)
        edited = self.edit_pass(edited, pass_number=2)
        edited = self.edit_pass(edited, pass_number=3)
        
        final = self.final_review(edited)
        
        return {
            "outline": outline,
            "draft": draft,
            "final": final,
            "metadata": {
                "word_count": len(final.split()),
                "reading_time_minutes": len(final.split()) / 200,
                "brand_compliance": self.check_brand_compliance(final),
            }
        }
    
    def create_outline(self, brief):
        resp = client.chat.completions.create(
            model="gpt-4o",
            messages=[{
                "role": "user",
                "content": f"""Create a detailed content outline from this brief:
{brief}

Return JSON with sections array: {{title, subsections: [title, key_points: []]}}"""
            }],
            response_format={"type": "json_object"},
        )
        return json.loads(resp.choices[0].message.content)
    
    def review_outline(self, outline, brief):
        resp = client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[{
                "role": "user",
                "content": f"""Review this outline against the brief:
Brief: {brief}
Outline: {json.dumps(outline)}

Is it approved? Return JSON: {{approved: bool, feedback: ""}}"""
            }],
            response_format={"type": "json_object"},
        )
        return json.loads(resp.choices[0].message.content)
    
    def revise_outline(self, outline, feedback):
        resp = client.chat.completions.create(
            model="gpt-4o",
            messages=[{
                "role": "user",
                "content": f"""Revise this outline based on feedback:
Outline: {json.dumps(outline)}
Feedback: {feedback}

Return revised outline as JSON."""
            }],
            response_format={"type": "json_object"},
        )
        return json.loads(resp.choices[0].message.content)
    
    def write_draft(self, outline):
        resp = client.chat.completions.create(
            model="gpt-4o",
            messages=[{
                "role": "system",
                "content": f"Write in this voice: {self.brand['voice']}. Tone: {self.brand['tone']}. Audience: {self.brand['audience']}. Avoid: {', '.join(self.brand['forbidden_terms'])}."
            }, {
                "role": "user",
                "content": f"Write a complete draft following this outline:\n{json.dumps(outline, indent=2)}"
            }],
            temperature=0.7,
            max_tokens=4000,
        )
        return resp.choices[0].message.content
    
    def edit_pass(self, text, pass_number):
        edit_instructions = {
            1: "Improve clarity, fix grammar, tighten sentences. Remove passive voice where active is better.",
            2: f"Apply brand voice: {self.brand['voice']}. Ensure tone matches: {self.brand['tone']}.",
            3: "Final polish: improve flow, transitions, add call-to-action. Remove any remaining issues.",
        }
        
        resp = client.chat.completions.create(
            model="gpt-4o",
            messages=[{
                "role": "user",
                "content": f"Edit pass {pass_number}: {edit_instructions[pass_number]}\n\nText:\n{text}"
            }],
            temperature=0.3,
        )
        return resp.choices[0].message.content
    
    def final_review(self, text):
        resp = client.chat.completions.create(
            model="gpt-4o",
            messages=[{
                "role": "user",
                "content": f"Do a final review. Check for: factual accuracy, brand voice consistency, grammar, flow. Return the final version:\n\n{text}"
            }],
            temperature=0.2,
        )
        return resp.choices[0].message.content
    
    def check_brand_compliance(self, text):
        violations = []
        for term in self.brand["forbidden_terms"]:
            if term.lower() in text.lower():
                violations.append(f"Contains forbidden term: '{term}'")
        return {"compliant": len(violations) == 0, "violations": violations}

# Usage
# cp = ContentPipeline()
# result = cp.generate("Write a blog post about best practices for building AI chatbots in production")
```

---

## 8. Architecture Pattern Implementations

### Orchestrator Pattern

```python
class Orchestrator:
    def __init__(self, tools: dict):
        self.tools = tools
    
    async def execute(self, task: str) -> str:
        messages = [{
            "role": "system",
            "content": f"""You are an orchestrator. You have these tools:
{json.dumps({k: v.description for k, v in self.tools.items()}, indent=2)}

Plan and execute steps. For each step, respond with:
TOOL: tool_name
ARGS: {{"arg": "value"}}

After getting results, continue or respond with:
FINAL: your answer"""
        }, {
            "role": "user",
            "content": task
        }]
        
        max_iterations = 10
        for i in range(max_iterations):
            resp = await client.chat.completions.create(
                model="gpt-4o",
                messages=messages,
                temperature=0,
            )
            
            content = resp.choices[0].message.content
            
            if content.startswith("FINAL:"):
                return content.replace("FINAL:", "").strip()
            
            if content.startswith("TOOL:"):
                lines = content.split("\n")
                tool_name = lines[0].replace("TOOL:", "").strip()
                args = json.loads("\n".join(lines[1:]).replace("ARGS:", "").strip())
                
                result = await self.tools[tool_name].execute(**args)
                messages.append({"role": "assistant", "content": content})
                messages.append({"role": "user", "content": f"Result: {json.dumps(result)}"})
        
        return "Max iterations reached"
```

### Pipeline Pattern

```python
from abc import ABC, abstractmethod

class Stage(ABC):
    @abstractmethod
    async def process(self, data: dict) -> dict:
        pass

class Pipeline:
    def __init__(self):
        self.stages: list[Stage] = []
    
    def add_stage(self, stage: Stage):
        self.stages.append(stage)
    
    async def execute(self, input_data: dict) -> dict:
        data = input_data
        for stage in self.stages:
            data = await stage.process(data)
        return data

# Example stages for document Q&A
class ParseStage(Stage):
    async def process(self, data):
        # Parse PDF to text
        data["text"] = data["pdf_path"]
        return data

class ChunkStage(Stage):
    async def process(self, data):
        # Split into chunks
        data["chunks"] = data["text"].split("\n\n")
        return data

class RetrieveStage(Stage):
    async def process(self, data):
        # Retrieve relevant chunks
        data["relevant"] = [c for c in data["chunks"] if data["query"].lower() in c.lower()]
        return data

class GenerateStage(Stage):
    async def process(self, data):
        data["answer"] = f"Based on the context: {data['relevant'][:3]}"
        return data

# Usage
# pipeline = Pipeline()
# pipeline.add_stage(ParseStage())
# pipeline.add_stage(ChunkStage())
# pipeline.add_stage(RetrieveStage())
# pipeline.add_stage(GenerateStage())
# result = await pipeline.execute({"pdf_path": "doc.pdf", "query": "What is AI?"})
```

### Router Pattern

```python
class Router:
    def __init__(self):
        self.routes = {}
        self.default_handler = None
    
    def register(self, condition_fn, handler):
        self.routes[condition_fn] = handler
    
    async def route(self, request):
        for condition, handler in self.routes.items():
            if await condition(request):
                return await handler(request)
        
        if self.default_handler:
            return await self.default_handler(request)
        
        raise ValueError("No handler found")

# Usage
async def is_technical(req): return "error" in req["text"].lower()
async def is_billing(req): return any(kw in req["text"].lower() for kw in ["refund", "charge", "payment"])

router = Router()
router.register(is_technical, lambda r: {"response": "Technical support response"})
router.register(is_billing, lambda r: {"response": "Billing support response"})
router.default_handler = lambda r: {"response": "General support response"}

# result = await router.route({"text": "I have a payment issue"})
```

---

## 9. Streaming Chat Endpoint

```python
from fastapi import FastAPI
from pydantic import BaseModel
from sse_starlette.sse import EventSourceResponse
import asyncio
import openai

app = FastAPI()
client = openai.OpenAI()

class ChatRequest(BaseModel):
    messages: list
    model: str = "gpt-4o-mini"
    temperature: float = 0.7

@app.post("/chat/stream")
async def chat_stream(req: ChatRequest):
    async def event_generator():
        collected = []
        usage = {"input_tokens": 0, "output_tokens": 0}
        
        stream = client.chat.completions.create(
            model=req.model,
            messages=req.messages,
            temperature=req.temperature,
            stream=True,
            stream_options={"include_usage": True},
        )
        
        for chunk in stream:
            if chunk.choices:
                delta = chunk.choices[0].delta
                if delta.content:
                    collected.append(delta.content)
                    yield {
                        "event": "token",
                        "data": delta.content,
                    }
                if delta.tool_calls:
                    yield {
                        "event": "tool_call",
                        "data": delta.tool_calls[0].model_dump_json(),
                    }
            
            if chunk.usage:
                usage = {
                    "input_tokens": chunk.usage.prompt_tokens,
                    "output_tokens": chunk.usage.completion_tokens,
                }
        
        yield {
            "event": "done",
            "data": json.dumps({
                "full_text": "".join(collected),
                "usage": usage,
            }),
        }
    
    return EventSourceResponse(event_generator())

# Run: uvicorn examples:app --reload
```

---

## 10. Cost-Aware Model Router

```python
import time
from dataclasses import dataclass

@dataclass
class ModelConfig:
    name: str
    provider: str
    input_price: float
    output_price: float
    avg_latency_ms: float
    capabilities: set

class CostAwareRouter:
    def __init__(self):
        self.models = {
            "cheap": ModelConfig("gpt-4o-mini", "openai", 0.15, 0.60, 500, {"simple", "classification"}),
            "balanced": ModelConfig("gpt-4o", "openai", 2.50, 10.00, 800, {"complex", "reasoning"}),
            "reasoning": ModelConfig("o3-mini", "openai", 10.00, 40.00, 5000, {"math", "code", "hard_reasoning"}),
        }
        self.budget = {"daily": 10.0, "monthly": 300.0}
        self.spent = {"daily": 0.0, "monthly": 0.0}
        self.reset_time = time.time() + 86400
    
    def select_model(self, task: dict) -> str:
        complexity = task.get("complexity", "simple")
        max_latency = task.get("max_latency_ms", 5000)
        requires_reasoning = task.get("requires_reasoning", False)
        
        candidates = []
        for name, config in self.models.items():
            if config.avg_latency_ms > max_latency:
                continue
            if requires_reasoning and "hard_reasoning" not in config.capabilities:
                continue
            candidates.append((config.input_price + config.output_price, name))
        
        if not candidates:
            return min(self.models.keys(), key=lambda m: self.models[m].avg_latency_ms)
        
        # Return cheapest that meets requirements
        return min(candidates, key=lambda c: c[0])[1]
    
    def estimate_cost(self, model_name: str, input_tokens: int, output_tokens: int) -> float:
        config = self.models[model_name]
        return (input_tokens * config.input_price + output_tokens * config.output_price) / 1_000_000
    
    def check_budget(self, cost: float) -> bool:
        if time.time() > self.reset_time:
            self.spent = {"daily": 0.0, "monthly": 0.0}
            self.reset_time = time.time() + 86400
        
        if self.spent["daily"] + cost > self.budget["daily"]:
            return False
        if self.spent["monthly"] + cost > self.budget["monthly"]:
            return False
        
        self.spent["daily"] += cost
        self.spent["monthly"] += cost
        return True

# Usage
# router = CostAwareRouter()
# model = router.select_model({"complexity": "simple", "max_latency_ms": 2000})
# cost = router.estimate_cost(model, 1000, 200)
# if router.check_budget(cost):
#     print(f"Using {model}, cost: ${cost:.5f}")
```
