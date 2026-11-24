# 03 — Clean Architecture

Level 3 of the C4 model for the Python codebase. See ADR-0003.

## The dependency rule

Dependencies point **inward**. The domain knows nothing about frameworks, models, databases or HTTP.

```mermaid
flowchart TB
    subgraph L4["interfaces — delivery"]
        HTTP["FastAPI routes"]
        CLIi["CLI"]
        MCPS["MCP server entrypoints"]
    end
    subgraph L3["infrastructure — adapters"]
        LLMA["LLM adapters<br/>Anthropic, OpenAI, Ollama"]
        VSA["pgvector store"]
        REPO["SQL repositories"]
        MCPA["MCP client"]
        OTEL["Telemetry"]
    end
    subgraph L2["application — use cases + ports"]
        UC["SubmitCase, ProcessCase,<br/>AnswerPolicyQuestion, DecideCase"]
        PORTS["Ports: LLMProvider, VectorStore,<br/>CaseRepository, RiskScorer,<br/>ToolGateway, AuditLog, Tracer"]
    end
    subgraph L1["domain — entities + rules"]
        ENT["Case, Application, Recommendation,<br/>Decision, PolicyRule, Citation"]
        RULES["Invariants, escalation rules,<br/>autonomy policy"]
    end

    L4 --> L2
    L3 --> L2
    L2 --> L1
    L3 -. implements .-> PORTS
```

## Where agents fit

Agents are **application services**: they orchestrate use cases through ports. Prompts and LLM calls are reached through the `LLMProvider` port; tools are reached through the `ToolGateway` port. This keeps agent logic testable with fakes.

```mermaid
classDiagram
    class LLMProvider {
        <<port>>
        +complete(request) LLMResponse
        +embed(texts) Vectors
    }
    class ToolGateway {
        <<port>>
        +call(tool, args, ctx) ToolResult
    }
    class Agent {
        <<abstract>>
        +name
        +prompt_ref
        +run(input, ctx) Output
    }
    class Orchestrator
    class PolicyAgent
    class RiskAgent
    class DocumentAgent
    class ReviewerAgent
    class AnthropicAdapter
    class OllamaAdapter
    class McpToolGateway
    class FakeLLM

    Agent <|-- Orchestrator
    Agent <|-- PolicyAgent
    Agent <|-- RiskAgent
    Agent <|-- DocumentAgent
    Agent <|-- ReviewerAgent
    Agent --> LLMProvider
    Agent --> ToolGateway
    LLMProvider <|.. AnthropicAdapter
    LLMProvider <|.. OllamaAdapter
    LLMProvider <|.. FakeLLM
    ToolGateway <|.. McpToolGateway
```

## Module map

```
src/lumen/
├── domain/
│   ├── intake/          # Application, Document, ExtractedField
│   ├── policy/          # Policy, PolicyRule, Citation
│   ├── risk/            # RiskAssessment, ModelVersion
│   ├── decisioning/     # Case, Plan, Recommendation, Decision, escalation rules
│   └── governance/      # AuditEvent, Role, Permission
├── application/
│   ├── ports/           # Interfaces only
│   ├── use_cases/       # One class per use case
│   └── agents/          # Orchestrator and specialized agents
├── infrastructure/
│   ├── llm/             # Provider adapters, router, cache, budgets
│   ├── retrieval/       # Chunker, embedder, vector store, reranker
│   ├── persistence/     # Repositories, audit log
│   ├── tools/           # MCP client
│   ├── guardrails/      # Guardrail chain implementations
│   └── telemetry/       # OpenTelemetry setup
└── interfaces/
    ├── api/             # FastAPI
    ├── cli/
    └── mcp_servers/     # lumen-policy, lumen-risk, lumen-docs, lumen-cases
```

## Design patterns used

| Pattern | Where | Why |
|---|---|---|
| Ports and Adapters | `application/ports` ↔ `infrastructure` | Swap providers without touching use cases |
| Strategy | Model router, chunking strategies, rerankers | Choose behavior by configuration and evals |
| Chain of Responsibility | Guardrails | Composable checks that pass, transform or block |
| Repository | Persistence | Domain-oriented data access |
| Factory | Agent and provider creation from config | Centralized wiring |
| Decorator | Retry, cache, tracing around `LLMProvider` | Cross-cutting concerns without changing adapters |
| State | Case lifecycle | Explicit, validated transitions |
