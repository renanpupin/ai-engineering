# Lumen — AI Credit Copilot

> A reference implementation of an **enterprise-grade, AI-native system**: designed, built, tested, deployed, scaled, monitored and governed end to end.

Lumen is a multi-agent copilot that helps credit analysts evaluate loan applications from small and medium businesses. It reads the credit policy (RAG), extracts data from financial documents (multimodal), computes a risk score with a model trained in this repository (MLOps), explains every decision with citations, and escalates borderline cases to a human reviewer (human-in-the-loop).

The domain is intentionally small and the business logic intentionally simple. **The point is not the credit product. The point is the engineering around it.**

---

## 1. What it does

Given a loan application and its supporting documents, Lumen:

1. **Extracts** financial data from PDFs and images (Document agent).
2. **Retrieves** the relevant rules from the credit policy and cites them (Policy agent).
3. **Scores** the application with a trained risk model exposed as an MCP tool (Risk agent).
4. **Reviews** the draft decision for hallucinations, missing citations and policy violations (Reviewer agent, LLM-as-a-judge).
5. **Decides or escalates**: returns a recommendation with an explanation, or routes borderline cases to a human reviewer.

Around that flow, the project implements and documents the engineering practices needed to run such a system in production:

| Engineering question | How Lumen addresses it |
|---|---|
| How are architectural decisions made? | Architecture Decision Records (`specs/adr/`) with context, options and trade-offs |
| How is agent quality and reliability measured? | Golden datasets, offline evals, LLM-as-a-judge, regression suite in CI (`evals/`) |
| How are inference costs controlled? | Token budgets, model routing, semantic caching, per-request cost tracking |
| How are failures and hallucinations handled? | Groundedness checks, mandatory citations, fallbacks, circuit breakers, autonomy levels |
| How does the infrastructure scale? | Stateless services, queues, Kubernetes autoscaling, load tests, self-hosted inference option |
| How are security and governance ensured? | Threat model (OWASP Top 10 for LLMs), guardrails, RBAC, audit trail, HITL, fairness reports |
| How is quality software delivered predictably? | Spec-driven development, TDD, contract tests, quality gates, versioned prompts and models |

### Why a credit domain?

- **It is a regulated domain.** Bias and fairness, explainability, auditability and human oversight are *requirements* here, not decoration.
- **It combines classical ML and LLMs.** A trained risk model (tabular ML) works alongside LLM agents, which is how many real AI systems look in production.

---

## 2. Guiding principles

1. **Spec first.** Documentation is the source of truth. Code, tests, evals and data analysis are derived from the specs in `specs/`.
2. **Hello world, then layers.** The project starts as a single working LLM call and grows one layer at a time. Every phase ships something that runs.
3. **Simple over clever.** Minimal domain, minimal dependencies, clean code. Complexity is allowed only where it teaches something.
4. **Everything is explained.** Each concept has its own Markdown file in `docs/concepts/`: what it is, why it matters, and how it is used here.
5. **Provider-agnostic.** LLM providers, vector stores and model servers sit behind ports (interfaces), so they can be swapped without touching the domain.
6. **Production mindset.** Observability, security, cost and failure modes are designed from day one, not bolted on.
7. **Bounded autonomy.** Agents act within explicit permissions. Risky or uncertain actions require human approval.

---

## 3. Core competencies demonstrated

| # | Area | What is demonstrated |
|---|---|---|
| 1 | **AI Engineering** | LLMs, RAG, embeddings, hybrid search, reranking, multimodality, structured outputs |
| 2 | **Agentic AI** | Multi-agent systems, tool calling, planning, memory, MCP |
| 3 | **Spec-Driven Development (SDD)** | Specs, ADRs and contracts as the center of the project; code, tests, evals and analysis derived from them |
| 4 | **Software Engineering** | Clean Architecture, Domain-Driven Design, SOLID, design patterns |
| 5 | **Quality Engineering** | TDD, unit, integration and contract tests, evals, regression testing |
| 6 | **MLOps** | Model lifecycle, training, experiment tracking, model registry, drift monitoring |
| 7 | **LLMOps** | Prompt versioning and management, LLM evaluation, tracing, cost tracking |
| 8 | **AI Infrastructure** | Docker, Kubernetes, CI/CD, Infrastructure as Code, GPU and inference serving |
| 9 | **AI Governance** | Guardrails, security, RBAC, audit, human-in-the-loop, bias and fairness |

