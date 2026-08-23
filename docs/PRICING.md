# AMNESIA FILE ENGINE
## Enterprise Pricing Model (Portable Memory Layer Architecture)

**Product:** Amnesia File Engine
**Architecture Codename:** Portable Memory Layer (PML)
**Founder & Owner:** Eric Marlon Echols
**Date:** August 22, 2026
**Status:** Confidential — Draft for investor/internal review

---

## 1. Why This Pricing Model Differs From the Original

The original tier structure priced against "optimized tokens/mo" — a unit tied to the old prompt-compression design. Under the PML architecture, the Memory Gateway is the core metered infrastructure through which Amnesia File delivers portable enterprise memory. Charging by raw token count would work against the product's own value proposition, since the whole point of retrieval-based memory is that per-turn token consumption should *not* scale with total history size.

Instead, pricing is built on the **actual cost-driving units of the Memory Gateway**:

| Usage Unit | What It Measures | Why It's the Right Metric |
|---|---|---|
| **Active Tenant Seats** | Distinct users/agents with provisioned memory access | Standard SaaS seat model; predictable revenue base |
| **Gateway Requests** | Read/write calls to the Memory Gateway | Directly tracks infrastructure load |
| **Retrieval Operations** | Vector search + reranking calls | The most compute-intensive gateway operation; scales with usage sophistication, not history size |
| **Stored Memory Volume (GB)** | Size of a tenant's structured + semantic + event stores | Directly tracks storage cost |
| **Model Adapter Calls** | Requests routed through adapters to customer's model provider | Pass-through routing cost; provider API costs are customer's own under BYOK |

This gives customers a bill that maps to what they're actually consuming, and gives Amnesia File a cost structure that scales with infrastructure spend rather than an arbitrary token quota that has no fixed relationship to actual cloud costs.

---

## 2. Commercial Architecture

```
                    CUSTOMER
                       │
                 Amnesia File
                       │
             ┌─────────▼─────────┐
             │   PML Gateway     │
             │                   │
             │ Memory Retrieval  │
             │ State Management  │
             │ Security / RBAC   │
             │ Audit / Export    │
             │ Model Adapters    │
             └─────────┬─────────┘
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
      GPT           Claude           Gemini
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                Private / Local AI
```

### BYOK (Bring Your Own Key) — Default Enterprise Model

Amnesia File charges for the PML infrastructure. The customer maintains its own relationship and billing with the underlying model provider (OpenAI, Anthropic, Google, etc.).

**Value proposition:**

> Amnesia File doesn't replace your AI providers. It sits above them and makes your organizational memory portable across them.

This produces a clean separation:
- Amnesia File bills for: gateway requests, retrieval operations, storage, seats, audit/export
- Customer bills directly with: OpenAI, Anthropic, Google, etc. for model API usage

This avoids turning Amnesia File into a giant pass-through billing operation before the gateway economics are proven.

### Optional Managed Routing (Future Product)

A later add-on where Amnesia File handles provider billing, routes across multiple providers, and charges a transparent usage margin. This is a Phase 2 revenue stream, not a launch feature.

---

## 3. Tier Structure

### Developer Tier — $200/mo base
- Up to 5 active tenant seats
- 50,000 gateway requests/mo included
- 10,000 retrieval operations/mo included
- 10 GB stored memory included
- 1 model adapter (choice of Claude, GPT, or Gemini)
- BYOK required (customer provides own API key)
- Standard AES-256 encryption at rest/in transit
- Community support

**Overage:** $0.002/request beyond included, $0.01/retrieval op beyond included, $2/GB/mo beyond included.

### Team Tier — $900/mo base
- Up to 25 active tenant seats
- 400,000 gateway requests/mo included
- 100,000 retrieval operations/mo included
- 100 GB stored memory included
- Up to 3 model adapters
- BYOK required
- RBAC (role-based access control)
- Audit log retention (90 days)
- Email support, 24hr SLA

**Overage:** $0.0015/request, $0.008/retrieval op, $1.75/GB/mo.

