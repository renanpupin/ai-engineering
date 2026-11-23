# Non-Functional Requirements

Targets are initial values for a single-region deployment. They are revisited with real measurements and changes are recorded in this file.

## Performance

| ID | Requirement | Target |
|---|---|---|
| NFR-P01 | Policy Q&A latency | p95 < 4 s |
| NFR-P02 | Full application recommendation latency (async) | p95 < 60 s |
| NFR-P03 | Risk scoring tool latency | p95 < 200 ms |
| NFR-P04 | Time to first token for streamed answers | p95 < 1.5 s |

## Reliability and availability

| ID | Requirement | Target |
|---|---|---|
| NFR-R01 | API availability | 99.5 % monthly |
| NFR-R02 | Successful case completion (recommendation or escalation, no crash) | ≥ 99 % |
| NFR-R03 | Provider failure handling | Retry with backoff, then fallback model, then escalate |
| NFR-R04 | Idempotent case submission | Same idempotency key → same case |
| NFR-R05 | No silent failure | Every failed step is visible in the trace and the case status |

## Scalability

| ID | Requirement | Target |
|---|---|---|
| NFR-S01 | Stateless API and agent workers | Horizontal scaling without session affinity |
| NFR-S02 | Throughput | 50 concurrent cases per worker pool, scaling linearly with replicas |
| NFR-S03 | Backpressure | Queue-based; requests above capacity are queued, not dropped |
| NFR-S04 | Autoscaling | Kubernetes HPA on CPU and queue depth |

## Cost

| ID | Requirement | Target |
|---|---|---|
| NFR-C01 | Average LLM cost per application | ≤ US$ 0.05 |
| NFR-C02 | Average LLM cost per policy question | ≤ US$ 0.005 |
| NFR-C03 | Hard token budget per request | Enforced before the call (see FR-003) |
| NFR-C04 | Model routing | Cheapest model that meets the eval threshold for each task |
| NFR-C05 | Semantic cache hit rate on policy Q&A | ≥ 30 % after warm-up |

## Security and privacy

| ID | Requirement | Target |
|---|---|---|
| NFR-SEC01 | Secrets | Never in code, logs or prompts; loaded from a secret store |
| NFR-SEC02 | PII | Redacted before logging, tracing and long-term memory |
| NFR-SEC03 | Transport | TLS everywhere outside the local environment |
| NFR-SEC04 | Least privilege | Agents and services get only the tools and data they need |
| NFR-SEC05 | Dependency hygiene | Automated dependency and container image scanning in CI |

## Observability

| ID | Requirement | Target |
|---|---|---|
| NFR-O01 | Trace coverage | 100 % of requests traced; LLM content sampled per policy |
| NFR-O02 | Structured logs | JSON logs with trace ID, case ID, actor |
| NFR-O03 | Dashboards | Latency, errors, cost, quality and escalation rate |
| NFR-O04 | Alerting | Alerts on SLO burn rate, cost anomalies and eval score drops |

## Maintainability and quality

| ID | Requirement | Target |
|---|---|---|
| NFR-M01 | Architecture | Clean Architecture; domain has zero framework imports (checked in CI) |
| NFR-M02 | Test coverage | ≥ 90 % domain and application layers; contract tests for every port |
| NFR-M03 | Static checks | Type checking and linting with zero errors on `main` |
| NFR-M04 | Reproducibility | One command to run locally; pinned dependencies; seeded evals |
| NFR-M05 | Documentation | Every concept, decision and spec item documented before release |

## Compliance and governance

| ID | Requirement | Target |
|---|---|---|
| NFR-G01 | Auditability | Every decision reproducible from audit trail: inputs, prompt, model and policy versions |
| NFR-G02 | Retention | Audit events retained for the configured period and immutable |
| NFR-G03 | Human oversight | No adverse decision issued without human approval |
| NFR-G04 | Fairness | Fairness metrics within thresholds before any model promotion |
