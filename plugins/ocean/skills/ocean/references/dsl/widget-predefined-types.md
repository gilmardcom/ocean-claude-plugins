# Predefined Widget Types

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/widget-predefined-types/

**Contents**

- [1. Overview](#1-overview)
- [2. Separator](#2-separator)
- [3. Table](#3-table)
- [4. Form](#4-form)
- [5. Button](#5-button)
- [6. Chart](#6-chart)
- [7. Text](#7-text)
- [8. Validation](#8-validation)
- [9. Related Knowledge](#9-related-knowledge)

Catalogs the predefined Ocean widget types and their implemented properties, defaults, and generated functions.

Source: https://atlas.ocean-lab.ai/dsl/widget-predefined-types/

Status: Approved · Authority: Normative

## 1. Overview

Ocean currently provides six widget types:

```text
Separator
Table
Form
Button
Chart
Text
```

This catalog is separate from `dsl.widget` so new types and type-specific behavior can evolve without obscuring the common widget grammar.

---

## 2. Separator

`Separator` renders static visual separation and has no dedicated functions.

| Property    | Default | Meaning                 |
| ----------- | ------- | ----------------------- |
| `label`     | empty   | Optional centered label |
| `color`     | `gray`  | Line or label color     |
| `size`      | `md`    | Line thickness          |
| `labelSize` | `md`    | Label size              |
| `style`     | `solid` | Line style              |
| `margin`    | `my-3`  | Vertical spacing        |

```ocean-dsl
Widget OrderDivider of type Separator
    label = "Order Actions"
    style = dashed
    size = lg
    margin = my-4
```

---

## 3. Table

`Table` presents structured rows. It requires `datatype` or `columns`.

| Property     | Default                | Meaning            |
| ------------ | ---------------------- | ------------------ |
| `datatype`   | —                      | Row datatype       |
| `columns`    | derived/required       | Column list or map |
| `title`      | `Title`                | Display title      |
| `rows`       | empty                  | Initial row data   |
| `selectable` | `false`                | Enables selection  |
| `sortable`   | `false`                | Enables sorting    |
| `filterable` | `false`                | Enables filtering  |
| `offset`     | `0`                    | Pagination offset  |
| `size`       | `10`                   | Visible row count  |
| `limit`      | `10`                   | Page limit         |
| `total`      | implementation default | Total-row bound    |

Generated functions:

```text
AddRow(input:<datatype>) Boolean
RemoveRow(id:String) Boolean
Clear() _
GetSelected() <datatype>
AddRows(input:List<<datatype>>) _
Refresh() _
IncOffset() _
DecOffset() _
GetOffset() Int
```

```ocean-dsl
Widget TodoTable of type Table
    datatype = TodoItem
    selectable = true
    sortable = true
```

---

## 4. Form

`Form` captures structured input. It requires `datatype` or `fields`.

| Property      | Default          | Meaning                                        |
| ------------- | ---------------- | ---------------------------------------------- |
| `datatype`    | —                | Form datatype                                  |
| `fields`      | derived/required | Field list or map                              |
| `title`       | `Title`          | Display title                                  |
| `buttons`     | —                | Labels keyed by `submit`, `cancel`, or `fetch` |
| `submitLabel` | `Submit`         | Submit label                                   |
| `cancelLabel` | `Cancel`         | Cancel label                                   |
| `fetchLabel`  | empty            | Fetch label; empty means no fetch by default   |

Generated functions:

```text
Submit(input:<datatype>) _
Validate(input:<datatype>) Boolean
Fill(input:<datatype>) _
Reset() _
```

```ocean-dsl
Widget CreateTodoForm of type Form
    datatype = TodoItem
    buttons:
        submit: "Create Todo"
        cancel: "Cancel"
```

---

## 5. Button

`Button` represents a triggerable control and is not data-bound.

| Property     | Default     | Meaning                        |
| ------------ | ----------- | ------------------------------ |
| `label`      | `Click Me!` | Display label                  |
| `color`      | `primary`   | Visual color variant           |
| `icon`       | `lightning` | Icon identifier                |
| `extraClass` | empty       | Additional presentation class  |
| `click`      | generated   | Target-specific click behavior |

Generated functions:

```text
SetLabel(label:String) _
Click() _
Enable() _
Disable() _
```

```ocean-dsl
Widget SubmitButton of type Button
    label = Submit
    color = primary
    icon = check
```

---

## 6. Chart

`Chart` presents structured series data. It requires `datatype` or `fields`.

| Property     | Default          | Meaning                     |
| ------------ | ---------------- | --------------------------- |
| `datatype`   | —                | Input row datatype          |
| `fields`     | derived/required | Inline chart data structure |
| `title`      | `Title`          | Chart title                 |
| `chartType`  | `line`           | Chart presentation type     |
| `xField`     | `x`              | X-axis field or fields      |
| `yField`     | `y`              | Y-axis field or fields      |
| `data`       | empty            | Initial chart data          |
| `showLegend` | `true`           | Legend visibility           |

Generated functions:

```text
SetData(input:List<<datatype>>) _
ToggleLegend(visible:Boolean) _
Refresh() _
```

```ocean-dsl
Widget SalesChart of type Chart
    datatype = SalesData
    chartType = line
    xField:
        - month
    yField:
        - total
        - average
    showLegend = true
```

---

## 7. Text

`Text` presents static or dynamically updated textual content.

| Property        | Default  | Meaning                                 |
| --------------- | -------- | --------------------------------------- |
| `title`         | empty    | Heading                                 |
| `subtitle`      | empty    | Subheading                              |
| `content`       | empty    | Body content                            |
| `align`         | `start`  | Alignment (`start`, `center`, or `end`) |
| `titleStyle`    | `bold`   | Title style                             |
| `subtitleStyle` | `italic` | Subtitle style                          |
| `contentStyle`  | `muted`  | Content style                           |
| `extraClass`    | empty    | Additional presentation class           |

Generated functions:

```text
SetTitle(text:String) _
SetSubtitle(text:String) _
SetContent(text:String) _
Clear() _
```

```ocean-dsl
Widget AboutText of type Text
    title = About
    subtitle = "Welcome to Todo App"
    content = "Manage your tasks with ease."
    align = center
```

---

## 8. Validation

- The widget type must be one of the catalogued names.
- Type names and property names are case-sensitive.
- Properties must have the scalar, list, or map shape expected by the type.
- Datatype references must resolve.
- Inline column or field definitions must be valid.
- `Table`, `Form`, and `Chart` require a datatype or their supported inline structure.
- Predefined functions use the exact names and signatures shown here.
- Unknown type-specific properties may be rejected by validation or generators.

---

## 9. Related Knowledge

- `dsl.widget` — defines common widget declaration syntax and semantics.
- `dsl.dashboard` — owns widget instances and layout.
- `dsl.datatype` — defines data structures used or inferred by data-bound widgets.
