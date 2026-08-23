# Amnesia File PML — Technical Execution Specification

**Product:** Amnesia File Engine — Portable Memory Layer (PML)
**Owner/Founder:** Eric Marlon Echols
**Date:** 2026-08-22
**Status:** Draft — Engineering Specification

---

## 1. Purpose

This document converts the PML architecture into actual services, APIs, database schemas, security boundaries, model adapters, deployment infrastructure, and acceptance criteria. It is the bridge:

```
Patent/IP concept → PML architecture → engineering implementation → benchmarks → pricing → investor diligence
```

---

## 2. System Architecture

```
Client / Agent (MCP-compatible host)
      │
      ▼
┌─────────────────────────────────┐
│        Memory Gateway           │  ← MCP Server
│                                 │
│  Auth / RBAC / Policy           │
│  Retrieval / Ranking            │
│  Memory Write Engine            │
│  Model Adapters (outbound)      │
│  Audit / Observability          │
│  Export Engine                  │
└──────────────┬──────────────────┘
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
Structured   Semantic    Event /
  State      Memory     Milestones
  Store       Store      Store
               │
               ▼
        MCP-Compatible
        Memory Interface
               │
    ┌──────────┼──────────┐
    ▼          ▼          ▼
   GPT      Claude      Gemini
    │          │          │
    └──────────┼──────────┘
               ▼
        Local / Private AI
```

### Inbound vs Outbound

- **Inbound:** MCP server handles all memory access from clients/agents (memory.search, memory.get_state, memory.store, etc.)
- **Outbound:** Model Adapter layer handles routing to GPT/Claude/Gemini/local models

These are separate concerns. MCP handles inbound memory access; adapters handle outbound model routing.

---

## 3. MCP Tool Surface

These tools are designed around the three complementary stores while remaining simple enough for models to use reliably. Any MCP-compatible host (Claude, Cursor, custom agents, local models) can use Amnesia File's portable memory without custom adapters.

### 3.1 Core Tools

#### memory.search
Primary retrieval (semantic + hybrid).

| Field | Value |
|---|---|
| **Purpose** | Retrieve relevant memory chunks and optional current structured state for the current operation. Decouples per-turn context from total history size. |
| **Key Inputs** | `query` (string, required), `top_k` (int, 1-20, default 5), `filters` (tags, time_range, store type), `include_state` (bool, default true) |
| **Routes To** | Semantic Memory Store (vector + hybrid) + Structured State Store (if include_state) |

```json
{
  "name": "memory.search",
  "title": "Search Portable Memory",
  "description": "Retrieve the most relevant memory chunks and optional current structured state for the current operation. Decouples per-turn context from total history size.",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Natural language or keyword query"
      },
      "top_k": {
        "type": "integer",
        "minimum": 1,
        "maximum": 20,
        "default": 5
      },
      "filters": {
        "type": "object",
        "properties": {
          "tags": { "type": "array", "items": { "type": "string" } },
          "time_range": {
            "type": "object",
            "properties": {
              "since": { "type": "string", "format": "date-time" },
              "until": { "type": "string", "format": "date-time" }
            }
          },
          "store": {
            "type": "string",
            "enum": ["semantic", "structured", "events", "all"],
            "default": "all"
          }
        }
      },
      "include_state": {
        "type": "boolean",
        "default": true,
        "description": "Also return current structured state"
      }
    },
    "required": ["query"]
  }
}
```

#### memory.get_state
Fetch authoritative structured state.

| Field | Value |
|---|---|
| **Purpose** | Lightweight fetch of current task/project/org state |
| **Key Inputs** | `keys` (array, optional — omit for all), `namespace` (string, optional) |
| **Routes To** | Structured State Store |

#### memory.store
Write new memory.

| Field | Value |
|---|---|
| **Purpose** | Persist new memory. Auto-routes to correct store based on type. Supports structured extraction. |
| **Key Inputs** | `content` (string, required), `type` (enum: fact / decision / event / state_update, required), `tags` (array), `metadata` (object), `namespace` (string) |
| **Routes To** | `fact` → Semantic Store, `decision` → Event Store + Semantic Store, `event` → Event Store, `state_update` → Structured State Store (+ Event Store for diff) |

#### memory.update
Correct or supersede existing memory.

| Field | Value |
|---|---|
| **Purpose** | Update memory while maintaining provenance. Supersedes, doesn't overwrite silently. |
| **Key Inputs** | `id` or `key` (required), `new_content` (string, required), `reason` (string, required) |
| **Routes To** | Original store of the memory + Event Store (records the supersession) |

#### memory.list
Browse / filter memories.

