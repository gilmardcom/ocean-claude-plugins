# Asset DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/asset/

**Contents**

- [Local file](#local-file)
- [Named asset](#named-asset)
- [Versioned asset](#versioned-asset)

Defines the `@asset` directive for attaching external files to DSL elements. Assets are copied verbatim into generated projects and are not interpreted or transformed.

Source: https://atlas.ocean-lab.ai/dsl/asset/

Status: WIP · Authority: Normative — not final: content may be incomplete or change.

# 1. Overview

The `@asset` directive declares an external file dependency that must be copied verbatim into the generated project.

Assets are:

- not generated;
- not interpreted;
- not transformed;
- copied exactly as they are.

An asset becomes part of the generated project and is owned by the DSL element in whose scope it is declared.

Unlike sections such as `@service` or `@expression`, `@asset` is **not a top-level DSL section**. It is a reusable directive that may appear inside supported DSL elements.

---

# 2. Syntax

General syntax:

```ocean-dsl
@asset <asset-ref>
```

where `<asset-ref>` is either:

```text
<path>
```

or

```text
<name>@<version>
```

The version is optional.

---

# 3. Asset References

An asset reference identifies the file to be copied.

Supported forms include:

### Local file

```ocean-dsl
@asset "templates/login.html"
```

### Named asset

```ocean-dsl
@asset "firebase-auth"
```

### Versioned asset

```ocean-dsl
@asset "firebase-auth@1.1.0"
```

Asset resolution is performed by the Ocean generation pipeline.

---

# 4. Scope

`@asset` is associated with the nearest enclosing DSL element.

Conceptually, it may be used inside DSL constructs such as:

- `@service`
- `@ui`
- `@expression`
- `@component`
- `@fsm`

The asset belongs to that enclosing element and is generated together with it.

---

# 5. Generation Semantics

During generation:

1. the asset reference is resolved;
2. the referenced file is located;
3. the file is copied into the generated project;
4. ownership is associated with the enclosing DSL element.

Assets are copied after generated files have been produced.

Generated files always take precedence.

Assets must never overwrite generated output.

Assets are treated as opaque files.

The DSL does not interpret their content or file format.

---

# 6. Validation Rules

The following validation rules apply:

1. **Resolvable**

   The asset reference must resolve to exactly one file.

2. **File Only**

   Directories are not supported.

3. **Scoped Ownership**

   An asset must be declared inside a valid DSL element.

4. **No Overwrite**

   Assets must not overwrite generated files or previously copied assets.

5. **Project Local**

   The destination path must remain inside the generated project.

Validation failures should be explicit and fail fast.

---

# 7. Purpose

`@asset` enables generated projects to safely include handwritten artifacts.

Typical uses include:

- configuration templates;
- HTML templates;
- static resources;
- handwritten source files;
- framework-specific artifacts;
- generated project supplements.

The goal is to allow generated and handwritten content to coexist without hidden conventions or post-generation scripts.

Ownership remains explicit in the DSL.

---

# 8. Example

```ocean-dsl
@expression

@asset "firebase-auth@1.1.0"

IsAuthenticated
    input  : token:String
    output : result:Boolean

    external
        source: "firebase-auth.go"
        method: "ValidateToken"
```

In this example:

- the expression owns the asset;
- the asset is copied unchanged into the generated project;
- the expression delegates its implementation to external code.

---

# 9. Rules and Constraints

The following rules apply:

- `@asset` is a directive, not a top-level DSL section.
- An asset must be declared inside a valid DSL element.
- Assets reference exactly one file.
- Assets are copied verbatim.
- Assets are never interpreted or transformed by Ocean.
- Assets must not overwrite generated files.
- Asset ownership belongs to the enclosing DSL element.
- Asset content is opaque to the DSL.
- Asset resolution occurs during project generation.

---

# 10. Related Knowledge

The `@asset` directive is related to:

- `dsl.service` — services may own external implementation assets.
- `dsl.ui` — UI definitions may include static resources.
- `dsl.expression` — expressions may reference external implementation assets.
- `dsl.component` — components may own supplementary artifacts.
- `dsl.fsm` — FSMs may reference external implementation assets.
- `dsl.import` — defines the mechanism for importing reusable definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
