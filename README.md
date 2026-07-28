# Enterprise Agent Governance Framework (EAGF)

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Governance Pillars](https://img.shields.io/badge/Pillars-6_Core_Domains-green.svg)](#core-governance-pillars)
[![Status](https://img.shields.io/badge/Status-Active_Draft-orange.svg)](#120-day-roadmap--index)

An open-source, production-grade architectural framework for governing Non-Human Agents (NHA), Autonomous AI Workflows, and Multi-Agent Orchestration in enterprise environments.

---

## Executive Summary

As enterprise organizations shift from copilot assistants to autonomous **Non-Human Agents (NHAs)** executing multi-step tool calls and transactions, traditional IAM and security controls break down. 

The **Enterprise Agent Governance Framework (EAGF)** provides a standardized taxonomy, threat model, and reference architecture across six key domains:

1. **Identity & Non-Human Agent (NHA) Authentication**: Cryptographic workload identity, RFC 8693 token exchange, and zero-trust session scoping.
2. **Policy & Action Boundaries**: 4-Tier Risk Classification, dynamic policy enforcement (OPA/Rego), and deterministic Human-in-the-Loop (HITL) gates.
3. **Governed Execution Gateways**: API sidecars, prompt injection protection, egress filtering, and tool-level sandboxing.
4. **Data Context & RAG Lineage**: Vector DB access control, document-level authorization, and data provenance tracking.
5. **Audit, Observability & Lineage**: OpenTelemetry distributed agent tracing, multi-agent causal graphs, and immutable event logging.
6. **Lifecycle, DevOps & Multi-Agent Mesh**: CI/CD eval flywheels, agent drift detection, and multi-agent communication guardrails.

---

## High-Level Architectural Overview

```mermaid
graph TD
    subgraph Client / Trigger
        User([Human / System Trigger])
    end

    subgraph Pillar 1: Identity & NHA
        Auth[OAuth2 / RFC 8693 Token Exchange]
        NHA[NHA Cryptographic Identity / SPIFFE]
    end

    subgraph Pillar 3: Governed Execution Gateway
        GW[Agent API Gateway / Sidecar]
        Guard[Prompt Injection & Egress Guardrails]
    end

    subgraph Pillar 2: Policy & Action Boundaries
        OPA{OPA / Rego Engine}
        HITL[Human-in-the-Loop Gate]
        Risk[4-Tier Risk Evaluator]
    end

    subgraph Pillar 4 & 5: Data, RAG & Lineage
        RAG[(Governed Vector DB / RAG)]
        OTel[OTel Agent Tracing & Audit Log]
    end

    subgraph Pillar 6: Execution Environment
        Agent[Autonomous Agent / Multi-Agent Mesh]
        Tools[Sandboxed Tool Execution]
    end

    User --> Auth
    Auth --> NHA
    NHA --> GW
    GW --> Guard
    Guard --> Risk
    Risk --> OPA
    OPA -- "Low/Medium Risk" --> Agent
    OPA -- "Tier 3/4 High Risk" --> HITL
    HITL -- "Approved" --> Agent
    Agent --> RAG
    Agent --> Tools
    Agent -.-> OTel
```

---

## Core Governance Pillars

| Pillar | Domain Name | Focus Areas | Key Specification |
| :--- | :--- | :--- | :--- |
| **01** | [Identity & NHA](pillars/01-identity-and-nha/README.md) | Cryptographic identity, short-lived tokens, impersonation defense | [RFC 8693 Token Exchange](pillars/01-identity-and-nha/2026-07-27-token-exchange-rfc8693.md) |
| **02** | [Policy & Action Boundaries](pillars/02-policy-and-action-boundaries/README.md) | Risk-based bounds, OPA rules, HITL approval workflows | [4-Tier Risk Classification](pillars/02-policy-and-action-boundaries/2026-07-28-4-tier-risk-classification.md) |
| **03** | [Governed Execution Gateways](pillars/03-governed-execution-gateways/README.md) | API sidecars, rate limits, tool payload verification | Gateway Reference Design |
| **04** | [Data Context & RAG Lineage](pillars/04-data-context-and-rag-lineage/README.md) | Document ACLs, vector grounding, provenance verification | RAG Lineage Spec |
| **05** | [Audit, Observability & Lineage](pillars/05-audit-observability-and-lineage/README.md) | OpenTelemetry spans, causal dependency trees, audit logs | Causal Agent Tracing |
| **06** | [Lifecycle, DevOps & Multi-Agent](pillars/06-lifecycle-devops-and-multi-agent/README.md) | Eval flywheels, multi-agent protocol safety, release gates | Agent Release Checklist |

---

## 120-Day Index & Roadmap

The framework is published as a progressive architectural synthesis over 120 days.

```
[Day 001 - Day 020]  Pillar 1: Identity, NHA & Token Exchange Mechanics
[Day 021 - Day 040]  Pillar 2: Action Boundaries, Risk Tiers & Policy Enforcers
[Day 041 - Day 060]  Pillar 3: Execution Gateways, Tool Sandboxes & Egress Inspection
[Day 061 - Day 080]  Pillar 4: Data Governance, RAG Provenance & ACL Filtering
[Day 081 - Day 100]  Pillar 5: Observability, OpenTelemetry Spans & Immutable Audit
[Day 101 - Day 120]  Pillar 6: Multi-Agent Mesh, DevOps Evaluation & CI/CD Release
```

### Recent Weekly Syntheses
- [2026-W30 Architecture Recap](weekly-recaps/2026-W30-weekly-synthesis.md): Foundations of Non-Human Agent Identity and RFC 8693 Delegation.
- [2026-W31 Architecture Recap](weekly-recaps/2026-W31-weekly-synthesis.md): 4-Tier Action Boundaries, OPA Policy Rules, and HITL Integration.

---

## Repository Structure

```
enterprise-agent-governance-framework/
├── README.md                           # Master Overview & Index
├── LICENSE                             # Apache 2.0 License
├── .github/
│   └── workflows/
│       └── publish-site.yml            # CI/CD Workflow for Docs
├── pillars/                            # Core Governance Pillars
│   ├── 01-identity-and-nha/            # Pillar 1 Specifications & Media
│   ├── 02-policy-and-action-boundaries/# Pillar 2 Specifications & Media
│   ├── 03-governed-execution-gateways/ # Pillar 3 Specifications
│   ├── 04-data-context-and-rag-lineage/# Pillar 4 Specifications
│   ├── 05-audit-observability-and-lineage/ # Pillar 5 Specifications
│   └── 06-lifecycle-devops-and-multi-agent/# Pillar 6 Specifications
├── weekly-recaps/                      # Weekly Architectural Summaries
├── social-captions/                    # Platform Specific Captions
└── templates/                          # Article & Publication Templates
```

---

## Contributing & License

Contributions are welcome! Please review our pillar guidelines before submitting pull requests.

Distributed under the **Apache 2.0 License**. See [LICENSE](LICENSE) for details.
