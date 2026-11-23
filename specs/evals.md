# Evaluation Specification

Evals measure AI behavior the way tests measure code. They run **offline** on versioned datasets (gate in CI) and **online** on sampled production traffic (monitoring).

## Datasets

| ID | Dataset | Content | Size (initial) | Labels |
|---|---|---|---|---|
| DS-01 | Policy Q&A golden set | Questions, expected sections, reference answers | 100 | Human |
| DS-02 | Out-of-scope questions | Questions the policy does not cover | 30 | Human |
| DS-03 | Extraction set | Synthetic documents with ground-truth fields | 50 docs | Generated + verified |
| DS-04 | Application cases | End-to-end cases with expected outcome | 100 | Human |
| DS-05 | Adversarial set | Direct and indirect prompt injections, jailbreaks, PII exfiltration | 100 | Expected behavior |
| DS-06 | Fairness pairs | Counterfactual pairs differing only in sensitive attributes | 100 pairs | Expected equality |
| DS-07 | Judge calibration set | Recommendations with human quality scores | 60 | Human |

Datasets are versioned. Production feedback (FR-061) can be promoted into datasets after review.

## Evaluations

| ID | Evaluation | Dataset | Method | Metrics | Gate |
|---|---|---|---|---|---|
| EV-01 | Retrieval quality | DS-01 | Deterministic | M-R01–M-R03 | Blocking |
| EV-02 | Policy answer quality | DS-01 | LLM judge + citation checks | M-Q03, M-Q04 | Blocking |
| EV-03 | Abstention | DS-02 | Deterministic | M-Q07 | Blocking |
| EV-04 | Extraction | DS-03 | Deterministic with tolerances | M-Q05, M-Q06 | Blocking |
| EV-05 | End-to-end decisions | DS-04 | Deterministic + judge | M-Q01, M-Q02, M-C01 | Blocking |
| EV-06 | Security | DS-05 | Deterministic + judge | M-S01, M-S02 | Blocking |
| EV-07 | Judge calibration | DS-07 | Agreement vs. human | M-Q09 | Blocking for judge changes |
| EV-08 | Fairness | DS-06 + holdout | Statistical | M-F01–M-F03 | Blocking for model promotion |
| EV-09 | Cost and latency | DS-04 | Measured | M-C01, M-O02 | Warning |

## Regression policy

- Any change to a **prompt, model, retrieval setting, tool description or agent code** triggers the relevant evals in CI.
- A change **fails** if a blocking metric falls below its target, or drops more than **2 percentage points** from the `main` baseline.
- Eval runs are deterministic where possible: fixed seeds, `temperature = 0` for graded steps, pinned model versions.
- Results are stored per run with prompt, model and dataset versions, so any two runs can be compared.

## LLM-as-a-judge rules

1. Deterministic checks first (schema, citation exists, numeric tolerance). The judge is used only for what cannot be checked deterministically.
2. Judges use explicit rubrics with scores and short justifications.
3. The judge model is different from, or at least prompted independently of, the model being judged.
4. Judges are themselves evaluated (EV-07) before being trusted.

## Online evaluation

- Sample of production traces is judged asynchronously (groundedness, citation validity).
- Scores are tracked on dashboards; drops trigger alerts (`NFR-O04`).
- Low-scoring traces are queued for human annotation.
