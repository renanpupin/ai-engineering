# MCP Tool Contracts — v1

Lumen exposes domain capabilities as **Model Context Protocol (MCP)** servers. Agents consume them as tools. Each tool declares an input schema, an output schema, required permission and side effects.

| Server | Tool | Input | Output | Side effects | Permission |
|---|---|---|---|---|---|
| `lumen-risk` | `score_application` | `features: dict[str, float]` | `RiskAssessment` (without explanation) | None | `risk:score` |
| `lumen-risk` | `explain_score` | `features`, `model_version` | `list[FeatureAttribution]` | None | `risk:score` |
| `lumen-policy` | `search_policy` | `query: str`, `top_k: int ≤ 10`, `policy_version?: str` | `list[{section_id, text, score}]` | None | `policy:read` |
| `lumen-docs` | `parse_pdf` | `document_id` | `pages: list[{page, text, tables}]` | None | `documents:read` |
| `lumen-docs` | `ocr_image` | `document_id` | `blocks: list[{text, bbox}]` | None | `documents:read` |
| `lumen-cases` | `escalate` | `case_id`, `reasons: list[str]` | `{status}` | Changes case status | `cases:escalate` |

## Rules

- Tools are **read-only by default**. Tools with side effects are explicitly marked and audited.
- Every tool validates its input and returns typed errors: `validation_error`, `not_found`, `forbidden`, `unavailable`.
- Every tool call carries the caller's identity and trace context.
- Tool descriptions are part of the contract and versioned, because the model reads them.
