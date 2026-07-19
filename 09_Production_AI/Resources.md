# Chapter 09: Production AI — Resources

## Books

| Title | Author | Focus |
|-------|--------|-------|
| **Building LLM Apps for Production** | Louis-François Bouchard et al. | End-to-end LLM application development |
| **AI Engineering** | Chip Huyen | Full-stack ML systems design |
| **Designing Data-Intensive Applications** | Martin Kleppmann | Distributed systems fundamentals |
| **Site Reliability Engineering** | Niall Richard Murphy et al. | Production reliability patterns |
| **The Security Engineering Toolkit** | Andrew J. Behr | AI-specific security patterns |
| **Prompt Engineering for LLMs** | Valentina Alto | Prompt design and testing |

## Official Documentation & Guides

### Monitoring & Observability
- **LangSmith**: https://docs.smith.langchain.com/
- **LangFuse**: https://langfuse.com/docs
- **Helicone**: https://docs.helicone.ai/
- **OpenTelemetry**: https://opentelemetry.io/docs/
- **Prometheus**: https://prometheus.io/docs/introduction/overview/
- **Grafana**: https://grafana.com/docs/grafana/latest/

### Evaluation
- **OpenAI Evals**: https://github.com/openai/evals
- **DeepEval**: https://docs.confident-ai.com/
- **LangChain Evaluation**: https://python.langchain.com/docs/guides/evaluation/
- **RAGAS (RAG Evaluation)**: https://docs.ragas.io/
- **BERTScore**: https://github.com/Tiiiger/bert_score
- **Hugging Face Evaluate**: https://huggingface.co/docs/evaluate/index

### Safety & Guardrails
- **Guardrails AI**: https://docs.guardrailsai.com/
- **NVIDIA NeMo Guardrails**: https://docs.nvidia.com/nemo/guardrails/
- **Lakera Guard**: https://docs.lakera.ai/
- **Rebuff**: https://docs.rebuff.ai/
- **Anthropic Safety**: https://docs.anthropic.com/claude/docs/safety-best-practices
- **OpenAI Safety**: https://platform.openai.com/docs/guides/safety-best-practices
- **MLCommons AI Safety**: https://mlcommons.org/working-groups/ai-safety/

### Deployment & Serving
- **vLLM**: https://docs.vllm.ai/
- **TGI (Text Generation Inference)**: https://huggingface.co/docs/text-generation-inference
- **NVIDIA Triton**: https://docs.nvidia.com/deeplearning/triton-inference-server/
- **Kubernetes**: https://kubernetes.io/docs/
- **Helm for ML**: https://github.com/bitnami/charts/tree/main/bitnami
- **Ray Serve**: https://docs.ray.io/en/latest/serve/index.html
- **BentoML**: https://docs.bentoml.com/

### Cost Management
- **OpenAI Cost Calculator**: https://openai.com/pricing
- **Anthropic Pricing**: https://www.anthropic.com/pricing
- **AWS Bedrock Pricing**: https://aws.amazon.com/bedrock/pricing/
- **Helicone Cost Tracking**: https://docs.helicone.ai/features/tracking-costs
- **Agenta**: https://docs.agenta.ai/ (prompt management + cost)

## Papers

### Evaluation & Quality
- **"Judging LLM-as-a-Judge"** (Zheng et al., 2024) — MT-Bench, evaluating evaluators
- **"BLEU: a Method for Automatic Evaluation of Machine Translation"** (Papineni et al., 2002)
- **"ROUGE: A Package for Automatic Evaluation of Summaries"** (Lin, 2004)
- **"BERTScore: Evaluating Text Generation with BERT"** (Zhang et al., 2020)
- **"G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment"** (Liu et al., 2023)

### Safety & Security
- **"Universal and Transferable Adversarial Attacks on Aligned Language Models"** (Zou et al., 2023)
- **"Jailbroken: How Does LLM Safety Training Fail?"** (Wei et al., 2024)
- **"The Janus Interface: How Fine-Tuning on Small Datasets Attacks LLM Safety"** (Qi et al., 2024)
- **"Ignore Previous Prompt: A Technique for Prompt Injection"** (Perez & Ribeiro, 2022)
- **"Not what you've signed up for: Compromising Real-World LLM-Integrated Applications"** (Greshake et al., 2023)

### Production Systems
- **"LLM in Production: A Systematic Literature Review"** (2024)
- **"Serving LLMs Efficiently: A Survey"** (Miao et al., 2024)
- **"Efficient Memory Management for Large Language Model Serving with PagedAttention"** (Kwon et al., 2023) — vLLM paper
- **"FlashAttention: Fast and Memory-Efficient Exact Attention"** (Dao et al., 2022)

