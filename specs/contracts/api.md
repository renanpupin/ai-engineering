# Public API Contract — v1

Base path: `/v1`. JSON over HTTPS. Authentication: bearer token (OIDC). Every response includes `X-Trace-Id`.

| Method | Path | Role | Description | Spec |
|---|---|---|---|---|
| `POST` | `/cases` | Analyst | Submit an application with documents. Header `Idempotency-Key` required. Returns `202` with `case_id` | FR-030 |
| `GET` | `/cases/{case_id}` | Analyst, Reviewer, Auditor | Case status, recommendation and decision | FR-033 |
| `POST` | `/cases/{case_id}/decision` | Reviewer | Approve, reject or override with reason | FR-050 |
| `POST` | `/cases/{case_id}/feedback` | Analyst | Rate a recommendation | FR-061 |
| `POST` | `/policy/questions` | Analyst | Ask a policy question; supports streaming | FR-011 |
| `GET` | `/audit/cases/{case_id}` | Auditor, Admin | Audit trail of a case | FR-052 |
| `GET` | `/health`, `/ready` | — | Liveness and readiness | NFR-R01 |

## Example — `GET /cases/{case_id}`

```json
{
  "case_id": "case_01J...",
  "status": "awaiting_review",
  "recommendation": {
    "outcome": "refer",
    "confidence": 0.62,
    "risk": { "probability_of_default": 0.18, "risk_band": "C", "model_version": "risk-model@3" },
    "rationale": "Debt service coverage is below the minimum required ...",
    "citations": [{ "type": "policy", "policy_version": "2026.1", "section_id": "4.2" }],
    "escalation_reasons": ["confidence_below_threshold"]
  },
  "versions": { "policy": "2026.1", "prompts": { "orchestrator": "1.3.0" }, "llm": { "mid": "provider/model-x" } }
}
```

## Errors

Errors follow RFC 9457 (`application/problem+json`) with `type`, `title`, `status`, `detail` and `trace_id`.

| Status | When |
|---|---|
| `400` | Schema validation failed |
| `401` / `403` | Not authenticated / not authorized (RBAC) |
| `409` | Idempotency key reused with a different payload |
| `422` | Input rejected by guardrails |
| `429` | Rate limit or budget exceeded |
| `503` | Dependency unavailable after retries and fallbacks |
