# Agent Specifications

## Overview

| ID | Agent | Pattern | Model tier | Autonomy |
|---|---|---|---|---|
| AG-01 | Orchestrator | Plan-and-execute + ReAct | Mid | Coordinates; cannot issue final adverse decisions |
| AG-02 | Document | Few-shot structured extraction, multimodal | Mid (vision) | Read-only |
| AG-03 | Policy | RAG with reranking, CoT | Small → Mid | Read-only |
| AG-04 | Risk | Tool use (MCP), zero-shot explanation | Small | Read-only |
| AG-05 | Reviewer | LLM-as-a-judge with rubric | Mid / reasoning | Can block or request revision |

Model tiers are logical. The concrete model for each tier is configuration, chosen by evals and cost (see ADR-0004 and `NFR-C04`).

## Common rules for all agents

- Each agent has a **versioned system prompt** defining role, behavior, scope and refusal rules.
- Inputs and outputs are **typed schemas** (see [`contracts/agent-io.md`](contracts/agent-io.md)). Invalid output is retried once with the validation error, then fails.
- Each agent has a **token budget** and a **max step count**.
- Each agent only sees the **tools it needs** (least privilege).
- Retrieved and user-provided content is wrapped as **untrusted data** and never treated as instructions.
- Every step emits a **trace span** with inputs, outputs, tokens, cost and latency.

---

## AG-01 — Orchestrator

| Field | Specification |
|---|---|
| **Role** | Plans and coordinates the case; assembles the recommendation |
| **Input** | `Case` with application and document references |
| **Output** | `Plan`, then `Recommendation` |
| **Tools** | `call_agent(document|policy|risk|reviewer)`, `read_memory`, `write_case_memory`, `escalate` |
| **Technique** | Plan first, then ReAct loop over the plan |
| **Limits** | Max 12 steps; budget from `NFR-C01` |
| **Escalates when** | Budget or step limit reached; any agent fails after fallback; reviewer blocks; escalation rules in `security-governance.md` |
| **Failure modes** | Loops, skipped steps, over-confident synthesis → mitigated by step limits, plan validation, judge review |

## AG-02 — Document agent

| Field | Specification |
|---|---|
| **Role** | Extract structured financial data from documents |
| **Input** | Document files (PDF, PNG, JPG) |
| **Output** | `ExtractionResult`: fields, confidence per field, source location |
| **Tools** | `parse_pdf`, `ocr_image` |
| **Technique** | Few-shot examples per document type; structured output |
| **Failure modes** | Invented values, wrong units or periods → field-level confidence, `missing` instead of guessing, unit normalization checks |

## AG-03 — Policy agent

| Field | Specification |
|---|---|
| **Role** | Find and interpret the policy rules that apply to the application |
| **Input** | Application summary or analyst question |
| **Output** | `PolicyFindings`: applicable rules, interpretation, citations |
| **Tools** | `search_policy` (hybrid search + rerank, via MCP) |
| **Technique** | RAG; Chain-of-Thought for rule interpretation; abstain when not covered |
| **Failure modes** | Wrong or outdated rule, unsupported claims → citation validation, policy version pinning, abstention threshold |

## AG-04 — Risk agent

| Field | Specification |
|---|---|
| **Role** | Obtain and explain the risk score |
| **Input** | Normalized features from the extraction |
| **Output** | `RiskAssessment` + plain-language explanation |
| **Tools** | `score_application` (MCP), `explain_score` (MCP) |
| **Technique** | Tool use; zero-shot explanation grounded in feature attributions |
| **Failure modes** | Explaining factors the model did not use → explanation must reference only returned attributions |

## AG-05 — Reviewer agent (AI judge)

| Field | Specification |
|---|---|
| **Role** | Grade the draft recommendation before it leaves the system |
| **Input** | Draft `Recommendation` + all evidence (extraction, policy findings, risk) |
| **Output** | `ReviewResult`: score per criterion, pass/fail, issues |
| **Criteria** | Groundedness, citation validity, policy compliance, completeness, consistency with risk score |
| **Technique** | Rubric-based LLM-as-a-judge; deterministic checks first (schema, citations exist) |
| **Failure modes** | Judge bias or leniency → judge itself is evaluated against human-labeled samples (see `EV-07`) |

## Memory

| Type | Scope | Content | Retention |
|---|---|---|---|
| **Working memory** | One agent step | Current context window | Discarded after step |
| **Short-term memory** | One case | Plan, intermediate results, agent messages | Lifetime of the case |
| **Long-term memory** | Across cases | Redacted case summaries, reviewer feedback patterns | Configurable; no PII |

Memory reads are retrieval-based and budgeted, so memory never floods the context.
