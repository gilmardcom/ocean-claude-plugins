# Scheduled Operations Monitor

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/examples/scheduled-operations-monitor/

**Contents**

- [Overview](#overview)
- [Schedule map](#schedule-map)
- [Files](#files)
- [DSL source files](#dsl-source-files)
- [Walkthrough](#walkthrough)

Model recurring operational work with reusable schedules, an API-backed service, and a generated monitoring UI.

Source: https://atlas.ocean-lab.ai/examples/scheduled-operations-monitor/

Level: medium · Estimated time: 20 minutes · Version: 1.0.0 · Onboarding example: no

Components: 4 services, 0 brokers, 0 databases

Tags: api, dashboard, expressions, operations, scheduling, timezone, ui

## Overview

This official Ocean DSL example demonstrates reusable, technology-independent
scheduling in a complete generated system: an API-backed service, dashboard,
HTMX UI, configuration, deployment model, datatypes, and expressions.

`OperationsMonitorService` demonstrates all logical schedule forms:

- an interval schedule reused by a scheduled connection;
- a daily operation with seconds and an IANA timezone;
- weekly connections with selected weekdays and an inline one-off schedule;
- monthly and inline weekly aggregate fan-out schedules;
- `skip`, `queue`, and `parallel` overlap behavior.

The UI is generated solely from the `@dashboard` and `@ui` definitions. Its
buttons call ordinary generated API endpoints, while the scheduler runs the
recurring flows independently in the generated service.

## Schedule map

| Logical schedule | Binding | Timezone | Overlap |
| --- | --- | --- | --- |
| `FastCollection` | `collect -> process` | n/a | skip |
| `DailyReporting` | `operation dailyReport` | Europe/Amsterdam | queue |
| `WeeklyReconciliation` | `collect -> reconcile` | UTC | parallel |
| `MonthlyClose` | aggregate API fan-out | UTC | skip |

The inline Friday and Sunday schedules intentionally show that one-off jobs do
not require a named definition.

## Files

DSL source comes first, then diagrams; the long Horizon and Voyage walkthroughs are at the end. You can stop after the source files.

| File | Section | Lines |
| --- | --- | --- |
| `00-som-info.ocn` | `@info` | 18 |
| `10-som-datatype.ocn` | `@datatype` | 26 |
| `20-som-api.ocn` | `@api` | 41 |
| `30-som-config.ocn` | `@config` | 23 |
| `40-som-expression.ocn` | `@expression` | 78 |
| `50-som-service.ocn` | `@service` | 50 |
| `51-som-data-service.ocn` | `@service` | 23 |
| `52-som-reporting-service.ocn` | `@service` | 21 |
| `55-som-dashboard.ocn` | `@dashboard` | 40 |
| `56-som-ui.ocn` | `@ui` | 41 |
| `60-som-deploy.ocn` | `@deploy` | 35 |

## DSL source files

### `00-som-info.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - scheduling
#   - operations-monitor
# perspective:
#   feature: scheduled-operations-monitor
# @ocean-meta-end

@info

name: Scheduled Operations Monitor
version: 1.0.0
title: Scheduled Operations Monitor
subtitle: Observe recurring operational work
shortDescription: A complete Ocean DSL example for reusable schedules and scheduled flows.

description: Demonstrates interval, daily, weekly, and monthly schedules through a generated service, API, dashboard, and UI.
```

### `10-som-datatype.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - datatype
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-monitor
# @ocean-meta-end

@datatype

OperationsOverview
    title       : String
    description : String
    timezone    : String

DailyOperationsReport
    title       : String
    generatedAt : String
    status      : String

MonthlyOperationsSummary
    pendingOperations    : String
    reconciliationState  : String
    monthlyStatus        : String
```

### `20-som-api.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - api
#   - rest-api
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-monitor
# @ocean-meta-end

@api

OperationsMonitorApi style:rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    basePath = /operations
    generateSwagger = true

    get /overview      getOverview()      : OperationsOverview
    get /daily-report  getDailyReport()   : DailyOperationsReport
    get /monthly-state getMonthlyState()  : MonthlyOperationsSummary

OperationsDataApi style:rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    basePath = /operations-data
    generateSwagger = true

    get /pending              getPendingOperations()    : String
    get /reconciliation-state getReconciliationState()  : String

OperationsReportingApi style:rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    basePath = /operations-reporting
    generateSwagger = true

    get /monthly-status getMonthlyStatus() : String
```

### `30-som-config.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - configuration
# perspective:
#   feature: scheduled-operations-monitor
#   service: all
# @ocean-meta-end

@config

@import config O.log.config@1.0.0 as LogConfig

OperationsMonitorConfig
    apiConfig : ApiConfig
    logConfig : LogConfig

ApiConfig
    port : Int (default=8080)

UiConfig
    port      : Int (default=8080)
    logConfig : LogConfig
```

### `40-som-expression.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - expression
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-monitor
# @ocean-meta-end

@expression

CollectPendingOperations
    input: _
    output: pending:String
    logic
        pending = "pending operational work"

ProcessPendingOperations
    input: pending:String
    output: _
    logic
        return

CreateDailyReport
    input: _
    output: _
    logic
        # message = "daily operational report created"
        return

ReconcileOperations
    input: pending:String
    output: _
    logic
        return

GetOperationsOverview
    input: _
    output: overview:OperationsOverview
    logic
        var result OperationsOverview
        result.title = "Scheduled Operations Monitor"
        result.description = "Interval, daily, weekly, and monthly jobs are configured in Ocean DSL."
        result.timezone = "UTC and Europe/Amsterdam"
        overview = result

GetDailyOperationsReport
    input: _
    output: report:DailyOperationsReport
    logic
        var result DailyOperationsReport
        result.title = "Daily operations report"
        result.generatedAt = "02:00:35 Europe/Amsterdam"
        result.status = "Scheduled"
        report = result

GetMonthlyOperationsSummary
    input: _
    output: summary:MonthlyOperationsSummary
    logic
        var result MonthlyOperationsSummary
        result.pendingOperations = "pending operational work"
        result.reconciliationState = "reconciliation scheduled"
        result.monthlyStatus = "monthly close scheduled"
        summary = result

GetReconciliationState
    input: _
    output: result:String
    logic
        result = "reconciliation scheduled"

GetMonthlyStatus
    input: _
    output: result:String
    logic
        result = "monthly close scheduled"
```

### `50-som-service.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - service
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-monitor
# @ocean-meta-end

@service

OperationsMonitorService

    @perspectives: version:1.0.0, lifestyle:stable

    use config OperationsMonitorConfig as svcCfg
    impl api OperationsMonitorApi as api on svcCfg.apiConfig.port
    use api OperationsDataApi as data
    use api OperationsReportingApi as reporting
    use expression CollectPendingOperations as collect
    use expression ProcessPendingOperations as process
    use expression CreateDailyReport as dailyReport
    use expression ReconcileOperations as reconcile
    use expression GetOperationsOverview
    use expression GetDailyOperationsReport
    use expression GetMonthlyOperationsSummary

    # Reusable, technology-independent schedule definitions.
    schedule FastCollection every 5m overlap skip
    schedule DailyReporting daily at 02:00:35 timezone Europe/Amsterdam overlap queue
    schedule WeeklyReconciliation weekly on Monday, Wednesday at 09:00 timezone UTC overlap parallel
    schedule MonthlyClose monthly on day 1 at 00:05 timezone UTC overlap skip

    # A direct scheduled operation.
    operation dailyReport schedule DailyReporting

    # Named and inline scheduled connections.
    connect collect -> process schedule FastCollection
    connect collect -> reconcile schedule WeeklyReconciliation
    connect collect -> reconcile schedule every Friday at 18:30 timezone Europe/Amsterdam overlap parallel

    # Scheduled aggregate fan-out calls dependent APIs, as aggregates do elsewhere in Ocean DSL.
    aggregate api.getMonthlyState -> data.getPendingOperations | data.getReconciliationState | reporting.getMonthlyStatus schedule MonthlyClose
    aggregate api.getMonthlyState -> data.getPendingOperations | data.getReconciliationState | reporting.getMonthlyStatus schedule weekly on Sunday at 23:30 timezone UTC overlap queue

    # UI-facing read endpoints are ordinary API connections.
    connect api.getOverview -> GetOperationsOverview
    connect api.getDailyReport -> GetDailyOperationsReport
    connect api.getMonthlyState -> GetMonthlyOperationsSummary
```

### `51-som-data-service.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - service
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-data
# @ocean-meta-end

@service

OperationsDataService

    @perspectives: version:1.0.0, lifestyle:stable

    use config OperationsMonitorConfig as svcCfg
    impl api OperationsDataApi as api on svcCfg.apiConfig.port
    use expression CollectPendingOperations
    use expression GetReconciliationState

    connect api.getPendingOperations -> CollectPendingOperations
    connect api.getReconciliationState -> GetReconciliationState
```

### `52-som-reporting-service.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - service
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-reporting
# @ocean-meta-end

@service

OperationsReportingService

    @perspectives: version:1.0.0, lifestyle:stable

    use config OperationsMonitorConfig as svcCfg
    impl api OperationsReportingApi as api on svcCfg.apiConfig.port
    use expression GetMonthlyStatus

    connect api.getMonthlyStatus -> GetMonthlyStatus
```

### `55-som-dashboard.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - dashboard
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-monitor-ui
# @ocean-meta-end

@dashboard

OperationsDashboard
    title: Scheduled Operations
    layout: SingleColumnLayout

    Widget OverviewButton of type Button
        label = Refresh schedule overview

    Widget DailyReportButton of type Button
        label = View daily report schedule
        color = secondary

    Widget MonthlyStateButton of type Button
        label = View monthly close schedule
        color = secondary

    Widget ScheduleGuide of type Text
        title = Reusable schedules
        subtitle = Jobs run independently with their own overlap policy.
        content = <p>The service demonstrates a five-minute interval, a daily report at 02:00:35 in Europe/Amsterdam, a weekly reconciliation, and a monthly close. The dashboard calls generated API endpoints while scheduled flows run independently in the generated service.</p>

AboutDashboard
    title: About scheduling
    layout: SingleColumnLayout

    Widget AboutText of type Text
        title = Ocean DSL scheduling
        subtitle = Logical schedules, target-specific generation
        content = <p>Schedule definitions describe recurrence, timezone, and overlap behavior without exposing a scheduler implementation. A definition may be reused by operations, connections, and aggregates; inline schedules remain available for one-off jobs.</p>
```

### `56-som-ui.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - ui
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: operations-monitor-ui
# @ocean-meta-end

@ui

OperationsMonitorUi

    @perspectives: version:1.0.0, lifestyle:stable

    framework: htmx
    styling: bootstrap
    template: go-html-template
    backend: go-gin
    static: static
    configType: UiConfig

    nav Operations dashboard = OperationsDashboard
    nav About dashboard = AboutDashboard

    header title = ⏱️ Scheduled Operations Monitor
    header subtitle = Observe the recurring work configured in Ocean DSL
    header align = center

    footer title = ⚡ Built with Ocean-lab
    footer subtitle = Reusable scheduling example
    footer align = center

    use dashboard OperationsDashboard as operations
    use dashboard AboutDashboard as about
    use api OperationsMonitorApi

    connect operations.OverviewButton.click -> OperationsMonitorApi.getOverview
    connect operations.DailyReportButton.click -> OperationsMonitorApi.getDailyReport
    connect operations.MonthlyStateButton.click -> OperationsMonitorApi.getMonthlyState
```

### `60-som-deploy.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - deployment
#   - scheduling
# perspective:
#   feature: scheduled-operations-monitor
#   service: all
# @ocean-meta-end

@deploy

Name: Scheduled-Operations-Monitor-deploy

OperationsMonitorDeploy
    service OperationsMonitorService
    replica 1
    export 9095:OperationsMonitorService.api
    dependsOn OperationsDataDeploy, OperationsReportingDeploy

OperationsDataDeploy
    service OperationsDataService
    replica 1
    export 9096:OperationsDataService.api

OperationsReportingDeploy
    service OperationsReportingService
    replica 1
    export 9097:OperationsReportingService.api

OperationsMonitorUiDeploy
    service OperationsMonitorUi
    replica 1
    export 8085:OperationsMonitorUi.config
    dependsOn OperationsMonitorDeploy
```

## Walkthrough

A long, step-by-step explanation (Horizon and Voyage) is in [Scheduled Operations Monitor walkthrough](scheduled-operations-monitor-walkthrough.md). Read it only if you need the reasoning behind the design.