| Field | Value |
|---|---|
| **Purpose** | Exploration and admin use |
| **Key Inputs** | `filters` (tags, store, time_range, namespace), `limit` (int, default 50), `cursor` (string, for pagination) |
| **Routes To** | All stores with filter routing |

#### memory.export
Full or scoped export (anti-lock-in feature).

| Field | Value |
|---|---|
| **Purpose** | Export memory in documented open schema for migration or self-hosting |
| **Key Inputs** | `format` (enum: json / jsonl / okf), `filters` (optional scope), `include_embeddings` (bool, default false) |
| **Routes To** | All stores (read-only export) |
| **RBAC** | Requires `export` role |

#### memory.delete
Scoped deletion.

| Field | Value |
|---|---|
| **Purpose** | Delete specific memories by ID or query |
| **Key Inputs** | `ids` (array) or `query` (string) + `confirm` (bool, required) |
| **Routes To** | Target store(s) + Event Store (records deletion) |
| **RBAC** | Requires `admin` role. Strongly gated. Audit-logged. |

### 3.2 Optional Higher-Value Tools (Phase 2)

| Tool | Purpose |
|---|---|
| `memory.summarize_period` | Semantic roll-up of older events into compressed summaries |
| `memory.get_related` | Multi-hop / graph-style retrieval across connected memories |
| `memory.health` | Tenant storage stats, last sync time, index health, quota usage |

### 3.3 Store Routing Map

```
memory.get_state          → Structured State Store
memory.search (default)   → Semantic Memory Store (vector + hybrid)
type: "state_update"      → Structured State Store + Event Store (diff)
type: "fact"             → Semantic Memory Store
type: "decision"         → Event Store + Semantic Memory Store
type: "event"            → Event Store
memory.export            → All stores (read-only)
memory.delete            → Target store + Event Store (deletion record)
```

---

## 4. MCP Resources

Resources give models passive, URI-addressable context.

| URI | Content | Notes |
|---|---|---|
| `memory://{tenant_id}/state` | Current structured state snapshot | Updated on every state_update write |
| `memory://{tenant_id}/summary` | High-level project/org summary | Generated from structured state + recent events |
| `memory://{tenant_id}/recent-events` | Recent milestones (last 30 days) | Ordered by timestamp |
| `memory://{tenant_id}/schema` | Open memory schema documentation | Helps models use tools correctly |

Supports: `resources/list`, `resources/read`, and optionally change notifications.

---

## 5. Database Schemas

### 5.1 Structured State Store

```json
{
  "tenant_id": "string (uuid)",
  "namespace": "string (default: 'default')",
  "key": "string",
  "value": "object (arbitrary JSON)",
  "updated_at": "timestamp",
  "updated_by": "string (user/agent id)",
  "version": "integer (incremental)",
  "encryption_key_id": "string (KMS key reference)"
}
```

Storage: Document database (MongoDB-compatible or PostgreSQL JSONB).

### 5.2 Semantic Memory Store

```json
{
  "tenant_id": "string (uuid)",
  "id": "string (uuid)",
  "content": "string (original text)",
  "embedding": "vector (float[])",
  "tags": ["string"],
  "metadata": "object",
  "namespace": "string",
  "created_at": "timestamp",
  "created_by": "string",
  "superseded_by": "string (nullable, id of superseding memory)",
  "encryption_key_id": "string"
}
```

Storage: Vector database (Pinecone, Weaviate, Qdrant, or pgvector).

### 5.3 Event/Milestone Store

```json
{
  "tenant_id": "string (uuid)",
  "id": "string (uuid)",
  "type": "enum (decision, state_change, milestone, deletion, supersession)",
  "actor": "string (user/agent id)",
  "timestamp": "timestamp (immutable, UTC)",
  "target_id": "string (id of affected memory)",
  "target_store": "enum (structured, semantic, events)",
  "before": "object (nullable, state before change)",
  "after": "object (nullable, state after change)",
  "reason": "string (nullable)",
  "audit_metadata": {
    "source_ip": "string",
    "request_id": "string",
    "auth_method": "string"
  }
}
```

Storage: Append-only log (PostgreSQL with partitioning by tenant + date, or DynamoDB).

---

## 6. Security Boundaries (Implementation)

### 6.1 Request Pipeline (Enforced in Order)

```
1. TLS termination (load balancer / ingress)
2. Rate limiting (per IP + per tenant)
3. Authentication (API key / OAuth token validation)
4. Tenant isolation enforcement (tenant_id extracted from auth, not request body)
5. RBAC policy evaluation (role + required permission check)
6. Scoped session token validation (if applicable)
7. Request logging (audit trail entry: received)
8. Store access (with per-tenant encryption key)
9. Response logging (audit trail entry: completed, with outcome)
10. Response returned to client
```

