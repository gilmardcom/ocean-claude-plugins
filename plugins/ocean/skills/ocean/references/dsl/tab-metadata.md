# Ocean UI Inline Tab Metadata

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/tab-metadata/

**Contents**

- [1. Overview](#1-overview)
- [2. Canonical Block Format](#2-canonical-block-format)
- [3. Fields](#3-fields)
- [4. Editing Metadata](#4-editing-metadata)
- [5. Bundle Grouping](#5-bundle-grouping)
- [6. Invalid and Malformed Blocks](#6-invalid-and-malformed-blocks)
- [7. Examples](#7-examples)
- [8. Non-Goals](#8-non-goals)
- [9. Related Knowledge](#9-related-knowledge)

Define the optional inline metadata block Ocean UI reads from DSL bundle tabs to organize tabs by tags and semantic perspectives.

Source: https://atlas.ocean-lab.ai/dsl/tab-metadata/

Status: Accepted · Authority: Normative for Ocean UI; non-semantic to the Ocean DSL engine

## 1. Overview

Ocean UI can read an optional metadata block from an Ocean DSL bundle tab. The
block remains ordinary Ocean DSL comments, so it is safe for systems that do
not recognize it.

In V1, inline tab metadata is a frontend-only convention. The Ocean DSL engine
and backend do not extract, validate, store, or otherwise act on it. Metadata
is part of the tab source content and is saved through the normal bundle save
flow.

Ocean UI uses this metadata to organize existing bundle tabs without changing
the `Bundle → Sections → Tabs` backend contract.

---

## 2. Canonical Block Format

Use exactly one complete block per DSL tab. Place it near the beginning of the
file so it is easy for people to find.

```ocean-dsl
# @ocean-meta-start
# tags:
#   - authentication
#   - security
# perspective:
#   feature: authentication
#   boundedContext: identity
#   phase: mvp
#   status: stable
# @ocean-meta-end

@service

UserService
    ...
```

The opening marker must be exactly `# @ocean-meta-start` and the closing marker
must be exactly `# @ocean-meta-end`. Every line inside the block must be a
full-line `#` comment. The comment content between the markers uses a small
YAML mapping.

Ocean UI recognizes metadata only for DSL tabs. The metadata button is disabled
for generated, binary, and other non-DSL files.

---

## 3. Fields

### 3.1 `tags`

`tags` is an optional list of non-empty strings. Write one tag per indented
line.

```ocean-dsl
# tags:
#   - authentication
#   - security
```

A tab can belong to multiple tag groups.

### 3.2 `perspective`

`perspective` is an optional mapping from a non-empty string key to a non-empty
string value.

```ocean-dsl
# perspective:
#   feature: authentication
#   boundedContext: identity
#   phase: mvp
#   status: stable
```

The conventional V1 keys are:

| Key | Example value | Meaning |
| --- | --- | --- |
| `feature` | `authentication` | Product capability or user-facing feature. |
| `boundedContext` | `identity` | Domain or bounded context. |
| `phase` | `mvp` | Delivery, lifecycle, or implementation phase. |
| `status` | `stable` | Current maturity or lifecycle status. |

Ocean UI also discovers additional perspective keys dynamically. For example,
the following is valid and can be selected as a Perspective grouping dimension:

```ocean-dsl
# perspective:
#   service: ui-service
#   journey: onboarding
```

Unknown top-level fields are ignored safely. They do not affect grouping.

---

## 4. Editing Metadata

Select a DSL tab and use its metadata icon to inspect the current state:

- Green: a valid metadata block.
- Red: a malformed metadata block that needs attention.
- Gray: no metadata block is present.

The dialog shows tags and the perspective fields actually present in the tab.
It is intentionally read-only in V1. Edit the metadata comments directly in
the Monaco editor, then save the bundle normally. If no block is present, the
dialog can insert a starter block at the beginning of the tab.

Manual editing remains supported at all times and is the single source of
truth.

---

## 5. Bundle Grouping

Ocean UI presents the same bundle through two navigation layers:

1. A group list in the left pane.
2. The tabs belonging to the selected group in the tab bar.

The view selector supports the following modes.

| View | Left-pane groups | Tab-bar contents |
| --- | --- | --- |
| Sections | Existing bundle section types. | Tabs in the selected section. |
| Tags | All discovered tag values. | Tabs carrying the selected tag. |
| Perspective | Values for a user-selected perspective key. | Tabs whose selected perspective key has that value. |

Perspective grouping is dynamic. Selecting `feature`, `boundedContext`,
`phase`, `status`, `service`, or any other discovered key changes the values
shown in the left pane. For example, selecting `service` can show
`ui-service`, then list every tab whose metadata contains
`perspective.service: ui-service`.

Tags fan out: a tab with both `authentication` and `security` appears in both
tag groups. Perspective fields do not fan out within one selected key: a tab
appears under the one value it defines for that key.

Ocean UI retains the current view and its latest valid group/tab selection per
bundle in local storage. Selection is restored when returning to the bundle or
switching back to a view.

### 5.1 Missing metadata

Sections remains the complete, default view and continues to show every tab.
Tabs without a matching tag or selected perspective key are omitted from that
metadata-based view; Ocean UI does not create an `Ungrouped` fallback group.
They remain available through Sections.

---

## 6. Invalid and Malformed Blocks

Malformed metadata never blocks tab opening, editing, saving, or normal bundle
navigation. Ocean UI treats the metadata as absent for grouping purposes and
shows a non-blocking red metadata state.

A block is malformed when, for example, it is incomplete, duplicated, contains
invalid YAML, has a non-string perspective value, or has an invalid tag value.

Fix the comments in the editor so that there is exactly one complete block with
valid values. No engine validation or automatic repair is performed in V1.

---

## 7. Examples

### 7.1 Group by tags

```ocean-dsl
# @ocean-meta-start
# tags:
#   - payments
#   - security
# perspective:
#   feature: checkout
#   boundedContext: billing
# @ocean-meta-end
```

This tab appears under both `payments` and `security` in Tags view.

### 7.2 Group by a custom perspective

```ocean-dsl
# @ocean-meta-start
# perspective:
#   service: ui-service
#   journey: checkout
# @ocean-meta-end
```

Choose `service` in Perspective view to group this tab under `ui-service`.
Choose `journey` to group it under `checkout`.

---

## 8. Non-Goals

V1 does not provide:

- Backend metadata extraction, validation, or datatype changes.
- A separately persisted metadata record.
- A visual metadata editor or separate metadata save action.
- Taxonomy governance or custom taxonomy administration.
- Automatic or assistant-generated metadata.
- Metadata support for generated Go, Java, SQL, YAML, or other non-DSL files.

---

## 9. Related Knowledge

- Ocean DSL comment rules — metadata uses ordinary `#` comments.
- Ocean UI bundle navigation — Sections is the stable default navigation view.
- Ocean example bundles — representative `.ocn` examples include inline metadata blocks.
