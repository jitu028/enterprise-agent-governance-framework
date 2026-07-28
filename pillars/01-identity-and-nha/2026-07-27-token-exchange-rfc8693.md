# RFC 8693 OAuth 2.0 Token Exchange for Non-Human Agents

**Date**: 2026-07-27  
**Status**: Approved Specification  
**Pillar**: 01 - Identity & Non-Human Agent (NHA) Authentication  

---

## 1. Problem Statement

When an autonomous AI agent acts on behalf of a user, standard OAuth 2.0 Authorization Code flows or client credentials flows present significant security risks:
- Passing raw user JWTs to multi-agent chains grants downstream tools full access to all user privileges.
- Standard client credentials obscure the identity of the human user on whose behalf the agent is acting.

**RFC 8693 (OAuth 2.0 Token Exchange)** solves this by standardizing token delegation, allowing an agent to exchange a human user's `subject_token` for a scoped, short-lived `delegated_token` where the actor is explicitly defined as the Non-Human Agent (NHA).

---

## 2. Token Exchange Protocol Flow

```
+----------------+          +-------------------+          +-------------------+
|  Human User    |          |  Non-Human Agent  |          | Authorization     |
| (Subject)      |          |  (NHA / Actor)    |          | Server (IdP)      |
+-------+--------+          +---------+---------+          +---------+---------+
        |                             |                              |
        |  1. Invoke Workflow         |                              |
        |  (with User Subject Token)  |                              |
        +---------------------------->|                              |
                                      |  2. Token Exchange Request   |
                                      |  grant_type=urn:ietf:params  |
                                      |  subject_token=<User JWT>    |
                                      |  actor_token=<NHA Cert>      |
                                      |  audience=target-tool-api    |
                                      +----------------------------->|
                                      |                              |
                                      |  3. Issued Token Response    |
                                      |  access_token=<Delegated>    |
                                      |  issued_token_type=...       |
                                      |<-----------------------------+
```

---

## 3. Request Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `grant_type` | `URI` | **Yes** | `urn:ietf:params:oauth:grant-type:token-exchange` |
| `subject_token` | `String` | **Yes** | The original JWT representing the delegating user. |
| `subject_token_type` | `URI` | **Yes** | `urn:ietf:params:oauth:token-type:access_token` or `urn:ietf:params:oauth:token-type:jwt` |
| `actor_token` | `String` | **Optional** | Token/certificate representing the NHA identity. |
| `actor_token_type` | `URI` | **Optional** | `urn:ietf:params:oauth:token-type:jwt` |
| `audience` | `String` | **Yes** | Specific downstream tool or service URI (e.g., `https://api.enterprise.com/v1/crm`). |
| `requested_token_type` | `URI` | **Yes** | `urn:ietf:params:oauth:token-type:access_token` |
| `scope` | `String` | **Yes** | Restricted scope requested for the agent task (e.g., `crm:read_contacts`). |

---

## 4. Delegated Token Payload Schema (JWT Claim Example)

```json
{
  "iss": "https://auth.enterprise.com",
  "sub": "user_987654321",
  "aud": "https://api.enterprise.com/v1/crm",
  "exp": 1785139200,
  "iat": 1785138300,
  "scope": "crm:read_contacts",
  "act": {
    "sub": "nha_agent_sales_qualifier_v2",
    "iss": "https://auth.enterprise.com/agents",
    "spiffe_id": "spiffe://enterprise.com/ns/ai/sa/sales-qualifier"
  },
  "nha_metadata": {
    "framework_version": "EAGF-v1.0",
    "risk_tier": "Tier-1-Read-Only",
    "session_id": "sess_agent_abc123"
  }
}
```

Notice the **`act` (Actor)** claim. Downstream services can verify that:
1. The action is performed on behalf of `sub` (`user_987654321`).
2. The direct executing workload is `act.sub` (`nha_agent_sales_qualifier_v2`).
3. The scope is tightly restricted to `crm:read_contacts`.

---

## 5. Security & Governance Rules

1. **Maximum TTL**: Delegated NHA access tokens MUST NOT exceed **15 minutes** TTL.
2. **Audience Restriction**: Tokens MUST be bound to a single audience URI. Tokens missing an `aud` claim MUST be rejected by the execution gateway.
3. **Revocation Cascading**: If the user's primary session is revoked, all active `act` tokens linked to `sub` MUST be invalidated immediately via distributed token introspection or real-time event bus.
