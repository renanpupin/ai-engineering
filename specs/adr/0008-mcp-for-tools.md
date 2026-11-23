# ADR-0008 — MCP for agent tools

- **Status:** Accepted
- **Date:** 2026-11-23
- **Related specs:** FR-021, contracts/mcp-tools.md

## Context
Agents need tools (risk scoring, policy search, document parsing). Tools should be reusable by other AI clients and isolated from agent code.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| In-process function tools | Simple, fast | Tied to one agent runtime; harder to secure and reuse |
| Model Context Protocol (MCP) servers | Open standard, reusable, process isolation, explicit schemas | Extra network hop and servers |

## Decision
Expose domain capabilities as MCP servers (`lumen-risk`, `lumen-policy`, `lumen-docs`, `lumen-cases`). Agents consume them through an MCP client adapter.

## Consequences
- Tools are independently deployable, testable (contract tests) and permissioned.
- The same tools can be used from other MCP clients (IDE assistants, chat apps).
- Added latency is measured against NFR-P03.
