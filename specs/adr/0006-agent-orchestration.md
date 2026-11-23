# ADR-0006 — Agent orchestration framework

- **Status:** Proposed
- **Date:** 2026-11-23
- **Related specs:** FR-032, AG-01

## Context
The Orchestrator needs planning, a ReAct loop, state per case, step limits, retries, human interrupts (HITL) and tracing.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| LangGraph | Graph-based state machines, checkpoints, human interrupts, widely adopted | Framework coupling, abstraction overhead |
| Custom lightweight orchestrator | Full control, teaches fundamentals, no lock-in | More code to maintain; must build checkpoints and interrupts |
| Provider agent SDKs | Simple, well integrated with one provider | Conflicts with ADR-0004 (provider-agnostic) |

## Decision (proposed)
Start with a **custom lightweight orchestrator** behind an `Orchestrator` port in Phase 4 to make the mechanics explicit, then add a **LangGraph adapter** to compare. The choice for production is made by evals, cost and code complexity.

## How we will know
Compare both on EV-05 quality, cost per case, latency and lines of code.