---

## 4. Concepts covered

Each concept below will have a dedicated file in `docs/concepts/` explaining the theory and pointing to where it is implemented.

### 4.1 LLM fundamentals

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **Large Language Models (LLMs)** | Transformer models that predict the next token, used for reasoning, extraction and generation | All agents; provider abstraction in `infrastructure/llm` |
| **Tokens** | The unit LLMs read, write and bill by | Token counting, budgets and cost per request |
| **LLM parameters** | Sampling and generation controls: `temperature`, `top_p`, `max_tokens`, stop sequences, seed | Per-agent configuration; deterministic settings for evals |
| **Context window** | The maximum number of tokens a model can attend to in one call | Context budgeting and compaction strategy |
| **Training** | Learning model weights from data (pre-training, fine-tuning, LoRA) | Risk model training in `ml/`; fine-tuning documented as an option |
| **Inference** | Running a trained model to produce outputs | LLM calls, risk model serving, latency and throughput tuning |
| **Embeddings** | Dense vectors that encode semantic meaning | Indexing the credit policy for retrieval |
| **Structured outputs** | Forcing model output to match a schema (JSON / Pydantic) | Every agent returns typed, validated objects |
| **Reasoning models** | Models that spend extra compute "thinking" before answering | Used selectively for hard cases, routed by cost |

### 4.2 Prompt and context engineering

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **Role & behavior** | System prompts defining identity, scope, tone and constraints | One versioned system prompt per agent |
| **Prompt engineering** | Designing instructions that produce reliable outputs | `prompts/` as versioned YAML with changelog |
| **Zero-shot** | Solving a task from instructions only | Simple classification steps |
| **Few-shot** | Providing worked examples in the prompt | Document extraction and decision formatting |
| **Chain-of-Thought (CoT)** | Asking the model to reason step by step | Risk explanation and policy interpretation |
| **ReAct** | Interleaving reasoning and actions (tool calls) in a loop | Orchestrator agent |
| **Context** | Everything the model sees in a call: instructions, history, retrieved data, tool results | Assembled by a dedicated context builder |
| **Context engineering** | Deciding *what* goes into the context, in what order and at what size | Retrieval, summarization, memory selection, token budgets |
| **Context evaluation** | Measuring whether the context was relevant and sufficient | Retrieval metrics: context precision, recall, relevance |
| **Context security** | Keeping secrets, PII and untrusted content from leaking or taking control | PII redaction, trust boundaries, content tagging |

### 4.3 Retrieval (RAG)

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **Retrieval-Augmented Generation (RAG)** | Grounding answers in retrieved documents | Policy agent answers only from the credit policy |
| **Vector database** | Storage and similarity search over embeddings | Qdrant or pgvector behind a port |
| **Chunking** | Splitting documents into retrievable units | Documented strategy with evaluated trade-offs |
| **Hybrid search** | Combining keyword (BM25) and vector search | Retrieval pipeline |
| **Reranking** | Re-scoring retrieved chunks with a stronger model | Cross-encoder rerank step before generation |
| **Citations & grounding** | Linking each claim to its source | Mandatory in every decision |
| **Multimodality** | Processing images and PDFs alongside text | Document agent reads balance sheets and statements |

