# Ocean DSL

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/

**Contents**

- [1. Overview](#1-overview)
- [2. Case Sensitivity](#2-case-sensitivity)
- [3. Source Files](#3-source-files)
- [4. Comments](#4-comments)
- [5. Canonical DSL References](#5-canonical-dsl-references)
- [6. Definitions Pending Migration](#6-definitions-pending-migration)
- [7. Directives](#7-directives)
- [8. Reserved and Future Sections](#8-reserved-and-future-sections)
- [9. Related Knowledge](#9-related-knowledge)

Entry point for the Ocean domain-specific language, its source-file rules, directives, section identifiers, and canonical references.

Source: https://atlas.ocean-lab.ai/dsl/

Status: Accepted · Authority: Normative

## 1. Overview

Ocean is a case-sensitive domain-specific language for modeling systems using
typed, declarative definitions.

An Ocean DSL source file declares one top-level DSL section, such as
`@datatype`, `@api`, `@broker`, or `@fsm`. Section-specific syntax and semantics
are defined by the canonical references linked from this index.

The broader architecture and Ocean package/import model will be documented in
the DSL overview and Ocean references as they are migrated into Ocean-Atlas.

---

## 2. Case Sensitivity

Ocean keywords, identifiers, names, and references are case-sensitive.

For example, these names are distinct:

```text
Order
order
ORDER
```

Authors must use the exact spelling and capitalization of declared names.

---

## 3. Source Files

Ocean DSL source files may use these extensions:

- `.ocn`
- `.ocean`

Every source file must begin with exactly one top-level section identifier:

```ocean-dsl
@<section>
```

Example:

```ocean-dsl
@datatype

PurchaseOrder
    id     : String
    status : OrderStatus
```

Directives such as `@include`, `@import`, `@asset`, and `@artifact` do not count
as additional top-level section identifiers.

---

## 4. Comments

Ocean supports comments beginning with `#` and continuing to the end of the
line.

### 4.1 Supported Forms

| Form | Example | Description |
| --- | --- | --- |
| Full-line | `# This is a comment` | The entire line is a comment. |
| End-of-line | `status : String # Current status` | The comment follows DSL syntax. |

Comments may be used throughout Ocean DSL files.

### 4.2 Rules

- A comment begins with `#` and extends to the end of its line.
- `#` inside a quoted string is a literal character rather than the start of a
  comment.
- A comment must not split a keyword or other lexical token.
- C-style multiline block comments such as `/* ... */` are not supported.

Valid:

```ocean-dsl
url : String (default="https://example.com/#/reset") # Reset endpoint
```

Invalid:

```ocean-dsl
connect#comment source -> target
```

---

## 5. Canonical DSL References

The following section references are currently available in Ocean-Atlas.

| Section | Reference | Purpose |
| --- | --- | --- |
| System model | [DSL System Design Model](description.md) | Explain how DSL sections combine into an architectural system model. |
| Metadata | [DSL Metadata](metadata.md) | Define item-level tags and perspectives and file-level groups. |
| `@datatype` | [Datatype](datatype.md) | Define reusable data contracts, enums, errors, envelopes, and interfaces. |
| `@context` | [Context](context.md) | Define volatile service-local shared state. |
| `@database` | [Database](database.md) | Define persistent entities, relationships, queries, and commands. |
| `@api` | [API](api.md) | Define public service interfaces. |
| `@broker` | [Broker](broker.md) | Define message brokers and topics. |
| `@expression` | [Expression](expression.md) | Define stateless reusable logic. |
| Scheduling and triggering | [Schedule and Trigger](schedule.md) | Define reusable logical schedules for executable items. |
| `@fsm` | [FSM](fsm.md) | Define finite-state machines and transitions. |
| `@component` | [Component](component.md) | Compose FSMs and nested components through typed connections. |

---

## 6. Definitions Pending Migration

These legacy DSL definitions do not yet have canonical Ocean-Atlas reference
pages. They will be migrated and reviewed individually.

| Section or concept | Planned reference | Purpose |
| --- | --- | --- |
| Ocean references and `@import` | `ocean.md` / `import.md` | Define registry references and imported definitions. |
| `@include` | `include.md` | Include definitions from another same-section source file. |
| `@asset` | `asset.md` | Declare opaque external files copied into generated output. |
| `@artifact` | `artifact.md` | Declare typed or transformable generation inputs. |
| Control structures | `control-structures.md` | Explain the expression, FSM, component, and service layers. |
| `@service` | `service.md` | Compose deployable services and external interfaces. |
| `@config` | `config.md` | Define typed reusable configuration schemas. |
| `@deploy` | `deploy.md` | Define deployment configuration and runtime wiring. |
| `@ui` | `ui.md` | Define deployable user-interface services. |
| `@dashboard` | `dashboard.md` | Define reusable UI pages composed of widgets. |
| Widget reference | `widget.md` | Define widget syntax and common properties. |
| Widget types | `widget-predefined-types.md` | Define the predefined widget catalog. |
| `@vault` | `vault.md` | Define vaults and secrets. |
| `@info` | `info.md` | Define project-level identification and documentation. |

---

## 7. Directives

Ocean directives extend the scope or materialization of the containing source
file.

| Directive | Scope extended | Purpose |
| --- | --- | --- |
| `@include` | Model scope | Statically include all definitions from another source file of the same section type. |
| `@import` | Dependency scope | Introduce one named external definition through a local alias. |
| `@asset` | Filesystem scope | Copy an opaque external file into generated output. |
| `@artifact` | Materialization scope | Interpret and transform an external artifact during generation. |

The canonical reference for each directive defines its placement, validation,
resolution, and type-compatibility rules.

### 7.1 Inclusion Summary

An include uses this form:

```ocean-dsl
@include <path.to.definition@version>
```

The including and included files must declare the same top-level section type.
For example, an `@datatype` file may include another `@datatype` file but must
not include an `@service` file.

Included definitions are made available as if they were declared in the
including file. Detailed transitive-inclusion, placement, and cycle rules belong
to the canonical `@include` reference.

---

## 8. Reserved and Future Sections

The legacy roadmap identifies these possible future sections:

- `@integration`
- `@repl`
- `@cli`
- access-control definitions such as `@role` or `@policy`
- testing definitions such as `@test` or `@scenario`

These names are not canonical DSL capabilities until their syntax and semantics
are accepted and documented.

---

## 9. Related Knowledge

- [Ocean-Atlas Reference Overview](reference-overview.md) — catalog of Ocean-Atlas knowledge and suggested reading paths.
- [Datatype DSL Reference](datatype.md) — canonical Ocean datatype and naming rules.
- [Expression DSL Reference](expression.md) — canonical stateless expression language.
- [FSM DSL Reference](fsm.md) — canonical state-machine model.
- [Component DSL Reference](component.md) — canonical component composition model.
