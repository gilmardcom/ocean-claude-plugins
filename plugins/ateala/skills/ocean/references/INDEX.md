# Reference Index

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai

Fallback map of every reference file. Prefer the routing table in `SKILL.md`.

| File | Covers |
| --- | --- |
| [dsl/QUICK.md](dsl/QUICK.md) | One card per DSL page with section titles and a short syntax excerpt. Read first for orientation. |
| [dsl/INDEX.md](dsl/INDEX.md) | Index of the per-section DSL references (one file each). |
| [examples/INDEX.md](examples/INDEX.md) | Catalog of complete example projects with full DSL source. |
| [getting-started.md](getting-started.md) | What Ocean is, setup, and the first end-to-end walkthrough. |
| [concepts.md](concepts.md) | Core Ocean concepts: bundle, control structure, repository, system design model. |
| [architecture.md](architecture.md) | How the Ocean platform and Ocean-Atlas are structured. |
| [patterns.md](patterns.md) | Reusable, proven designs (FSMs, CRUD, request-response, approval workflows, secrets). |
| [troubleshooting.md](troubleshooting.md) | Symptoms, causes and resolutions. Coverage is still limited. |
| [glossary.md](glossary.md) | Canonical Ocean terminology. |
| [decisions.md](decisions.md) | ADR/RFC digest and the roadmap (planned, not yet available). |

## Getting Started

File: [getting-started.md](getting-started.md)

- Getting Started — A compact, browser-based onboarding path for understanding Ocean, running the onboarding example in Ocean App, and learning how its Bundle drives generation.
- What Is Ocean? — A short, conceptual introduction to Ocean, its workflow, and the roles in its ecosystem — enough to start, with links to the full concepts.
- Prerequisites — Ocean requires no local CLI or toolchain: modeling, validation, and generation happen in Ocean App.
- Run the First Example — Open, generate, and run “Onboarding: Task Manager” entirely inside Ocean App.
- Understand the First Bundle — A guided reading of the Task Manager Bundle using Ocean's own Horizon, Voyage, and Implementation facets.
- Make the First Change — Edit the Bundle inside Ocean App, regenerate, and confirm one enum value reaching the API and every generated form that references it.
- The Everyday Workflow — The development loop you'll repeat for every Ocean change, who owns which files, essential terminology, and the errors beginners hit most.
- Next Steps — Goal-oriented routes into the rest of Ocean-Atlas, and which examples to try next by difficulty.

## Concepts

File: [concepts.md](concepts.md)

- Concepts — Curated entry point to Ocean's foundational system model, project boundary, behavioral composition, and reusable-definition concepts.
- Control Structure — Explains how Ocean services, components, FSMs, and expressions form a layered behavioral composition model.
- Ocean Bundle — Defines the Ocean Bundle — the complete, identifiable Ocean DSL project that groups a system's DSL definitions under one project-level identity.
- Ocean Repository — Defines the Ocean Repository and local Pre-baked Registry as versioned sources of reusable definitions, together with the canonical `O.` and `P.` reference formats used to identify them.
- Ocean System Design Model — Explains how Ocean models software systems through technology-independent DSL constructs for data, behavior, interfaces, persistence, messaging, orchestration, configuration, and deployment.

## Architecture

File: [architecture.md](architecture.md)

- Architecture — Landing page for Ocean platform architecture — how its major layers relate, and where to continue for each architectural concern.
- Ocean-Atlas Knowledge API — Reference for the Ocean-Atlas Knowledge API — a private, read-only runtime HTTP service exposing navigation, documents, examples, keyword and semantic search, taxonomy, relationships, and version information from builders/core's normalized AtlasModel and a Postgres/pgvector knowledge index.

## Patterns

File: [patterns.md](patterns.md)

