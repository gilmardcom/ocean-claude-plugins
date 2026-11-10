# Tag DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/tag/

Canonical reference ID for Ocean classification tags. Tags are defined by the generic metadata model in `dsl.metadata`; this document identifies the concept and points to its normative definition.

Source: https://atlas.ocean-lab.ai/dsl/tag/

Status: Accepted · Authority: Normative

## 1. Overview

A **tag** is a free-form label attached to one DSL item, used to classify it by
concern, ownership, lifecycle, visibility, or another tooling-defined category.

Tags are one of two generic metadata mechanisms Ocean DSL items may declare,
the other being perspectives (see [`dsl.perspective`](perspective.md)).

---

## 2. Normative Definition

The complete syntax, placement, and validation rules for tags are defined by
[`dsl.metadata`](metadata.md), Section 3 ("Tags"). This document does not
duplicate that specification; consult `dsl.metadata` as the single normative
source.

The canonical keyword is plural and prefixed: `@tags`. The legacy singular
spelling `tag` is not canonical DSL syntax.

---

## 3. Minimal Example

```ocean-dsl
NotificationFlow
    @tags : messaging, core, external
```

See `dsl.metadata` for the full item-level placement rules, validation
requirements, and a complete worked example.

---

## 4. Related Knowledge

- [DSL Metadata](metadata.md) — normative definition of `@tags`, `@perspectives`, and file-level `group`.
- [Perspective](perspective.md) — the companion key-value metadata mechanism.
- [Vault](vault.md) — supports item-level tags.
- [Broker](broker.md) — supports item-level tags.
- [Dashboard](dashboard.md) — supports item-level tags.

The primary semantic relationships are declared in the document metadata.
