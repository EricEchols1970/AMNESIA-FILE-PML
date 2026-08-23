# AMNESIA FILE ENGINE
## 12-Month Technical Execution Roadmap (Portable Memory Layer Architecture)

**Product:** Amnesia File Engine
**Architecture Codename:** Portable Memory Layer (PML)
**Founder & Owner:** Eric Marlon Echols
**Prepared for:** Investor / Technical Diligence Package
**Date:** August 22, 2026
**Status:** Confidential
**Architecture basis:** Memory Gateway + Structured/Semantic/Event stores + MCP-compatible model adapters (see *Amnesia File Engine — Portable Memory Layer Architecture*)

---

## 1. Roadmap Summary Table

| # | Milestone | Duration | Quarter | Key Deliverable | Exit Criteria |
|---|---|---|---|---|---|
| M1 | PML Core Specification | Weeks 1–4 | Q1 | Written spec: data schemas, API contracts, adapter interface | Spec reviewed and frozen for v1 |
| M2 | Memory Gateway Alpha | Weeks 3–10 | Q1 | Working gateway: auth, request routing, basic read/write | Gateway handles read/write for 1 test tenant |
| M3 | Structured + Semantic Memory | Weeks 8–16 | Q1–Q2 | State store (DB) + vector store integrated | Round-trip write/retrieve works across both stores |
| M4 | First Model Adapter + MCP Interface | Weeks 14–20 | Q2 | Claude adapter live via MCP-compatible tool calls | End-to-end agent session using gateway memory |
| M5 | Multi-Provider Interoperability | Weeks 18–26 | Q2 | GPT + Gemini adapters added | Same memory corpus usable across ≥3 providers |
| M6 | Security Architecture Validation | Weeks 22–28 | Q2–Q3 | Tenant isolation, encryption at rest/in transit, key mgmt | Passes internal pen test; no cross-tenant leakage |
| M7 | Enterprise Tenant/RBAC Layer | Weeks 26–34 | Q3 | Role-based access, scoped session credentials | Multi-tenant demo with isolated permissions |
| M8 | Observability + Audit + Export | Weeks 30–38 | Q3 | Audit trail, usage metrics, documented export schema | Customer can export full memory corpus and re-import |
| M9 | Enterprise Private Beta | Weeks 36–42 | Q3–Q4 | 3–5 design partners onboarded | Partners running real workloads for 2+ weeks |
| M10 | Scale/Load Testing | Weeks 40–46 | Q4 | Load tests at target concurrency, retrieval latency benchmarks | Meets defined SLA (e.g., p95 retrieval latency target) |
| M11 | Security & Compliance Readiness | Weeks 44–50 | Q4 | SOC 2 readiness assessment, third-party pen test | Findings remediated or risk-accepted |
| M12 | Production Enterprise Launch | Weeks 48–52 | Q4 | GA release, pricing live, support process live | First paying enterprise customer live in production |

*(Note: durations overlap intentionally — this reflects realistic parallel workstreams, not sequential handoffs. Adjust week numbers once team size and hiring timeline are confirmed; the sequencing and dependency logic below is the part that matters most to technical reviewers.)*

---

## 2. Dependency Logic (Why This Order)

- **M1 before everything:** the adapter interface and schema decisions in M1 determine whether M4/M5 (multi-provider support) are cheap or expensive later. This is the highest-leverage early investment.
- **M2/M3 in parallel:** the gateway shell (auth, routing) and the storage layers can be built concurrently by separate engineers once the M1 contracts are fixed.
- **M4 before M5:** prove the MCP-compatible interface works end-to-end with one provider before generalizing to three. Adding providers is a validation of the adapter pattern, not new architecture.
- **M6 gates M7:** access control and tenant isolation must be validated before building the RBAC layer on top of it — building enterprise permissions on an unvalidated security foundation is the most common cause of expensive rework.
- **M8 (export) before M9 (beta):** design partners should be able to leave at any time with their data intact. This is also the feature that most directly demonstrates the anti-lock-in claim to design partners themselves.
- **M11 before M12:** compliance readiness should be validated before, not after, the first enterprise contract — retrofitting SOC 2 controls post-launch is significantly more expensive than building to them from M6 onward.

### Dependency Graph

```
M1 ─┬─→ M2 ─┬─→ M4 ──→ M5
    │       │
    └─→ M3 ─┘    M6 ──→ M7 ──→ M8 ──→ M9 ──→ M10 ──→ M11 ──→ M12
```

**Critical path:** M1 → M2 → M4 → M6 → M7 → M8 → M9 → M10 → M11 → M12

---

## 3. Team & Resourcing Implications

This sequencing assumes a small core team scaling over the year:

| Phase | Approx. headcount | Key roles needed |
|---|---|---|
| M1–M3 (Q1) | 2–3 engineers | Backend/infra engineer, DB/retrieval engineer |
| M4–M6 (Q2) | 3–5 engineers | + AI/model integration engineer, security engineer |
| M7–M9 (Q3) | 5–8 engineers | + platform/DevOps, solutions engineer for design partners |
| M10–M12 (Q4) | 8–12 engineers | + SRE/on-call, compliance/security lead, support |

This is a reasonable base case for illustration — actual hiring should be tied to funding milestones, and a technical investor will expect this table to flex with the size of the raise.

---

## 4. What This Roadmap Deliberately Does Not Claim

To keep this credible under technical diligence:

- It does **not** claim full multi-provider parity by M5 — only that the same memory corpus is *usable* across providers, since adapter quality (e.g., how well each model uses tool results) will vary by provider and improve iteratively.
- It does **not** claim SOC 2 *certification* by M11 — certification typically requires an observation period after readiness; M11 targets audit-readiness, with certification as a post-launch milestone.
- It does **not** assume retrieval latency or cost is "constant" regardless of scale — M10 exists specifically to benchmark and publish real numbers rather than assert them in advance.

---

## 5. Roadmap Gantt Visualization

```
Q1  │ M1 ████               │ M2     ████████  │ M3          ██████████
Q2  │          M4 ██████     │ M5       ████████  │ M6     ██████
Q3  │                    M7 ████████  │ M8       ████████  │ M9     ██████
Q4  │                              M10 ██████  │ M11     ██████  │ M12 ████
```

---

## 6. Pricing Integration

- M8 produces usage metering infrastructure → enables pricing model implementation
- M10 produces real cost-to-serve data → validates/adjusts overage rates
- M11 produces compliance docs → unlocks Enterprise tier sales
- M12 launches all tiers → pricing goes live

The pricing model (see PRICING.md) is designed to be finalized after M10, when real infrastructure costs are measured. All pricing numbers are draft hypotheses until validated by M10 benchmarks and market research.

---

## 7. Immediate Next Step

With this roadmap in place, the pricing model is built against **real usage units** defined by this architecture — gateway requests, retrieval operations, stored memory volume, and active tenant seats — rather than flat token-quota tiers.

The investor package now consists of:
1. **Architecture** — PML (see PML_SPECIFICATION.md)
2. **Engineering** — Technical Execution Spec (see TECHNICAL_EXECUTION_SPEC.md)
3. **Roadmap** — This document
4. **Pricing** — Gateway-based SaaS with BYOK (see PRICING.md)
5. **Security** — Security Architecture (see SECURITY.md)

---

## Changelog

- v0.1 (2026-08-22): Initial roadmap draft — Eric Marlon Echols
- v0.2 (2026-08-22): Finalized roadmap with overlapping timelines, dependency logic, resourcing, and explicit non-claims — Eric Marlon Echols
