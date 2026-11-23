# ADR-0005 — Python, FastAPI and Pydantic

- **Status:** Accepted
- **Date:** 2026-11-23

## Context
The project combines ML training, LLM orchestration and an API. The ecosystem for ML, evals and observability is strongest in Python.

## Decision
Python 3.12+, FastAPI for the API, Pydantic for all schemas (API, agent I/O, structured LLM outputs), `uv` for dependency management, `pytest`, `ruff` and `mypy` for quality.

## Consequences
- One language for ML, agents and API.
- Pydantic models are the single definition for contracts and structured outputs.
- Python performance limits are handled by async I/O and horizontal scaling; heavy inference runs in dedicated servers.