## Tools & Frameworks

### Comprehensive Platforms
| Tool | Description | Cost |
|------|-------------|------|
| **LangSmith** | Full LLM lifecycle platform | Free tier, paid SaaS |
| **LangFuse** | Open-source LLM observability | Free, self-host or cloud |
| **Helicone** | LLM API proxy with monitoring | Free tier, usage-based |
| **MLflow** | ML lifecycle management | Free, open-source |
| **Weights & Biases** | Experiment tracking | Free tier, paid SaaS |
| **Arize AI** | ML observability | Free tier, paid SaaS |
| **WhyLabs** | AI observability | Free tier, paid SaaS |

### Evaluation Tools
| Tool | Description |
|------|-------------|
| **DeepEval** | Unit testing for LLMs |
| **RAGAS** | RAG pipeline evaluation |
| **OpenAI Evals** | Open-source eval framework |
| **LM Evaluation Harness** | Standardized model eval |
| **AlpacaEval** | Instruction-following eval |

### Guardrails
| Tool | Description |
|------|-------------|
| **Guardrails AI** | Python guardrail framework |
| **NeMo Guardrails** | Enterprise guardrails toolkit |
| **Lakera Guard** | API-based guardrail service |
| **Rebuff** | Prompt injection protection |
| **Azure AI Content Safety** | Microsoft's content filter |

### Deployment
| Tool | Description |
|------|-------------|
| **vLLM** | High-throughput LLM serving |
| **TGI** | Hugging Face inference server |
| **Triton** | NVIDIA inference server |
| **Ollama** | Local model serving (dev) |
| **LocalAI** | Self-hosted OpenAI alternative |
| **llama.cpp** | CPU-optimized inference |

### Security
| Tool | Description |
|------|-------------|
| **Vault (HashiCorp)** | Secrets management |
| **AWS Secrets Manager** | Cloud secrets storage |
| **OAuth2 Proxy** | Authentication gateway |
| **Fail2Ban** | Brute force protection |

## Communities & Conferences

### Communities
- **LangChain Discord**: https://discord.gg/langchain
- **Hugging Face Discord**: https://discord.gg/huggingface
- **r/LocalLLaMA**: https://reddit.com/r/LocalLLaMA
- **r/MachineLearning**: https://reddit.com/r/MachineLearning
- **AI Engineering Discord**: Various communities

### Conferences
- **NeurIPS** — ML/AI research
- **ICML** — ML research
- **MLOps World** — Production ML
- **KubeCon** — Kubernetes ecosystem
- **REWORK AI Summit** — Applied AI
- **AI Safety Summit** — Safety research

## Courses & Tutorials

- **"Full Stack LLM Bootcamp"** (Hugging Face): https://fullstackllmbootcamp.com/
- **"LLM Engineering"** (DeepLearning.AI): https://www.deeplearning.ai/courses/
- **"MLOps Specialization"** (DeepLearning.AI): https://www.deeplearning.ai/courses/machine-learning-engineering-for-production-mlops/
- **"Kubernetes for ML"** (AIBorne): https://training.kubernetes.agileops.dev/
- **Stanford CS329S: ML Systems Design**: https://stanford-cs329s.github.io/
- **Berkeley CS188: AI Safety**: Various resources

## Templates & Starter Kits

- **LangChain Template**: https://github.com/langchain-ai/langchain-template
- **LLM App Starter**: https://github.com/your-repo/llm-production-starter
- **Helicone Proxy Config**: https://docs.helicone.ai/getting-started/quick-start
- **Kubernetes LLM Manifests**: https://github.com/ray-project/kuberay
- **Guardrails Starter**: https://github.com/guardrails-ai/starter-kit

## Blogs & Articles

- **"The Building Blocks of LLM Observability"** (LangSmith Blog)
- **"Production LLM Monitoring"** (LangFuse Blog)
- **"How to Evaluate LLM Applications"** (Weights & Biases)
- **"Prompt Injection: What It Is and How to Prevent It"** (Lakera)
- **"LLM Guardrails: A Practical Guide"** (NVIDIA Developer)
- **"Scaling LLM Inference with vLLM"** (Anyscale Blog)
- **"Cost Optimization for LLM APIs"** (Helicone Blog)
- **"CI/CD for LLM Applications"** (Galileo AI)
- **"Incident Response for AI Systems"** (Honeycomb Blog)
- **"OpenTelemetry for LLM Applications"** (Lightstep Blog)
