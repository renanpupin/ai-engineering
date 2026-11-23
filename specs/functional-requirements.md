# Functional Requirements

Each requirement has an ID, a priority (MoSCoW), the phase in which it is delivered, and acceptance criteria written as testable statements (Given / When / Then).

## Summary

| ID | Requirement | Priority | Phase |
|---|---|---|---|
| FR-001 | Single LLM completion through a provider-agnostic port | Must | 1 |
| FR-002 | Versioned prompts loaded from the prompt registry | Must | 1 |
| FR-003 | Token usage and cost recorded per call | Must | 1 |
| FR-010 | Ingest and index the credit policy | Must | 2 |
| FR-011 | Answer policy questions with citations (RAG) | Must | 2 |
| FR-012 | Abstain when the policy does not cover the question | Must | 2 |
| FR-020 | Train and register the risk model | Must | 3 |
| FR-021 | Score an application through an MCP tool | Must | 3 |
| FR-022 | Explain the risk score with feature attributions | Should | 3 |
| FR-030 | Submit a loan application with documents | Must | 4 |
| FR-031 | Extract structured financial data from documents | Must | 4 |
| FR-032 | Orchestrate agents with an explicit plan | Must | 4 |
| FR-033 | Produce a decision recommendation with rationale | Must | 4 |
| FR-034 | Short-term and long-term memory | Should | 4 |
| FR-040 | Review recommendations with an AI judge | Must | 5 |
| FR-050 | Escalate to a human reviewer (HITL) | Must | 6 |
| FR-051 | Enforce role-based access control | Must | 6 |
| FR-052 | Record an audit trail for every decision | Must | 6 |
| FR-053 | Apply input and output guardrails | Must | 6 |
| FR-060 | Trace every request end to end | Must | 7 |
| FR-061 | Collect user feedback on recommendations | Should | 7 |

---

## Phase 1 — Hello LLM

### FR-001 — Provider-agnostic LLM completion
The system sends a prompt to an LLM through the `LLMProvider` port and returns a typed response.
- **Given** a configured provider, **when** a completion is requested, **then** the response contains text, model name, input tokens, output tokens and latency.
- **Given** the provider is changed in configuration, **when** the same request runs, **then** no code outside `infrastructure/llm` changes.
- **Given** the provider times out or returns a retryable error, **then** the call is retried with exponential backoff up to the configured limit.

### FR-002 — Versioned prompts
Prompts are loaded from `prompts/` by name and semantic version.
- **Given** a prompt `name@version`, **when** it is loaded, **then** its template, parameters (`temperature`, `max_tokens`, …) and metadata are returned.
- **Given** a request, **then** the prompt name and version used are recorded with the call.

### FR-003 — Token and cost tracking
- **Given** any LLM call, **then** input tokens, output tokens and estimated cost (from a versioned price table) are recorded.
- **Given** a request exceeds its token budget, **then** the call is refused before it is sent.

## Phase 2 — RAG

### FR-010 — Policy ingestion
- **Given** a policy document (Markdown or PDF) with a version, **when** it is ingested, **then** it is chunked, embedded and indexed with metadata: `policy_version`, `section_id`, `chunk_id`.
- **Given** a new policy version is ingested, **then** the previous version remains queryable for auditing.

### FR-011 — Policy Q&A with citations
- **Given** a policy question, **when** it is answered, **then** every factual claim cites at least one `section_id`.
- **Given** retrieval returns candidates, **then** they are reranked before generation and only the top-k are placed in context.

### FR-012 — Abstention
- **Given** no retrieved chunk passes the relevance threshold, **then** the system answers that the policy does not cover the question and does not generate an unsupported answer.

## Phase 3 — Risk model

### FR-020 — Train and register
- **Given** the training dataset and config, **when** the training pipeline runs, **then** parameters, metrics, dataset hash and artifacts are logged and the model is registered with a version.
- **Given** a candidate model, **then** it is promoted to `production` only if it meets the thresholds in `M-ML*` and the fairness thresholds in `M-F*`.

### FR-021 — Score via MCP
- **Given** a valid feature set, **when** the `score_application` MCP tool is called, **then** it returns `probability_of_default`, `risk_band` and `model_version`.
- **Given** an invalid feature set, **then** the tool returns a typed validation error and no score.

### FR-022 — Explain the score
- **Given** a scored application, **then** the top contributing features and their direction are returned.

## Phase 4 — Agentic workflow

### FR-030 — Submit application
- **Given** an authenticated analyst, **when** an application with documents is submitted, **then** a `Case` is created with status `received` and an ID is returned.

### FR-031 — Extract financial data
- **Given** supported documents (PDF, PNG, JPG), **then** the Document agent returns the fields defined in `contracts/agent-io.md` with a confidence per field.
- **Given** a required field cannot be extracted with sufficient confidence, **then** it is marked `missing` and never invented.

### FR-032 — Planned orchestration
- **Given** a case, **then** the Orchestrator produces an explicit plan (list of steps and agents) before acting, and the plan is stored with the case.
- **Given** a step fails, **then** the Orchestrator retries, uses a fallback, or escalates; it never silently skips the step.
- **Given** the maximum number of steps or the token budget is reached, **then** the run stops and the case is escalated.

### FR-033 — Decision recommendation
- **Given** all steps complete, **then** a `Recommendation` is produced with: outcome (`approve`, `reject`, `refer`), confidence, risk score, cited policy rules and a rationale.

### FR-034 — Memory
- **Given** an ongoing case, **then** the agents share short-term memory scoped to that case.
- **Given** a closed case, **then** a summary is stored in long-term memory and can be retrieved for similar future cases. PII is never stored in long-term memory.

## Phase 5 — Quality

### FR-040 — AI judge review
- **Given** a draft recommendation, **then** the Reviewer agent scores it for groundedness, citation validity, policy compliance and completeness against a rubric.
- **Given** any score below threshold, **then** the recommendation is revised once or escalated.

## Phase 6 — Governance

### FR-050 — Human-in-the-loop
- **Given** a recommendation that is `refer`, below the confidence threshold, above the amount threshold, or flagged by the judge, **then** the case moves to `awaiting_review` and no final decision is issued without a human.
- **Given** a reviewer overrides a recommendation, **then** a reason is required and stored.

### FR-051 — RBAC
- **Given** a user, **then** permitted actions follow the matrix in `security-governance.md`. Any other action is denied and logged.

### FR-052 — Audit trail
- **Given** any decision, tool call, prompt version, model version, approval or override, **then** an append-only audit event is recorded with actor, timestamp, inputs reference and outcome.

### FR-053 — Guardrails
- **Given** any user input or retrieved content, **then** input guardrails run before it reaches a model (injection detection, PII redaction, schema validation).
- **Given** any model output, **then** output guardrails run before it is returned (schema, PII, policy, toxicity).

## Phase 7 — Observability

### FR-060 — End-to-end tracing
- **Given** any request, **then** a single trace ID links API call, agent steps, tool calls, retrieval, LLM calls and the final outcome.

### FR-061 — Feedback
- **Given** a recommendation, **then** analysts can rate it and add a comment; feedback is linked to the trace and can be promoted to the eval dataset.
