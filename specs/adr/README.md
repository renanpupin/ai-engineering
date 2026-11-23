# Architecture Decision Records

An ADR records one significant decision: the context, the options considered, the choice and its consequences. ADRs are immutable once `Accepted`; a later ADR can supersede an earlier one.

**Statuses:** `Proposed` → `Accepted` → (`Deprecated` | `Superseded by ADR-XXXX`)

**When to write one:** a choice that is hard to reverse, affects several components, changes a contract, or trades off quality attributes (cost, latency, safety, maintainability).

| ADR | Title | Status |
|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted |
| [0002](0002-spec-driven-development.md) | Spec-driven development | Accepted |
| [0003](0003-clean-architecture-and-ddd.md) | Clean Architecture with DDD (ports and adapters) | Accepted |
| [0004](0004-provider-agnostic-llm-and-model-routing.md) | Provider-agnostic LLM port and model routing | Accepted |
| [0005](0005-python-fastapi-pydantic.md) | Python, FastAPI and Pydantic | Accepted |
| [0006](0006-agent-orchestration.md) | Agent orchestration framework | Proposed |
| [0007](0007-vector-store.md) | Vector store | Proposed |
| [0008](0008-mcp-for-tools.md) | MCP for agent tools | Accepted |
| [0009](0009-ml-lifecycle-mlflow.md) | MLflow for experiment tracking and model registry | Proposed |
| [0010](0010-observability-stack.md) | Observability: OpenTelemetry and Langfuse | Proposed |
| [0011](0011-human-in-the-loop-by-default.md) | Human-in-the-loop by default | Accepted |

Use [`template.md`](template.md) for new ADRs.
