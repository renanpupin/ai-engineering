# 05 — RAG Pipeline

How the credit policy becomes grounded, cited answers. Specs: FR-010–FR-012, EV-01–EV-03, ADR-0007.

## Ingestion

```mermaid
flowchart LR
    SRC["Policy document<br/>Markdown / PDF + version"] --> REVW{"Reviewed and<br/>approved?"}
    REVW -- no --> STOP([Rejected])
    REVW -- yes --> PARSE["Parse<br/>keep section ids"]
    PARSE --> CHUNK["Chunk<br/>by section, size cap, overlap"]
    CHUNK --> META["Attach metadata<br/>policy_version, section_id,<br/>chunk_id, permissions"]
    META --> EMB["Embed<br/>embedding model via LLM port"]
    META --> LEX["Build lexical index<br/>BM25 / full-text"]
    EMB --> VDB[("pgvector")]
    LEX --> VDB
    VDB --> SNAP["Index version recorded"]
```

## Query

```mermaid
flowchart LR
    Q["Question or<br/>application summary"] --> GIN["Input guardrails"]
    GIN --> CACHE{"Semantic cache hit?"}
    CACHE -- yes --> ANS
    CACHE -- no --> REWR["Query rewriting<br/>optional"]
    REWR --> HY["Hybrid search<br/>vector + BM25<br/>filtered by version + permissions"]
    HY --> FUSE["Reciprocal rank fusion"]
    FUSE --> RR["Rerank<br/>cross-encoder, top-k"]
    RR --> THR{"Relevance ≥ threshold?"}
    THR -- no --> ABS["Abstain:<br/>policy does not cover this"]
    THR -- yes --> CB["Context builder<br/>order, dedupe, token budget,<br/>wrap as untrusted"]
    CB --> GEN["Generate with citations<br/>structured output"]
    GEN --> CHK["Citation check<br/>each claim → section_id exists"]
    CHK --> ANS["Answer + citations"]
```

## Design choices

| Choice | Default | Evaluated by |
|---|---|---|
| Chunking | Section-aware, ~400 tokens, 15 % overlap | M-R01, M-R02 |
| Retrieval | Hybrid (vector + BM25) with RRF | M-R01–M-R03 vs. vector-only |
| Reranking | Cross-encoder over top 20 → top 5 | M-R04 (rerank uplift) |
| Abstention threshold | Tuned on DS-02 | M-Q07 |
| Context budget | Max share of the window for retrieved text | M-C02, M-Q03 |

Each default becomes an ADR once measured.
