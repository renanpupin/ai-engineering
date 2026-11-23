# ADR-0003 — Clean Architecture with DDD (ports and adapters)

- **Status:** Accepted
- **Date:** 2026-11-23
- **Related specs:** NFR-M01, FR-001

## Context
LLM providers, vector stores, orchestration frameworks and model servers change fast. Business rules (decision invariants, escalation rules) must not depend on them.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| Framework-centric (build inside an agent framework) | Fastest start | Business logic coupled to the framework; hard to test and replace |
| Layered with ports and adapters | Testable domain, swappable infrastructure | More interfaces and wiring |

## Decision
Use Clean Architecture: `domain` ← `application` ← `infrastructure` / `interfaces`. External systems are accessed through ports defined in `application` and implemented as adapters in `infrastructure`. Model the domain with DDD bounded contexts (see `domain-model.md`).

## Consequences
- Domain and use cases are unit-tested without network or models (TDD).
- Providers can be swapped by configuration.
- An import-linter rule in CI forbids framework imports in `domain`.
