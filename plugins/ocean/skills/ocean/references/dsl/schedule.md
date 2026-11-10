# Schedule and Trigger DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/dsl/schedule/

**Contents**

- [1. Overview](#1-overview)
- [2. Scope and Applicable Items](#2-scope-and-applicable-items)
- [3. Named Schedule Definitions](#3-named-schedule-definitions)
- [4. Schedule Values](#4-schedule-values)
- [5. Timezones and Daylight Saving Time](#5-timezones-and-daylight-saving-time)
- [6. Overlap Behavior](#6-overlap-behavior)
- [7. Using a Schedule](#7-using-a-schedule)
- [8. Complete Service Example](#8-complete-service-example)
- [9. Validation Rules](#9-validation-rules)
- [10. Generator Responsibility](#10-generator-responsibility)
- [11. Related Knowledge](#11-related-knowledge)

Defines reusable, technology-independent schedules and their use as triggers for executable Ocean DSL items.

Source: https://atlas.ocean-lab.ai/dsl/schedule/

Status: Accepted · Authority: Normative

## 1. Overview

Scheduling describes **when** an executable Ocean DSL item runs. It does not
describe a timer library, cron implementation, operating-system service, or
cloud scheduler. A generator resolves the logical schedule into the target
runtime implementation.

Schedules are local to an executable section instance. They may be declared by
name and reused in that instance, or written inline on a scheduled item.

The common model supports fixed intervals, daily, weekly, and monthly calendar
schedules; times with optional seconds; IANA timezones; and overlap behavior.

---

## 2. Scope and Applicable Items

A schedule definition belongs to the containing executable model, rather than
to a global `@schedule` section. A name declared in one service, UI, or FSM is
not visible in another one.

Schedules may be attached to executable items such as:

- `operation` — runs one injected expression without an event payload;
- `connect` — invokes a source and routes its value to one or more targets;
- `aggregate` — triggers an aggregate flow.

The current Go generator executes scheduled bindings in `@service`. Schedule
definitions and uses are also represented by the common parser/AST in `@ui`
and `@fsm`; generators for those section lifecycles may be added separately.

---

## 3. Named Schedule Definitions

General syntax:

```ocean-dsl
schedule <name> <schedule-value> [timezone <IANA-zone>] [overlap <behavior>]
```

Examples:

```ocean-dsl
schedule FastSync every 10s overlap skip
schedule DailyExport daily at 14:00:35 timezone Europe/Amsterdam overlap queue
schedule WeeklyCleanup weekly on Monday, Wednesday at 09:00 timezone UTC
schedule MonthlyBilling monthly on day 1 at 00:05 overlap parallel
```

`<name>` follows the normal Ocean identifier rules and must be unique within
its containing executable model.

If overlap is omitted, the normalized default is `skip`. If timezone is
omitted for a calendar schedule, the default is `UTC`.

---

## 4. Schedule Values

### 4.1 Fixed interval

```ocean-dsl
every <positive-integer><unit>
```

Supported units are `ms`, `s`, `m`, and `h`.

```ocean-dsl
schedule PollInventory every 30s
schedule CompactCache every 5m
schedule RefreshProjection every 1h
```

Intervals describe elapsed time. They cannot use `at` or `timezone`.

### 4.2 Daily

```ocean-dsl
daily at HH:mm[:ss]
```

```ocean-dsl
schedule DailyExport daily at 14:00
schedule NightlyReport daily at 02:00:35 timezone Europe/Amsterdam
```

### 4.3 Weekly

```ocean-dsl
weekly on <weekday>[, <weekday>...] at HH:mm[:ss]
```

```ocean-dsl
schedule Reconcile weekly on Monday, Wednesday, Friday at 09:00 timezone UTC
```

Weekday names are `Monday`, `Tuesday`, `Wednesday`, `Thursday`, `Friday`,
`Saturday`, and `Sunday`. Each weekday may occur at most once.

For one weekday, this equivalent alias is accepted:

```ocean-dsl
schedule FridayClose every Friday at 18:30 timezone Europe/Amsterdam
```

### 4.4 Monthly

```ocean-dsl
monthly on day <1..31> at HH:mm[:ss]
```

```ocean-dsl
schedule MonthStart monthly on day 1 at 00:05 timezone UTC
```

For a day that does not exist in a particular month, such as day `31` in
February, that month's occurrence is skipped; it does not roll into the next
month.

---

## 5. Timezones and Daylight Saving Time

`timezone` is optional for daily, weekly, and monthly schedules and must be an
IANA timezone ID or `UTC`.

```ocean-dsl
schedule AmsterdamReport daily at 02:00:35 timezone Europe/Amsterdam
schedule USClose weekly on Friday at 17:00 timezone America/New_York
```

Abbreviations such as `CET`, the host-dependent value `Local`, and unknown
timezone IDs are invalid. Calendar schedules use their declared timezone, so
local daylight-saving transitions are handled by the target runtime's timezone
rules. Use `UTC` when a fixed global instant is required.

---

## 6. Overlap Behavior

An overlap occurs when a trigger fires while a previous execution of the same
scheduled binding is still running.

```ocean-dsl
overlap skip
overlap queue
overlap parallel
```

| Behavior | Meaning |
| --- | --- |
| `skip` | Do not start another execution while one is running. This is the default. |
| `queue` | Request one follow-up execution after the current execution completes. Repeated triggers are coalesced into that pending run. |
| `parallel` | Start every trigger independently; concurrent executions are allowed. |

Choose `skip` for idempotent maintenance, `queue` when one eventual follow-up
matters, and `parallel` only when concurrent execution is safe.

---

## 7. Using a Schedule

Attach a trailing `schedule` clause to an executable statement. The clause
contains either a local schedule name or an inline schedule value.

### 7.1 Operations

```ocean-dsl
operation createDailyReport schedule DailyExport
operation cleanupExpiredSessions schedule daily at 02:00 timezone UTC overlap skip
```

The scheduled operation must not require an input payload.

### 7.2 Connections

```ocean-dsl
connect collectPendingOperations -> processPendingOperations schedule FastSync
connect collectPendingOperations -> reconcileOperations schedule weekly on Monday at 09:00 timezone UTC overlap parallel
```

The source must be invocable without an input payload. A scheduled connection
can supply its one produced domain value to targets that accept one input. The
generator rejects incompatible source and target signatures instead of emitting
an inert or invalid job.

### 7.3 Aggregates

```ocean-dsl
aggregate api.getMonthlyState -> data.getPendingOperations | reporting.getMonthlyStatus schedule MonthlyBilling
```

In the current Go service generator, scheduled aggregates trigger the same
generated aggregate handler used by the normal service API flow. Aggregate
sources and targets must therefore meet the aggregate/API contract.

---

## 8. Complete Service Example

```ocean-dsl
@service

OperationsMonitorService
    use expression CollectPendingOperations as collect
    use expression ProcessPendingOperations as process
    use expression CreateDailyReport as dailyReport
    use expression ReconcileOperations as reconcile
    impl api OperationsMonitorApi as api on myCfg.apiConfig.port
    use api OperationsDataApi as data
    use api OperationsReportingApi as reporting

    schedule FastSync every 5m overlap skip
    schedule DailyReport daily at 02:00:35 timezone Europe/Amsterdam overlap queue
    schedule Reconciliation weekly on Monday, Wednesday at 09:00 timezone UTC overlap parallel
    schedule MonthlyClose monthly on day 1 at 00:05 timezone UTC overlap skip

    operation dailyReport schedule DailyReport
    connect collect -> process schedule FastSync
    connect collect -> reconcile schedule Reconciliation
    aggregate api.getMonthlyState -> data.getPendingOperations | reporting.getMonthlyStatus schedule MonthlyClose
```

The example demonstrates named schedule reuse, interval and calendar schedules,
seconds, timezones, all three overlap policies, operations, connections, and
aggregates.

---

## 9. Validation Rules

The parser and validator reject invalid schedule declarations and uses with a
diagnostic at the relevant source line.

| Invalid form | Reason |
| --- | --- |
| `schedule Bad every 0s` | Interval must be positive. |
| `schedule Bad every 10d` | `d` is not a supported interval unit. |
| `schedule Bad daily at 9:00` | Time must use `HH:mm` or `HH:mm:ss`. |
| `schedule Bad weekly at 09:00` | Weekly schedules require one or more weekdays. |
| `schedule Bad monthly on day 32 at 09:00` | Day must be between `1` and `31`. |
| `schedule Bad every 10s timezone UTC` | Timezone is not valid for intervals. |
| `schedule Bad daily at 09:00 overlap unknown` | Overlap must be `skip`, `queue`, or `parallel`. |
| `connect A -> B schedule Missing` | `Missing` is not defined in the local schedule scope. |

Named schedules cannot be duplicated in the same executable model. A schedule
clause must be trailing and may not be contradictory, for example by declaring
multiple timezone or overlap clauses.

---

## 10. Generator Responsibility

The DSL stores only logical intent. It never exposes cron syntax, a Go package,
or a cloud scheduling product.

The Go service generator currently emits a reusable scheduler runtime and one
job per scheduled binding. It resolves intervals and calendar occurrences,
loads declared timezones, applies overlap behavior, starts jobs after service
dependencies are ready, propagates handler errors to the service logger, and
stops jobs during shutdown.

Other generators may resolve the same model differently, for example through a
Java scheduler, Kubernetes CronJob, workflow engine, or managed platform
trigger, without changing the DSL source.

---

## 11. Related Knowledge

- [Service DSL Reference](service.md) — service composition and executable connections.
- [Expression DSL Reference](expression.md) — reusable expression definitions.
- [FSM DSL Reference](fsm.md) — state-machine definitions.
- [UI DSL Reference](ui.md) — deployable UI services.
- [Integration DSL Reference](integration.md) — integration-oriented flow composition.
