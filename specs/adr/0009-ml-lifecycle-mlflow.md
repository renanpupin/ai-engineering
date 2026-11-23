# ADR-0009 — MLflow for experiment tracking and model registry

- **Status:** Proposed
- **Date:** 2026-11-23
- **Related specs:** FR-020, M-ML*, M-F*

## Context
The risk model needs reproducible training, comparable experiments, versioned artifacts and controlled promotion to production.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| MLflow | Open source, tracking + registry + serving, runs locally | Self-hosted server to operate |
| Weights & Biases | Excellent UI and collaboration | SaaS, cost |
| Custom (files + Git) | No dependencies | Reinventing tracking and registry |

## Decision (proposed)
MLflow for tracking and registry. Promotion to `production` is gated by M-ML* and M-F* thresholds in CI.
