# Datatype DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/datatype/

**Contents**

- [1. Overview](#1-overview)
- [2. Datatype Definition](#2-datatype-definition)
- [3. Primitive Datatypes](#3-primitive-datatypes)
- [4. Compound Datatypes](#4-compound-datatypes)
- [5. Naming Conventions](#5-naming-conventions)
- [6. ID Fields](#6-id-fields)
- [7. Enums](#7-enums)
- [8. Errors](#8-errors)
- [9. Envelopes](#9-envelopes)
- [10. Interfaces](#10-interfaces)
- [11. Imports](#11-imports)
- [12. Complete Example](#12-complete-example)
- [13. Related Knowledge](#13-related-knowledge)

Defines the Ocean `@datatype` DSL section, including datatypes, primitive and compound types, IDs, enums, errors, envelopes, interfaces, and datatype imports.

Source: https://atlas.ocean-lab.ai/dsl/datatype/

Status: Accepted · Authority: Normative

## 1. Overview

The `@datatype` section defines reusable data structures used throughout Ocean.

Datatypes provide technology-independent data-transfer-object (DTO) contracts
that can be used by APIs, services, brokers, expressions, databases, and other
Ocean constructs.

A datatype file:

- starts with `@datatype`;
- must define at least one datatype-related item;
- may contain multiple definitions;
- may contain comments;
- must not nest datatype definitions.

Example:

```ocean-dsl
@datatype

PurchaseOrder
    buyer  : Party
    seller : Party
    item   : Item
```

---

## 2. Datatype Definition

A datatype consists of a name followed by its fields.

### 2.1 Syntax

```text
<DatatypeName>
    <fieldName> : [*]<FieldType>
    ...
```

Each field:

- is declared on a separate line;
- consists of a field name and type separated by `:`;
- may use the `*` prefix to declare the field as a pointer.

Example:

```ocean-dsl
PurchaseOrder
    buyer  : Party
    seller : *Party
    item   : Item
```

Datatype definitions may appear sequentially in the same `@datatype` file but must not be nested.

### 2.2 Self-Referential and Recursive Datatypes

A datatype may reference itself, directly or indirectly, to model recursive
structures such as trees, linked lists, and nested categories.

A direct single-value self-reference must use the `*` pointer prefix so the
datatype does not require infinite inline expansion:

```ocean-dsl
Category
    name   : String
    parent : *Category
```

Self-reference through a compound datatype is also allowed because the
collection provides the required indirection:

```ocean-dsl
Category
    name     : String
    children : List<Category>
```

Indirect recursive relationships are allowed when the cycle contains at least
one pointer or compound-type edge:

```ocean-dsl
Employee
    team : *Team

Team
    members : List<Employee>
```

The following direct inline self-reference is invalid:

```ocean-dsl
Category
    parent : Category
```

Rules:

- The referenced datatype must resolve within the file or through an available
  datatype definition.
- A direct single-value self-reference must use `*`.
- Every indirect recursive cycle must contain at least one pointer or compound
  datatype edge.
- Recursive collections may use `List`, `Set`, or `Map` where their element or
  value types are otherwise valid.
- Recursive datatype structures are distinct from interface cycles; circular
  interface relationships remain prohibited.
- Generators must preserve recursive structure without infinitely expanding the
  datatype definition.

---

## 3. Primitive Datatypes

Primitive datatypes may be used directly without explicit definition.

| Datatype    | Definition                                                                                 | Example                        |
| ----------- | ------------------------------------------------------------------------------------------ | ------------------------------ |
| `Int`       | Whole number without a decimal point                                                       | `42`                           |
| `Float`     | Single-precision decimal number                                                            | `3.14`                         |
| `Double`    | Double-precision decimal number                                                            | `3.1415926535`                 |
| `Boolean`   | Boolean value                                                                              | `true`                         |
| `String`    | Sequence of characters                                                                     | `"Hello"`                      |
| `Date`      | Calendar date without time                                                                 | `2025-04-16`                   |
| `Time`      | Time of day without a date                                                                 | `14:30:00`                     |
| `DateTime`  | Date and time without timezone information                                                 | `2025-04-16T14:30:00`          |
| `Timestamp` | Date and time including timezone or offset                                                 | `2025-04-16T14:30:00Z`         |
| `Json`      | Arbitrary structured JSON data whose schema is not explicitly modeled as an Ocean datatype | `{"name":"Ocean","version":1}` |

`Json` represents structured JSON data rather than a JSON-encoded `String`.

The concrete representation of each primitive is determined by the target technology or language generator.

---

## 4. Compound Datatypes

Compound datatypes represent collections of other datatypes.

| Datatype          | Definition                                             | Example            |
| ----------------- | ------------------------------------------------------ | ------------------ |
| `List<Datatype>`  | Ordered collection that may contain duplicate elements | `List<Int>`        |
| `Set<Datatype>`   | Collection of unique elements                          | `Set<String>`      |
| `Map<Key, Value>` | Collection of key-value pairs                          | `Map<String, Int>` |

Examples:

```ocean-dsl
Order
    items      : List<Item>
    tags       : Set<String>
    attributes : Map<String, String>
```

Compound types may contain primitive or user-defined datatypes where supported by the type.

### 4.1 Paginated Results: `Page<Datatype>`

`Page<Datatype>` represents one page of a paginated result: the page's items
plus the true total count across every page, not just the current one.

| Field   | Type            | Meaning                                  |
| ------- | --------------- | ----------------------------------------- |
| `items` | `List<Datatype>` | The current page's rows                   |
| `total` | `Int`            | The true total row count, across all pages |

`Page<Datatype>` is used as a return type, most often for a REST/SOAP/gRPC
operation or an expression's output — anywhere a caller needs both a page of
results and enough information to know whether another page follows:

```ocean-dsl
@api

OrderApi style:rest
    get /orders/{offset}/{limit} listOrders(offset:Int, limit:Int) : Page<OrderRow>
```

Unlike `List<T>`, `Set<T>`, and `Map<K,V>`, which lower directly to a native
container in every generated language, `Page<Datatype>` has no native
equivalent: the engine generates one concrete `{items, total}` datatype per
distinct `Datatype` actually used as a page's element type (e.g.
`Page<OrderRow>` synthesizes an `OrderRow` page datatype), the first time that
combination appears. That generated datatype behaves exactly like any other
declared datatype from then on — it can be inspected, referenced, and shows
up in generated docs the same way.

Only a single, simple element type is supported (`Page<OrderRow>`, not
`Page<List<OrderRow>>` or `Page<Map<String, OrderRow>>`), and `Page<T>` may
not itself appear as a field inside another datatype — it is a return type
only.

A table widget's `listRows` binding recognizes when the operation it is
bound to returns `Page<T>`: it reports the true total to the UI, so pagination
past the current page works correctly. Bound to a bare `List<T>` instead, the
same widget still renders the page it receives, but has no way to learn the
true total, so it cannot know whether another page follows.

---

## 5. Naming Conventions

### 5.1 Datatype Names

Datatype names must match:

```text
^[A-Z][A-Za-z0-9_]*$
```

They must:

- begin with an uppercase letter;
- contain only letters, digits, and `_`.

Examples:

| Input       | Valid |
| ----------- | ----- |
| `Apple`     | Yes   |
| `A_123`     | Yes   |
| `X1`        | Yes   |
| `Test_Name` | Yes   |
| `apple`     | No    |
| `_Test`     | No    |
| `1Start`    | No    |

### 5.2 Field Names

Field names must match:

```text
^[a-z][A-Za-z0-9_]*$
```

They must:

- begin with a lowercase letter;
- contain only letters, digits, and `_`.

Examples:

| Input       | Valid |
| ----------- | ----- |
| `apple`     | Yes   |
| `a_123`     | Yes   |
| `x1`        | Yes   |
| `test_Name` | Yes   |
| `Apple`     | No    |
| `_test`     | No    |
| `1start`    | No    |

### 5.3 Field Types

A field type must resolve to one of:

- a primitive datatype;
- a compound datatype;
- a user-defined datatype;
- an enum;
- another compatible datatype construct supported by `@datatype`.

---

## 6. ID Fields

Datatypes may declare an ID field with a structured generation pattern.

### 6.1 Syntax

```text
<DatatypeName>
    id pattern <pattern>
```

Example:

```ocean-dsl
PurchaseOrder
    id pattern PO-UUID
```

A pattern may consist of:

```text
[prefix-]<strategy>
```

The prefix is optional and provides a domain-specific identifier prefix.

### 6.2 ID Generation Strategies

| Strategy    | Description                                           | Example                                |
| ----------- | ----------------------------------------------------- | -------------------------------------- |
| `UUID`      | Universally unique identifier                         | `550e8400-e29b-41d4-a716-446655440000` |
| `ULID`      | Lexicographically sortable unique identifier          | `01F8MECHZX3TBDSZ7XRADM79XE`           |
| `AutoInc`   | Auto-incrementing integer within the applicable scope | `42`                                   |
| `Timestamp` | Milliseconds since epoch                              | `1684170839000`                        |
| `Random6`   | Six-character random alphanumeric identifier          | `X7F3C9`                               |

Examples:

| Pattern        | Example                                   |
| -------------- | ----------------------------------------- |
| `UUID`         | `7f3a9b12-8d54-4a5c-a3e1-d3c6ea9d4123`    |
| `PO-UUID`      | `PO-7f3a9b12-8d54-4a5c-a3e1-d3c6ea9d4123` |
| `USR-Random6`  | `USR-X8Z1QW`                              |
| `TX-Timestamp` | `TX-1684170839000`                        |
| `DOC-AutoInc`  | `DOC-42`                                  |

---

## 7. Enums

Enums define a fixed set of named values that may be used as field types.

They are suitable for statuses, categories, types, and other constrained value sets.

### 7.1 Syntax

```text
enum <EnumName>
    <value>
    ...
```

Example:

```ocean-dsl
@datatype

enum OrderStatus
    created
    confirmed
    shipped
    delivered
    cancelled

PurchaseOrder
    status : OrderStatus
```

### 7.2 Rules

- Enum names use `UpperCamelCase`.
- Enum values use `lowerCamelCase`.
- Enum values are simple and unparameterized.
- Enum values are represented as strings.
- Enums may be referenced as datatype field types.
- An enum must be defined or made available before it is used by a datatype.

### 7.3 Generated Unknown Value

Ocean generators ensure that an enum has an `unknown` value suitable for use as its zero/default value.

If `unknown` is not explicitly declared, it is automatically added as the first enum value.

---

## 8. Errors

An `error` defines a standardized first-class failure model.

Errors provide a consistent, transport-independent representation of failures across APIs, brokers, expressions, services, and other boundaries.

Errors are data rather than language-level exceptions.

They are serializable, transport-independent, composable, and stackable. Their
controlled structure allows generators and applications to expose errors safely
across service boundaries. Errors are typically carried by an `envelope`, but
may also be used independently where appropriate.

### 8.1 Syntax

```text
error <ErrorName>
    code     : <code>
    message  : <message>
    httpCode : <httpCode>
```

Example:

```ocean-dsl
@datatype

error NotFoundError
    code     : NOT_FOUND
    message  : Not found
    httpCode : 404
```

### 8.2 Fields

| Field      | Type           | Required | Description                                       |
| ---------- | -------------- | -------- | ------------------------------------------------- |
| `code`     | `String`       | Yes      | Stable machine-readable error identifier          |
| `message`  | `String`       | Yes      | Human-readable error summary                      |
| `details`  | `List<String>` | No       | Contextual details accumulated during propagation |
| `httpCode` | `Int`          | No       | HTTP status hint; defaults to `500`               |

### 8.3 Rules

- `code` should use `UPPER_SNAKE_CASE`.
- `code` is a stable, machine-readable identifier and `message` is its
  human-readable summary.
- `httpCode` defaults to `500` when omitted.
- `details` may be populated while an error propagates through runtime layers.
- Error definitions are immutable, while runtime error instances may be enriched with contextual details.
- `httpCode` is a transport hint and does not make the Error model HTTP-specific.

The same error definition may be reused across different transport and execution technologies.

---

## 9. Envelopes

An `envelope` defines a standardized operation result containing either successful data or an error.

It provides consistent result semantics across APIs, brokers, asynchronous messaging, and internal service boundaries.

### 9.1 Syntax

```text
envelope <EnvelopeName>
    data <Datatype>
```

Example:

```ocean-dsl
@datatype

envelope LoginResult
    data AuthTokens
```

### 9.2 Logical Structure

Every envelope conceptually contains:

```text
ok    : Boolean
data  : <Datatype>
error : Error
meta  : Map<String, String>
```

These fields are implicit and are not redeclared for each envelope.

### 9.3 Runtime Invariants

When:

```text
ok == true
```

then:

- `data` must be present;
- `error` must be null.

When:

```text
ok == false
```

then:

- `error` must be present;
- `data` must be null.

Generated code and validators enforce these invariants.

### 9.4 Design Rules

- An envelope has one explicitly declared successful data type.
- At runtime an envelope contains either successful data or an error, never both.
- Error handling is standardized through the Error model.
- The implicit `meta` map is optional and supports concerns such as tracing,
  correlation IDs, and warnings.
- Ocean uses explicitly named envelopes rather than DSL generic syntax such as `Envelope<T>`.

---

## 10. Interfaces

`@datatype` supports interface-based structural composition.

Interfaces define reusable field contracts. They provide structural compatibility without behavior or method inheritance.

### 10.1 Syntax

An interface may be declared explicitly:

```ocean-dsl
interface OrderAware
    order : String
```

A datatype may conform to one or more interfaces using `:`:

```ocean-dsl
PurchaseOrder : OrderAware, Timestamped
    buyer  : Party
    seller : Party
```

Fields contributed by interfaces become part of the resulting datatype structure.

### 10.2 Example

```ocean-dsl
@datatype

interface OrderAware
    order : String

Timestamped
    createdAt : Date
    updatedAt : Date

PurchaseOrder : OrderAware, Timestamped
    buyer    : Party
    seller   : Party
    items    : List<Item>
    issuedOn : Date
    status   : String
```

`PurchaseOrder` therefore contains:

- `order`;
- `createdAt`;
- `updatedAt`;
- its explicitly declared fields.

It may also be used wherever structural compatibility with `OrderAware` or `Timestamped` is required.

### 10.3 Constraints

- Interfaces must not be nested.
- Interfaces define fields only and must not define behavior or methods.
- Conformance is structural.
- Duplicate fields contributed by multiple interfaces must be deduplicated consistently.
- Circular interface relationships are not allowed.

The `interface` keyword is optional where Ocean can infer interface usage from structural composition.

---

## 11. Imports

A `@datatype` file may import datatypes from Ocean packages.

### 11.1 Syntax

```ocean-dsl
@import datatype <reference> as <LocalName>
```

Example:

```ocean-dsl
@datatype

@import datatype O.domain.payment.CardInfo@1.2.0 as CardInfo

Payment
    card : CardInfo
```

### 11.2 Rules

- Only datatypes may be introduced with `@import` in an `@datatype` file.
- The `as <LocalName>` alias is required and is used to reference the imported
  datatype within the file.
- Imported datatypes may be referenced like locally available datatypes.
- Enums are included as part of datatype imports.
- Behavioral or service-level constructs must not be imported as datatype definitions.

This restriction preserves the self-contained structural nature of datatype knowledge.

---

## 12. Complete Example

The following is a complete `@datatype` file
(`ocean-examples/0008-invoice-approval-system/10-datatype.ocn`):

```ocean-dsl
@datatype

Invoice
    id pattern INV-UUID
    title        : String
    customer     : *Customer
    totalAmount  : Float
    submittedAt  : DateTime
    approvedAt   : DateTime
    rejectedAt   : DateTime
    status       : InvoiceStatus
    createdBy    : *User
    approvedBy   : *User
    rejectedBy   : *User
    remark       : String

enum InvoiceStatus
    pendingApproval
    approved
    rejected
    #paid

User
    id pattern USR-UUID
    name: String
    role: String

Customer
    id pattern CUS-UUID
    name       : String
    department : String

SubmitInvoiceReq
    title        : String
    customerId   : String
    totalAmount  : Float
    createdBy    : String  # userId

ApproveInvoiceReq
    approvedBy : String  # userId
    remark     : String

RejectInvoiceReq
    rejectedBy : String  # userId
    reason     : String

InvoiceSummary
    id           : String
    title        : String
    customerName : String
    totalAmount  : Float
    status       : InvoiceStatus
```

This example demonstrates:

- user-defined datatypes;
- primitive datatypes (`String`, `Float`, `DateTime`);
- pointer fields (`*Customer`, `*User`);
- an enum used as a field type;
- structured ID fields with prefixed generation patterns;
- request and summary DTOs alongside the core domain types;
- line and trailing comments.

For compound datatypes (`List`, `Set`, `Map`), the `Json` primitive, errors,
envelopes, and interfaces, see Sections 3, 4, 8, 9, and 10.

---
## 13. Related Knowledge

The `@datatype` DSL is related to:

- `dsl.import` — defines the mechanism for importing selected datatype definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable datatype definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
