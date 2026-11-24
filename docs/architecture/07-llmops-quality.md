# 07 — LLMOps and Quality

How prompts, models and agent code change safely. Specs: FR-002, `evals.md`, M-Q*, NFR-M02.

## Test pyramid for an AI system

```mermaid
flowchart TB
    E2E["End-to-end evals<br/>DS-04, few, slow, costly"]
    AIEV["Component evals<br/>retrieval, extraction, judge, security"]
    CT["Contract tests<br/>API, agent I/O, MCP tools"]
    IT["Integration tests<br/>adapters with real dependencies"]
    UT["Unit tests — TDD<br/>domain and use cases with fakes"]
    E2E --- AIEV --- CT --- IT --- UT
```

Lower layers are deterministic and run on every commit. Upper layers call models, so they run on relevant changes and are seeded and pinned for repeatability.

## Prompt lifecycle

```mermaid
flowchart LR
    ED["Edit prompt<br/>prompts/agent/name.yaml"] --> VER["Bump semantic version<br/>+ changelog entry"]
    VER --> PR["Pull request"]
    PR --> CI["CI: run affected evals"]
    CI --> CMP{"Compare with main baseline"}
    CMP -- regression --> FAIL([Blocked])
    CMP -- ok --> REVW["Human review<br/>diff + eval report"]
    REVW --> MERGE["Merge"]
    MERGE --> CANARY["Canary in production<br/>% of traffic"]
    CANARY --> ONLINE{"Online evals OK?"}
    ONLINE -- yes --> FULL([Rollout])
    ONLINE -- no --> RB([Rollback to previous version])
```

## CI quality gates

```mermaid
flowchart LR
    C["Commit / PR"] --> LINT["Lint + types<br/>ruff, mypy"]
    LINT --> ARCH["Architecture rules<br/>import-linter"]
    ARCH --> UNIT["Unit + integration tests<br/>coverage gate"]
    UNIT --> CONT["Contract tests"]
    CONT --> SEC["Security scans<br/>deps, image, secrets"]
    SEC --> CHG{"Prompt / model /<br/>retrieval / agent<br/>changed?"}
    CHG -- no --> OK([Mergeable])
    CHG -- yes --> EVALS["Eval suite<br/>EV-01..EV-09"]
    EVALS --> REP["Eval report<br/>posted on PR"]
    REP --> GATE{"Blocking metrics<br/>within target and<br/>≤ 2 pp drop?"}
    GATE -- yes --> OK
    GATE -- no --> NO([Blocked])
```

## What is versioned

| Artifact | Where | Version format |
|---|---|---|
| Prompts | `prompts/` | Semantic version per prompt |
| Tool descriptions | MCP servers | Contract version |
| Model routing config | `config/models.yaml` | Git commit |
| Eval datasets | `evals/datasets/` | Dataset version + hash |
| Risk model | MLflow registry | Model version |
| Policy | Ingestion index | Policy version |

Every case stores the versions it used, so any output can be reproduced (GOV-04).
