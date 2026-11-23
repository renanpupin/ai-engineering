# ADR-0011 — Human-in-the-loop by default

- **Status:** Accepted
- **Date:** 2026-11-23
- **Related specs:** FR-050, NFR-G03, security-governance.md §3–4

## Context
Credit decisions have real impact on people and businesses, and adverse decisions are regulated. LLMs can be confidently wrong.

## Options considered
| Option | Pros | Cons |
|---|---|---|
| Full autonomy with monitoring | Fastest | Unacceptable risk for adverse decisions |
| Human reviews everything | Safest | No efficiency gain |
| Autonomy levels with escalation rules | Balances efficiency and safety; tunable with data | More logic and configuration |

## Decision
Default autonomy is **L1 (recommend)**. No adverse decision is issued without a human. Escalation rules are explicit, versioned and measured (M-H01, M-H02). Higher autonomy can only be enabled by an Admin and a new ADR backed by eval and production data.

## Consequences
- Safety and auditability by design.
- Escalation and override rates become key metrics for tuning the system.
