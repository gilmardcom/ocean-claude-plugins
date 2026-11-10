# Glossary

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/glossary/

**Contents**

- [1. Overview](#1-overview)
- [2. How the Glossary Differs from Other Knowledge](#2-how-the-glossary-differs-from-other-knowledge)
- [3. Editorial Rules](#3-editorial-rules)
- [4. Browse by Domain](#4-browse-by-domain)
- [5. Entry Format](#5-entry-format)
- [6. Glossary A-Z](#6-glossary-a-z)
- [7. Roadmap](#7-roadmap)

## Glossary

Canonical terminology layer for Ocean — short, disambiguated definitions with aliases and cross-links for core platform, architecture, DSL, and learning-model terms.

Source: https://atlas.ocean-lab.ai/glossary/

Status: Accepted · Authority: Authoritative

### 1. Overview

The Glossary provides short, canonical definitions for terms that appear
across concepts, DSL references, architecture, and examples — for example
Bundle, Horizon, Voyage, System Design Model, Ocean Repository, Perspective,
Control Structure, Service, Component, Broker, Vault, and FSM.

It is a terminology layer, not a duplicate of the pages it links to. Each
entry gives a reader — or a search and retrieval system — enough to
disambiguate a term at a glance, then sends them to the page that owns the
full meaning.

This matters most for search and retrieval-augmented generation: glossary
entries normalize terminology and disambiguation, so a query for "FSM",
"finite state machine", or "Voyage" resolves to one specific Ocean meaning
instead of a generic industry one.

---

### 2. How the Glossary Differs from Other Knowledge

| Knowledge type | Purpose |
| --- | --- |
| **Glossary entry** | Quick, authoritative definition of a term — one or two sentences, plus aliases and disambiguation. |
| **Concept page** | Deeper explanation: relationships, rationale, and examples. |
| **DSL reference** | Precise language syntax and semantics. |
| **Architecture page** | How major components and layers relate, and where to continue. |

When a term needs more than a short definition, the glossary entry links out
rather than growing to cover that ground itself.

---

### 3. Editorial Rules

This page follows a fixed set of rules so it stays a reliable terminology
layer rather than a second copy of other pages:

1. **One canonical meaning per term.** Every Ocean-specific term has exactly one preferred definition on this page.
2. **Short definitions.** Usually one or two sentences. Detailed explanation belongs on the linked concept or reference page.
3. **Canonical links.** Every entry points to the page that owns the full meaning; the glossary summarizes, it does not redefine.
4. **Explicit, unambiguous aliases.** Synonyms, abbreviations, and former names are recorded only when they can safely resolve to one Ocean term. Generic search phrases are not aliases.
5. **Disambiguation.** Terms that also have a common software meaning — component, service, context, bundle, repository, config — state their Ocean-specific meaning under "Not to be confused with."
6. **Related terms.** Entries link to related terms so a reader can navigate the terminology graph, not just one definition.
7. **Stable anchors.** Every term has a stable heading anchor (for example `/glossary/#system-design-model`) that search results and citations can target directly.
8. **Alphabetical and domain navigation.** [Section 4](#4-browse-by-domain) groups terms by domain; [Section 6](#6-glossary-a-z) lists every term alphabetically.
9. **Search normalization.** Recorded aliases such as "FSM" / "finite state machine" are meant to resolve to the same entry regardless of which form is searched.
10. **Status synchronization.** An entry's status mirrors its canonical reference and must be updated whenever that reference changes status.

An additional rule — automated validation of duplicate terms, duplicate aliases,
ambiguous alias mappings, and broken canonical links — is intended for the
builder pipeline once the glossary grows past one hand-maintained page. That
validation, a structured per-term data model, and generated alphabetical and
domain views are roadmap work, not current behavior; see
[Section 7](#7-roadmap).

---

### 4. Browse by Domain

**Platform** — [Generator](#generator) · [Ocean](#ocean) · [Ocean App](#ocean-app) · [Ocean-Atlas](#ocean-atlas) · [Ocean DSL](#ocean-dsl) · [Ocean-Lab](#ocean-lab) · [Ocean Repository](#ocean-repository)

**Architecture** — [Bundle](#bundle) · [Control Structure](#control-structure) · [System Design Model](#system-design-model)

**Learning Model** — [Example](#example) · [Horizon](#horizon) · [Implementation](#implementation) · [Voyage](#voyage)

**DSL** — [API](#api) · [Artifact](#artifact) · [Asset](#asset) · [Broker](#broker) · [Component](#component) · [Config](#config) · [Context](#context) · [Dashboard](#dashboard) · [Database](#database) · [Datatype](#datatype) · [Definition](#definition) · [Deploy](#deploy) · [Engine](#engine) · [Expression](#expression) · [FSM](#fsm) · [Import](#import) · [Include](#include) · [Info](#info) · [Integration](#integration) · [Metadata](#metadata) · [Perspective](#perspective) · [Predefined Widget Types](#predefined-widget-types) · [Section](#section) · [Service](#service) · [Tag](#tag) · [UI](#ui) · [Vault](#vault) · [Widget](#widget)

---

### 5. Entry Format

Each entry below follows the same shape:

```md
### Term

- **Domain:** ...
- **Aliases:** ... (only when aliases exist)
- **Definition:** One or two sentences — the Ocean-specific meaning.
- **Not to be confused with:** ... (only when disambiguation is needed)
- **Related:** other glossary terms
- **Canonical reference:** the page that owns the full meaning
- **Status:** the current status of the canonical reference
```

Sections that do not apply to a given term (typically Aliases or "Not to be
confused with") are omitted rather than left empty.

---

### 6. Glossary A-Z

#### API

- **Domain:** DSL
- **Definition:** The technology-independent `@api` section defining a service's public operations, payload representations, and transport configuration.
- **Related:** [Service](#service), [Integration](#integration), [UI](#ui)
- **Canonical reference:** [API DSL Reference](dsl/api.md)
- **Status:** Accepted

#### Artifact

- **Domain:** DSL
- **Definition:** The `@artifact` directive for resolving, interpreting, transforming, and materializing an external semantic file as part of a generated project.
- **Not to be confused with:** [Asset](#asset), which attaches a file verbatim without interpretation or transformation.
- **Related:** [Asset](#asset), [Service](#service), [UI](#ui)
- **Canonical reference:** [Artifact Directive Reference](dsl/artifact.md)
- **Status:** Approved

#### Asset

- **Domain:** DSL
- **Definition:** The `@asset` directive for attaching an external file to a DSL element, copied verbatim into the generated project without interpretation.
- **Not to be confused with:** [Artifact](#artifact), which is interpreted and transformed rather than copied verbatim.
- **Related:** [Artifact](#artifact), [Service](#service), [UI](#ui)
- **Canonical reference:** [Asset DSL Reference](dsl/asset.md)
- **Status:** WIP

#### Broker

- **Domain:** DSL
- **Definition:** The `@broker` section defining message brokers, topics, message payloads, messaging patterns, and delivery semantics.
- **Related:** [Service](#service), [Integration](#integration)
- **Canonical reference:** [Broker DSL Reference](dsl/broker.md)
- **Status:** Accepted

#### Bundle

- **Domain:** Architecture
- **Aliases:** Ocean DSL project
- **Definition:** The complete, identifiable Ocean DSL project — the full set of local and reused DSL definitions that together describe one system, identified by its `@info` section.
- **Not to be confused with:** a bundler or packaging build output in general software tooling; an Ocean Bundle is the DSL project itself, not a build artifact.
- **Related:** [System Design Model](#system-design-model), [Ocean Repository](#ocean-repository), [Import](#import), [Include](#include), [Service](#service)
- **Canonical reference:** [Ocean Bundle](concepts.md#ocean-bundle)
- **Status:** Approved

#### Component

- **Domain:** DSL
- **Definition:** The `@component` section for composing FSMs, expressions, and nested components through typed inputs, outputs, and explicit connections.
- **Not to be confused with:** a UI/frontend component; an Ocean Component composes behavioral logic, not presentation.
- **Related:** [FSM](#fsm), [Expression](#expression), [Service](#service), [Control Structure](#control-structure)
- **Canonical reference:** [Component DSL Reference](dsl/component.md)
- **Status:** Draft

#### Config

- **Domain:** DSL
- **Aliases:** Configuration
- **Definition:** The `@config` section for declaring named, reusable, typed configuration schemas used by deployable services.
- **Related:** [Vault](#vault), [Service](#service)
- **Canonical reference:** [Config DSL Reference](dsl/config.md)
- **Status:** Approved

#### Context

- **Domain:** DSL
- **Definition:** The `@context` section for service-local volatile shared state used by service logic and orchestration.
- **Not to be confused with:** "context" in general programming (request context, dependency-injection context); Ocean Context is a declared, typed, service-local state section.
- **Related:** [Service](#service), [Config](#config)
- **Canonical reference:** [Context DSL Reference](dsl/context.md)
- **Status:** Accepted

#### Control Structure

- **Domain:** Architecture
- **Definition:** How Ocean services, components, FSMs, and expressions form a layered behavioral composition model.
- **Not to be confused with:** control-flow statements (if/loop) in general programming; Ocean's Control Structure is the layering of behavioral building blocks, not statement-level flow control.
- **Related:** [Service](#service), [Component](#component), [FSM](#fsm), [Expression](#expression)
- **Canonical reference:** [Control Structure](concepts.md#control-structure)
- **Status:** Approved

#### Dashboard

- **Domain:** DSL
- **Definition:** The `@dashboard` section for composing widgets into reusable, backend-agnostic pages or screens.
- **Not to be confused with:** [UI](#ui), which owns application-level integration and composes dashboards into a deployable user-interface service.
- **Related:** [Widget](#widget), [Predefined Widget Types](#predefined-widget-types), [UI](#ui)
- **Canonical reference:** [Dashboard DSL Reference](dsl/dashboard.md)
- **Status:** Approved

#### Database

- **Domain:** DSL
- **Definition:** The `@database` section defining persistent entities, relationships, keys, indexes, encryption, queries, and commands.
- **Related:** [Service](#service), [Context](#context), [Integration](#integration)
- **Canonical reference:** [Database DSL Reference](dsl/database.md)
- **Status:** Accepted

#### Datatype

- **Domain:** DSL
- **Aliases:** Data type
- **Definition:** The `@datatype` section defining reusable, technology-independent data contracts used across Ocean APIs, services, brokers, expressions, databases, and other constructs.
- **Related:** [API](#api), [Service](#service), [Database](#database), [Import](#import), [Include](#include)
- **Canonical reference:** [Datatype DSL Reference](dsl/datatype.md)
- **Status:** Accepted

#### Definition

- **Domain:** DSL
- **Definition:** One named item declared inside a top-level DSL [section](#section) — for example one `Service`, one `Datatype`, or one `FSM` — as opposed to the section itself.
- **Not to be confused with:** [Section](#section), the top-level block (`@service`, `@datatype`) that contains one or more definitions.
- **Related:** [Section](#section), [Bundle](#bundle)
- **Canonical reference:** [Ocean DSL](dsl/overview.md)
- **Status:** Accepted

#### Deploy

- **Domain:** DSL
- **Aliases:** Deployment
- **Definition:** The `@deploy` section defining how Ocean services, UIs, and brokers are materialized for a runtime environment, including replicas, configuration, ports, volumes, dependencies, and runtime connections.
- **Related:** [Service](#service), [UI](#ui), [Broker](#broker), [Config](#config)
- **Canonical reference:** [Deploy DSL Reference](dsl/deploy.md)
- **Status:** Approved

#### Engine

- **Domain:** DSL
- **Definition:** The concrete technology selected to implement a construct, declared through an `engine` attribute — for example `engine = postgres` on a database, or `engine = nats` on a broker.
- **Not to be confused with:** [Generator](#generator), the tool that produces implementation code; an engine is the selected target technology a generator produces code for.
- **Related:** [Generator](#generator), [Database](#database), [Broker](#broker)
- **Canonical reference:** [Ocean System Design Model](concepts.md#ocean-system-design-model)
- **Status:** Approved

#### Example

- **Domain:** Learning Model
- **Definition:** A validated, runnable Ocean project that demonstrates specific DSL capabilities end-to-end, cataloged with a Horizon and a Voyage.
- **Related:** [Horizon](#horizon), [Voyage](#voyage), [Bundle](#bundle)
- **Canonical reference:** [Examples](https://atlas.ocean-lab.ai/examples/)
- **Status:** Accepted

#### Expression

- **Domain:** DSL
- **Definition:** Ocean's stateless, side-effect-free reusable logic model — named or inline expressions with typed inputs and outputs.
- **Not to be confused with:** a math or regular expression; an Ocean Expression is a typed, reusable logic unit that may wrap built-in functions or external implementations.
- **Related:** [Component](#component), [FSM](#fsm), [Control Structure](#control-structure)
- **Canonical reference:** [Expression DSL Reference](dsl/expression.md)
- **Status:** Accepted

#### FSM

- **Domain:** DSL
- **Aliases:** Finite State Machine, finite-state machine
- **Definition:** The `@fsm` section for modeling stateful domain behavior through states, events, triggers, transitions, and actions.
- **Not to be confused with:** a generic finite-state-machine library or pattern; Ocean's FSM is a specific DSL section with entity-controlled and in-memory modes.
- **Related:** [Component](#component), [Control Structure](#control-structure), [Expression](#expression)
- **Canonical reference:** [FSM DSL Reference](dsl/fsm.md)
- **Status:** Accepted

#### Generator

- **Domain:** Platform
- **Definition:** A tool that translates a validated, technology-independent Ocean model into a concrete implementation for a selected [engine](#engine) or target technology.
- **Not to be confused with:** [Engine](#engine), the selected target technology; a generator is the tool, an engine is what it targets.
- **Related:** [Engine](#engine), [System Design Model](#system-design-model)
- **Canonical reference:** [Ocean System Design Model](concepts.md#ocean-system-design-model)
- **Status:** Approved

#### Horizon

- **Domain:** Learning Model
- **Definition:** The concise orientation view of an Ocean example — its scenario, architecture, demonstrated Ocean concepts, and expected result — described independently of implementation technology.
- **Not to be confused with:** [Voyage](#voyage), which teaches the step-by-step construction of the same example.
- **Related:** [Voyage](#voyage), [Example](#example), [Implementation](#implementation)
- **Canonical reference:** [Examples](https://atlas.ocean-lab.ai/examples/)
- **Status:** Accepted

#### Implementation

- **Domain:** Learning Model
- **Definition:** The source-inspection facet of an Ocean example, preserving its original folders and files so readers can inspect the complete executable model and its supporting artifacts.
- **Not to be confused with:** [Voyage](#voyage), which explains the construction process and decisions rather than presenting the source itself.
- **Related:** [Voyage](#voyage), [Horizon](#horizon), [Example](#example), [Bundle](#bundle)
- **Canonical reference:** [Examples](https://atlas.ocean-lab.ai/examples/)
- **Status:** Accepted

#### Import

- **Domain:** DSL
- **Definition:** The `@import` directive for resolving a selected reusable definition and exposing it through a local alias.
- **Not to be confused with:** [Include](#include), which statically composes a same-section definition rather than exposing it through an alias.
- **Related:** [Include](#include), [Ocean Repository](#ocean-repository), [Bundle](#bundle)
- **Canonical reference:** [Import DSL Reference](dsl/import.md)
- **Status:** Draft

#### Include

- **Domain:** DSL
- **Definition:** The `@include` directive for statically composing a same-section DSL definition from the Ocean Repository or local Pre-baked Registry.
- **Not to be confused with:** [Import](#import), which resolves a reusable definition into a local alias rather than statically composing it into the current file.
- **Related:** [Import](#import), [Ocean Repository](#ocean-repository), [Bundle](#bundle)
- **Canonical reference:** [Include DSL Reference](dsl/include.md)
- **Status:** Draft

#### Info

- **Domain:** DSL
- **Aliases:** Project information
- **Definition:** The `@info` section declaring the identity, version, and descriptive metadata of the complete Ocean DSL project.
- **Not to be confused with:** [Metadata](#metadata), which classifies individual DSL definitions and files rather than identifying the complete project.
- **Related:** [Bundle](#bundle), [Metadata](#metadata)
- **Canonical reference:** [Info DSL Reference](dsl/info.md)
- **Status:** Approved

#### Integration

- **Domain:** DSL
- **Definition:** The `@integration` section defining typed, directional flows between implemented contracts and used APIs, brokers, databases, files, and other supported resources.
- **Related:** [Service](#service), [API](#api), [Broker](#broker), [Database](#database)
- **Canonical reference:** [Integration](dsl/integration.md)
- **Status:** Accepted

#### Metadata

- **Domain:** DSL
- **Definition:** Generic classification attached to Ocean DSL definitions or files through item-level `@tags` and `@perspectives` and the optional file-level `group`.
- **Not to be confused with:** [Info](#info), which identifies and describes the complete Ocean DSL project.
- **Related:** [Tag](#tag), [Perspective](#perspective), [Info](#info)
- **Canonical reference:** [DSL Metadata Reference](dsl/metadata.md)
- **Status:** Accepted

#### Ocean

- **Domain:** Platform
- **Definition:** The overall Ocean platform and ecosystem name for DSL-first, technology-independent system modeling, reuse, and generation.
- **Not to be confused with:** [Ocean DSL](#ocean-dsl), which names the language specifically rather than the platform as a whole.
- **Related:** [Ocean DSL](#ocean-dsl), [Ocean-Atlas](#ocean-atlas), [Ocean-Lab](#ocean-lab), [Ocean Repository](#ocean-repository)
- **Canonical reference:** [Architecture](architecture.md#architecture-1)
- **Status:** Accepted

#### Ocean App

- **Domain:** Platform
- **Definition:** The Ocean-Lab application used to work with Ocean projects directly.
- **Related:** [Ocean-Lab](#ocean-lab), [Ocean-Atlas](#ocean-atlas)
- **Canonical reference:** [Ocean App](https://app.ocean-lab.ai)
- **Status:** External — no dedicated Ocean-Atlas page; described on the [Ocean-Atlas home page](https://atlas.ocean-lab.ai/).

#### Ocean DSL

- **Domain:** Platform
- **Aliases:** Ocean language
- **Definition:** Ocean's case-sensitive domain-specific language for modeling systems through typed, declarative, technology-independent definitions.
- **Not to be confused with:** [Ocean](#ocean), the platform name that also encompasses the Ocean Repository, generators and engine, Ocean-Atlas, and Ocean-Lab.
- **Related:** [Ocean](#ocean), [System Design Model](#system-design-model), [Bundle](#bundle)
- **Canonical reference:** [Ocean DSL](dsl/overview.md)
- **Status:** Accepted

#### Ocean Repository

- **Domain:** Architecture
- **Definition:** The versioned source of reusable Ocean definitions, referenced through the canonical `O.` (Ocean Repository) and `P.` (local Pre-baked Registry) formats.
- **Not to be confused with:** a source-code repository (for example a Git repository); the Ocean Repository specifically holds versioned, reusable DSL definitions.
- **Related:** [Bundle](#bundle), [Import](#import), [Include](#include)
- **Canonical reference:** [Ocean Repository](concepts.md#ocean-repository)
- **Status:** Approved

#### Ocean-Atlas

- **Domain:** Platform
- **Definition:** The canonical knowledge base for the Ocean platform and its domain-specific language.
- **Not to be confused with:** [Ocean-Lab](#ocean-lab), the broader Ocean application and ecosystem that Ocean-Atlas documents but does not implement.
- **Related:** [Ocean](#ocean), [Ocean-Lab](#ocean-lab)
- **Canonical reference:** [Ocean-Atlas](https://atlas.ocean-lab.ai/)
- **Status:** Approved

#### Ocean-Lab

- **Domain:** Platform
- **Aliases:** Ocean Lab
- **Definition:** The broader Ocean application and ecosystem in which generated systems and services are built and run.
- **Not to be confused with:** [Ocean-Atlas](#ocean-atlas), which documents Ocean but is a separate knowledge platform.
- **Related:** [Ocean](#ocean), [Ocean-Atlas](#ocean-atlas), [Ocean App](#ocean-app)
- **Canonical reference:** [Ocean-Lab](https://ocean-lab.ai)
- **Status:** External — no dedicated Ocean-Atlas page; described on the [Ocean-Atlas home page](https://atlas.ocean-lab.ai/).

#### Perspective

- **Domain:** DSL
- **Definition:** Context-dependent, key-value metadata (`@perspectives`) attached to a DSL item, such as version, lifecycle stage, or region.
- **Not to be confused with:** [Tag](#tag), which attaches free-form labels rather than key-value pairs; the legacy singular `perspective` keyword is not canonical syntax.
- **Related:** [Tag](#tag)
- **Canonical reference:** [Perspective DSL Reference](dsl/perspective.md)
- **Status:** Accepted

#### Predefined Widget Types

- **Domain:** DSL
- **Aliases:** Widget types
- **Definition:** The canonical catalog of built-in Ocean widget types and their implemented properties, defaults, and generated functions.
- **Related:** [Widget](#widget), [Dashboard](#dashboard), [UI](#ui)
- **Canonical reference:** [Predefined Widget Types](dsl/widget-predefined-types.md)
- **Status:** Approved

#### Section

- **Domain:** DSL
- **Definition:** One top-level DSL block — `@datatype`, `@api`, `@service`, and so on — that a source file declares exactly one of, containing one or more [definitions](#definition).
- **Not to be confused with:** [Definition](#definition), one named item declared inside a section.
- **Related:** [Definition](#definition), [Ocean DSL](#ocean-dsl)
- **Canonical reference:** [Ocean DSL](dsl/overview.md)
- **Status:** Accepted

#### Service

- **Domain:** DSL
- **Definition:** The `@service` section composing domain behavior, external contracts, infrastructure, configuration, secrets, and typed connections into one deployable unit.
- **Not to be confused with:** a microservice in general architecture; an Ocean Service is a DSL composition unit that may or may not map one-to-one to a deployed microservice.
- **Related:** [Component](#component), [Bundle](#bundle), [Control Structure](#control-structure)
- **Canonical reference:** [Service DSL Reference](dsl/service.md)
- **Status:** Approved

#### System Design Model

- **Domain:** Architecture
- **Definition:** How Ocean DSL constructs for data, behavior, interfaces, persistence, messaging, orchestration, configuration, and deployment combine to model one technology-independent system.
- **Related:** [Bundle](#bundle), [Control Structure](#control-structure), [Ocean DSL](#ocean-dsl)
- **Canonical reference:** [Ocean System Design Model](concepts.md#ocean-system-design-model)
- **Status:** Approved

#### Tag

- **Domain:** DSL
- **Definition:** A free-form label (`@tags`) attached to a DSL item to classify it by concern, ownership, lifecycle, or visibility.
- **Not to be confused with:** [Perspective](#perspective), which attaches key-value metadata rather than free-form labels; the legacy singular `tag` keyword is not canonical syntax.
- **Related:** [Perspective](#perspective)
- **Canonical reference:** [Tag DSL Reference](dsl/tag.md)
- **Status:** Accepted

#### UI

- **Domain:** DSL
- **Aliases:** User Interface
- **Definition:** The `@ui` section composing dashboards into deployable user-interface services connected to APIs and brokers.
- **Related:** [Widget](#widget), [API](#api), [Broker](#broker)
- **Canonical reference:** [UI DSL Reference](dsl/ui.md)
- **Status:** Approved

#### Vault

- **Domain:** DSL
- **Aliases:** Secret Vault
- **Definition:** The `@vault` section for modeling logical secret stores, typed secret schemas, and physical secret-backend generation.
- **Not to be confused with:** a specific secret-backend product (for example HashiCorp Vault); Ocean's Vault is a technology-independent logical model that generates to a chosen backend.
- **Related:** [Config](#config), [Service](#service), [Perspective](#perspective), [Tag](#tag)
- **Canonical reference:** [Vault DSL Reference](dsl/vault.md)
- **Status:** Approved

#### Voyage

- **Domain:** Learning Model
- **Definition:** The step-by-step learning facet of an Ocean example, explaining how the system is constructed and why each implementation decision is made.
- **Not to be confused with:** [Horizon](#horizon), which explains the completed system without teaching its construction sequence.
- **Related:** [Horizon](#horizon), [Implementation](#implementation), [Example](#example)
- **Canonical reference:** [Examples](https://atlas.ocean-lab.ai/examples/)
- **Status:** Accepted

#### Widget

- **Domain:** DSL
- **Definition:** A declared element inside an Ocean dashboard, with a predefined type, scalar/list/map properties, datatype binding, and generated functions.
- **Related:** [UI](#ui)
- **Canonical reference:** [Widget DSL Reference](dsl/widget.md)
- **Status:** Approved

---

### 7. Roadmap

The current page completes the first two stages of a three-stage plan:

1. **Schema and editorial rules** (this page) — the term shape, disambiguation, alias, and cross-link conventions defined in [Section 3](#3-editorial-rules) and [Section 5](#5-entry-format).
2. **Foundational term set** (this page) — an initial term set spanning the platform, architecture, DSL, and learning-model domains.
3. **Builder validation and generated views** (not yet built) — a structured per-term data model, builder-level validation (duplicate terms, duplicate or ambiguous aliases, missing definitions, broken canonical links), and generated alphabetical/domain views, feeding future search, APIs, and RAG output.

Until stage three exists, this single page is the authoritative glossary, and
new terms should be added here as additional `### Term` entries rather than
as separate files.
