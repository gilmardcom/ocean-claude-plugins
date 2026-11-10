# Import DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/import/

**Contents**

- [1. Overview](#1-overview)
- [2. Syntax](#2-syntax)
- [3. Examples](#3-examples)
- [4. Semantics](#4-semantics)
- [5. Aliases](#5-aliases)
- [6. Type Compatibility](#6-type-compatibility)
- [7. Version Resolution](#7-version-resolution)
- [8. Validation](#8-validation)
- [9. Import vs Include](#9-import-vs-include)
- [10. Related Knowledge](#10-related-knowledge)

Defines the Ocean `@import` directive for resolving a selected reusable definition and exposing it through a local alias.

Source: https://atlas.ocean-lab.ai/dsl/import/

Status: Draft · Authority: Normative — not final: content may be incomplete or change.

## 1. Overview

The `@import` directive resolves one selected reusable definition and makes it available in the current DSL file through a local alias.

Importing preserves the definition's repository identity while allowing the receiving file to use a concise local name.

---

## 2. Syntax

```ocean-dsl
@import <type> <RepositoryRef> as <Alias>
```

| Element         | Meaning |
| --------------- | ------- |
| `type`          | Kind of DSL definition being imported, such as `datatype`, `api`, `expression`, `fsm`, `component`, or `service` |
| `RepositoryRef` | Qualified reusable-item reference in an accepted repository reference space |
| `Alias`         | Local name through which the imported definition is referenced |

The canonical Ocean Repository reference form is:

```text
O.<namespace>.<name>@<version>
```

A construct may also accept a compatible local Pre-baked reference:

```text
P.<namespace>.<name>@<version>
```

The source spaces and their resolution rules are defined by `concept.ocean-repository`.

---

## 3. Examples

```ocean-dsl
@import fsm O.auth.loginFlow@1.2.0 as login
@import expression O.std.IsEmail@latest as isEmail
@import component O.billing.InvoiceLogic@2.0.0 as invoice
```

A datatype import can be used as if the aliased type were declared locally:

```ocean-dsl
@datatype

@import datatype O.std.auth.UserToken@1.0.0 as Token

Session
    user    : Token
    expires : Int
```

Here, `O.std.auth.UserToken@1.0.0` remains the global repository identity and `Token` is its local DSL identity.

---

## 4. Semantics

An import must:

- identify exactly one reusable definition;
- declare the expected DSL type;
- resolve the requested version deterministically;
- validate that the resolved definition matches the declared type;
- introduce a valid, unambiguous local alias;
- preserve the imported definition's semantics.

An imported definition is referenced through its alias after resolution. The full repository reference does not replace ordinary local DSL naming.

---

## 5. Aliases

The alias must follow the naming rules of the receiving DSL construct.

Aliases separate global and local identity:

```text
Global identity : O.std.auth.UserToken@1.0.0
Local identity  : Token
```

An alias must not collide with another visible local or imported name in the same scope.

---

## 6. Type Compatibility

The declared import type is part of the import contract.

For example:

```ocean-dsl
@import datatype O.domain.payment.CardInfo@1.2.0 as CardInfo
```

is valid only when the reference resolves to a datatype definition. Resolving it to an API, component, or other incompatible item is invalid.

Each DSL reference defines which import types are accepted in that file's scope.

---

## 7. Version Resolution

An explicit version selects that version:

```text
O.auth.loginFlow@1.2.0
```

Where supported, `latest` selects the latest applicable version:

```text
O.auth.loginFlow@latest
```

If the version is omitted, resolution follows the rules of the selected repository. Resolution must remain explicit and deterministic.

---

## 8. Validation

An import is invalid when:

- the reference syntax is malformed;
- the source prefix is unsupported;
- the item or requested version cannot be resolved;
- resolution is ambiguous;
- the resolved item has the wrong DSL type;
- the alias is invalid or collides with another visible name;
- the receiving DSL construct does not permit that imported type.

---

## 9. Import vs Include

`@import` and `@include` provide different reuse semantics.

| Directive  | Effect |
| ---------- | ------ |
| `@import`  | Selects a definition and exposes it under a local alias |
| `@include` | Statically composes a compatible same-section definition into the current file |

The complete inclusion contract is defined by `dsl.include`.

---

## 10. Related Knowledge

- `concept.ocean-repository` — defines repository identity, namespaces, versions, and the `O.` and `P.` source spaces.
- `dsl.include` — defines static same-section composition without an import alias.
- `dsl.datatype` — defines datatype imports and alias usage.
- `dsl.api` — defines the imports accepted by API files.
- `dsl.database` — defines database import scope.
- `dsl.broker` — defines broker import scope.
- `dsl.expression` — defines expression import scope.
- `dsl.fsm` — defines FSM import scope.
- `dsl.component` — defines component import scope.
- `dsl.service` — defines service import scope.

These semantic relationships are declared in the document metadata.
