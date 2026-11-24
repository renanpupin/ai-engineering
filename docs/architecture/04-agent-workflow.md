# 04 — Agent Workflow

How a case moves through the multi-agent system. Specs: `agents.md`, FR-032–FR-034, FR-040, FR-050.

## Orchestration graph

The Orchestrator first builds a **plan**, then executes it. Independent steps run in parallel. Every path ends either in a recommendation or an escalation, never in a silent failure.

```mermaid
flowchart TB
    START([Case received]) --> PLAN["Orchestrator: build plan"]
    PLAN --> VAL{"Plan valid?"}
    VAL -- no --> ESC
    VAL -- yes --> DOC["Document agent<br/>extract fields"]
    DOC --> PAR{{"parallel"}}
    PAR --> POL["Policy agent<br/>find applicable rules"]
    PAR --> RISK["Risk agent<br/>score + explain"]
    POL --> SYN["Orchestrator: synthesize<br/>draft recommendation"]
    RISK --> SYN
    SYN --> REV["Reviewer agent<br/>AI judge"]
    REV -- accept --> RULES{"Escalation rules"}
    REV -- revise --> SYN2["Revise once"] --> REV2["Reviewer agent"]
    REV2 -- accept --> RULES
    REV2 -- fail --> ESC
    REV -- escalate --> ESC
    RULES -- none met --> OUT([Recommended])
    RULES -- any met --> ESC([Awaiting human review])

    DOC -. failure after retry + fallback .-> ESC
    POL -. failure .-> ESC
    RISK -. failure .-> ESC
    BUDGET["Budget / step limit guard"] -. exceeded .-> ESC
```

## Sequence of a case

```mermaid
sequenceDiagram
    autonumber
    participant W as Worker
    participant O as Orchestrator
    participant M as Case memory
    participant D as Document agent
    participant P as Policy agent
    participant R as Risk agent
    participant J as Reviewer agent
    participant T as MCP tools
    participant L as LLM gateway

    W->>O: process(case)
    O->>M: read similar past cases (long-term)
    O->>L: plan (structured output)
    O->>M: store plan
    O->>D: extract(documents)
    D->>T: parse_pdf / ocr_image
    D->>L: few-shot extraction (vision tier)
    D-->>O: ExtractionResult
    par policy and risk
        O->>P: findings(application)
        P->>T: search_policy (hybrid + rerank)
        P->>L: interpret with CoT, cite sections
        P-->>O: PolicyFindings
    and
        O->>R: assess(features)
        R->>T: score_application, explain_score
        R->>L: explain from attributions
        R-->>O: RiskAssessment
    end
    O->>L: synthesize Recommendation
    O->>J: review(draft, evidence)
    J->>J: deterministic checks
    J->>L: rubric grading
    J-->>O: ReviewResult
    O->>M: store summary (redacted)
    O-->>W: Recommendation or Escalation
```

## ReAct loop inside an agent

```mermaid
flowchart LR
    TH["Thought<br/>what do I need?"] --> AC["Action<br/>call a tool"]
    AC --> OB["Observation<br/>tool result, untrusted"]
    OB --> TH
    TH --> FIN["Final answer<br/>structured output"]
    LIM["Max steps / budget"] -. stops .-> TH
```

## Prompting techniques by agent

| Agent | Technique | Why |
|---|---|---|
| Orchestrator | Plan-and-execute + ReAct | Explicit, auditable plan; adaptive when a step fails |
| Document | Few-shot, structured output, vision | Examples anchor field formats and units |
| Policy | RAG + Chain-of-Thought | Rules need interpretation; reasoning is checked against citations |
| Risk | Zero-shot, tool use | The model only explains numbers returned by the tool |
| Reviewer | Rubric-based LLM-as-a-judge | Repeatable grading with explicit criteria |

## Memory

```mermaid
flowchart LR
    subgraph Working["Working memory — one call"]
        CTX["Context window<br/>budgeted"]
    end
    subgraph Short["Short-term — one case"]
        PLANM["Plan"]
        RES["Intermediate results"]
    end
    subgraph Long["Long-term — across cases"]
        SUM["Redacted case summaries<br/>embedded"]
        FB["Reviewer feedback patterns"]
    end
    Short -->|selected, summarized| Working
    Long -->|retrieved top-k| Working
    Short -->|case closed: redact + summarize| Long
```
