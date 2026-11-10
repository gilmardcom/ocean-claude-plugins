# UI DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/ui/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax Overview](#3-syntax-overview)
- [4. UI Names](#4-ui-names)
- [5. Technology Properties](#5-technology-properties)
- [6. Framework](#6-framework)
- [7. Styling](#7-styling)
- [8. Template](#8-template)
- [9. Backend](#9-backend)
- [10. Static Path](#10-static-path)
- [11. Configuration Type](#11-configuration-type)
- [12. Navigation](#12-navigation)
- [13. Navigation Examples](#13-navigation-examples)
- [14. Header](#14-header)
- [15. Footer](#15-footer)
- [16. Using Dashboards](#16-using-dashboards)
- [17. Using APIs](#17-using-apis)
- [18. Using Brokers](#18-using-brokers)
- [19. Connections](#19-connections)
- [20. Widget-to-backend Flow](#20-widget-to-backend-flow)
- [21. Backend-to-widget Flow](#21-backend-to-widget-flow)
- [22. Mappers and Fan-out](#22-mappers-and-fan-out)
- [23. Type Compatibility](#23-type-compatibility)
- [24. Imports](#24-imports)
- [25. Includes](#25-includes)
- [26. Complete Example](#26-complete-example)
- [27. Deployment Boundary](#27-deployment-boundary)
- [28. Validation](#28-validation)
- [29. Invalid Examples](#29-invalid-examples)
- [30. Rules and Constraints](#30-rules-and-constraints)
- [31. Related Knowledge](#31-related-knowledge)

Defines the Ocean `@ui` section for composing dashboards into deployable user-interface services and connecting widget functions to APIs and brokers.

Source: https://atlas.ocean-lab.ai/dsl/ui/

Status: Approved · Authority: Normative

## 1. Overview

The `@ui` section defines deployable user-interface services.

A UI composes one or more dashboards, selects generation technologies, defines navigation and page chrome, and connects widget functions to APIs and brokers.

Dashboards remain backend-agnostic. The UI owns runtime backend wiring and the generated presentation-service boundary.

---

## 2. Core Principle

```text
Dashboards and Widgets
          │
          ▼
        @ui
     ├── framework
     ├── styling
     ├── templates
     ├── backend
     ├── navigation
     └── typed connections
          │
          ▼
      APIs / Brokers
```

The UI turns reusable presentation models into an interactive application without moving backend integration into dashboard definitions.

---

## 3. Syntax Overview

```ocean-dsl
@ui

<UiName>
    @perspectives: <key>:<value>, ...

    framework: <Framework>
    styling: <Styling>
    template: <Template>
    backend: <Backend>
    static: <Path>
    configType: <ConfigName>

    nav <Label> <dashboard|link|file> = <Target>

    header title = <Value>
    header subtitle = <Value>
    header align = <Value>

    footer title = <Value>
    footer subtitle = <Value>
    footer align = <Value>

    use dashboard <DashboardName> [as <alias>]
    use api <ApiName> [as <alias>]
    use broker <BrokerName> as <alias>

    connect <source> -> <target>
```

A file may define multiple UIs.

The technology properties (`framework:`, `styling:`, `template:`, `backend:`,
`static:`, `configType:`) use a colon. The `nav`, `header`, and `footer` lines
do not; their values are written unquoted and run to the end of the line. An
optional `@perspectives:` line may follow the UI name.

---

## 4. UI Names

UI names follow the Ocean type-name convention and are unique within the resolved UI scope.

Examples:

```text
TodoUI
AdminPortal
OperationsConsole
```

The UI name identifies the logical deployable presentation unit. A generator may derive target-specific artifact and image names from it.

---

## 5. Technology Properties

A UI may select four complementary generation concerns:

| Property    | Responsibility                       |
| ----------- | ------------------------------------ |
| `framework` | Client interaction or behavior model |
| `styling`   | Presentation styling system          |
| `template`  | Markup or template generation system |
| `backend`   | Runtime used to serve the UI         |

These properties are case-sensitive and accept only implemented values.

---

## 6. Framework

```ocean-dsl
framework: htmx
```

Supported values:

```text
htmx
appsmith
custom
```

The framework determines how generated UI behavior and interaction are represented.

---

## 7. Styling

```ocean-dsl
styling: bootstrap
```

Supported values:

```text
bootstrap
tailwind
none
```

Styling affects generated presentation, not the semantic types of widget connections.

---

## 8. Template

```ocean-dsl
template: go-html-template
```

Supported values:

```text
go-html-template
static-html
custom
```

The template selects how the UI's markup or view artifacts are produced.

---

## 9. Backend

```ocean-dsl
backend: go-gin
```

Supported values:

```text
go-gin
go-std-http
custom
```

The backend determines how the generated UI service is hosted and how server-side UI behavior is implemented.

---

## 10. Static Path

The optional `static` property identifies the folder used for static UI content:

```ocean-dsl
static: ./generated/ui
```

The path is interpreted by the active UI generator. Materialized output must remain within the applicable generated project boundary.

Static content must not overwrite protected generated output or other owned files.

---

## 11. Configuration Type

The optional `configType` property references the UI's configuration schema:

```ocean-dsl
configType: UiConfig
```

The reference must resolve to a valid `@config` schema. Nested configuration schemas are resolved transitively according to `dsl.config`.

Unlike `@service`, the current UI grammar uses the `configType` property rather than `use config ... as ...`.

---

## 12. Navigation

Navigation entries use:

```ocean-dsl
nav <Label> <Type> = <Target>
```

(`navigation:` is also accepted where supported by the grammar; the examples use
`nav`.)

Supported types are:

| Type        | Target                         |
| ----------- | ------------------------------ |
| `dashboard` | A dashboard used by the UI     |
| `link`      | An external or application URL |
| `file`      | A static file path             |

---

## 13. Navigation Examples

```ocean-dsl
nav View dashboard = TodoViewDashboard
nav Company link = https://company.com
nav Schema file = static/schema.pdf
```

Dashboard targets must resolve to dashboards available to the UI.

The default dashboard is selected from navigation: an entry labelled `Home` is preferred; otherwise the first entry is used. When navigation is empty, no navigated dashboard target is available.

---

## 14. Header

Header properties use `header` (no colon), one field per line, with unquoted
values:

```ocean-dsl
header title = 📝 ToDo App
header subtitle = Simple Task Manager
header align = center
```

Supported header fields are:

- `title`;
- `subtitle`;
- `align`.

They control generated page chrome and do not affect connection semantics.

---

## 15. Footer

Footer properties use `footer` (no colon), one field per line, with unquoted
values:

```ocean-dsl
footer title = ⚡ Built with Ocean-lab
footer subtitle = Version 1.0.0
footer align = center
```

Supported footer fields are `title`, `subtitle`, and `align`.

---

## 16. Using Dashboards

```ocean-dsl
use dashboard TodoDashboard
```

or:

```ocean-dsl
use dashboard TodoDashboard as todo
```

Dashboard aliases are optional. Every used dashboard must resolve, and effective local names must be unique.

Widgets are referenced through the dashboard's local name:

```text
<dashboard>.<widget>.<function>
```

---

## 17. Using APIs

```ocean-dsl
use api TodoAPI
```

or:

```ocean-dsl
use api TodoAPI as api
```

APIs represent backend contracts called or observed by the UI. API aliases are optional.

The current UI model consumes APIs; it does not use the service-level `impl api ... on ...` form. The examples declare `use api <ApiName>` without an alias and reference operations by the API's full name (`TodoApi.listItems`).

---

## 18. Using Brokers

```ocean-dsl
use broker TodoBroker as broker
```

The broker alias is mandatory. Broker topics may serve as sources or targets of UI connections according to their messaging patterns.

The current implementation assumes one principal broker in some generation paths; models should not rely on multiple-broker behavior unless the selected target supports it.

---

## 19. Connections

The canonical UI connection syntax is:

```ocean-dsl
connect <source> -> <target>
    authorize context.<name>
        <authorization-policy>
```

Example:

```ocean-dsl
connect submit.CreateTodoForm.submit -> TodoApi.createItem
```

Connection endpoints are resolved from the APIs, brokers, and dashboards used by
the UI. Widget function names are lower camel case (`submit`, `fetch`, `click`,
`listRows`); see `dsl.widget-predefined-types`.

`bind <destination> <- context.<name>` is not valid beneath a UI connection.
Trusted request-context binding is currently implemented only for `@service`
connections, where the target is service logic with a typed parameter. The UI
uses the public API boundary and must not establish or project `Ocean-Context`
claims itself.

A UI connection may declare `authorize context.<name>` as a declarative policy
for UI runtimes that receive verified Ocean context. The UI never creates that
context; its hosting runtime must provide it. Policy syntax and semantics are
defined by [`dsl.authorization`](authorization.md).

---

## 20. Widget-to-backend Flow

A widget function may trigger an API operation or broker topic. Table widgets
load their data by connecting their `listRows` function to a list operation:

```ocean-dsl
connect view.TodoTable.listRows        -> TodoApi.listItems
connect submit.CreateTodoForm.submit   -> TodoApi.createItem
connect edit.EditTodoForm.fetch        -> TodoApi.getItem
connect about.InfoButton.click         -> TodoApi.getInfo
```

The widget function's output must be compatible with the target input.

Predefined widget functions and signatures are defined by `dsl.widget-predefined-types`.

---

## 21. Backend-to-widget Flow

Where supported, a backend output may target a widget function that accepts the
corresponding data:

```ocean-dsl
connect broker.todoUpdated -> view.TodoTable.refresh
```

Direction is determined by the source and target endpoints. Separate `publish`
and `subscribe` keywords are not part of the current canonical parser; both
directions use `connect`. In the current examples every connection runs from a
widget function to an API operation (including table loads via `listRows`).

---

## 22. Mappers and Fan-out

A connection may use the common mapper form when explicit type conversion is required:

```ocean-dsl
connect view.TodoTable.listRows -MapTodoRows-> TodoApi.listItems
```

A source may have multiple targets:

```ocean-dsl
connect submit.CreateTodoForm.submit -> TodoApi.createItem, broker.todoCreated
```

The mapper must resolve to a compatible expression. Every expanded target connection is validated independently.

---

## 23. Type Compatibility

UI connections are typed.

Ocean validates:

- endpoint existence;
- endpoint direction;
- function existence;
- input and output signatures;
- source-to-target compatibility;
- mapper compatibility;
- messaging-pattern compatibility.

An incompatible connection without a valid mapper is invalid.

---

## 24. Imports

The UI section supports imports of:

- `api`;
- `broker`;
- `config`;
- `dashboard`.

```ocean-dsl
@ui

@import api O.std.todo.TodoAPI@1.0.0 as TodoAPI
@import broker O.std.event.TodoBroker@1.0.0 as TodoBroker
@import dashboard O.std.ui.TodoDashboard@1.0.0 as TodoDashboard
@import config O.std.ui.UiConfig@1.0.0 as UiConfig
```

`P.` references may be used for compatible local Pre-baked items. Imported definitions are referenced through their import aliases.

---

## 25. Includes

A UI file may include compatible `@ui` definitions through `@include`.

Included and local UIs share one logical section scope and are validated together. Cross-section inclusion and cyclic inclusion are invalid.

---

## 26. Complete Example

A complete `@ui` file
(`ocean-examples/0001-task-manager/app-todo-ui.ocn`):

```ocean-dsl
@ui

TodoUi

    @perspectives: version:0.1.0, lifestyle:stable

    framework: htmx
    styling: bootstrap
    template: go-html-template
    backend: go-gin
    static: static
    configType: TodoUiConfig

    nav View dashboard = TodoViewDashboard
    nav Search dashboard = TodoSearchDashboard
    nav Submit dashboard = TodoSubmitDashboard
    nav Edit dashboard = TodoEditDashboard
    nav Delete dashboard = TodoDeleteDashboard
    nav About dashboard = AboutDashboard
    nav Company link = https://company.com
    nav Schema file = static/schema.pdf

    header title = 📝 ToDo App
    header subtitle = Simple Task Manager
    header align = center

    footer title = ⚡ Built with Ocean-lab
    footer subtitle = Version 1.0.0
    footer align = center

    use dashboard TodoViewDashboard as view
    use dashboard TodoSearchDashboard as search
    use dashboard TodoSubmitDashboard as submit
    use dashboard TodoEditDashboard as edit
    use dashboard TodoDeleteDashboard as delete
    use dashboard AboutDashboard as about

    use api TodoApi

    connect view.TodoTable.listRows -> TodoApi.listItems
    connect search.TodoSearchForm.submit -> TodoApi.findItemByTitleOrPriority # could be AND
    connect submit.CreateTodoForm.submit -> TodoApi.createItem
    connect edit.EditTodoForm.submit -> TodoApi.adjustItem
    connect edit.EditTodoForm.fetch -> TodoApi.getItem
    connect delete.TodoDeleteForm.submit -> TodoApi.deleteItem
    connect about.InfoButton.click -> TodoApi.getInfo
```

This example demonstrates:

- a `@perspectives:` line and the colon-style technology properties;
- colon-less `nav`, `header`, and `footer` lines with unquoted values (including emoji);
- `dashboard`, `link`, and `file` navigation targets;
- one aliased `use dashboard` per dashboard and an un-aliased `use api`;
- widget-function-to-API connections using the API's full name and lower-camel-case
  functions (`listRows`, `submit`, `fetch`, `click`).

---

## 27. Deployment Boundary

An `@ui` definition represents a deployable presentation service. It determines the technology combination and integration model; `@deploy` determines environment-specific materialization.

```text
@ui     → logical presentation service
@deploy → target and environment deployment
```

The generated UI may include templates, static content, backend code, configuration, and connection integration.

---

## 28. Validation

Validation includes:

- valid and unique UI name;
- supported framework, styling, template, and backend values;
- valid static path;
- resolvable configuration type;
- valid navigation type and target;
- valid header and footer fields;
- resolvable API, broker, and dashboard uses;
- required broker aliases;
- unique effective local names;
- valid connection endpoints and functions;
- source/target type compatibility;
- mapper compatibility;
- supported imports and includes.

Unknown UI properties are invalid.

---

## 29. Invalid Examples

Unsupported framework:

```ocean-dsl
framework: react
```

Missing broker alias:

```ocean-dsl
use broker TodoBroker
```

Unsupported UI property:

```ocean-dsl
layout: SimpleLayout
```

Layouts belong to dashboards; `layout` is not a current UI property.

Non-canonical connection keyword:

```ocean-dsl
publish todo.Form.Submit -> api.create
```

Use `connect` in the current grammar.

---

## 30. Rules and Constraints

- A UI file starts with `@ui`.
- A file may define multiple UIs.
- UI names follow Ocean identifier conventions and are unique in scope.
- Framework values are `htmx`, `appsmith`, or `custom`.
- Styling values are `bootstrap`, `tailwind`, or `none`.
- Template values are `go-html-template`, `static-html`, or `custom`.
- Backend values are `go-gin`, `go-std-http`, or `custom`.
- `configType` references a valid config schema.
- An optional `@perspectives:` line may follow the UI name.
- Technology properties use a colon (`framework:`); `nav`, `header`, and `footer` lines do not, and their values are unquoted.
- Navigation entries use `nav <Label> <Type> = <Target>`; types are `dashboard`, `link`, and `file`.
- Headers and footers support `title`, `subtitle`, and `align`.
- Widget function names are lower camel case; connections reference API operations by the API's full name (or alias, when declared).
- API and dashboard aliases are optional.
- Broker aliases are required.
- Connections use `connect`; `publish` and `subscribe` are not current keywords.
- Connection endpoints must belong to used APIs, brokers, or dashboards.
- Connections are directional and typed.
- Incompatible types require an explicit compatible mapper.
- Fan-out is allowed where supported.
- UI imports support API, broker, config, and dashboard definitions.
- Dashboard layout is defined by `dsl.dashboard`, not `dsl.ui`.
- Direct `use service` is not part of the current UI grammar.
- Environment-specific deployment belongs to `dsl.deploy`.

---

## 31. Related Knowledge

- `dsl.dashboard` — defines reusable pages and screens composed by a UI.
- `dsl.widget` — defines widgets contained in dashboards.
- `dsl.widget-predefined-types` — defines built-in widget properties and functions.
- `dsl.api` — defines backend contracts used by the UI.
- `dsl.broker` — defines messaging topics used by UI connections.
- `dsl.config` — defines the UI configuration schema.
- `dsl.import` and `dsl.include` — define reuse mechanisms.
- `dsl.deploy` — defines deployment materialization.

These relationships are declared in the metadata.
