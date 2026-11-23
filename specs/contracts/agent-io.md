# Agent I/O Contracts — v1

All agent outputs are structured and validated. Field names below are normative; types are expressed in Python / Pydantic notation.

## ExtractionResult (AG-02)

```python
class ExtractedField:
    name: str                 # e.g. "annual_revenue"
    value: float | str | None
    unit: str | None          # e.g. "EUR"
    period: str | None        # e.g. "FY2025"
    confidence: float         # 0..1
    status: Literal["ok", "missing", "low_confidence"]
    source: Citation          # document_id + page + bounding box

class ExtractionResult:
    fields: list[ExtractedField]
    document_types: list[str]
```

Required fields: `annual_revenue`, `ebitda`, `total_debt`, `cash`, `years_in_business`, `monthly_debt_service`.

## PolicyFindings (AG-03)

```python
class Citation:
    type: Literal["policy", "document"]
    policy_version: str | None
    section_id: str | None
    document_id: str | None
    page: int | None

class PolicyFinding:
    rule_summary: str
    applies: bool
    satisfied: bool | None
    citations: list[Citation]   # min 1

class PolicyFindings:
    findings: list[PolicyFinding]
    covered: bool               # False → abstain
```

## RiskAssessment (AG-04)

```python
class FeatureAttribution:
    feature: str
    contribution: float         # signed

class RiskAssessment:
    probability_of_default: float   # 0..1
    risk_band: Literal["A", "B", "C", "D", "E"]
    model_version: str
    top_factors: list[FeatureAttribution]
    explanation: str
```

## Recommendation (AG-01)

```python
class Recommendation:
    outcome: Literal["approve", "reject", "refer"]
    confidence: float
    risk: RiskAssessment
    findings: PolicyFindings
    rationale: str
    citations: list[Citation]       # min 1
    escalation_reasons: list[str]
```

## ReviewResult (AG-05)

```python
class CriterionScore:
    criterion: Literal["groundedness", "citation_validity", "policy_compliance",
                       "completeness", "risk_consistency"]
    score: float      # 0..1
    issues: list[str]

class ReviewResult:
    scores: list[CriterionScore]
    passed: bool
    action: Literal["accept", "revise", "escalate"]
```
