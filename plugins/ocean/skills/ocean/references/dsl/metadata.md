# DSL Metadata Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/metadata/

**Contents**

- [1. Overview](#1-overview)
- [2. Item-Level Metadata](#2-item-level-metadata)
- [3. Tags](#3-tags)
- [4. Perspectives](#4-perspectives)
- [5. Canonical Spellings](#5-canonical-spellings)
- [6. File-Level Group](#6-file-level-group)
- [7. Placement and Validation](#7-placement-and-validation)
- [8. Complete Example](#8-complete-example)
- [9. Related Knowledge](#9-related-knowledge)

Defines generic Ocean DSL metadata using item-level tags and perspectives and an optional file-level group.

Source: https://atlas.ocean-lab.ai/dsl/metadata/

Status: Accepted · Authority: Normative

## 1. Overview

Ocean DSL metadata classifies and organizes definitions without changing their
core domain meaning.

The generic metadata model provides:

- `@tags` — free-form labels attached to one DSL item;
- `@perspectives` — key-value metadata attached to one DSL item;
- `group` — one logical classification applied to every item in a source file.

Metadata may be used by tooling for filtering, documentation, discovery,
generation, environment-specific views, or deployment selection.

---

## 2. Item-Level Metadata

Every top-level DSL item may declare tags, perspectives, both, or neither.

Syntax:

```ocean-dsl
@<section>

<ItemName>
    @tags         : <tag1>, <tag2>, ...
    @perspectives : <key1>:<value1>, <key2>:<value2>, ...
    <item-specific definitions>
```

Item-level metadata appears immediately after the item header and before the
item's section-specific declarations.

Example:

```ocean-dsl
@component

NotificationFlow
    @tags         : messaging, core
    @perspectives : channel:email, region:US

    input  : trigger:Trigger
    output : result:NotificationResult

    use fsm EmailFSM as email
    connect trigger -> email.start
    connect email.done -> result
```

---

## 3. Tags

`@tags` attaches one or more free-form labels to an item.

Syntax:

```ocean-dsl
@tags : <tag1>, <tag2>, ...
```

Tags are comma-separated and may classify an item by concern, ownership,
lifecycle, visibility, or another tooling-defined category.

Example:

```ocean-dsl
@tags : messaging, core, external
```

Rules:

- `@tags` is optional.
- A tag declaration belongs only to the item that contains it.
- Duplicate tags on the same item should be normalized to one logical value.
- Tag interpretation must not silently alter the item's normative DSL semantics.

---

## 4. Perspectives

`@perspectives` attaches context-dependent key-value metadata to an item.

Syntax:

```ocean-dsl
@perspectives : <key1>:<value1>, <key2>:<value2>, ...
```

Example:

```ocean-dsl
@perspectives : version:0.1.0, lifestyle:stable, region:US
```

Perspectives may describe variants or views used by tooling, documentation,
generation, or deployment selection.

Rules:

- `@perspectives` is optional.
- Entries are comma-separated `key:value` pairs.
- A perspective key must occur at most once in one item declaration.
- Perspective interpretation is performed by tooling that recognizes the key.
- Unknown perspective keys remain metadata and must not invalidate the item's
  core definition solely because a consumer does not recognize them.

---

## 5. Canonical Spellings

The canonical generic metadata keywords are plural and include the `@` prefix:

```text
@tags
@perspectives
```

The legacy singular spellings `tag` and `perspective` are not canonical generic
metadata syntax.

A section may separately define a property named `tags`, such as a
section-specific `tags = ...` attribute. Such a property is governed by that
section's reference and is not interchangeable with generic `@tags` metadata.

---

## 6. File-Level Group

A source file may assign all of its items to one logical group.

Syntax:

```ocean-dsl
@<section> (group:<GroupName>)
```

Example:

```ocean-dsl
@fsm (group:PaymentFlow)

PaymentFSM controls Payment.status
    key: id
    database: PaymentDB
    event confirm in(_) out(_)

    state pending
        on event confirm:
            next paid
```

The group declaration belongs to the file rather than to one item. Every item
declared in that file belongs to the same group.

### 6.1 Group Naming

A group name follows Ocean datatype naming rules:

```text
^[A-Z][A-Za-z0-9_]*$
```

It begins with an uppercase letter and contains only letters, digits, and `_`.

### 6.2 Group Rules

- `group` is optional.
- A file may declare at most one group.
- The group is declared inline with the top-level section identifier.
- A group may be used with any top-level DSL section.
- The declared group applies to every item in the file.
- A group is organizational metadata and has no intrinsic runtime semantics.
- Tooling may use groups for filtering, documentation, packaging, or modular
  deployment selection.

---

## 7. Placement and Validation

The following placement order applies:

```ocean-dsl
@<section> [(group:<GroupName>)]

<ItemName>
    [@tags : ...]
    [@perspectives : ...]
    <item-specific definitions>
```

Validation must reject:

- more than one file-level group declaration;
- a group declaration placed on an item rather than the section header;
- item metadata placed outside an item block;
- duplicate perspective keys within the same item;
- group names that violate datatype naming rules.

---

## 8. Complete Example

```ocean-dsl
@component (group:NotificationDomain)

NotificationFlow
    @tags         : messaging, core
    @perspectives : channel:email, region:US

    input  : trigger:Trigger
    output : result:NotificationResult

    use fsm EmailFSM as email

    connect trigger -> email.start
    connect email.done -> result

AuditFlow
    @tags         : messaging, audit
    @perspectives : region:US

    input  : event:AuditEvent
    output : result:AuditResult

    use fsm AuditFSM as audit

    connect event -> audit.record
    connect audit.done -> result
```

Both component items belong to `NotificationDomain`, while each retains its own
tags and perspectives.

---

## 9. Related Knowledge

- [Ocean DSL](overview.md) — source-file structure and canonical DSL references.
- [Datatype](datatype.md) — naming rules used by group names.
- [Component](component.md) — component items used in the complete example.
- [FSM](fsm.md) — FSM items and group usage.
