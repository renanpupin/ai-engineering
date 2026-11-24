# 02 — Containers

Level 2 of the C4 model: the deployable units, their responsibilities and how they communicate.

```mermaid
flowchart LR
    subgraph Clients
        UI["Review UI<br/>minimal web"]
        CLI["CLI"]
    end

    subgraph Lumen
        API["<b>API</b><br/>FastAPI<br/>auth, RBAC, guardrails,<br/>case submission"]
        Q[["<b>Queue</b><br/>Redis Streams"]]
        WK["<b>Agent workers</b><br/>orchestrator + agents"]
        RT["<b>LLM gateway</b><br/>router, budgets,<br/>semantic cache"]

        subgraph MCP["MCP servers"]
            MP["lumen-policy"]
            MR["lumen-risk"]
            MD["lumen-docs"]
            MC["lumen-cases"]
        end

        MS["<b>Model server</b><br/>risk model inference"]
        ING["<b>Ingestion job</b><br/>policy chunk + embed"]
        TR["<b>Training pipeline</b><br/>batch job"]

        DB[("<b>PostgreSQL</b><br/>cases, audit,<br/>pgvector, memory")]
        OS[("<b>Object storage</b><br/>documents, artifacts")]
        CACHE[("<b>Redis</b><br/>cache, rate limits")]
        MLF["<b>MLflow</b><br/>tracking + registry"]
    end

    LLM["LLM providers"]
    OBS["OTel collector →<br/>Langfuse / Prometheus / Grafana"]

    UI --> API
    CLI --> API
    API --> DB
    API --> OS
    API --> Q
    Q --> WK
    WK --> RT --> LLM
    RT --> CACHE
    WK --> MP & MR & MD & MC
    MP --> DB
    MD --> OS
    MC --> DB
    MR --> MS
    MS --> MLF
    ING --> DB
    ING --> RT
    TR --> MLF
    API & WK & RT & MCP & MS -. OTLP .-> OBS
```

| Container | Technology | Responsibility | Scales by |
|---|---|---|---|
| API | FastAPI | Auth, RBAC, input guardrails, idempotent submission, reads | Replicas (stateless) |
| Queue | Redis Streams | Decouple submission from processing; backpressure | — |
| Agent workers | Python | Run the orchestrator and agents per case | Replicas on queue depth |
| LLM gateway | Python module or service | Tier routing, fallback, token budgets, cost tracking, semantic cache | Replicas |
| MCP servers | MCP (Python SDK) | Expose policy search, risk scoring, document parsing, case actions | Replicas |
| Model server | FastAPI / MLflow serving | Low-latency risk model inference | Replicas |
| Ingestion job | Batch | Chunk, embed and index policy versions | On demand |
| Training pipeline | Batch | Train, evaluate and register risk models | On demand |
| PostgreSQL + pgvector | PostgreSQL | Cases, audit log, vectors, long-term memory | Vertical, read replicas |
| Object storage | S3 / MinIO | Documents and artifacts | Managed |
| MLflow | MLflow | Experiments and model registry | Single instance |

**Why a queue:** a full case takes tens of seconds and calls external models. Async processing keeps the API fast, absorbs spikes and allows retries without losing work (NFR-S03, NFR-R04).
