# Include DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/include/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax](#3-syntax)
- [4. Placement](#4-placement)
- [5. Static Inclusion](#5-static-inclusion)
- [6. Section-Type Compatibility](#6-section-type-compatibility)
- [7. Supported Section Types](#7-supported-section-types)
- [8. Ocean Repository References](#8-ocean-repository-references)
- [9. Local Pre-baked References](#9-local-pre-baked-references)
- [10. Version Resolution](#10-version-resolution)
- [11. Transitive Inclusion](#11-transitive-inclusion)
- [12. Cyclic Inclusion](#12-cyclic-inclusion)
- [13. Name Scope and Uniqueness](#13-name-scope-and-uniqueness)
- [14. Reference Resolution](#14-reference-resolution)
- [15. Validation](#15-validation)
- [16. Include vs Import](#16-include-vs-import)
- [17. Complete Example](#17-complete-example)
- [18. Invalid Examples](#18-invalid-examples)
- [19. Purpose](#19-purpose)
- [20. Rules and Constraints](#20-rules-and-constraints)
- [21. Related Knowledge](#21-related-knowledge)

Defines the Ocean `@include` directive for statically composing same-section Ocean DSL definitions from versioned Ocean Repository or local Pre-baked references.

Source: https://atlas.ocean-lab.ai/dsl/include/

Status: Draft · Authority: Normative — not final: content may be incomplete or change.

## 1. Overview

The `@include` directive composes reusable Ocean DSL definitions into the current DSL file.

Each Ocean DSL file defines exactly one section type through a top-level `@<type>` declaration, such as:

```ocean-dsl
@datatype
```

```ocean-dsl
@api
```

```ocean-dsl
@service
```

An Ocean DSL file may include other definitions of the **same section type**.

Included definitions are treated as if they were declared directly in the including file.

---

## 2. Core Principle

Inclusion is static, structural composition.

Conceptually:

```text
Current DSL file
       │
       ├── local definitions
       │
       └── @include reference
                   │
                   ▼
          reusable DSL definitions
                   │
                   ▼
          one combined DSL section
```

`@include` composes DSL definitions. It does not include implementation source code, generated artifacts, or arbitrary files.

---

## 3. Syntax

General syntax:

```ocean-dsl
@include <OceanRef>
```

An inclusion reference uses one of the canonical Ocean reference formats:

```text
O.<namespace>.<name>@<version>
P.<namespace>.<name>@<version>
```

Where:

- `O.` identifies an item in the Ocean Repository;
- `P.` identifies a local Pre-baked item;
- `namespace` identifies the logical hierarchy containing the item;
- `name` identifies the reusable item;
- `version` identifies the requested version.

Examples:

```ocean-dsl
@include O.common.baseTypes@1.1.0
```

```ocean-dsl
@include P.organization.auditTypes@1.0.0
```

---

## 4. Placement

`@include` directives must appear immediately after the top-level section declaration and before any local DSL definitions.

Example:

```ocean-dsl
@datatype

@include O.common.baseTypes@1.1.0
@include P.organization.auditTypes@1.0.0

User
    id    : String
    email : String
```

The following is invalid because a local definition appears before the inclusion:

```ocean-dsl
@datatype

User
    id : String

@include O.common.baseTypes@1.1.0
```

Multiple `@include` directives are allowed. Each directive is declared on its own line.

---

## 5. Static Inclusion

`@include` statically incorporates all DSL definitions from the resolved item into the current file.

For validation and model construction, the included definitions behave as though they were written inline.

Conceptually:

```ocean-dsl
@datatype

@include O.common.identityTypes@1.0.0

User
    identity : Identity
```

is processed as one logical datatype section containing both the included definitions and `User`.

Inclusion does not create a runtime dependency or dynamic lookup. Resolution occurs while the Ocean model is built.

---

## 6. Section-Type Compatibility

The including file and every included item must declare the same top-level DSL section type.

Examples:

- an `@datatype` file may include only `@datatype` items;
- an `@api` file may include only `@api` items;
- an `@expression` file may include only `@expression` items;
- an `@service` file may include only `@service` items.

Cross-section inclusion is invalid.

For example, an `@service` file cannot include an `@datatype` item:

```ocean-dsl
@service

@include O.common.baseTypes@1.1.0
```

If `O.common.baseTypes@1.1.0` resolves to an `@datatype` item, validation must fail with a section-type compatibility error.

---

## 7. Supported Section Types

`@include` applies to Ocean section types whose definitions can be composed as one logical section.

These may include:

- datatypes;
- APIs;
- databases;
- brokers;
- expressions;
- FSMs;
- components;
- services;
- other section types that explicitly support inclusion.

Support for `@include` does not relax the rules of the receiving section. The combined definitions remain subject to that section's grammar and semantic constraints.

---

## 8. Ocean Repository References

An `O.` reference resolves an item from the Ocean Repository.

Canonical format:

```text
O.<namespace>.<name>@<version>
```

Example:

```ocean-dsl
@include O.common.baseTypes@1.1.0
```

The resolved repository item must:

- exist;
- contain a valid Ocean DSL definition;
- have a compatible section type;
- satisfy the requested version;
- be accessible to the current project or environment.

---

## 9. Local Pre-baked References

A `P.` reference resolves a local Pre-baked item.

Canonical format:

```text
P.<namespace>.<name>@<version>
```

Example:

```ocean-dsl
@include P.organization.auditTypes@1.0.0
```

Pre-baked items are reusable definitions available through the local Ocean environment. They use the same inclusion semantics and section-type compatibility rules as Ocean Repository items.

The `P.` prefix is part of the item's identity. A `P.` reference must not be silently resolved as an `O.` reference, or vice versa.

---

## 10. Version Resolution

The inclusion reference identifies the version of the reusable item to resolve.

Example:

```ocean-dsl
@include O.common.baseTypes@1.1.0
```

Where supported by the Ocean reference model, `latest` may be used:

```ocean-dsl
@include O.common.baseTypes@latest
```

Resolution must be deterministic. The selected item and version must be established before the combined section is validated.

For reproducible models, an explicit immutable version is preferred over `latest`.

---

## 11. Transitive Inclusion

Inclusion is transitive.

If item A includes item B, and item B includes item C, then a file that includes A also receives the definitions from B and C.

Conceptually:

```text
Current file
    │
    └── includes A
            │
            └── includes B
                    │
                    └── includes C
```

The resulting logical section contains:

```text
local definitions + definitions from A + definitions from B + definitions from C
```

Every item in the transitive chain must have a section type compatible with the original including file.

---

## 12. Cyclic Inclusion

Cyclic inclusion is not allowed.

A direct cycle is invalid:

```text
A includes B
B includes A
```

An indirect cycle is also invalid:

```text
A includes B
B includes C
C includes A
```

Ocean must detect cycles while resolving the inclusion graph and report a validation error.

A cyclic graph must not be partially expanded or accepted.

---

## 13. Name Scope and Uniqueness

Included and local definitions share one logical section scope.

Names must therefore remain unique according to the rules of that section.

For example, if an included datatype item defines `User`, the including file cannot define another incompatible `User` in the same section.

```text
Included definitions ─┐
                      ├── one name scope
Local definitions ────┘
```

Name collisions introduced through direct or transitive inclusion are validation errors.

`@include` does not provide an alias and does not create a separate namespace inside the receiving file.

---

## 14. Reference Resolution

After inclusion, references are resolved across the combined logical section.

This allows a local definition to reference an included definition directly, and an included definition to reference other definitions available through its valid inclusion graph.

Resolution conceptually follows this sequence:

```text
Resolve O. and P. references
            ↓
Validate section compatibility
            ↓
Expand transitive inclusions
            ↓
Detect cycles
            ↓
Build combined section scope
            ↓
Resolve DSL references
            ↓
Run section validation
```

An unresolved, inaccessible, ambiguous, or incompatible inclusion reference is invalid.

---

## 15. Validation

After all inclusions have been resolved, the complete logical section is validated using the standard rules for its section type.

Validation includes, where applicable:

- inclusion reference resolution;
- source-prefix resolution;
- version resolution;
- section-type compatibility;
- cycle detection;
- global name uniqueness within the combined section;
- DSL reference resolution;
- unused-definition detection;
- section-specific syntax and semantic validation.

Errors may originate in local definitions, directly included definitions, or transitively included definitions.

---

## 16. Include vs Import

`@include` and `@import` are distinct reuse mechanisms.

### `@include`

- incorporates all definitions from a compatible same-section item;
- merges those definitions into the current logical section;
- does not assign a local alias;
- is intended for composition of cohesive sets of definitions.

### `@import`

- selects a specific reusable definition;
- exposes that definition through a local alias;
- may be used where the receiving DSL section permits the imported definition type;
- does not merge an entire same-section file into the local scope.

Conceptually:

```text
@include  → compose a complete compatible DSL section
@import   → bind a selected reusable definition to a local alias
```

The exact `@import` contract is defined by `dsl.import`.

---

## 17. Complete Example

```ocean-dsl
@datatype

@include O.common.baseTypes@1.1.0
@include P.organization.auditTypes@1.0.0

User
    id        : Identifier
    email     : String
    createdAt : AuditTimestamp
```

In this example:

- `O.common.baseTypes@1.1.0` is resolved from the Ocean Repository;
- `P.organization.auditTypes@1.0.0` is resolved from the local Pre-baked registry;
- both items must define `@datatype` sections;
- all included and local datatype definitions form one logical section;
- `Identifier` and `AuditTimestamp` may be referenced directly;
- the combined section is validated as a whole.

---

## 18. Invalid Examples

### Include after a local definition

```ocean-dsl
@datatype

User
    id : String

@include O.common.baseTypes@1.1.0
```

Invalid because all `@include` directives must appear before local DSL definitions.

### Cross-section inclusion

```ocean-dsl
@service

@include O.common.baseTypes@1.1.0
```

Invalid when the referenced item declares `@datatype`.

### Cyclic inclusion

```text
O.example.a@1.0.0 includes P.example.b@1.0.0
P.example.b@1.0.0 includes O.example.a@1.0.0
```

Invalid because the source prefix does not affect cycle detection. The complete inclusion graph is evaluated as one graph.

### Duplicate definition

```text
O.example.customerTypes@1.0.0 defines Customer
P.organization.sharedTypes@2.0.0 defines Customer
```

Invalid when both are included into the same logical section and the section requires globally unique names.

---

## 19. Purpose

`@include` enables modular reuse of cohesive Ocean DSL definitions while preserving strict section boundaries.

It supports:

- splitting large sections into reusable modules;
- sharing complete groups of related definitions;
- composing repository and local Pre-baked content;
- deterministic versioned reuse;
- transitive composition;
- validation of the final model as one logical section.

This avoids copying reusable definitions between projects while keeping their resulting DSL structure explicit and validatable.

---

## 20. Rules and Constraints

The following rules apply:

- Each Ocean DSL file declares exactly one top-level section type.
- `@include` statically incorporates all definitions from the referenced item.
- Inclusion references use either `O.<namespace>.<name>@<version>` or `P.<namespace>.<name>@<version>`.
- `O.` identifies an Ocean Repository item.
- `P.` identifies a local Pre-baked item.
- The source prefix is part of the reference identity.
- The including and included items must declare the same section type.
- Cross-section inclusion is not allowed.
- `@include` directives appear immediately after the top-level section declaration.
- No local DSL definition may appear before an `@include` directive.
- Multiple includes are allowed.
- Inclusion is transitive.
- Cyclic inclusion is invalid.
- Included and local definitions share one logical section scope.
- Included definitions are not assigned aliases.
- Name collisions are validated across the complete combined section.
- Inclusion affects DSL structure only.
- Inclusion does not copy or generate implementation source code or other artifacts.
- All definitions are validated together after inclusion is resolved.

---

## 21. Related Knowledge

The `@include` directive is related to:

- `concept.ocean-repository` — defines the Ocean reuse and reference model, including `O.` and `P.` references.
- `dsl.import` — imports a selected reusable definition under a local alias.
- `dsl.datatype` — datatype sections may include other datatype sections.
- `dsl.api` — API sections may include other API sections.
- `dsl.database` — database sections may include other database sections.
- `dsl.broker` — broker sections may include other broker sections.
- `dsl.expression` — expression sections may include other expression sections.
- `dsl.fsm` — FSM sections may include other FSM sections.
- `dsl.component` — component sections may include other component sections where supported.
- `dsl.service` — service sections may include other service sections.

These semantic relationships are declared in the document metadata.
