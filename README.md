# AMNESIA FILE — Portable Memory Layer (PML)

**Owner/Founder:** Eric Marlon Echols
**Status:** Architecture Phase
**License:** MIT

## The Problem

Every AI model has amnesia. Each new conversation starts blank. No memory of prior sessions, decisions, relationships, or context. This is the fundamental limitation blocking enterprise-grade AI autonomy.

## The Solution

Amnesia File is a **Portable Memory Layer (PML)** — a gateway architecture that sits between clients/agents and model providers, giving any AI persistent, searchable, portable memory without modifying the model itself.

```
                    AMNESIA FILE
                 PORTABLE MEMORY LAYER
                         │
             ┌───────────▼───────────┐
             │    Memory Gateway     │
             │                       │
             │ Auth / RBAC / Policy  │
             │ Retrieval / Ranking   │
             │ Memory Write Engine   │
             │ Model Adapters        │
             │ Audit / Observability │
             └───────────┬───────────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   Structured State   Semantic       Event/
       Store          Memory         Milestones
                         │
                         ▼
                 MCP-Compatible
                 Memory Interface
                         │
       ┌─────────────────┼─────────────────┐
       ▼                 ▼                 ▼
     GPT              Claude           Gemini
       │                 │                 │
       └─────────────────┼─────────────────┘
                         ▼
                  Local / Private
                     Models
```

## Core Principles

1. **Memory Gateway is the product boundary.** Client/agent → PML Gateway → model provider. The gateway controls retrieval, writes, authorization, tenant isolation, model adapters, observability, and export.

2. **Portability is first-class.** A provider adapter interface (`ModelAdapter → GPT / Claude / Gemini / local / future`) lets you add providers without changing the memory system.

3. **Memory lives outside the model.** Three complementary stores:
   - **Structured State Store** — authoritative current state
   - **Semantic Memory Store** — searchable historical knowledge
   - **Event/Milestone Store** — decisions, changes, and important events
   - The model retrieves only what it needs.

4. **Security is architecture, not marketing.** Tenant isolation → authentication → authorization → scoped session credentials → encryption → audit trail → key management. **Honest statement:** once plaintext memory is deliberately sent to an external model provider, that provider can process that plaintext. This transparency is a feature, not a bug.

5. **Export and self-host.** "Your memory belongs to you." Customers can export memory using a documented schema and migrate to another Amnesia deployment or self-hosted environment.

## Scaling Claim (Corrected)

PML decouples per-turn context consumption from the total historical memory corpus by retrieving only context relevant to the current operation.

Retrieval cost can still vary with retrieved context size, query complexity, reranking, and model/tool calls. We do not claim "roughly constant per-turn cost" as an absolute guarantee.

## Documentation

- [PML Core Specification](docs/PML_SPECIFICATION.md)
- [12-Month Technical Roadmap](docs/ROADMAP.md)
- [Security Architecture](docs/SECURITY.md)

## License

MIT License — © 2026 Eric Marlon Echols