### 4.4 Agentic AI

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **Agentic AI** | LLMs that plan and act with tools toward a goal | The whole copilot |
| **Multi-agent systems** | Specialized agents coordinated by an orchestrator | Orchestrator, Policy, Risk, Document and Reviewer agents |
| **Tool calling** | Letting the model invoke functions with typed arguments | Risk scoring, document parsing, policy search |
| **Planning** | Decomposing a goal into steps before acting | Orchestrator builds an explicit plan per application |
| **Memory** | Short-term (conversation) and long-term (persisted facts and past cases) | Memory module with explicit retention rules |
| **MCP (Model Context Protocol)** | Open standard for exposing tools and data to AI systems | Risk model and policy search exposed as MCP servers |
| **Autonomy levels** | Explicit limits on what an agent may do without approval | Policy-driven: suggest, act with approval, act |
| **Human-in-the-loop (HITL)** | A human reviews or approves before an action is final | Borderline and high-value decisions are escalated |

### 4.5 Quality and evaluation

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **TDD** | Writing the test before the code | Domain and application layers |
| **Integration tests** | Testing components together with real dependencies | Adapters: LLM, vector DB, registry |
| **Contract tests** | Verifying the agreements between services and tools | API schemas, MCP tool contracts, agent I/O schemas |
| **Evals** | Systematic measurement of AI output quality on a dataset | `evals/` with golden datasets per agent |
| **AI judge (LLM-as-a-judge)** | Using a model to grade outputs against a rubric | Reviewer agent and offline grading |
| **Regression testing** | Ensuring a change to a prompt or model does not degrade quality | Eval suite runs in CI as a quality gate |
| **Evaluation tools** | Frameworks for running and tracking evals | e.g. Langfuse datasets, Ragas, promptfoo |
| **Hallucination handling** | Detecting and preventing unsupported claims | Groundedness checks, citation validation, abstention |

### 4.6 Operations (MLOps, LLMOps, AIOps)

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **MLOps** | Lifecycle of ML models: data, training, validation, deployment, monitoring | Risk model in `ml/` |
| **Experiment tracking** | Recording parameters, metrics and artifacts of each run | MLflow |
| **Model registry** | Versioned catalog of models with stages (staging, production) | MLflow Model Registry |
| **Data and model drift** | Changes in input data or model behavior over time | Drift checks on the risk model |
| **LLMOps** | Operating LLM applications: prompts, evals, tracing, cost | Prompt registry, eval pipeline, tracing |
| **Prompt / version management** | Treating prompts as versioned, reviewed, tested artifacts | `prompts/` with semantic versions |
| **AIOps** | Applying AI and automation to operations (alerting, triage, anomaly detection) | Automated anomaly alerts on quality, cost and latency |
| **Observability** | Understanding system behavior from logs, metrics and traces | OpenTelemetry instrumentation everywhere |
| **Tracing** | Following one request across agents, tools and model calls | Langfuse / OpenTelemetry spans per agent step |
| **Production monitoring** | Dashboards and alerts on SLOs | Latency, error rate, cost, eval scores, escalation rate |
| **Cost control** | Keeping inference spend predictable | Token budgets, model routing, semantic caching, batching |

### 4.7 Security and governance

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **Guardrails** | Input and output checks that block unsafe or invalid content | Before and after every agent run |
| **Prompt injection attacks** | Untrusted input that tries to override instructions (direct or indirect) | Threat model, input isolation, red-team test suite |
| **PII protection** | Detecting and masking personal data | Redaction before logging and before external calls |
| **RBAC** | Role-based access control | Analyst, reviewer and admin roles over actions and data |
| **Audit trail** | Immutable record of who (or which agent) did what and why | Every decision, tool call and approval is logged |
| **Bias and fairness** | Measuring and mitigating unfair outcomes across groups | Fairness metrics on the risk model and decisions |
| **Explainability** | Making decisions understandable | Feature attributions (SHAP) plus natural-language rationale |
| **Red teaming** | Adversarial testing of the system | Automated attack suite in `evals/security/` |

