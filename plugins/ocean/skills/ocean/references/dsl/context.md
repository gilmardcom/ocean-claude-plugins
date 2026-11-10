# Context DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/dsl/context/

**Contents**

- [1. Overview](#1-overview)
- [2. Definition](#2-definition)
- [3. Syntax](#3-syntax)
- [4. Context Fields](#4-context-fields)
- [5. Naming Conventions](#5-naming-conventions)
- [6. Purpose and Semantics](#6-purpose-and-semantics)
- [7. Use in `@service`](#7-use-in-service)
- [8. Imports](#8-imports)
- [9. Rules and Constraints](#9-rules-and-constraints)
- [10. Complete Example](#10-complete-example)
- [11. Related Knowledge](#11-related-knowledge)

Defines the Ocean `@context` DSL section for service-local volatile shared state used by service logic and orchestration.

Source: https://atlas.ocean-lab.ai/dsl/context/

Status: Accepted · Authority: Normative

## 1. Overview

The `@context` section defines named service-local shared state used to coordinate internal behavior across API calls, expressions, handlers, FSMs, and other service logic.

Contexts contain structured data based on Ocean datatypes. Context fields can be read or written by explicitly connected service logic.

Context is not persisted to external storage. It is volatile and exists for the duration determined by its execution context, such as a request, session, or orchestration.

---

## 2. Definition

A context file:

- starts with `@context`;
- must define at least one context;
- may define multiple contexts sequentially;
- must not contain nested context definitions;
- requires each context to have a unique name;
- defines fields using Ocean datatypes;
- may define default values for fields;
- may contain comments throughout the file.

---

## 3. Syntax

A context is defined as:

```ocean-dsl
@context

<ContextName>
    <fieldName> : <FieldType> (default=<value>)
```

The default value is optional.

Example:

```ocean-dsl
@context

TipContext
    day  : String (default=Drink water!)
    date : String
    time : String
    tip  : String
```

Each field is declared on one line. A default literal may contain spaces and
punctuation and is written without quotes (`default=Drink water!`).

---

## 4. Context Fields

A context field has:

- a field name;
- a field type;
- an optional default value.

Syntax:

```ocean-dsl
<fieldName> : <FieldType>
```

or:

```ocean-dsl
<fieldName> : <FieldType> (default=<value>)
```

Field types must resolve to valid Ocean datatypes, including:

- primitive datatypes;
- compound datatypes;
- user-defined datatypes;
- built-in datatypes.

Example:

```ocean-dsl
MyContext
    message : String
    items   : List<Item>
    config  : AppConfig
```

---

## 5. Naming Conventions

Context names follow the same naming convention as datatype names:

```text
^[A-Z][A-Za-z0-9_]*$
```

They:

- begin with an uppercase letter;
- contain only letters, digits, and `_`.

Examples:

```text
RequestContext
OrderContext
MyContext
```

Context field names follow the standard Ocean datatype field naming convention:

```text
^[a-z][A-Za-z0-9_]*$
```

They begin with a lowercase letter and contain only letters, digits, and `_`.

---

## 6. Purpose and Semantics

| Concept        | Description                                                                  |
| -------------- | ---------------------------------------------------------------------------- |
| Local scope    | A context is available only to the service that uses it.                     |
| Guarded access | Service logic accesses context fields through explicit service connections.  |
| Datatype-based | Context fields use valid Ocean datatypes.                                    |
| Volatile       | Context data is not persisted to external storage.                           |
| Declarative    | Context enables explicit coordination of shared state between service logic. |

A context provides shared state without introducing implicit access between unrelated parts of service logic.

---

## 7. Use in `@service`

A service may declare usage of one or more contexts. Every `use context`
declaration requires an explicit `as <alias>` clause.

Example:

```ocean-dsl
@service

MyService

    @perspectives: version:0.1.0, lifestyle:stable

    ...

    use expression GetItem as get
    use context MyContext as ctx

    ...

    connect get -> ctx.item

    ...
```

In this example:

```ocean-dsl
use context MyContext as ctx
```

makes `MyContext` available to the service using the alias `ctx`.

The connection:

```ocean-dsl
connect get -> ctx.item
```

writes the output of the `get` expression into the `item` field of the context.

The exact connection semantics are defined by the `@service` DSL.

---

## 8. Imports

A `@context` file may import datatypes from Ocean.

Imported datatypes can then be used as context field types.

Example:

```ocean-dsl
@context

@import datatype O.domain.payment.CardInfo@1.2.0 as CardInfo

PaymentContext
    paymentCreated : CardInfo
```

Only datatypes may be imported into an `@context` file. The `as <alias>` clause
is required, and the alias is used as the field type within the file.

Context definitions themselves are not imported through this mechanism.

---

## 9. Rules and Constraints

The following rules apply:

- A context file starts with `@context`.
- Each file must define at least one context.
- A file may define multiple contexts.
- Context definitions must not be nested.
- Each context must have a unique name.
- Each field is declared on one line.
- Context fields must use valid Ocean datatypes.
- Context fields may define optional default values.
- Context state is volatile and is not persisted as database state.
- Context access from service logic is explicit rather than implicit.
- Every service `use context` declaration specifies an explicit alias.
- Only datatypes may be introduced with `@import` in a context file.
- Imported datatype definitions may be used as context field types.

---

## 10. Complete Example

A complete `@context` file
(`ocean-examples/0003-answer-async/01-ans-context.ocn`):

```ocean-dsl
@context

AnswerContext
    tip : String (default=enjoy your day!)
```

This example demonstrates:

- a single service-local context;
- a primitive datatype field;
- an optional default value whose literal contains spaces and is written unquoted;
- a one-line field declaration.

Real contexts are frequently this small. Section 4 shows a context with
compound and user-defined field types, and Section 8 shows a context that
imports a datatype for use as a field type.

---

## 11. Related Knowledge

The `@context` DSL is related to:

- `dsl.datatype` — defines the datatype system used by context fields.
- `dsl.service` — defines how contexts are used, aliased, connected, read, and written within services.
- `dsl.import` — defines the mechanism for importing selected datatype definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
