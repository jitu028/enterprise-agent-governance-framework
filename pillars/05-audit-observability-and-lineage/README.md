# Pillar 05: Audit, Observability & Lineage

[![Status](https://img.shields.io/badge/Status-Specification_Draft-blue.svg)](#)

---

## Overview

Unlike standard microservices with deterministic request-response lifecycles, autonomous agent systems make dynamic decisions, spawn subagents, and loop across multi-step execution graphs. Debugging and auditing agent workflows requires distributed tracing capable of tracking non-deterministic reasoning steps.

Pillar 05 defines specifications for:
1. **OpenTelemetry Agent Semantic Conventions**: Standardized span attributes for LLM prompts, completions, tool invocations, and subagent handoffs.
2. **Multi-Agent Causal Lineage Graphs**: Parent-child span relationships tracking subagent delegation trees.
3. **Immutable Audit Trails**: Cryptographically signed audit logs stored in append-only storage for compliance and forensics.
4. **Real-time Anomaly Detection**: Flagging agent hallucination loops, excessive token usage, or unexpected action sequence drift.

---

## OpenTelemetry Span Taxonomy

```
trace_id: 4bf92f3577b34da6a3ce929d0e0e4736
└── span_id: 00f067aa0ba902b7 (Root Agent Execution)
    ├── attributes:
    │   ├── agent.id: "sales-qualifier-v2"
    │   ├── agent.session_id: "sess_998877"
    │   └── user.id: "user_123"
    │
    ├── span_id: 11a022bb0ba902c8 (LLM Inference Step)
    │   └── attributes:
    │       ├── gen_ai.system: "gemini"
    │       ├── gen_ai.request.model: "gemini-1.5-pro"
    │       └── gen_ai.usage.total_tokens: 1420
    │
    └── span_id: 22c033cc0ba902d9 (Subagent Spawn: Researcher)
        └── attributes:
            ├── subagent.id: "web-researcher-agent"
            ├── tool.name: "search_web"
            └── action.tier: "Tier-1-Read-Only"
```

---

## Compliance & Forensic Rules

1. **Non-Repudiation**: Every Tier 3 and Tier 4 action record MUST be signed with the Gateway's private key and stored in write-once-read-many (WORM) storage.
2. **Privacy Compliance (GDPR/CCPA)**: Prompts and completions logged in traces MUST redact sensitive PII before persistence unless explicitly authorized under encrypted vault policy.
