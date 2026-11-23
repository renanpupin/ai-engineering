# ADR-0002 — Spec-driven development

- **Status:** Accepted
- **Date:** 2026-11-23

## Context
AI behavior is probabilistic. Without explicit acceptance criteria and metrics, "it works" is a matter of opinion and regressions go unnoticed.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| Code first, document later | Fast start | Docs drift; no measurable target for AI behavior |
| Spec first (SDD) | Testable criteria, traceability, shared language | Upfront effort |

## Decision
Specs in `specs/` are the source of truth. Each requirement has an ID and testable criteria; tests, evals and prompts reference those IDs.

## Consequences
- Evals and quality gates are derived directly from specs.
- Spec changes are reviewed like code.
- Requires discipline to keep IDs and references up to date (checked in CI later).
