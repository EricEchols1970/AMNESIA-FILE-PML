# PML Core Specification v0.1

**Author:** Eric Marlon Echols, Owner/Founder
**Date:** 2026-08-22
**Status:** Draft — Architecture Phase

---

## 1. Overview

The Portable Memory Layer (PML) is a gateway-based memory system for AI agents and applications. It provides persistent, searchable memory across sessions and model providers without modifying the underlying models.

### 1.1 Design Goals

- **Model-agnostic:** Works with GPT, Claude, Gemini, local models, and future providers
- **Portable:** Memory can be exported and migrated between deployments
- **Secure:** Tenant isolation, RBAC, audit trails, and encryption by design
- **Scalable:** Retrieval is decoupled from total memory corpus size
- **Transparent:** Honest about limitations of sending plaintext to external providers

### 1.2 Product Boundary

```
Client/Agent → PML Gateway → Model Provider
```

The Memory Gateway is the core product. All memory operations flow through it.

---

## 2. Architecture Components

### 2.1 Memory Gateway

The central control plane. All requests pass through the gateway before reaching any model provider.

**Responsibilities:**
- Authentication and authorization (Auth/RBAC/Policy)
- Memory retrieval and ranking
- Memory write operations
- Model adapter routing
- Audit logging and observability
- Tenant isolation enforcement
- Export and migration handling

### 2.2 Three Memory Stores

#### Structured State Store
The authoritative source of current state. Stores:
- Current entity state (users, projects, preferences, relationships)
- Key-value pairs representing "what is true now"
- Overwritten on state change; previous values archived to Event Store

**Schema:** Document/JSON with typed fields per entity class.

#### Semantic Memory Store
Searchable historical knowledge. Stores:
- Conversation summaries and embeddings
- Historical context and background knowledge
- Vector-indexed for semantic retrieval
- Supports similarity search, keyword search, and hybrid retrieval

**Schema:** Vector embeddings + metadata + original text.

#### Event/Milestone Store
Immutable log of decisions, changes, and important events. Stores:
- State transitions with before/after values
- Decision records (what was decided, when, by whom, why)
- Milestone markers for significant events
- Append-only; never modified or deleted

**Schema:** Event log with timestamp, actor, event type, before/after diff, and rationale.

### 2.3 Model Adapter Interface

```typescript
interface ModelAdapter {
  provider: string;           // "openai" | "anthropic" | "google" | "local" | string
  model: string;              // specific model identifier

  // Send context to model and get response
  complete(prompt: string, context: RetrievedContext): Promise<ModelResponse>;

  // Generate embeddings for semantic storage
  embed(text: string): Promise<number[]>;

  // Health check
  health(): Promise<boolean>;
}
```

Adapters are pluggable. Adding a new provider means implementing this interface — no changes to the memory system.

### 2.4 MCP-Compatible Memory Interface

The gateway exposes an MCP (Model Context Protocol) compatible interface so any MCP-aware agent can use PML as a memory provider.

**Operations:**
- `memory.recall` — retrieve relevant context for current operation
- `memory.write` — persist new state, events, or semantic knowledge
- `memory.export` — export full memory in documented schema
- `memory.search` — explicit semantic/keyword search
- `memory.event` — record a decision or milestone

---

## 3. Security Model

### 3.1 Layers (in order of enforcement)

1. **Tenant Isolation** — every request is scoped to a tenant ID. No cross-tenant access is possible at the storage layer.
2. **Authentication** — API key or OAuth token validated on every request.
3. **Authorization (RBAC)** — role-based access control. Roles: admin, writer, reader, export.
4. **Scoped Session Credentials** — short-lived tokens for individual sessions with scoped permissions.
5. **Encryption** — data encrypted at rest (AES-256) and in transit (TLS 1.3).
6. **Audit Trail** — every read, write, export, and admin action logged with actor, timestamp, and outcome.
7. **Key Management** — per-tenant encryption keys with rotation support.

### 3.2 Honest Security Statement

> Once plaintext memory is deliberately sent to an external model provider for processing, that provider can access and process that plaintext. PML does not claim to protect memory from the model provider it is sent to. PML protects memory at rest, in transit, and from unauthorized access — but cannot control what an external provider does with plaintext it receives.

This statement is included in all technical documentation and investor materials. It is a trust feature, not a liability.

---

## 4. Export and Self-Host

### 4.1 Export Schema

Memory exports use a documented JSON schema:

```json
{
  "schema_version": "1.0",
  "exported_at": "2026-08-22T00:00:00Z",
  "tenant_id": "string",
  "structured_state": { ... },
  "semantic_memory": [
    { "id": "string", "embedding": [], "text": "string", "metadata": {} }
  ],
  "events": [
    { "id": "string", "timestamp": "string", "type": "string", "data": {} }
  ]
}
```

### 4.2 Migration

Customers can:
1. Export memory from any Amnesia deployment
2. Import into another Amnesia deployment
3. Self-host the entire PML stack using open-source components
4. Retain full ownership of all memory data

**"Your memory belongs to you."**

---

## 5. Scaling Statement

PML decouples per-turn context consumption from the total historical memory corpus by retrieving only context relevant to the current operation.

Retrieval cost varies with:
- Amount of retrieved context
- Query complexity
- Reranking requirements
- Model/tool call overhead

PML does not guarantee "roughly constant per-turn cost." Instead, PML ensures that per-turn cost is bounded by the size of relevant context, not the size of the total memory corpus.

---

## 6. Provider Adapters

### 6.1 Initial Providers
- OpenAI (GPT-4, GPT-4o, o1, etc.)
- Anthropic (Claude 3.5 Sonnet, Claude 3 Opus, etc.)
- Google (Gemini 1.5 Pro, Gemini 2.0, etc.)

### 6.2 Local/Private Models
- Ollama-compatible local models
- vLLM-compatible local deployments
- Custom enterprise model deployments

### 6.3 Adding a Provider

1. Implement the `ModelAdapter` interface
2. Register the adapter in the gateway configuration
3. No changes to memory stores or retrieval logic required

---

## 7. Pricing Approach

Pricing will be derived from the resulting architecture and measurable usage units, not arbitrary token quotas.

**Potential metering dimensions:**
- Memory operations (reads/writes per month)
- Storage volume (structured + semantic + events)
- Retrieval complexity (number of retrieved chunks, reranking operations)
- Model adapter calls (proxied to external providers)
- Export operations
- Tenant seats and RBAC roles

Final pricing model to be defined after M8 (Observability + Audit + Export).

---

## Changelog

- v0.1 (2026-08-22): Initial draft specification — Eric Marlon Echols
