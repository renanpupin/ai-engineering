# 10 — Deployment

How Lumen runs locally, in CI and on Kubernetes. Specs: NFR-S01–NFR-S04, NFR-M04.

## Environments

| Environment | Purpose | How it runs | LLMs |
|---|---|---|---|
| Local | Development | `docker compose up` | Local (Ollama) or hosted with own key |
| CI | Tests and evals | Ephemeral containers | Fakes for unit tests; pinned models for evals |
| Staging | Pre-production, canary source | Kubernetes namespace | Hosted |
| Production | Real traffic | Kubernetes namespace | Hosted + optional self-hosted |

## Kubernetes topology

```mermaid
flowchart TB
    ING["Ingress<br/>TLS"] --> API

    subgraph NS["Namespace: lumen"]
        API["Deployment: api<br/>HPA on CPU / RPS"]
        WK["Deployment: workers<br/>KEDA / HPA on queue depth"]
        GW["Deployment: llm-gateway<br/>HPA on RPS"]
        MCPD["Deployments: mcp-*<br/>HPA on RPS"]
        MS["Deployment: model-server<br/>HPA on latency"]
        ING_J["CronJob / Job: ingestion"]
        TR_J["Job: training"]
        COL["DaemonSet / Deployment:<br/>otel-collector"]
        INF["Optional: vLLM on GPU node pool"]
    end

    subgraph Data["Managed services"]
        PG[("PostgreSQL + pgvector")]
        RD[("Redis")]
        S3[("Object storage")]
        MLF["MLflow"]
        SM["Secret manager"]
    end

    API --> RD
    API --> PG
    WK --> RD
    WK --> GW
    WK --> MCPD
    GW --> INF
    MCPD --> PG
    MCPD --> MS
    MS --> MLF
    API & WK & GW --> SM
```

## Scaling strategy

| Component | Bottleneck | Strategy |
|---|---|---|
| API | Requests per second | Stateless replicas; rate limits per user |
| Workers | Concurrent cases, external LLM latency | Scale on queue depth; async I/O; per-provider concurrency limits |
| LLM gateway | Provider rate limits, cost | Routing to cheaper tiers, semantic cache, fallback providers, request batching |
| Self-hosted inference | GPU memory and throughput | vLLM continuous batching, KV cache, quantization; GPU node pool scales to zero when idle |
| Model server | CPU latency | Small model, replicas |
| PostgreSQL | Writes (audit), vector queries | Connection pooling, read replicas, index tuning; dedicated vector engine if needed (ADR-0007) |

## CI/CD

```mermaid
flowchart LR
    PR["Pull request"] --> CI["CI: lint, types, tests,<br/>contracts, scans, evals"]
    CI --> M["Merge to main"]
    M --> BUILD["Build images<br/>tag = commit SHA, SBOM, sign"]
    BUILD --> TF["Terraform plan / apply<br/>infrastructure changes"]
    BUILD --> STG["Deploy to staging<br/>Helm"]
    STG --> SMOKE["Smoke tests + eval subset"]
    SMOKE --> CAN["Canary in production<br/>10 % traffic"]
    CAN --> CHECK{"SLOs + online evals OK?"}
    CHECK -- yes --> PROD["Full rollout"]
    CHECK -- no --> RB["Automatic rollback"]
```

## Infrastructure as Code

```
infra/
├── docker/          # Dockerfiles per service
├── compose/         # docker-compose for local development
├── helm/lumen/      # Helm chart: deployments, HPA, config, secrets refs
└── terraform/
    ├── modules/     # network, cluster, database, storage, observability
    └── envs/        # staging, production
```
