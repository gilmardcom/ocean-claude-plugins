# Perspective DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/perspective/

Canonical reference ID for Ocean perspective metadata. Perspectives are defined by the generic metadata model in `dsl.metadata`; this document identifies the concept and points to its normative definition.

Source: https://atlas.ocean-lab.ai/dsl/perspective/

Status: Accepted · Authority: Normative

## 1. Overview

A **perspective** is context-dependent, key-value metadata attached to one DSL
item, used to describe variants or views consumed by tooling — for example
version, lifecycle stage, region, or channel.

Perspectives are one of two generic metadata mechanisms Ocean DSL items may
declare, the other being tags (see [`dsl.tag`](tag.md)).

---

## 2. Normative Definition

The complete syntax, placement, and validation rules for perspectives are
defined by [`dsl.metadata`](metadata.md), Section 4 ("Perspectives"). This
document does not duplicate that specification; consult `dsl.metadata` as the
single normative source.

The canonical keyword is plural and prefixed: `@perspectives`. The legacy
singular spelling `perspective` is not canonical DSL syntax.

---

## 3. Minimal Example

```ocean-dsl
NotificationFlow
    @perspectives : channel:email, region:US
```

See `dsl.metadata` for the full item-level placement rules, key uniqueness
requirements, and a complete worked example.

---

## 4. Related Knowledge

- [DSL Metadata](metadata.md) — normative definition of `@tags`, `@perspectives`, and file-level `group`.
- [Tag](tag.md) — the companion free-form label metadata mechanism.
- [Vault](vault.md) — supports item-level perspectives.
- [Broker](broker.md) — supports item-level perspectives.
- [Context](context.md) — supports item-level perspectives.

The primary semantic relationships are declared in the document metadata.