### Enterprise Tier — $4,500/mo base
- Up to 250 active tenant seats
- 3M gateway requests/mo included
- 750,000 retrieval operations/mo included
- 1 TB stored memory included
- Unlimited model adapters (all supported providers + local/private models)
- BYOK default; optional managed routing available
- Dedicated tenant isolation (dedicated compute, not shared cluster)
- Full audit trail + export/self-host capability
- SOC 2 readiness documentation provided
- Dedicated support, 4hr SLA

**Overage:** $0.001/request, $0.006/retrieval op, $1.50/GB/mo.

### Scale-Up / Private Cloud Tier — starting at $18,000/mo
- Unlimited seats
- Custom request/retrieval volume, negotiated
- Single-tenant private cloud deployment
- Custom model adapter development for proprietary/internal models
- BYOK default; managed routing negotiable
- Dedicated security architecture review + compliance support (SOC 2, HIPAA, etc. as applicable)
- Named technical account manager

*(Pricing at this tier is necessarily negotiated per deployment — infrastructure and compliance costs vary significantly by customer requirements.)*

---

## 4. Cost-to-Serve Reasoning (Illustrative)

This is a framework, not audited financials. All pricing numbers should be treated as draft hypotheses, not validated market prices, until competitor/customer research and actual cost-to-serve measurements are completed (target: M10 Scale/Load Testing).

| Cost Driver | Approx. Driver Type | Notes |
|---|---|---|
| Vector DB (retrieval ops) | Variable, per-operation | Cost depends on chosen vector DB provider and index size |
| Structured state DB | Semi-fixed, scales with storage GB | Standard managed DB pricing |
| Model adapter routing | Variable, per-call | Under BYOK, provider API costs are the customer's own; Amnesia File only bears routing infrastructure cost |
| Compute (gateway itself) | Semi-fixed, scales with request volume | Standard containerized service scaling |
| Storage (memory volume) | Variable, per-GB | Cloud object/DB storage pricing |

**BYOK advantage:** Because the customer pays the model provider directly, Amnesia File's cost structure does not include model API costs. This keeps margins clean and predictable — Amnesia File's costs scale with infrastructure (compute, storage, vector DB), not with expensive model API consumption.

---

## 5. Patent/IP Alignment Note

The original patent concept centers on state injection, cryptographic envelopes, and token packing. PML centers on external retrieval, structured state, interoperability, and a gateway. These are materially different technical approaches.

**Recommendation:** Do not present the original patent language as covering the PML implementation without patent counsel analyzing the disclosure and claims. The provisional patent should be reviewed and potentially revised to match the retrieval-based PML architecture before the pricing sheet or investor package references IP protection.

---

## 6. Why This Is a Stronger Investor Narrative

- **Ties revenue to real infrastructure cost drivers**, so gross margin claims can be defended under diligence rather than asserted.
- **BYOK keeps Amnesia File out of the pass-through billing trap** — revenue comes from infrastructure value, not from reselling OpenAI/Anthropic credits.
- **Rewards the architecture's actual advantage** (retrieval instead of full-history replay) by making "efficient usage" cheaper for the customer.
- **Scales credibly** — dedicated tenant isolation and private cloud are priced as what they cost to run.
- **Pricing numbers are explicitly draft hypotheses** — defensible because they're labeled as such, to be validated by M10 benchmarks and market research.

---

## 7. Next Steps

The pricing model, roadmap, architecture doc, and technical execution specification together form a coherent investor package. Before presenting:

1. **Patent review** — Have patent counsel analyze whether the original provisional covers PML or needs revised claims.
2. **Market research** — Validate pricing against competitors (LangChain, Mem0, Zep, etc.) and customer willingness-to-pay.
3. **Cost-to-serve measurement** — M10 load testing produces real infrastructure costs that validate or adjust overage rates.
4. **BYOK vs. managed routing decision** — Confirm BYOK as default; decide timeline for managed routing add-on.

---

## Changelog

- v0.1 (2026-08-22): Initial pricing model draft — Eric Marlon Echols
- v0.2 (2026-08-22): Added BYOK model, revised gateway positioning language, added patent alignment note, pricing marked as draft hypotheses — Eric Marlon Echols
