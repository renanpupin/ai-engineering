# Contracts

Contracts define the agreements between components. Every contract is enforced by schemas in code (Pydantic) and verified by **contract tests** in CI.

| Contract | Between | File |
|---|---|---|
| Public API | Clients ↔ Lumen API | [`api.md`](api.md) |
| Agent I/O | Orchestrator ↔ agents | [`agent-io.md`](agent-io.md) |
| MCP tools | Agents ↔ MCP servers | [`mcp-tools.md`](mcp-tools.md) |

## Versioning rules

- Contracts use semantic versioning.
- **Additive** changes (new optional field) → minor version.
- **Breaking** changes (removed or renamed field, changed type or meaning) → major version and an ADR.
- Consumers must ignore unknown fields.
