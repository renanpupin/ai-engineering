# ADR-0007 — Vector store

- **Status:** Proposed
- **Date:** 2026-11-23
- **Related specs:** FR-010, FR-011, SEC-09

## Context
The policy corpus is small. Requirements: hybrid search, metadata filtering (policy version, permissions), simple operations.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| pgvector (PostgreSQL) | One database for app data, audit and vectors; transactional; simple ops | Fewer vector-specific features at large scale |
| Qdrant | Purpose-built, fast filtering, hybrid search | Extra service to operate |
| Managed service | No ops | Cost, lock-in, harder to run locally |

## Decision (proposed)
**pgvector** for simplicity, behind a `VectorStore` port. Revisit if corpus size or latency requires a dedicated engine.

## How we will know
M-R01–M-R03 and NFR-P01 met at expected corpus size.
