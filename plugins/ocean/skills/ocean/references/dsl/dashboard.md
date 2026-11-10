# Dashboard DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/dsl/dashboard/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax](#3-syntax)
- [4. Dashboard Definition](#4-dashboard-definition)
- [5. Dashboard Names](#5-dashboard-names)
- [6. Title and Subtitle](#6-title-and-subtitle)
- [7. Layout](#7-layout)
- [8. Widgets](#8-widgets)
- [9. Widget Properties](#9-widget-properties)
- [10. Metadata](#10-metadata)
- [11. Multiple Dashboards](#11-multiple-dashboards)
- [12. Usage in UI](#12-usage-in-ui)
- [13. Backend Independence](#13-backend-independence)
- [14. Complete Example](#14-complete-example)
- [15. Imports](#15-imports)
- [16. Validation](#16-validation)
- [17. Invalid Examples](#17-invalid-examples)
- [18. Purpose](#18-purpose)
- [19. Rules and Constraints](#19-rules-and-constraints)
- [20. Related Knowledge](#20-related-knowledge)

Defines the Ocean `@dashboard` section for composing widgets into reusable, backend-agnostic pages or screens.

Source: https://atlas.ocean-lab.ai/dsl/dashboard/

Status: Approved · Authority: Normative

## 1. Overview

The `@dashboard` section defines reusable user-interface pages or screens.

A dashboard groups related widgets—such as tables, forms, buttons, charts, text blocks, and separators—into one logical presentation unit. It may define a title, subtitle, layout, metadata, and any number of widgets.

A dashboard remains backend-agnostic. API, broker, service, and other external connections are owned by the `@ui` layer.

---

## 2. Core Principle

A dashboard owns presentation composition, while a UI owns application-level integration.

```text
@ui
  │
  ├── uses Dashboard
  ├── connects widget functions
  └── connects backend capabilities
              │
              ▼
@dashboard
  ├── title and layout
  └── Widgets
       ├── Table
       ├── Form
       ├── Button
       ├── Chart
       ├── Text
       └── Separator
```

This separation allows dashboards to be reused without embedding service or transport details in their presentation model.

---

## 3. Syntax

```ocean-dsl
@dashboard

<DashboardName>
    title: <Title>
    subtitle: <Subtitle>
    layout: <LayoutName>

    Widget <WidgetName> of type <WidgetType>
        <scalarProperty> = <value>
        <listProperty>:
            - <value>
        <mapProperty>:
            <key>: <value>
```

A file may define multiple dashboards.

---

## 4. Dashboard Definition

A dashboard consists of:

- a unique name;
- an optional display title;
- an optional display subtitle;
- an optional predefined layout;
- optional Ocean metadata;
- zero or more widget declarations.

Example:

```ocean-dsl
TodoDashboard
    title: Todo Management
    subtitle: Organize tasks
    layout: TwoColumnLayout
```

---

## 5. Dashboard Names

Dashboard names follow the Ocean type-name convention.

Examples:

```text
TodoDashboard
OrderDashboard
OperationsOverview
```

Names must be unique within the resolved dashboard section scope and must be valid Ocean identifiers.

---

## 6. Title and Subtitle

`title` and `subtitle` are optional human-readable display values.

```ocean-dsl
title: Todo Items
subtitle: See all your tasks!
```

The value runs to the end of the line and is written without quotes; it may
contain spaces, punctuation, and emoji (`title: 📦 Inventory`). These properties
affect presentation and documentation, not backend behavior.

---

## 7. Layout

The optional `layout` property selects a predefined dashboard layout.

```ocean-dsl
layout: TwoColumnLayout
```

Supported layouts are:

| Layout                  | Meaning                            |
| ----------------------- | ---------------------------------- |
| `Default`               | Generator-default arrangement      |
| `SingleColumnLayout`    | One vertical content column        |
| `TwoColumnLayout`       | Two content columns                |
| `ThreeColumnLayout`     | Three content columns              |
| `SidebarWithMainLayout` | Sidebar with a main content region |

When omitted, the layout defaults to `Default`. Layout names are case-sensitive.

---

## 8. Widgets

A widget is declared inside its dashboard:

```ocean-dsl
Widget <WidgetName> of type <WidgetType>
```

Example:

```ocean-dsl
Widget TodoTable of type Table
    datatype = TodoItem
    selectable = true
```

Widget names must be unique within the dashboard. Widget syntax and shared semantics are defined by `dsl.widget`; supported types, properties, defaults, and functions are defined by `dsl.widget-predefined-types`.

---

## 9. Widget Properties

Widget properties may be scalar, list, or map values.

Scalar:

```ocean-dsl
title = Tasks Table
limit = 5
```

List — table `columns` and form `fields` entries pair a field name with its datatype:

```ocean-dsl
columns:
    - id      String
    - title   String
    - status  TaskLifecycle
    - dueDate DateTime
```

Map — `buttons` maps a button role to its label; the separator is ` : `:

```ocean-dsl
buttons:
    submit : Create Todo
    cancel : Cancel
    fetch  : 🔄 Fetch by ID
```

Scalar and label values are written without quotes and run to the end of the
line. Property names use lower camel case and are specific to the selected
widget type.

---

## 10. Metadata

Dashboards may use supported Ocean metadata, including groups, tags, and perspectives.

Metadata may support documentation, discovery, filtering, organization, and generation. It does not replace dashboard properties or UI connections.

The exact metadata syntax and semantics are defined by the corresponding metadata references.

---

## 11. Multiple Dashboards

One `@dashboard` file may define multiple dashboards:

```ocean-dsl
@dashboard

TodoListDashboard
    layout: SingleColumnLayout

    Widget TodoTable of type Table
        datatype = TodoItem

TodoEditDashboard
    layout: SingleColumnLayout

    Widget EditTodoForm of type Form
        datatype = TodoItem
```

Each dashboard forms a separate reusable presentation unit.

---

## 12. Usage in UI

A UI uses a dashboard through a dashboard-use declaration:

```ocean-dsl
@ui

TodoUI
    use dashboard TodoDashboard
```

The UI may connect widget functions to application capabilities:

```ocean-dsl
connect CreateTodoForm.Submit -> CreateTodo
```

The exact use and connection grammar is defined by `dsl.ui`. Dashboard declarations do not directly define API, broker, or service connections.

---

## 13. Backend Independence

A dashboard describes what is displayed and how its widgets are arranged.

It does not own:

- API endpoints;
- broker subscriptions or publications;
- service bindings;
- database access;
- deployment configuration;
- infrastructure credentials.

Those integrations are defined by the UI and other applicable DSL sections.

---

## 14. Complete Example

A complete `@dashboard` file
(`ocean-examples/0001-task-manager/app-todo-dashborad.ocn`):

```ocean-dsl
@dashboard

TodoViewDashboard
    title: Todo Items
    subtitle: See all your tasks!
    layout: TwoColumnLayout

    Widget TodoTable of type Table
        title = Tasks Table
        columns:
            - id String
            - title String
            - status TaskLifecycle
            - dueDate DateTime

TodoSearchDashboard
    title: Search Todos
    subtitle: Find tasks!
    layout: SingleColumnLayout

    Widget TodoSearchForm of type Form
        title = Find Task
        fields:
            - title String
            - priority Priority
        buttons:
            submit : Search
            cancel : Reset

TodoDeleteDashboard
    title: Delete Todos
    subtitle: Delete tasks!
    layout: SingleColumnLayout

    Widget TodoDeleteForm of type Form
        title = Delete Task
        fields:
            - id String
        buttons:
            submit : Delete
            cancel : Reset

TodoSubmitDashboard
    title: Create Todo
    subtitle: Add tasks!
    layout: SingleColumnLayout

    Widget CreateTodoForm of type Form
        title = Create Task
        fields:
            - title String
            - priority Priority
            - dueDate DateTime
        buttons:
            submit : Create Todo
            cancel : Cancel

TodoEditDashboard
    title: Edit Todo
    subtitle: change tasks!
    layout: TwoColumnLayout

    Widget EditTodoForm of type Form
        title = Edit Task
        fields:
            - id String
            - title String
            - dueDate DateTime
            - priority Priority
            - status TaskLifecycle
        buttons:
            submit : Update
            cancel : Cancel
            fetch : 🔄 Fetch by ID

AboutDashboard
    title: About
    subtitle: App information!
    layout: SingleColumnLayout

    Widget AboutText of type Text
        title = About ToDo App
        subtitle = This app helps you to organize your tasks 📝
        content = This is the Todo App. Version 1.0.0 ⚡ Built with Ocean-lab.
        # the source file repeats `content =` with a longer paragraph; abbreviated here

    Widget InfoButton of type Button
        color = secondary
```

This example demonstrates:

- multiple reusable dashboards in one file;
- unquoted `title` / `subtitle` values with spaces, punctuation, and emoji;
- `SingleColumnLayout` and `TwoColumnLayout`;
- a `Table` widget with typed `columns` entries (`- <field> <Type>`);
- `Form` widgets with typed `fields` and a `buttons` map (`submit`, `cancel`, `fetch`);
- `Text` and `Button` widgets with scalar properties.

---

## 15. Imports

The current dashboard contract does not define supported reusable item types for `@import`.

Dashboard imports should therefore not be treated as available merely because the parser can recognize generic import syntax. A supported import type must first be added to the normative dashboard contract and resolution model.

Reusable same-section composition may be performed through `@include` where supported by the common inclusion contract.

---

## 16. Validation

Dashboard validation includes:

- valid `@dashboard` header;
- valid and unique dashboard names;
- recognized dashboard properties;
- valid predefined layout;
- valid metadata;
- valid and unique widget names within each dashboard;
- supported widget types;
- valid widget property names and shapes;
- resolvable widget datatypes;
- valid widget-specific configuration;
- valid UI references to dashboards and widget functions.

An unknown dashboard property or unsupported layout is invalid.

---

## 17. Invalid Examples

Unknown layout:

```ocean-dsl
TodoDashboard
    layout: FourColumnLayout
```

Unknown dashboard property:

```ocean-dsl
TodoDashboard
    theme: OceanBlue
```

Invalid widget name:

```ocean-dsl
Widget todo-table of type Table
```

Unsupported widget type:

```ocean-dsl
Widget Calendar of type Calendar
```

---

## 18. Purpose

`@dashboard` exists to:

- model a reusable page or screen;
- group related widgets into one logical unit;
- define presentation titles and layouts;
- keep presentation composition backend-agnostic;
- allow UI definitions to reuse dashboards;
- provide generators with a typed, structured UI model;
- keep application integration at the `@ui` boundary.

---

## 19. Rules and Constraints

- A dashboard file starts with `@dashboard`.
- A file may define multiple dashboards.
- Dashboard names follow Ocean identifier conventions and are unique in scope.
- `title` and `subtitle` are optional.
- `layout` is optional and defaults to `Default`.
- Layouts must use a supported case-sensitive name.
- A dashboard may contain multiple widgets.
- Widget names are unique within their dashboard.
- Widget types must be supported by `dsl.widget-predefined-types`.
- Widget properties are type-specific and may be scalar, list, or map values.
- Dashboards are backend-agnostic.
- API, broker, service, and infrastructure connections belong to `@ui` or other relevant sections.
- The current contract supports no dashboard import item types.
- Dashboard use and widget-function connections are governed by `dsl.ui`.

---

## 20. Related Knowledge

- `dsl.widget` — defines widget declaration syntax and common semantics.
- `dsl.widget-predefined-types` — catalogs supported widgets, properties, defaults, and functions.
- `dsl.ui` — uses dashboards and owns application-level connections.
- `dsl.datatype` — defines structures used by data-bound widgets.
- `dsl.include` — defines same-section reusable composition where supported.
- `dsl.import` — defines selective reusable imports, though dashboard currently supports no import types.

These relationships are declared in the metadata.