### 6.2 Tenant Isolation Implementation

- `tenant_id` is extracted from the authenticated session, NEVER from the request body
- All database queries include `WHERE tenant_id = ?` as a mandatory filter
- Vector DB queries include tenant-scoped namespaces
- No cross-tenant queries are possible at the storage layer

### 6.3 Audit Trail Implementation

- Every audit event written to append-only store BEFORE response is returned
- Audit writes are synchronous for write operations, async for read operations (with fallback to sync on failure)
- Audit log is immutable — no UPDATE or DELETE operations permitted
- Audit log is per-tenant but stored in a separate audit database

---

## 7. Model Adapter Layer

### 7.1 Adapter Interface

```typescript
interface ModelAdapter {
  provider: string;           // "openai" | "anthropic" | "google" | "local"
  model: string;              // specific model identifier

  // Send retrieved context + prompt to model, get response
  complete(
    prompt: string,
    context: RetrievedContext,
    options?: CompletionOptions
  ): Promise<ModelResponse>;

  // Generate embeddings for semantic storage
  embed(text: string): Promise<number[]>;

  // Health check
  health(): Promise<boolean>;
}

interface RetrievedContext {
  semantic_results: SemanticResult[];
  structured_state: object | null;
  recent_events: EventResult[];
}

interface ModelResponse {
  content: string;
  usage: {
    prompt_tokens: number;
    completion_tokens: number;
    total_tokens: number;
  };
  model: string;
  provider: string;
}
```

### 7.2 BYOK Implementation

Under BYOK, the customer provides their own API key for the model provider. The adapter uses the customer's key — Amnesia File does not see, store, or bill for the model provider's API costs.

```
Customer API Key → stored encrypted in tenant config → passed to adapter on each call
```

### 7.3 Initial Adapters

| Adapter | Provider | Models | Status |
|---|---|---|---|
| OpenAIAdapter | OpenAI | GPT-4o, GPT-4, o1, o3 | M4 |
| AnthropicAdapter | Anthropic | Claude 3.5 Sonnet, Claude 3 Opus | M5 |
| GoogleAdapter | Google | Gemini 1.5 Pro, Gemini 2.0 | M5 |
| LocalAdapter | Ollama / vLLM | Any local model | M5 |

---

## 8. Deployment Infrastructure

### 8.1 Stack (Recommended)

| Component | Technology | Rationale |
|---|---|---|
| MCP Server | Python (official mcp SDK v2) or TypeScript | Fastest path to MCP compatibility |
| Transport | Streamable HTTP | Enterprise remote deployment |
| Structured State Store | PostgreSQL (JSONB) | Mature, scalable, JSON support |
| Semantic Memory Store | pgvector (PostgreSQL) or Pinecone | pgvector reduces infra complexity; Pinecone for scale |
| Event Store | PostgreSQL (partitioned append-only) | ACID guarantees, partitioning for scale |
| Cache | Redis | Hot state cache, rate limiting |
| KMS | AWS KMS / Google Cloud KMS / Vault | Per-tenant key management |
| Container Orchestration | Kubernetes or Cloud Run | Auto-scaling gateway |
| Observability | OpenTelemetry → Grafana/CloudWatch | Standard observability stack |

### 8.2 Deployment Tiers

| Tier | Deployment Model |
|---|---|
| Developer | Shared cluster, shared DB (tenant isolation at query level) |
| Team | Shared cluster, dedicated schema per tenant |
| Enterprise | Dedicated compute, dedicated DB instance per tenant |
| Private Cloud | Single-tenant private deployment, customer's cloud or dedicated VPC |

### 8.3 Gateway Service Architecture

```
                    ┌─────────────┐
                    │  Load Bal.  │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         ┌─────────┐ ┌─────────┐ ┌─────────┐
         │ Gateway  │ │ Gateway  │ │ Gateway  │
         │ Pod 1    │ │ Pod 2    │ │ Pod N    │
         └────┬────┘ └────┬────┘ └────┬────┘
              │            │            │
              └────────────┼────────────┘
                           │
         ┌─────────────────┼─────────────────┐
         ▼                 ▼                 ▼
   ┌──────────┐    ┌──────────────┐    ┌──────────┐
   │PostgreSQL│    │ Vector Store │    │  Redis   │
   │(state +  │    │ (semantic)   │    │ (cache)  │
   │ events)  │    │              │    │          │
   └──────────┘    └──────────────┘    └──────────┘
```

---

## 9. Acceptance Criteria (Per Milestone)

### M1 — PML Core Specification
- [ ] Specification document reviewed and approved
- [ ] ModelAdapter interface defined
- [ ] MCP tool schemas defined for all 7 core tools
- [ ] Export schema v1.0 defined
- [ ] Tenant isolation and RBAC models defined

