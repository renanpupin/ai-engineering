# ADR-0010 — Observability: OpenTelemetry and Langfuse

- **Status:** Proposed
- **Date:** 2026-11-23
- **Related specs:** FR-060, NFR-O01–NFR-O04

## Context
We need standard traces, metrics and logs for the whole system, plus LLM-specific views: prompts, completions, tokens, cost, eval scores and datasets.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| OpenTelemetry only | Vendor-neutral standard | No LLM-specific UX (prompt diffs, evals) |
| LLM platform only (Langfuse, LangSmith, …) | Great LLM UX | Weak for non-LLM services |
| OpenTelemetry + Langfuse | Standard instrumentation, LLM views, open source, self-hostable | Two systems |
| Commercial APM (e.g. Datadog LLM Observability) | One pane of glass | Cost; less portable |

## Decision (proposed)
Instrument with **OpenTelemetry** (GenAI semantic conventions) and export LLM spans to **Langfuse**; system metrics to Prometheus / Grafana. Backends stay swappable because instrumentation is standard.
