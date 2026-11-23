# ADR-0001 — Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-11-23

## Context
Design choices in AI systems involve trade-offs (cost vs. quality, autonomy vs. safety) that are invisible in the code. Without a record, the reasoning is lost and decisions are re-litigated.

## Decision
Record every significant decision as an ADR in `specs/adr/`, using the template, reviewed through pull requests.

## Consequences
- Decisions and their trade-offs are explicit and reviewable.
- Small overhead per decision; mitigated by a short template.
