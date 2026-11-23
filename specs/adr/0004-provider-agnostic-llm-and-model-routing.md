# ADR-0004 — Provider-agnostic LLM port and model routing

- **Status:** Accepted
- **Date:** 2026-11-23
- **Related specs:** FR-001, NFR-C01, NFR-C04, NFR-R03

## Context
Model quality, price and availability change constantly. Different tasks need different capability levels. Depending on one provider creates cost, availability and lock-in risk.

## Decision
- Define an `LLMProvider` port with adapters per provider (hosted and local).
- Agents request a **logical tier** (`small`, `mid`, `reasoning`, `vision`), not a model name.
- A router maps tier → concrete model by configuration, with a fallback chain.
- The model assigned to each task is the cheapest one that passes that task's evals.

## Consequences
- Cost optimization becomes a configuration change validated by evals.
- Provider outages are handled by fallback (NFR-R03).
- Provider-specific features (e.g. prompt caching) are exposed as optional capabilities on the port.
