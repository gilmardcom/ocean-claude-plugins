# Widget DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/widget/

**Contents**

- [1. Overview](#1-overview)
- [2. Syntax](#2-syntax)
- [3. Dashboard Scope](#3-dashboard-scope)
- [4. Widget Identity](#4-widget-identity)
- [5. Property Shapes](#5-property-shapes)
- [6. Datatype Binding](#6-datatype-binding)
- [7. Predefined Functions](#7-predefined-functions)
- [8. Defaults](#8-defaults)
- [9. Complete Example](#9-complete-example)
- [10. Validation](#10-validation)
- [11. Rules and Constraints](#11-rules-and-constraints)
- [12. Related Knowledge](#12-related-knowledge)

Defines widget declarations inside Ocean dashboards, including names, predefined types, scalar/list/map properties, datatype binding, and generated functions.

Source: https://atlas.ocean-lab.ai/dsl/widget/

Status: Approved · Authority: Normative

## 1. Overview

A widget is a named UI building block declared inside an Ocean dashboard. Widgets describe presentation and interaction in a target-independent model; generators translate them into concrete user-interface output.

`Widget` is not a top-level DSL section. Its enclosing `@dashboard` owns it.

---

## 2. Syntax

```ocean-dsl
Widget <WidgetName> of type <WidgetType>
    <scalarProperty> = <value>
    <listProperty>:
        - <value>
    <mapProperty>:
        <key>: <value>
```

Example:

```ocean-dsl
Widget TodoTable of type Table
    title = "Tasks"
    datatype = TodoItem
    selectable = true
```

---

## 3. Dashboard Scope

Widgets occur inside a dashboard definition:

```ocean-dsl
@dashboard

TodoDashboard
    title: "Todo Management"
    layout: TwoColumnLayout

    Widget TodoTable of type Table
        datatype = TodoItem
```

Widget names must be unique within their dashboard. The dashboard determines placement and participates in UI-level connections.

---

## 4. Widget Identity

The declaration header contains a widget name and type:

```text
Widget <WidgetName> of type <WidgetType>
```

Names follow the Ocean type-name convention. Types are case-sensitive predefined identifiers. Currently supported types are:

```text
Separator
Table
Form
Button
Chart
Text
```

Unknown types are invalid.

---

## 5. Property Shapes

Widget properties have three structural forms.

Scalar:

```ocean-dsl
title = "Tasks"
```

List:

```ocean-dsl
columns:
    - id
    - title
    - status
```

Map:

```ocean-dsl
buttons:
    submit: "Create Todo"
    cancel: "Cancel"
```

Each widget type defines its accepted properties and expected shapes. A property supplied in the wrong shape is invalid.

---

## 6. Datatype Binding

Data-bound widgets use the lowercase `datatype` property:

```ocean-dsl
datatype = TodoItem
```

The reference must resolve to an Ocean datatype. `Table`, `Form`, and `Chart` can derive their structural fields from it.

Where supported, structure may instead be declared inline through the widget-specific property:

- `Table` uses `columns`;
- `Form` uses `fields`;
- `Chart` uses `fields`.

The widget processor derives an internal datatype from inline structure. A data-bound widget missing both a resolvable `datatype` and its required inline structure is invalid.

---

## 7. Predefined Functions

Widget behavior is represented by functions predefined by the widget type. These functions are generated during model processing and can participate in supported UI connections.

Examples include:

```text
Table.AddRow
Form.Submit
Button.Click
Chart.SetData
Text.SetContent
```

Functions are not declared through generic `actions` and `events` lists in the current model. Their exact signatures are defined in `dsl.widget-predefined-types`.

---

## 8. Defaults

Widget types may supply defaults for omitted presentation or behavioral properties. Defaults are applied by widget-type processing and do not need to be repeated in every declaration.

Defaults belong to the predefined widget catalog because they evolve with each built-in type.

---

## 9. Complete Example

```ocean-dsl
@dashboard

TodoDashboard
    title: "Todo Management"
    layout: TwoColumnLayout

    Widget TodoTitle of type Text
        title = "ToDo App"
        subtitle = "Keeping your tasks organized"

    Widget TodoTable of type Table
        datatype = TodoItem
        selectable = true

    Widget CreateTodoForm of type Form
        datatype = TodoItem
        buttons:
            submit: "Create Todo"
            cancel: "Cancel"
```

---

## 10. Validation

Validation includes:

- placement inside a dashboard;
- valid and unique widget name;
- supported widget type;
- recognized type-specific properties;
- correct scalar, list, or map property shape;
- datatype resolution;
- valid inline field definitions;
- type-specific required properties;
- valid property values;
- valid references to predefined functions.

---

## 11. Rules and Constraints

- A widget is declared inside an `@dashboard` definition.
- The header is `Widget <WidgetName> of type <WidgetType>`.
- Widget names are unique within their dashboard.
- Widget types are case-sensitive and predefined.
- Properties may be scalar, list, or map values.
- Property names and shapes are type-specific.
- The canonical datatype property is lowercase `datatype`.
- `Table`, `Form`, and `Chart` require a datatype or supported inline structure.
- Built-in behavior is exposed through predefined functions.
- Generic `actions` and `events` declarations are not part of the current canonical contract.
- Defaults and function signatures are defined by the widget type catalog.

---

## 12. Related Knowledge

- `dsl.dashboard` — owns widget declarations and their layout.
- `dsl.widget-predefined-types` — catalogs supported widget types, properties, defaults, and functions.
- `dsl.datatype` — defines datatypes used by data-bound widgets.
- `dsl.ui` — defines higher-level UI composition and supported connections.

These relationships are declared in the metadata.
