# Info DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/dsl/info/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax](#3-syntax)
- [4. Project-level Section](#4-project-level-section)
- [5. Fields](#5-fields)
- [6. Name](#6-name)
- [7. Name Representation](#7-name-representation)
- [8. Version](#8-version)
- [9. Version Meaning](#9-version-meaning)
- [10. Semantic Versioning](#10-semantic-versioning)
- [11. Description](#11-description)
- [12. Description Representation](#12-description-representation)
- [13. Field Ordering](#13-field-ordering)
- [14. Cardinality](#14-cardinality)
- [15. Scope and Identity](#15-scope-and-identity)
- [16. Documentation Generation](#16-documentation-generation)
- [17. Generated Output](#17-generated-output)
- [18. Complete Example](#18-complete-example)
- [19. Minimal Example](#19-minimal-example)
- [20. Validation](#20-validation)
- [21. Invalid Examples](#21-invalid-examples)
- [22. Info vs Service Metadata](#22-info-vs-service-metadata)
- [23. Info vs Repository Identity](#23-info-vs-repository-identity)
- [24. Purpose](#24-purpose)
- [25. Rules and Constraints](#25-rules-and-constraints)
- [26. Related Knowledge](#26-related-knowledge)

Defines the Ocean `@info` section for declaring project-level identity, version, and descriptive metadata for a system or application.

Source: https://atlas.ocean-lab.ai/dsl/info/

Status: Approved · Authority: Normative

## 1. Overview

The `@info` section defines high-level metadata about the system or application described by an Ocean DSL project.

It provides a stable place for basic project identification and documentation, including:

- the system name;
- the system-definition version;
- a short description of the system's purpose.

This metadata can be used to label generated documentation, diagrams, configuration output, and generated projects consistently.

`@info` describes the overall DSL project. It does not define runtime behavior, deployment topology, or a reusable domain model.

---

## 2. Core Principle

Every Ocean DSL project should be identifiable independently of its generated implementation.

Conceptually:

```text
Ocean DSL project
       │
       ├── name
       ├── version
       └── description
                │
                ▼
       consistent project identity
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
      docs   diagrams   generated output
```

Project metadata should remain concise, human-readable, and independent of target technology.

---

## 3. Syntax

General structure:

```ocean-dsl
@info

name: <SystemName>
version: <Version>
[title: <Title>]
[subtitle: <Subtitle>]
[shortDescription: <ShortDescription>]
[description: <Description>]
```

Example:

```ocean-dsl
@info

name: Inventory Management
version: 1.0.0
description: Manages inventory and stock movements.
```

Field values are written without quotes and run to the end of the line.

---

## 4. Project-level Section

`@info` is a project-level section.

It identifies the system represented by the complete Ocean DSL project rather than one individual datatype, API, FSM, component, or service.

Conceptually:

```text
DSL Project
    ├── @info
    ├── @datatype definitions
    ├── @api definitions
    ├── @component definitions
    ├── @service definitions
    └── other DSL sections
```

An `@info` section typically appears once per DSL project.

---

## 5. Fields

The `@info` section defines the following fields:

| Field              | Required | Meaning                                                                          |
| ------------------ | -------- | -------------------------------------------------------------------------------- |
| `name`             | Yes      | Human-readable identifier for the system or application                          |
| `version`          | Yes      | Semantic version of the current system definition                                |
| `title`            | No       | Display title for documentation and UI headers                                   |
| `subtitle`         | No       | Short tagline shown alongside the title                                          |
| `shortDescription` | No       | One-line summary of the system                                                   |
| `description`      | No       | Explanation of the system's purpose or responsibilities; may span multiple lines |

Unknown fields are invalid unless they are introduced by a later version of the `@info` contract.

---

## 6. Name

The `name` field identifies the system or application.

Syntax:

```ocean-dsl
name: <SystemName>
```

Example:

```ocean-dsl
name: Inventory Management
```

The name is required.

It should be:

- short enough to use in documentation headers and generated metadata;
- meaningful to people working with the system;
- stable across generated targets;
- distinct within the organizational or project context in which it is used.

The name is descriptive metadata. It should not be assumed to be a filesystem-safe directory name, programming-language package name, or deployment identifier.

---

## 7. Name Representation

The `name` value is written as-is, without quotes, and may contain spaces:

```ocean-dsl
name: Inventory Management
```

```ocean-dsl
name: ToDo App
```

Tooling should preserve the declared display name when generating human-readable output.

Target-specific identifiers derived from the name must be normalized according to the target's own rules and must not change the canonical project name.

---

## 8. Version

The `version` field identifies the version of the system definition.

Syntax:

```ocean-dsl
version: <Version>
```

Example:

```ocean-dsl
version: 1.0.0
```

The version is required and uses semantic versioning.

Canonical core format:

```text
<major>.<minor>.<patch>
```

Examples:

```text
1.0.0
1.4.2
2.0.0
```

---

## 9. Version Meaning

The version applies to the Ocean system definition represented by the project.

It is not automatically the version of:

- Ocean itself;
- the Ocean DSL grammar;
- a generator;
- an individual service;
- a generated container image;
- a third-party dependency.

Those elements may have their own independent versions.

Conceptually:

```text
@info version
      └── version of this system definition
```

How system-definition version changes are governed may be defined by project lifecycle or release-management rules outside this reference.

---

## 10. Semantic Versioning

The three core semantic-version components communicate the scale of a change:

- **major** — incompatible or breaking changes to the system definition;
- **minor** — backward-compatible additions or capabilities;
- **patch** — backward-compatible corrections or refinements.

The `@info` section records the version. It does not independently calculate whether a change requires a major, minor, or patch increment.

That determination belongs to the project's compatibility and release policy.

---

## 11. Description

The `description` field briefly explains the system's purpose or primary responsibilities.

Syntax:

```ocean-dsl
description: <Description>
```

Example:

```ocean-dsl
description: Manages inventory and stock movements.
```

The description is optional.

A useful description should:

- state what the system does;
- remain concise;
- avoid target-specific implementation details;
- be appropriate for documentation summaries and generated metadata.

---

## 12. Description Representation

The `description` value is written without quotes. It may run to the end of the
line, span several lines, and contain simple inline HTML such as `<br>` for use
in generated documentation and UI:

```ocean-dsl
description: Processes customer orders and coordinates fulfillment.
```

```ocean-dsl
description: The Inventory Management application enables complete visibility and
control of materials across the supply network.<br><br>It provides multi-location
management, material movements, and threshold-based alerts.
```

For a one-line summary, prefer `shortDescription`. Long-form architecture,
operating instructions, ownership data, and design rationale still belong in
dedicated Ocean-Atlas documents rather than `@info`.

---

## 13. Field Ordering

The canonical field order is:

```text
name
version
title
subtitle
shortDescription
description
```

Example:

```ocean-dsl
@info

name: Math App
version: 1.0.0
title: Math-app
subtitle: To perform your math
shortDescription: Answers all your math questions
description: This app answers your math questions
```

The parser accepts fields in other orders; tooling should normalize them to the
canonical order when formatting or generating documentation.

---

## 14. Cardinality

An Ocean DSL project should contain no more than one effective `@info` section.

Multiple competing project identities are ambiguous and invalid.

```text
DSL Project
    └── zero or one @info section
```

The section is recommended for every complete system project even where compatibility with incomplete or fragmentary DSL inputs allows it to be absent.

The `name` and `version` fields are required whenever the section is present.

---

## 15. Scope and Identity

The `@info` section describes the root project scope.

It does not rename or version every definition contained in that project.

For example:

```text
Project: Inventory Management @ 1.0.0
    ├── InventoryService
    ├── StockComponent
    ├── Product datatype
    └── Inventory API
```

The contained definitions retain their own Ocean names and, where applicable, repository identities and versions.

---

## 16. Documentation Generation

Ocean tooling may use `@info` metadata to generate:

- documentation titles;
- project summaries;
- diagram captions;
- version labels;
- metadata pages;
- generated project manifests;
- release or build information.

For example:

```text
Inventory Management
Version 1.0.0
Manages inventory and stock movements.
```

Generated uses must preserve the semantic meaning of each field.

---

## 17. Generated Output

Generators may expose project information in generated output when appropriate for the target.

Examples may include:

- application metadata;
- package manifests;
- build labels;
- generated README files;
- service information endpoints;
- deployment annotations.

The mapping from `@info` fields to target-specific output is generator-defined.

A generator should not reinterpret the project version as a dependency or runtime version without an explicit target rule.

---

## 18. Complete Example

A complete `@info` file
(`ocean-examples/0005-math-example/00-info.ocn`):

```ocean-dsl
@info

name: Math App
version: 1.0.0
title: Math-app
subtitle: To perform your math
shortDescription: Answers all your math questions

description: This app answers your math questions
```

This example declares:

- the required system name and version;
- optional `title`, `subtitle`, and `shortDescription` display fields;
- a `description`;
- unquoted values throughout.

Tooling may use this metadata consistently across documentation and generated targets.

---

## 19. Minimal Example

```ocean-dsl
@info

name: Inventory
version: 1.0.0
```

This is valid because `description` is optional.

The section still provides the required system identity and version.

---

## 20. Validation

Validation of `@info` includes:

- project-level placement;
- section cardinality;
- presence of `name`;
- presence of `version`;
- absence of duplicate fields;
- absence of unknown fields;
- non-empty name value;
- valid semantic-version syntax;
- valid string representation for the optional description.

Validation errors should identify the field and the violated rule explicitly.

---

## 21. Invalid Examples

### Missing name

```ocean-dsl
@info

version: 1.0.0
description: Manages inventory.
```

Invalid because `name` is required.

### Missing version

```ocean-dsl
@info

name: Inventory
description: Manages inventory.
```

Invalid because `version` is required.

### Invalid version

```ocean-dsl
@info

name: Inventory
version: first-release
```

Invalid because the version is not a semantic version.

### Duplicate field

```ocean-dsl
@info

name: Inventory
name: Warehouse
version: 1.0.0
```

Invalid because `name` is declared more than once.

### Multiple project identities

```text
@info name: Inventory ...
@info name: Warehouse ...
```

Invalid when both sections belong to the same effective DSL project.

---

## 22. Info vs Service Metadata

`@info` describes the complete system or application project.

A service definition describes one deployable orchestration boundary within that system.

```text
@info
    └── project identity

@service
    └── deployable service definition
```

A project may contain multiple services while retaining one project-level `@info` identity.

Service-specific names, contracts, configuration, and deployment behavior belong to `dsl.service` and related references.

---

## 23. Info vs Repository Identity

The `@info` version describes the current system definition.

It is distinct from the version in a reusable Ocean reference:

```text
O.<namespace>.<name>@<version>
P.<namespace>.<name>@<version>
```

Repository and Pre-baked reference versions identify specific reusable items. The `@info` version identifies the project as a whole.

Changing one does not automatically require changing the other unless the project's release policy says so.

---

## 24. Purpose

`@info` exists to:

- give an Ocean DSL project a clear human-readable identity;
- record the version of the system definition;
- summarize the system's purpose;
- provide consistent metadata for documentation and diagrams;
- support version labeling across generated outputs;
- distinguish project identity from implementation-specific identifiers;
- make project metadata available to Ocean tooling.

---

## 25. Rules and Constraints

The following rules apply:

- `@info` is a project-level DSL section.
- An Ocean DSL project has no more than one effective `@info` section.
- Complete system projects should provide an `@info` section.
- `name` is required when `@info` is present.
- `version` is required when `@info` is present.
- `title`, `subtitle`, `shortDescription`, and `description` are optional.
- Each supported field may appear only once.
- Unknown fields are invalid unless introduced by a later contract version.
- `name` must be non-empty.
- Field values are written without quotes and run to the end of the line.
- `version` identifies the version of the system definition.
- `version` uses semantic-version syntax.
- `description` may span multiple lines and may contain simple inline HTML such as `<br>`.
- The canonical field order is `name`, `version`, `title`, `subtitle`, `shortDescription`, then `description`.
- `@info` does not define runtime behavior.
- `@info` does not replace service-specific metadata.
- The project version is distinct from Ocean Repository and local Pre-baked item versions.
- Target-specific identifiers derived from `name` do not replace the canonical declared name.

---

## 26. Related Knowledge

The `@info` DSL is related to:

- `concept.bundle` — defines the Ocean Bundle (the Ocean DSL project) that the section identifies.
- `dsl.service` — defines deployable services contained within the identified system.
- `concept.ocean-repository` — defines reusable item identities and versions, which are distinct from the project-level `@info` version.

These semantic relationships are declared in the document metadata.
