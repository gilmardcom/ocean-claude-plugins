# Ocean-Atlas Reference Overview

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/reference-overview/

**Contents**

- [1. Overview](#1-overview)
- [2. Start Here](#2-start-here)
- [3. DSL Reference Catalog](#3-dsl-reference-catalog)
- [4. DSL Directives](#4-dsl-directives)
- [5. Concepts](#5-concepts)
- [6. Reusable References](#6-reusable-references)
- [7. Behavioral Composition](#7-behavioral-composition)
- [8. Presentation Composition](#8-presentation-composition)
- [9. Deployment Model](#9-deployment-model)
- [10. File Structure](#10-file-structure)
- [11. File Extensions](#11-file-extensions)
- [12. Case Sensitivity](#12-case-sensitivity)
- [13. Comments](#13-comments)
- [14. Document Authority](#14-document-authority)
- [15. Suggested Reading Paths](#15-suggested-reading-paths)
- [16. Canonical Inventory](#16-canonical-inventory)
- [17. Maintenance](#17-maintenance)

Consolidated overview of Ocean DSL references, directives, foundational concepts, identifiers, and maintenance conventions.

Source: https://atlas.ocean-lab.ai/reference-overview/

Status: Approved · Authority: Authoritative

## 1. Overview

Ocean-Atlas is the canonical knowledge base for the Ocean platform and its domain-specific language.

It contains:

- normative DSL references;
- supporting DSL catalogs;
- foundational concepts;
- shared syntax and metadata rules;
- relationships between language constructs;
- guidance for understanding and evolving Ocean.

Ocean-Atlas documents the language and platform. It is distinct from the Ocean Repository, which contains reusable machine-consumable Ocean definitions.

---

## 2. Start Here

| Document                                           | ID                          | Purpose                                                                 |
| -------------------------------------------------- | --------------------------- | ----------------------------------------------------------------------- |
| [System Design Model](../concepts.md#ocean-system-design-model) | `concept.system-design-model` | Explains how Ocean models software systems through technology-independent DSL constructs |
| [Ocean Repository](../concepts.md#ocean-repository)   | `concept.ocean-repository`  | Defines reusable Ocean items and the `O.` and `P.` reference model      |
| [Control Structure](../concepts.md#control-structure) | `concept.control-structure` | Explains how services, components, FSMs, and expressions work together  |
| [Metadata](metadata.md)                        | `dsl.metadata`              | Defines shared metadata available across Ocean definitions              |

---

## 3. DSL Reference Catalog

Most entries below are top-level DSL sections declared through an `@<section>` header. The user-interface family also includes the nested `Widget` construct and its supporting predefined-type catalog; their kind is identified explicitly.

### Project and shared models

| Section                        | ID             | Status   | Purpose                                                                    |
| ------------------------------ | -------------- | -------- | -------------------------------------------------------------------------- |
| [`@info`](dsl/info.md)         | `dsl.info`     | Approved | Declares project-level name, version, and description                      |
| [`@datatype`](dsl/datatype.md) | `dsl.datatype` | Approved | Defines reusable types, fields, enums, and type relationships              |
| [`@context`](dsl/context.md)   | `dsl.context`  | Approved | Defines typed contextual information available to supported runtime scopes |
| [`@config`](dsl/config.md)     | `dsl.config`   | Approved | Defines reusable typed runtime-configuration schemas                       |
| [`@vault`](dsl/vault.md)       | `dsl.vault`    | Approved | Defines logical vaults and typed secret schemas                            |

### Interfaces and infrastructure

| Section                              | ID                | Status   | Purpose                                                                      |
| ------------------------------------ | ----------------- | -------- | ---------------------------------------------------------------------------- |
| [`@api`](dsl/api.md)                 | `dsl.api`         | Approved | Defines typed API contracts and operations                                   |
| [`@broker`](dsl/broker.md)           | `dsl.broker`      | Approved | Defines brokers, topics, and messaging patterns                              |
| [`@database`](dsl/database.md)       | `dsl.database`    | Approved | Defines database models, entities, queries, and commands                     |
| [`@integration`](dsl/integration.md) | `dsl.integration` | Accepted | Defines typed, directional integration flows between contracts and resources |

### Behavior and composition

| Section                            | ID               | Status   | Purpose                                                                   |
| ---------------------------------- | ---------------- | -------- | ------------------------------------------------------------------------- |
| [`@expression`](dsl/expression.md) | `dsl.expression` | Approved | Defines reusable stateless and side-effect-free logic                     |
| [Authorization policies](authorization.md) | `dsl.authorization` | Approved | Defines declarative `authorize` policies attached to expressions and connections |
| [`@fsm`](dsl/fsm.md)               | `dsl.fsm`        | Approved | Defines stateful behavior using states, events, and transitions           |
| [`@component`](dsl/component.md)   | `dsl.component`  | Approved | Defines stateless composition of FSMs, expressions, and nested components |
| [`@service`](dsl/service.md)       | `dsl.service`    | Approved | Defines deployable orchestration boundaries and system connections        |

### User interfaces

| Construct                                                 | ID                            | Kind                 | Status   | Purpose                                                            |
| --------------------------------------------------------- | ----------------------------- | -------------------- | -------- | ------------------------------------------------------------------ |
| [`@dashboard`](dsl/dashboard.md)                          | `dsl.dashboard`               | Section              | Approved | Defines reusable backend-agnostic pages composed of widgets        |
| [`Widget`](dsl/widget.md)                                 | `dsl.widget`                  | Nested construct     | Approved | Defines named UI building blocks inside dashboards                 |
| [Predefined Widget Types](widget-predefined-types.md) | `dsl.widget-predefined-types` | Supporting reference | Approved | Defines built-in widget types, properties, defaults, and functions |
| [`@ui`](dsl/ui.md)                                        | `dsl.ui`                      | Section              | Approved | Defines deployable presentation services and backend wiring        |

The presentation hierarchy is:

```text
@ui
└── uses @dashboard
    └── contains Widget
        └── uses predefined widget types
```

### Deployment

| Section                    | ID           | Status   | Purpose                                                 |
| -------------------------- | ------------ | -------- | ------------------------------------------------------- |
| [`@deploy`](dsl/deploy.md) | `dsl.deploy` | Approved | Defines environment-specific deployment materialization |

---

## 4. DSL Directives

Directives extend the scope or materialization behavior of a DSL file. They are not independent top-level section types.

| Directive                      | ID             | Status   | Scope extended        | Purpose                                                              |
| ------------------------------ | -------------- | -------- | --------------------- | -------------------------------------------------------------------- |
| [`@import`](dsl/import.md)     | `dsl.import`   | Approved | Dependency scope      | Imports a selected reusable definition under a local alias           |
| [`@include`](dsl/include.md)   | `dsl.include`  | Approved | Model scope           | Statically composes a compatible same-section definition             |
| [`@asset`](dsl/asset.md)       | `dsl.asset`    | WIP      | Filesystem scope      | Materializes an opaque external file without semantic transformation |
| [`@artifact`](dsl/artifact.md) | `dsl.artifact` | Approved | Materialization scope | Interprets, transforms, and materializes a typed semantic file       |

Conceptually:

```text
@import   → selected reusable definition
@include  → complete compatible DSL section
@asset    → opaque external file
@artifact → type-aware external semantic file
```

---

## 5. Concepts

Concept documents explain cross-cutting Ocean models that span multiple DSL references.

| Concept                                            | ID                          | Status   | Purpose                                                                              |
| -------------------------------------------------- | --------------------------- | -------- | ------------------------------------------------------------------------------------ |
| [System Design Model](../concepts.md#ocean-system-design-model) | `concept.system-design-model` | Approved | Explains how Ocean models software systems through technology-independent DSL constructs |
| [Ocean Repository](../concepts.md#ocean-repository)   | `concept.ocean-repository`  | Approved | Defines reusable-definition sources, identity, versioning, publishing, and discovery |
| [Control Structure](../concepts.md#control-structure) | `concept.control-structure` | Approved | Defines the roles of services, components, FSMs, and expressions                     |

---

## 6. Reusable References

Ocean uses two qualified reference spaces:

```text
O.<namespace>.<name>@<version>
P.<namespace>.<name>@<version>
```

Where:

- `O.` identifies a reusable item from the Ocean Repository;
- `P.` identifies a local Pre-baked item;
- `namespace` provides the item's logical hierarchy;
- `name` identifies the item;
- `version` selects its version.

The source prefix is part of the item's identity. Complete semantics are defined by [`concept.ocean-repository`](concepts/ocean-repository.md).

---

## 7. Behavioral Composition

Ocean separates deployment, composition, state, and stateless logic:

```text
Service     → deployable orchestration boundary
Component   → stateless behavioral composition
FSM         → stateful behavior and transitions
Expression  → stateless logic and transformation
```

See [Control Structure](../concepts.md#control-structure) for the complete model.

---

## 8. Presentation Composition

Ocean UI definitions are composed as follows:

```text
@ui
  ├── technology and configuration
  ├── navigation and page chrome
  ├── uses @dashboard
  │       └── contains Widget
  └── connects widget functions to APIs and brokers
```

Relevant references:

- [`dsl.ui`](dsl/ui.md);
- [`dsl.dashboard`](dsl/dashboard.md);
- [`dsl.widget`](dsl/widget.md);
- [`dsl.widget-predefined-types`](dsl/widget-predefined-types.md).

---

## 9. Deployment Model

Logical deployable definitions and environment-specific deployment are separate:

```text
@service / @ui / @broker
             │
             ▼
          @deploy
             │
             ▼
orchestrator and runtime artifacts
```

See [`dsl.deploy`](dsl/deploy.md) for replicas, configuration bindings, ports, environment values, volumes, dependencies, brokers, and deployment connections.

---

## 10. File Structure

An Ocean DSL file uses one top-level section header:

```ocean-dsl
@<section>
```

Examples:

```ocean-dsl
@datatype
```

```ocean-dsl
@service
```

```ocean-dsl
@deploy
```

Section parameters or metadata may be supported where defined by the applicable reference.

---

## 11. File Extensions

Recognized Ocean DSL file extensions include:

```text
.ocean
.ocn
```

The canonical extension for a project may be narrowed by tooling or repository conventions.

---

## 12. Case Sensitivity

Ocean DSL keywords, identifiers, names, types, properties, aliases, and references are case-sensitive unless a specific reference explicitly states otherwise.

For example:

```text
OrderService ≠ orderService
Table        ≠ table
Submit       ≠ submit
```

Authors should use the exact casing defined by the applicable DSL contract.

---

## 13. Comments

Comments begin with `#` and continue to the end of the line.

Full-line comment:

```ocean-dsl
# Route newly created orders into the workflow.
```

End-of-line comment:

```ocean-dsl
connect api.createOrder -> flow.create  # Start the order flow.
```

`#` inside a quoted string is treated as literal content:

```ocean-dsl
url: "https://example.com/#/reset"  # Valid comment after the value.
```

Inline comments inserted between tokens and multi-line block comments are not supported.

---

## 14. Document Authority

Ocean-Atlas uses document metadata to communicate authority and maturity.

Current status convention:

- all established DSL references and concepts are **Approved**;
- `dsl.asset` remains **WIP**.

Normative DSL references govern syntax, validation, and semantics for their constructs. Concept documents explain cross-cutting models without replacing the normative grammar.

---

## 15. Suggested Reading Paths

### Learning the language

1. [System Design Model](../concepts.md#ocean-system-design-model) and the [DSL overview](overview.md)
2. [Metadata](metadata.md)
3. [Datatype](datatype.md)
4. [Import](import.md) and [Include](include.md)
5. [Control Structure](../concepts.md#control-structure)
6. [Expression](expression.md), [FSM](fsm.md), [Component](component.md), and [Service](service.md)

### Building backend systems

1. [Datatype](datatype.md)
2. [API](api.md), [Broker](broker.md), and [Database](database.md)
3. [Expression](expression.md), [FSM](fsm.md), and [Component](component.md)
4. [Service](service.md)
5. [Config](config.md) and [Vault](vault.md)
6. [Deploy](deploy.md)

### Building user interfaces

1. [Datatype](datatype.md)
2. [Widget](widget.md) and [Predefined Widget Types](widget-predefined-types.md)
3. [Dashboard](dashboard.md)
4. [UI](ui.md)
5. [Config](config.md)
6. [Deploy](deploy.md)

---

## 16. Canonical Inventory

### DSL IDs

```text
dsl.info
dsl.metadata
dsl.datatype
dsl.context
dsl.config
dsl.vault
dsl.api
dsl.broker
dsl.database
dsl.integration
dsl.expression
dsl.authorization
dsl.fsm
dsl.component
dsl.service
dsl.dashboard
dsl.widget
dsl.widget-predefined-types
dsl.ui
dsl.deploy
dsl.import
dsl.include
dsl.asset
dsl.artifact
```

### Concept IDs

```text
concept.system-design-model
concept.ocean-repository
concept.control-structure
```

---

## 17. Maintenance

When adding or changing an Ocean-Atlas document:

- assign a stable ID;
- place DSL contracts in `dsl/`;
- place cross-cutting concepts in `concepts/`;
- declare semantic relationships in metadata;
- link the document from this index;
- avoid legacy numeric filename prefixes;
- keep examples aligned with the implemented grammar;
- distinguish implemented rules from future considerations;
- update related documents when an ID or contract changes.