### M2 — Memory Gateway Alpha
- [ ] Gateway service scaffold running
- [ ] Auth middleware (API key validation)
- [ ] Tenant isolation enforcement on all queries
- [ ] Health check endpoint
- [ ] Basic request logging
- [ ] Mock memory stores (in-memory) for testing

### M3 — Structured + Semantic Memory
- [ ] Structured State Store: CRUD operations with tenant isolation
- [ ] Semantic Memory Store: embedding + similarity search
- [ ] Event Store: append-only write, query by time/type
- [ ] Retrieval engine: query, rank, return context
- [ ] Write engine: route to correct store based on type
- [ ] All stores enforce encryption at rest

### M4 — First Model Adapter + MCP Interface
- [ ] OpenAI adapter implemented (complete + embed)
- [ ] BYOK: customer API key stored encrypted, used on calls
- [ ] MCP server: all 7 core tools implemented
- [ ] MCP resources: state, summary, recent-events, schema
- [ ] End-to-end: client → MCP → gateway → store → adapter → model → response
- [ ] Audit trail: all operations logged

### M5 — Multi-Provider Interoperability
- [ ] Anthropic adapter implemented and tested
- [ ] Google adapter implemented and tested
- [ ] Local model adapter (Ollama) implemented and tested
- [ ] Same memory works across all 4 providers
- [ ] Provider switching without memory migration
- [ ] Adapter health monitoring

### M6 — Security Architecture Validation
- [ ] Cross-tenant access: impossible at storage layer (verified by test suite)
- [ ] RBAC: all roles tested with denied/allowed scenarios
- [ ] Scoped session tokens: creation, validation, revocation, expiry
- [ ] Encryption: AES-256 at rest verified, TLS 1.3 in transit verified
- [ ] Key rotation: zero-downtime rotation tested
- [ ] Audit trail: completeness verified (100% of operations logged)
- [ ] Penetration test: basic automated + manual review

### M7 — Enterprise Tenant/RBAC Layer
- [ ] Multi-tenant management dashboard
- [ ] Role creation and permission scoping (UI)
- [ ] Tenant provisioning and deprovisioning
- [ ] Per-tenant configuration (adapters, storage limits, retention)
- [ ] Seat management and usage tracking

### M8 — Observability + Audit + Export
- [ ] Observability dashboard (request volume, latency, error rates)
- [ ] Audit log search and filtered export
- [ ] Memory export: full schema-compliant JSON/JSONL output
- [ ] Self-host deployment guide published
- [ ] Usage metering: all 5 pricing dimensions tracked

### M9 — Enterprise Private Beta
- [ ] 3-5 beta customers onboarded
- [ ] Production deployment running (not staging)
- [ ] Usage data and performance metrics collected
- [ ] Customer feedback documented and prioritized
- [ ] At least 1 iteration cycle completed from feedback

### M10 — Scale/Load Testing
- [ ] Gateway handles 10x peak projected request volume
- [ ] Retrieval performance with 1M+ memory chunks per tenant
- [ ] Per-turn context cost benchmarked vs. raw context replay
- [ ] Real infrastructure costs measured per tier
- [ ] Overage pricing rates validated or adjusted
- [ ] Load test report published (internal)

### M11 — Security & Compliance Readiness
- [ ] SOC 2 Type I documentation complete
- [ ] External security review completed
- [ ] Penetration testing completed (third-party)
- [ ] HIPAA readiness documentation (if applicable)
- [ ] DPA template finalized
- [ ] Incident response plan documented

### M12 — Production Enterprise Launch
- [ ] General availability — all tiers live
- [ ] Self-serve onboarding (Developer + Team tiers)
- [ ] Enterprise sales process operational
- [ ] Developer documentation and SDKs published
- [ ] Case studies from beta customers published
- [ ] Pricing model finalized based on M10 data

---

## 10. Open Questions (Resolve Before M4)

1. **pgvector vs. Pinecone** — pgvector reduces infrastructure complexity but may not scale to Enterprise tier. Decision needed by M3.
2. **Python vs. TypeScript MCP server** — Python has the official mcp SDK v2; TypeScript has broader ecosystem. Decision needed by M2.
3. **Patent claims alignment** — Original provisional patent describes compression/injection, not retrieval/gateway. Patent counsel review needed before M12.
4. **Managed routing timeline** — BYOK confirmed as default. When does managed routing add-on launch? Post-M12 or parallel?

---

## Changelog

- v0.1 (2026-08-22): Initial technical execution specification — Eric Marlon Echols. Includes MCP tool surface, store schemas, security boundaries, deployment infrastructure, and acceptance criteria.
