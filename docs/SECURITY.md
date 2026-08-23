# Security Architecture

**Product:** Amnesia File Engine — Portable Memory Layer (PML)
**Owner/Founder:** Eric Marlon Echols
**Date:** 2026-08-22
**Status:** Draft — Architecture Phase

---

## 1. Security Philosophy

Security is an architecture, not a marketing phrase. Every layer of PML is designed with security as a first-class concern, not a bolt-on.

**Core principle:** Be honest about what we protect and what we don't. Trust comes from transparency, not from overpromising.

---

## 2. Security Layers (Enforced in Order)

### Layer 1: Tenant Isolation
Every request is scoped to a tenant ID at the storage layer. Cross-tenant access is structurally impossible — not just policy-prevented.

- Each tenant's data lives in isolated storage partitions
- Query scoping is enforced at the database/vector-DB level, not just at the API
- No tenant can query, read, or enumerate another tenant's memory

### Layer 2: Authentication
- API key or OAuth 2.0 token required on every request
- Keys are scoped to a tenant and role
- Failed auth attempts are rate-limited and logged

### Layer 3: Authorization (RBAC)
Roles:
- **admin** — full access including user management, export, configuration
- **writer** — can read and write memory, cannot manage tenants or export
- **reader** — read-only access to memory stores
- **export** — can trigger memory export (separate from admin for delegation)

Policies are evaluated on every request, not cached across sessions.

### Layer 4: Scoped Session Credentials
- Short-lived tokens (15-60 minute TTL) for individual sessions
- Scoped to specific operations (e.g., read-only session, write-only session)
- Revocable by admin at any time
- Reduces blast radius of credential compromise

### Layer 5: Encryption
- **At rest:** AES-256 encryption for all stored data
- **In transit:** TLS 1.3 for all network traffic
- **Per-tenant keys:** each tenant has dedicated encryption keys
- **Key rotation:** keys can be rotated without downtime; old data re-encrypted lazily

### Layer 6: Audit Trail
Every action is logged:
- Actor (tenant ID + user/agent ID)
- Action type (read, write, export, admin, auth)
- Target (which memory store, which record)
- Timestamp (UTC, immutable)
- Outcome (success/failure)
- Source IP and request ID

Audit logs are:
- Append-only (cannot be modified or deleted)
- Retained per tier (Developer: 30 days, Team: 90 days, Enterprise: 1 year+, Private Cloud: custom)
- Exportable for compliance

### Layer 7: Key Management
- Per-tenant encryption keys
- Key rotation support (automatic and manual)
- Key storage in dedicated KMS (AWS KMS, Google Cloud KMS, or HashiCorp Vault for self-hosted)
- Key access logged in audit trail

---

## 3. Honest Security Statement

> Once plaintext memory is deliberately sent to an external model provider for processing, that provider can access and process that plaintext. PML does not claim to protect memory from the model provider it is sent to.
>
> PML protects memory:
> - At rest (encryption)
> - In transit (TLS 1.3)
> - From unauthorized access (auth, RBAC, tenant isolation)
> - From tampering (audit trail, append-only event store)
>
> PML cannot control what an external model provider does with plaintext it receives. This limitation is documented, disclosed to customers, and included in all technical due diligence materials.

This statement is a trust feature. Enterprise customers respect vendors who clearly delineate what they protect and what they don't.

---

## 4. Compliance Readiness

### SOC 2 Type I (Target: M11)
- Control documentation
- Tenant isolation evidence
- Audit trail completeness
- Encryption verification
- Access control policies

### HIPAA (If Applicable)
- BAA (Business Associate Agreement) available
- PHI handling documentation
- Encryption at rest and in transit (already standard)
- Audit trail for all PHI access

### GDPR
- Data export (already a core feature)
- Right to deletion (tenant data purge)
- Data processing agreement (DPA) template
- EU data residency option (Enterprise tier)

---

## 5. Threat Model

| Threat | Mitigation |
|---|---|
| Cross-tenant data access | Tenant isolation at storage layer (L1) |
| Credential theft | Short-lived scoped tokens (L4), key rotation (L7) |
| Privilege escalation | RBAC enforced per-request (L3) |
| Data exfiltration | Audit trail on all reads/exports (L6), export role separation |
| Plaintext exposure to provider | Documented limitation, customer opts in per-adapter |
| Key compromise | Per-tenant keys, rotation support (L7) |
| Insider threat | Audit trail, RBAC, scoped credentials (L3, L4, L6) |

---

## Changelog

- v0.1 (2026-08-22): Initial security architecture draft — Eric Marlon Echols
