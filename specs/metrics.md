# Metrics

Metrics turn specs into numbers. Each metric has an ID, a definition, how it is measured, a target and where it is used (eval gate, dashboard, alert). Targets are initial and revised with data.

## Quality (agents and LLM outputs)

| ID | Metric | Definition | Source | Target | Used in |
|---|---|---|---|---|---|
| M-Q01 | Decision agreement | % of recommendations matching golden label | Offline eval | ≥ 85 % | Eval gate |
| M-Q02 | Adverse decision precision | Of `reject` recommendations, % correct | Offline eval | ≥ 95 % | Eval gate |
| M-Q03 | Groundedness | % of claims supported by cited evidence | Judge + human sample | ≥ 95 % | Eval gate, dashboard |
| M-Q04 | Citation validity | % of citations that exist and support the claim | Deterministic + judge | ≥ 98 % | Eval gate |
| M-Q05 | Extraction accuracy | Field-level exact / tolerance match | Offline eval | ≥ 95 % | Eval gate |
| M-Q06 | Hallucinated field rate | % of extracted fields not present in source | Offline eval | ≤ 0.5 % | Eval gate |
| M-Q07 | Abstention correctness | % of out-of-policy questions correctly abstained | Offline eval | ≥ 90 % | Eval gate |
| M-Q08 | Schema validity | % of agent outputs valid on first attempt | Production | ≥ 99 % | Dashboard |
| M-Q09 | Judge–human agreement | Agreement between AI judge and human labels | Offline eval | ≥ 85 % (Cohen's κ ≥ 0.7) | Eval gate |

## Retrieval (RAG)

| ID | Metric | Definition | Target |
|---|---|---|---|
| M-R01 | Context recall@k | % of required sections present in top-k | ≥ 90 % |
| M-R02 | Context precision@k | % of top-k chunks that are relevant | ≥ 70 % |
| M-R03 | MRR | Mean reciprocal rank of first relevant chunk | ≥ 0.8 |
| M-R04 | Rerank uplift | Change in precision@k with vs. without reranker | Positive; reported per change |

## Cost and efficiency

| ID | Metric | Definition | Target |
|---|---|---|---|
| M-C01 | Cost per application | Sum of LLM + embedding costs per case | ≤ US$ 0.05 |
| M-C02 | Cost per policy question | LLM + embedding cost | ≤ US$ 0.005 |
| M-C03 | Tokens per case | Input and output tokens by agent | Tracked; alert on +30 % week over week |
| M-C04 | Cache hit rate | Semantic cache hits / lookups | ≥ 30 % |
| M-C05 | Routing efficiency | % of calls served by the small tier | Tracked |

## Operational (SLIs)

| ID | Metric | Definition | Target (SLO) |
|---|---|---|---|
| M-O01 | Availability | Successful requests / total (non-4xx) | 99.5 % |
| M-O02 | Latency | p50 / p95 / p99 per endpoint and agent | See `NFR-P*` |
| M-O03 | Error rate | 5xx and failed agent steps | < 1 % |
| M-O04 | Case completion | Cases reaching `recommended` or `awaiting_review` | ≥ 99 % |
| M-O05 | Provider fallback rate | Calls served by fallback model | Tracked; alert > 5 % |

## Human-in-the-loop and product

| ID | Metric | Definition | Target |
|---|---|---|---|
| M-H01 | Escalation rate | % of cases sent to human review | 20–40 % (too low = risky, too high = useless) |
| M-H02 | Override rate | % of recommendations changed by humans | ≤ 15 % |
| M-H03 | Time to decision | Submission → final decision | Tracked |
| M-H04 | Analyst satisfaction | Average feedback rating | ≥ 4 / 5 |

## Safety and security

| ID | Metric | Definition | Target |
|---|---|---|---|
| M-S01 | Prompt-injection resistance | % of attack suite blocked or neutralized | ≥ 98 % |
| M-S02 | PII leakage | PII found in logs, traces, memory or outputs | 0 |
| M-S03 | Guardrail false positives | Legitimate inputs blocked | ≤ 2 % |
| M-S04 | Unauthorized action attempts | Denied RBAC checks | Tracked; alert on spikes |

## Risk model (MLOps)

| ID | Metric | Definition | Target |
|---|---|---|---|
| M-ML01 | AUC-ROC | Discrimination on holdout set | ≥ 0.75 |
| M-ML02 | Brier score / calibration | Probability calibration | Brier ≤ 0.15; calibration plot reviewed |
| M-ML03 | KS statistic | Separation between good and bad | ≥ 0.35 |
| M-ML04 | Data drift (PSI) | Population Stability Index per feature | Alert when PSI > 0.2 |
| M-ML05 | Prediction drift | Shift in score distribution | Alert when PSI > 0.2 |

## Fairness

| ID | Metric | Definition | Target |
|---|---|---|---|
| M-F01 | Disparate impact ratio | Approval rate of group / reference group | 0.8 – 1.25 |
| M-F02 | Equal opportunity difference | TPR difference between groups | ≤ 0.1 |
| M-F03 | Rationale consistency | Same case with only sensitive attribute changed → same outcome | ≥ 98 % |

Sensitive attributes are never model features. They exist only in the evaluation dataset (synthetic) to measure fairness.