### 4.8 Infrastructure

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **Docker** | Containerized, reproducible runtime | Every service |
| **Kubernetes** | Container orchestration and autoscaling | Helm charts, horizontal pod autoscaling |
| **CI/CD** | Automated build, test, eval and deploy | GitHub Actions with quality gates |
| **Infrastructure as Code (IaC)** | Infrastructure defined in versioned code | Terraform |
| **GPU / inference serving** | Running open models efficiently (batching, KV cache, quantization) | Optional self-hosted path with vLLM or Ollama |
| **Semantic caching** | Reusing answers for semantically similar requests | Cache layer in front of LLM calls |
| **Model routing** | Choosing the cheapest model that meets the quality bar | Router based on task type and difficulty |

### 4.9 Software engineering

| Concept | What it is | Where it appears in Lumen |
|---|---|---|
| **Clean Architecture** | Dependencies point inward; domain is independent of frameworks | `domain → application → infrastructure / interfaces` |
| **Domain-Driven Design (DDD)** | Modeling software around the business domain and its language | Bounded contexts: Application, Risk, Policy, Decision |
| **SOLID** | Five principles for maintainable object-oriented design | Ports and adapters, small focused classes |
| **Design patterns** | Reusable solutions to recurring problems | Strategy (model routing), Adapter (providers), Chain of Responsibility (guardrails), Repository, Factory |
| **Clean code** | Readable, intention-revealing, small units | Enforced by linters, type checks and reviews |

---

## 5. Roadmap

Every phase ships something that runs and is documented before it is coded.

| Phase | Deliverable | Main competencies |
|---|---|---|
| **0. Specs & foundation** | Prerequisites, specs, ADRs, concept docs, project skeleton, CI | SDD, Software Engineering |
| **1. Hello LLM** | One agent, provider abstraction, versioned prompts, token and cost tracking | LLM fundamentals, prompting |
| **2. RAG** | Embeddings, vector DB, hybrid search, reranking, citations | AI Engineering, context engineering |
| **3. ML model** | Risk model training, experiment tracking, registry, served via MCP | MLOps |
| **4. Agentic** | Multi-agent orchestration, ReAct, CoT, planning, memory, tool calling | Agentic AI, MCP |
| **5. Quality** | TDD, contract tests, evals, AI judge, prompt regression in CI | Quality Engineering, LLMOps |
| **6. Governance** | Guardrails, prompt-injection defenses, RBAC, audit, HITL, fairness | AI Governance |
| **7. Observability** | Tracing, cost / latency / quality dashboards, alerting | LLMOps, AIOps |
| **8. Scale** | Docker, Kubernetes, Terraform, autoscaling, caching, routing, load tests | AI Infrastructure |

**Current status:** Phase 0, specs and documentation.

---

## 6. Planned repository structure

```
lumen/
├── specs/              # Source of truth: requirements, ADRs, contracts, eval criteria, metrics
├── docs/
│   ├── architecture/   # Diagrams and architecture overview
│   └── concepts/       # One file per concept listed above
├── src/
│   ├── domain/         # Entities and business rules (no framework dependencies)
│   ├── application/    # Use cases and ports (interfaces)
│   ├── agents/         # Orchestrator, specialized agents, planning, memory
│   ├── infrastructure/ # Adapters: LLM providers, vector DB, MCP, model registry
│   └── interfaces/     # API (FastAPI) and CLI
├── ml/                 # Training, experiment tracking, model registry
├── prompts/            # Versioned prompts and changelog
├── evals/              # Golden datasets, judges, regression and security suites
├── tests/              # Unit, integration and contract tests
└── infra/              # Docker, Kubernetes / Helm, Terraform, CI/CD
```

---

## 7. Planned stack

Python · FastAPI · Pydantic · LangGraph · Qdrant / pgvector · scikit-learn / XGBoost · MLflow · Langfuse · OpenTelemetry · pytest · Docker · Kubernetes · Terraform · GitHub Actions

LLM providers sit behind a port: Anthropic, OpenAI, or local models via Ollama / vLLM, so the project can run without paid APIs.

Final choices and their trade-offs will be recorded as ADRs.

