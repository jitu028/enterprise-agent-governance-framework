# 4-Tier Risk Classification Framework for Autonomous AI Actions

**Date**: 2026-07-28  
**Status**: Approved Specification  
**Pillar**: 02 - Policy & Action Boundaries  

---

## 1. Overview

To prevent catastrophic autonomous failure while maintaining developer velocity, the **Enterprise Agent Governance Framework (EAGF)** classifies every tool call and API action into four distinct **Risk Tiers**.

Each risk tier specifies:
- Reversibility requirements
- Enforcement mechanisms (Automated vs. Human-in-the-loop)
- Audit logging density
- SLA for execution approval

---

## 2. Risk Tier Matrix

| Tier Level | Name | Reversibility | Enforcement | Approval Mechanism | Max Allowed Frequency |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Tier 1** | **Read-Only** | N/A (No state change) | Fully Autonomous | Automated Policy Pass | 100 req/min |
| **Tier 2** | **Reversible Write** | High (Can be rolled back) | Policy Gate + Auto Log | OPA Rego Evaluation | 20 req/min |
| **Tier 3** | **Critical State Change** | Low / Irreversible | Human-in-the-Loop | Single Approver Interactive Prompt | 5 req/hour |
| **Tier 4** | **High-Impact Autonomous** | Irreversible / Multi-System | Dual-Admin HITL | Dual Approver + Quorum Signature | 1 req/day |

---

## 3. Tier Definitions & Examples

### Tier 1: Read-Only / Information Retrieval
- **Description**: Actions that query or summarize existing knowledge bases, database records, or public search engines without changing persistent state.
- **Examples**:
  - Searching internal RAG knowledge base for policy documents.
  - Fetching user contact details from CRM.
  - Summarizing open GitHub issues.
- **Control Rules**: Rate limited by Gateway; output scrubbed for PII/PHI.

### Tier 2: Reversible Write / Low-Impact State Changes
- **Description**: Actions that modify non-critical state or create draft objects that can easily be undone or discarded.
- **Examples**:
  - Creating a draft email or support ticket.
  - Adding a tag or note to a CRM lead.
  - Staging a pull request on a non-production branch.
- **Control Rules**: Automated Rego policy evaluation; asynchronous notification sent to channel owner.

### Tier 3: Critical State Change / Financial & PII Actions
- **Description**: Direct modifications to production databases, financial transfers below $10,000, sending customer-facing communications, or altering security permissions.
- **Examples**:
  - Executing SQL `UPDATE` or `INSERT` on production databases.
  - Sending an email blast to > 50 external customers.
  - Triggering a payment or wire transfer.
- **Control Rules**: Synchronous **Human-in-the-Loop (HITL)** gate. Execution suspends until an authorized human user signs off via Slack/Teams/Web Portal.

### Tier 4: High-Impact Autonomous / Destructive Operations
- **Description**: Operations with catastrophic blast radius, including mass data deletion, production cluster teardowns, financial transactions > $10,000, or root key rotation.
- **Examples**:
  - Executing `DROP TABLE`, `TRUNCATE`, or bulk `DELETE`.
  - Deleting Cloud Infrastructure (GCP Projects, AWS VPCs, K8s Clusters).
  - Production software deployment to live traffic.
- **Control Rules**: Mandatory **Dual-Admin Quorum Approval** + WebAuthn/YubiKey hardware challenge.

---

## 4. OPA / Rego Implementation Example

Below is a reference Open Policy Agent (OPA) policy that enforces Tier classification and HITL intercept requirements:

```rego
package agent.governance.action_boundary

default allow = false
default require_hitl = false

# Allow Tier 1 Read-Only actions automatically
allow {
    input.action_tier == "Tier-1-Read-Only"
    input.authenticated == true
}

# Allow Tier 2 Reversible Writes if within rate limits
allow {
    input.action_tier == "Tier-2-Reversible-Write"
    input.authenticated == true
    input.requests_last_minute < 20
}

# Require HITL approval for Tier 3 Critical State Changes
require_hitl {
    input.action_tier == "Tier-3-Critical-State-Change"
}

allow {
    input.action_tier == "Tier-3-Critical-State-Change"
    input.hitl_approval.approved == true
    input.hitl_approval.approver_role == "security_admin"
}

# Reject if missing required attributes
deny_reason["Missing authenticated NHA identity"] {
    not input.authenticated
}

deny_reason["Tier 4 requires dual human approval"] {
    input.action_tier == "Tier-4-High-Impact"
    count(input.hitl_approval.approvers) < 2
}
```

---

## 5. Audit Logging Requirements

Every action evaluation must emit an OpenTelemetry event containing:
- `event.type`: `action_boundary_evaluation`
- `agent.id`: Unique Non-Human Agent identifier
- `action.tier`: `Tier-1` through `Tier-4`
- `policy.result`: `ALLOW`, `DENY`, or `SUSPEND_FOR_HITL`
- `hitl.approver`: User ID of approving human (if applicable)
