# Artifact Directive Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/artifact/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Directive, Not Section](#3-directive-not-section)
- [4. Syntax](#4-syntax)
- [5. Artifact Type](#5-artifact-type)
- [6. Artifact Reference](#6-artifact-reference)
- [7. Local File References](#7-local-file-references)
- [8. Named and Versioned References](#8-named-and-versioned-references)
- [9. Scoped Ownership](#9-scoped-ownership)
- [10. Valid Scopes](#10-valid-scopes)
- [11. Resolution](#11-resolution)
- [12. Type-aware Interpretation](#12-type-aware-interpretation)
- [13. Transformation](#13-transformation)
- [14. Materialization](#14-materialization)
- [15. Generation Order](#15-generation-order)
- [16. Output Ownership and Conflicts](#16-output-ownership-and-conflicts)
- [17. Artifact vs Asset](#17-artifact-vs-asset)
- [18. Go Artifact Example](#18-go-artifact-example)
- [19. Local Artifact Example](#19-local-artifact-example)
- [20. Validation](#20-validation)
- [21. Failure Behavior](#21-failure-behavior)
- [22. Determinism and Safety](#22-determinism-and-safety)
- [23. Purpose](#23-purpose)
- [24. Rules and Constraints](#24-rules-and-constraints)
- [25. Related Knowledge](#25-related-knowledge)

Defines the Ocean `@artifact` directive for resolving, interpreting, transforming, and materializing external semantic files as part of a generated project.

Source: https://atlas.ocean-lab.ai/dsl/artifact/

Status: Approved · Authority: Normative

## 1. Overview

The `@artifact` directive declares an external file dependency that participates in Ocean generation.

An artifact is not generated from scratch. Ocean resolves an existing source file, interprets it according to its declared artifact type, optionally transforms it, and materializes the result in the generated project.

Artifacts are **semantic files**. Their meaning and processing depend on their artifact type.

Examples may include:

- Go source files;
- Java source files;
- configuration files with type-specific transformation rules;
- other supported files whose structure Ocean understands.

---

## 2. Core Principle

`@artifact` allows hand-written or externally maintained files to participate in a generated system without treating them as opaque copies.

Conceptually:

```text
Artifact declaration
        │
        ▼
Resolve source file
        │
        ▼
Interpret by artifact type
        │
        ▼
Apply type-aware transformation
        │
        ▼
Materialize in generated project
```

The artifact remains externally supplied, but Ocean owns the rules by which it enters the generated output.

---

## 3. Directive, Not Section

`@artifact` is a directive. It is not a top-level DSL section.

It appears within the scope of a DSL element, such as an expression, service, or UI definition.

Example:

```ocean-dsl
@expression

@artifact go "firebase-auth@1.1.0"
```

The enclosing DSL element owns the artifact and determines its output context.

---

## 4. Syntax

General syntax:

```ocean-dsl
@artifact <artifact-type> <artifact-ref>
```

Where:

- `<artifact-type>` identifies how the artifact is interpreted, transformed, and materialized;
- `<artifact-ref>` identifies the source file.

Conceptual grammar:

```text
<artifact-type> ::= <identifier>

<artifact-ref>  ::= <path> | <name>[@<version>]
```

Examples:

```ocean-dsl
@artifact go "firebase-auth@1.1.0"
```

```ocean-dsl
@artifact go "./artifacts/firebase_auth.go"
```

---

## 5. Artifact Type

The artifact type declares how Ocean must interpret the resolved file.

Syntax:

```text
<artifact-type>
```

Example:

```ocean-dsl
@artifact go "firebase-auth@1.1.0"
```

Here, `go` indicates that the resolved source is a Go artifact.

An artifact type may define rules for:

- supported source-file formats;
- parsing;
- structural validation;
- transformations;
- naming;
- package or namespace adaptation;
- output location;
- file extension;
- conflict detection.

The artifact type must be recognized by the active Ocean toolchain. Unknown or unsupported types are invalid.

---

## 6. Artifact Reference

The artifact reference identifies the external source file.

Two forms are supported:

```text
<path>
```

```text
<name>[@<version>]
```

A path identifies a local file:

```ocean-dsl
@artifact go "./artifacts/firebase_auth.go"
```

A named reference identifies an artifact through the configured artifact-resolution mechanism:

```ocean-dsl
@artifact go "firebase-auth@1.1.0"
```

The reference must resolve to exactly one file.

---

## 7. Local File References

A local artifact reference points to a file available to the current Ocean project.

Example:

```ocean-dsl
@artifact go "./implementation/calculate_tax.go"
```

Local path resolution must be deterministic and constrained to locations permitted by the project and toolchain configuration.

The resolved target must be a file. Directories are not supported in version 1 of the artifact model.

The artifact's generated destination must remain within the generated project root.

---

## 8. Named and Versioned References

A named artifact reference may include a version.

Format:

```text
<name>@<version>
```

Example:

```ocean-dsl
@artifact go "firebase-auth@1.1.0"
```

The name and version are passed to the configured artifact resolver.

Resolution must produce one unambiguous source file. Missing, ambiguous, or inaccessible references are invalid.

If a version is provided, the resolver must honor it. Resolution behavior must remain deterministic.

---

## 9. Scoped Ownership

Every artifact belongs to the DSL element in whose scope it is declared.

The owning scope provides the semantic and generation context for the artifact.

It may determine:

- the artifact's output directory;
- the target package or namespace;
- permitted artifact types;
- available generated symbols;
- transformations applied before materialization.

Conceptually:

```text
Owning DSL element
        │
        ├── generated output
        └── declared artifact
                   │
                   └── materialized into the owner's output context
```

An artifact declared outside a valid owning scope is invalid.

---

## 10. Valid Scopes

`@artifact` may appear in any DSL scope that explicitly supports artifacts.

Examples may include:

- `@service`;
- `@ui`;
- `@expression`;
- other generation-capable DSL scopes.

Support is determined by the enclosing DSL section and the active generator.

The availability of `@artifact` does not imply that every artifact type is valid in every scope. A scope or generator may restrict the supported artifact types.

---

## 11. Resolution

Artifact processing begins by resolving the reference.

Conceptually:

```text
Artifact reference
        │
        ├── local path
        │
        └── named and optionally versioned reference
                    │
                    ▼
             one source file
```

Resolution must establish:

- that the reference exists;
- that it is accessible;
- that it resolves to exactly one file;
- that the file is compatible with the declared artifact type.

No transformation or materialization occurs when resolution fails.

---

## 12. Type-aware Interpretation

After resolution, Ocean interprets the source file according to the declared artifact type.

Interpretation may include parsing the file into a type-specific structural model.

For a Go artifact, Ocean may understand constructs such as:

- package declarations;
- imports;
- functions;
- types;
- identifiers.

The exact interpretation contract belongs to the artifact type.

An artifact that cannot be parsed or validated according to its declared type is invalid.

---

## 13. Transformation

An artifact may be transformed before it is written to the generated project.

Transformations are artifact-type-specific and must preserve the semantic validity of the result.

Examples may include:

- adapting a package name to the owning generated scope;
- rewriting a namespace;
- normalizing imports;
- applying target-specific naming rules;
- adapting metadata required by the generated project.

Transformations must be controlled by the artifact processor. `@artifact` does not authorize arbitrary post-processing scripts.

---

## 14. Materialization

Materialization writes the interpreted and transformed artifact into the generated project's output filesystem.

The destination is determined by:

- the owning DSL scope;
- the artifact type;
- the active generator;
- the generated project layout.

The output path must remain inside the generated project root.

An artifact must not escape the project through absolute paths, parent-directory traversal, symbolic-link resolution, or equivalent mechanisms.

---

## 15. Generation Order

Artifacts are applied after code generation and before assets.

Conceptually:

```text
Generate code
      ↓
Resolve, transform, and materialize artifacts
      ↓
Materialize assets
      ↓
Final generated project
```

This order allows artifact processors to adapt semantic files to the generated code context while preserving a clear distinction from later opaque asset materialization.

---

## 16. Output Ownership and Conflicts

Every output path must have one unambiguous owner.

An artifact must not overwrite:

- generated files;
- another artifact;
- an asset;
- any other protected output.

Conflict detection must consider the final normalized output path, not only the source reference or unnormalized destination.

When two inputs resolve to the same output path, generation must fail explicitly. Processing order must not be used to choose a winner.

---

## 17. Artifact vs Asset

Artifacts and assets are external files with different semantics.

### Artifact

- has a declared artifact type;
- is interpreted by Ocean;
- may be parsed and transformed;
- participates semantically in generation;
- is materialized according to type-specific rules.

### Asset

- is treated as opaque content;
- is copied or materialized without semantic interpretation;
- must preserve its content according to the asset contract.

Conceptually:

```text
Artifact → interpret → transform → materialize
Asset    → preserve  → copy/materialize
```

An external file that requires language-aware or format-aware adaptation should be modeled as an artifact rather than an asset.

---

## 18. Go Artifact Example

```ocean-dsl
@expression

@artifact go "firebase-auth@1.1.0"
```

In this example:

- the artifact is owned by the expression scope;
- `go` selects the Go artifact processor;
- `firebase-auth@1.1.0` is resolved to one Go source file;
- the source is parsed as Go;
- the source may be adapted to the owning output context, such as by changing its package declaration;
- the transformed file is emitted into the expression's generated output directory.

The artifact is not generated from an Ocean definition. It is an external implementation file incorporated through controlled, type-aware processing.

---

## 19. Local Artifact Example

```ocean-dsl
@expression

@artifact go "./implementation/firebase_auth.go"
```

In this example, Ocean resolves a project-local source file rather than a named artifact.

The same type-aware processing, transformation, ownership, path-safety, and conflict rules apply.

---

## 20. Validation

Artifact validation includes the following rules.

### Resolvable

The artifact reference must resolve successfully and unambiguously.

### File-only

The resolved source must be a file. Directories are not supported in version 1.

### Known type

The artifact type must be recognized and supported by the active toolchain and generation context.

### Type-compatible

The resolved file must be parseable and valid according to the declared artifact type.

### Scoped ownership

The artifact must be declared within a valid DSL element scope that supports artifacts.

### No overwrite

The artifact must not overwrite generated files, other artifacts, assets, or protected output.

### Project-local target

The final normalized output path must remain within the generated project root.

Validation errors must be fail-fast, explicit, and identify the artifact declaration that caused the failure.

---

## 21. Failure Behavior

Ocean must stop artifact processing when a required artifact cannot be safely and deterministically materialized.

Failure cases include:

- an unresolved reference;
- a reference resolving to multiple files;
- a directory reference;
- an unknown artifact type;
- invalid type-specific content;
- declaration in an unsupported scope;
- a transformation failure;
- an output collision;
- an output path outside the generated project root.

Ocean must not silently skip a required artifact or emit a partially transformed result.

---

## 22. Determinism and Safety

Given the same DSL model, artifact sources, versions, generator configuration, and target, artifact processing should produce the same result.

Artifact handling must preserve:

- deterministic resolution;
- explicit versions where used;
- type-aware validation;
- controlled transformation;
- normalized output paths;
- project-root containment;
- conflict-free ownership;
- fail-fast behavior.

These guarantees allow external semantic files to participate in generation without weakening the integrity of the generated project.

---

## 23. Purpose

`@artifact` exists to:

- integrate type-aware hand-written files into generated systems;
- enable controlled transformation of external code, such as package rewriting;
- preserve a strict distinction between opaque assets and semantic artifacts;
- avoid generator-specific hacks and arbitrary post-processing scripts;
- keep transformation intent explicit in the Ocean model;
- materialize external implementations without allowing unsafe output mutation.

It provides a controlled boundary between generated output and externally supplied semantic files.

---

## 24. Rules and Constraints

The following rules apply:

- `@artifact` is a directive, not a DSL section.
- An artifact is declared within a valid owning DSL scope.
- An artifact declaration contains an artifact type and an artifact reference.
- An artifact reference is either a local file path or a named reference with an optional version.
- The reference must resolve to exactly one file.
- Directories are not supported in version 1.
- The artifact type must be known and supported.
- The resolved file must be compatible with the declared artifact type.
- Artifacts are interpreted using artifact-type-specific rules.
- Artifacts may be transformed before materialization.
- Artifacts are materialized into the owning element's generated output context.
- Artifacts are applied after code generation and before assets.
- Artifacts must not overwrite generated files, other artifacts, or assets.
- Output conflicts are errors and must not be resolved by processing order.
- The normalized output path must remain within the generated project root.
- Artifact processing must be deterministic.
- Validation failures must be explicit and fail-fast.
- Artifacts are semantic and type-aware; assets are opaque.

---

## 25. Related Knowledge

The `@artifact` directive is related to:

- `dsl.asset` — defines opaque external files that are preserved rather than semantically interpreted.
- `dsl.service` — service generation may incorporate owned artifacts.
- `dsl.ui` — UI generation may incorporate owned artifacts.
- `dsl.expression` — expression implementations may be supplied as type-aware artifacts.

These semantic relationships are declared in the document metadata.