- Patterns — Browsable catalogue of reusable Ocean modeling patterns — recurring design problems, the recommended DSL structure, and where each is demonstrated.
- Approval Workflow — Model human approve/reject decisions as FSM transitions that record the deciding actor and decision data in the same transition handler.
- Asynchronous Request-Response — Answer a synchronous caller through typed broker request-response topics with explicit timeouts instead of direct API calls.
- Database-Backed CRUD — Connect a public API directly to a persistent entity's queries and commands, using expressions only where a rule must run before storage.
- Entity-Backed FSM — Govern a persistent entity's lifecycle through an FSM that defines the permitted transitions for one controlled field.
- In-Memory FSM — Model transient, mode-dependent behavior with an FSM that has no persisted entity, using `in-memory` states and a service-local context.
- Secure Secret Access — Give a service typed, named access to secret values through focused vaults composed into one primary vault, without ever writing secret values into the DSL.
- Synchronous API Composition — Compose several synchronous API results into one coordinated answer using a coordinating service and `aggregate`.

## Troubleshooting

File: [troubleshooting.md](troubleshooting.md)

- Troubleshooting — Entry point for diagnosing and resolving common Ocean problems.

## Glossary

File: [glossary.md](glossary.md)

- Glossary — Canonical terminology layer for Ocean — short, disambiguated definitions with aliases and cross-links for core platform, architecture, DSL, and learning-model terms.

## Decisions and Roadmap

File: [decisions.md](decisions.md)

- Atlas Builder Boundary — Establishes a technology-independent Atlas Builder Core between canonical Ocean-Atlas knowledge and presentation-specific builders such as Astro and Starlight.
- Canonical Markdown and Structured Build Model — Ocean-Atlas knowledge is authored as plain Markdown files with a structured `## Metadata` block and typed relationships, discovered and validated by a build-time pipeline rather than stored in a database or CMS.
- Example Knowledge Facets: Horizon, Voyage, and Implementation — Every Ocean example is published as three distinct modeled facets — Horizon, Voyage, and Implementation — backed by separate explanation and source responsibilities so each can be read and reused independently.
- External Examples Repository Integration — Runnable Ocean examples live in the separate `ocean-examples` repository, consumed by Ocean-Atlas as a git submodule and published through a dedicated example pipeline rather than as ordinary canonical Markdown.
- Knowledge API Service — Ocean-Atlas exposes a private, read-only runtime HTTP API over builders/core's normalized AtlasModel, reachable only from other Ocean services on Fly's private network, superseding the static-artifact approach proposed in rfc.versioned-atlas-model now that a concrete consumer set exists.
- Knowledge Chunking and Indexing Pipeline — Ocean-Atlas chunks canonical knowledge along heading boundaries with stable, ID-derived chunk identity, and keeps the resulting embeddings current through a versioned, lock-safe background reconciliation — published atomically in one transaction only after all required embedding calls succeed — that adds and removes chunks incrementally rather than re-embedding the whole corpus on every process start.
- Search Today and Structured Retrieval Tomorrow — Ocean-Atlas uses Starlight's Pagefind-based static search with Atlas-specific query and example indexing enhancements, while future retrieval or RAG must consume Core's structured `AtlasModel` rather than rendered HTML.
- Stable Knowledge Identity Independent of Filesystem Layout — Every canonical knowledge item carries an author-assigned, immutable stable ID that relationships reference directly, kept distinct from its source file path and its derived presentation route.
- Vector Storage and Embedding Provider — Ocean-Atlas stores knowledge embeddings in a dedicated database on the existing shared Fly Postgres cluster using the pgvector extension, and generates them with Voyage AI's voyage-4 model (1024 dimensions) behind a provider-independent embedding interface.
- Publish a Versioned AtlasModel for Search, APIs, and RAG — Publish a safe, versioned JSON projection of Core's `AtlasModel` so search, future APIs, and RAG ingestion share one supported structured distribution format without exposing internal build fields.
- Roadmap — Directional overview of Ocean's planned capability streams — language, generation, AI assistance, reuse, UI, new DSL domains, hardware, and AI-enhanced generation — organized into roadmap horizons.
