# Weekly Architectural Synthesis: Week 30, 2026

**Focus Domain**: Non-Human Agent (NHA) Identity & RFC 8693 Token Delegation  
**Published**: Sunday, July 26, 2026  
**Author**: Enterprise Agent Governance Framework (EAGF) Working Group  

---

## Executive Summary

Week 30 marked the official baseline release of **Pillar 01 (Identity & NHA Authentication)**. As enterprise adoption of autonomous agent systems accelerates, legacy static API keys and long-lived OAuth refresh tokens are rapidly becoming top security vulnerabilities.

This week's architectural consensus establishes **RFC 8693 OAuth 2.0 Token Exchange** as the enterprise standard for delegating user credentials to Non-Human Agents.

---

## Key Achievements & Architectural Decisions

### 1. Cryptographic Workload Identity for NHAs
- **Decision**: Every autonomous agent instance must be assigned an ephemeral cryptographic identity (e.g., SPIFFE ID or short-lived X.509 certificate) rather than sharing application-wide service account keys.
- **Rationale**: Isolates blast radius to single workflow instances and enables exact attribution during security forensics.

### 2. Standardized RFC 8693 Token Exchange Flow
- Successfully drafted and approved the specification for [RFC 8693 Token Exchange Mechanics](../pillars/01-identity-and-nha/2026-07-27-token-exchange-rfc8693.md).
- Enforced mandatory `act` (Actor) claims in all delegated JWTs, allowing downstream tool gateways to verify both the delegating human user (`sub`) and the executing agent (`act.sub`).

### 3. Strict 15-Minute Token Lifetimes
- All access tokens issued to NHAs are capped at a **15-minute maximum TTL**.
- Re-authentication and token exchange must occur automatically via background sidecars for long-running agentic tasks.

---

## Weekly Metric & Community Artifacts

- **New Specifications Released**: 2
- **Core Pillars Active**: Pillar 01 (Identity & NHA)
- **Infographic Published**: [NHA Token Exchange Diagram](../pillars/01-identity-and-nha/assets/2026-07-27-nha-infographic.png)
- **Social Captions Published**: [July 27 Social Release](../social-captions/2026-07-27-captions.md)

---

## Next Week Outlook (Week 31)

Transitioning from Identity to **Pillar 02: Policy & Action Boundaries**. We will publish the **4-Tier Risk Classification Framework** and reference OPA/Rego policies for restricting agent tool call capabilities.
