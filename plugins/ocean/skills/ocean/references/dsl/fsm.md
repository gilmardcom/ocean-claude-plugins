# FSM DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/dsl/fsm/

**Contents**

- [1. Overview](#1-overview)
- [2. FSM Types](#2-fsm-types)
- [3. FSM Declaration](#3-fsm-declaration)
- [4. FSM Declaration Elements](#4-fsm-declaration-elements)
- [5. Events](#5-events)
- [6. States](#6-states)
- [7. Special States](#7-special-states)
- [8. State Triggers](#8-state-triggers)
- [9. Handler Types](#9-handler-types)
- [10. Supported Actions](#10-supported-actions)
- [11. State Transitions](#11-state-transitions)
- [12. `this`](#12-this)
- [13. Expressions](#13-expressions)
- [14. Context](#14-context)
- [15. Database](#15-database)
- [16. Execution Model](#16-execution-model)
- [17. Complete Entity-Controlled Example](#17-complete-entity-controlled-example)
- [18. Complete In-Memory Example](#18-complete-in-memory-example)
- [19. Internal Normalization](#19-internal-normalization)
- [20. Imports](#20-imports)
- [21. Rules and Constraints](#21-rules-and-constraints)
- [22. Related Knowledge](#22-related-knowledge)

Defines the Ocean `@fsm` DSL section for modeling stateful domain behavior using entity-controlled or in-memory finite-state machines, states, events, triggers, transitions, actions, and reusable logic.

Source: https://atlas.ocean-lab.ai/dsl/fsm/

Status: Accepted · Authority: Normative

## 1. Overview

The `@fsm` section defines a Finite-State Machine (FSM) that models domain behavior over time through states and transitions.

FSMs respond to abstract events, execute logic, transition between states, and may emit outputs.

Ocean supports two types of FSM:

- Entity-Controlled FSM;
- In-Memory FSM.

A file begins with:

```ocean-dsl
@fsm
```

---

## 2. FSM Types

### 2.1 Entity-Controlled FSM

An Entity-Controlled FSM controls a specific field of an existing datatype, typically an enum field representing lifecycle state.

Syntax:

```ocean-dsl
<FSMName> controls <Datatype>.<field>
```

Example:

```ocean-dsl
OrderFSM controls Order.status
```

Characteristics:

- controls a field of an entity;
- state is persisted through the database;
- execution itself is stateless;
- states are normally inferred from the controlled enum;
- requires an entity key;
- may name a database explicitly with `database:`; when omitted, the entity's
  database is inferred.

Typical use cases include lifecycle management such as:

```text
Order.status
Task.state
Shipment.status
```

Example:

```ocean-dsl
@fsm

OrderFSM controls Order.status
    key: id
    database: OrderDB
    event confirm in(_) out(_)

    state Created
        on event confirm : logic
            next Confirmed

    state Confirmed
        on enter : external
            source: "order_hooks.go"
            method: "NotifyConfirmed"
```

---

### 2.2 In-Memory FSM

An In-Memory FSM owns its state in memory.

Syntax:

```ocean-dsl
<FSMName> in-memory
```

Example:

```ocean-dsl
CalcFSM in-memory
```

An in-memory FSM uses a context and explicitly defines its states and default state.

Example:

```ocean-dsl
@fsm

CalcFSM in-memory
    use context CalcCtx as ctx
    states  : NoOp, Adding
    default : NoOp
    event calculate in(a:Float & b:Float) out(r:Float)

    state NoOp
        on event calculate:
            emit r = 0

    state Adding
        on event calculate : logic
            emit r = Add(a, b)

        on enter : external
            source: "calc_hooks.go"
            method: "OnEnterAdding"

        on each 5s : placeholder

        on exit : empty
```

Characteristics:

- owns its internal state;
- uses a context;
- state is held in memory;
- state is not persisted between runs;
- states must be explicitly declared;
- a default state is required.

Typical use cases include:

- session flows;
- wizards;
- bots;
- temporary workflows;
- short-lived orchestration.

---

## 3. FSM Declaration

### 3.1 Entity-Controlled FSM

General structure:

```ocean-dsl
<FSMName> controls <Datatype>.<field>
    key: <field>
    [database: <DatabaseName>]
    event <eventName> in(<inputs>) out(<outputs>)

    state <StateName>
        <handlers>
```

The `key:`, optional `database:`, and `event` lines, and any `use expression`
declarations, precede the `state` blocks; their relative order is not
significant.

### 3.2 In-Memory FSM

General structure:

```ocean-dsl
<FSMName> in-memory
    use context <ContextName> as <alias>
    states  : <State1>, <State2>, ...
    default : <DefaultState>
    event <eventName> in(<inputs>) out(<outputs>)
    use expression <ExpressionName>

    state <StateName>
        <handlers>
```

---

## 4. FSM Declaration Elements

| Element      | Entity-Controlled FSM                           | In-Memory FSM    |
| ------------ | ----------------------------------------------- | ---------------- |
| `controls`   | `Datatype.field`                                | Not used         |
| `context`    | Not used                                        | Required         |
| `key`        | Required to locate the entity                   | Not used         |
| `database`   | Optional; inferred from the entity when omitted | Not used         |
| `states`     | Inferred from controlled enum                   | Must be declared |
| `default`    | Optional                                        | Required         |
| `event`      | Supported                                       | Supported        |
| `expression` | Reusable logic                                  | Reusable logic   |

---

## 5. Events

FSMs receive abstract events.

Syntax:

```ocean-dsl
event <eventName> in(<inputs>) out(<outputs>)
```

Example:

```ocean-dsl
event calculate in(a:Float & b:Float) out(r:Float)
```

Multiple inputs or outputs are separated using `&`.

No input:

```ocean-dsl
event confirm in(_) out(_)
```

No output:

```ocean-dsl
event cancel in(id:String) out(_)
```

`_` indicates the absence of input or output.

### 5.1 Entity-Controlled FSM Key

Each event of an Entity-Controlled FSM implicitly receives the field declared by `key:` as input.

Example:

```ocean-dsl
OrderFSM controls Order.status
    key: id
```

The entity key is therefore available when processing FSM events without being explicitly repeated in every event declaration.

---

## 6. States

A state is declared using:

```ocean-dsl
state <StateName>
```

Example:

```ocean-dsl
state Created
```

Each state may define handlers that react to events or lifecycle triggers.

Example:

```ocean-dsl
state Created
    on enter:
        emit started

    on event confirm:
        next Confirmed

    on exit:
        emit completed
```

---

## 7. Special States

Two state names have special meaning:

```text
Every
Error
```

### `Every`

`Every` may define behavior that applies across states according to FSM runtime semantics.

### `Error`

`Error` may define behavior associated with FSM error handling.

These names are recognized specially by the FSM system.

---

## 8. State Triggers

Supported triggers are:

| Trigger | Syntax                 | Description                              |
| ------- | ---------------------- | ---------------------------------------- |
| `enter` | `on enter:`            | Triggered when entering a state.         |
| `exit`  | `on exit:`             | Triggered when leaving a state.          |
| `event` | `on event <name>:`     | Triggered when an event is received.     |
| `after` | `on after <duration>:` | Triggered once after a duration.         |
| `each`  | `on each <duration>:`  | Triggered repeatedly while in the state. |
| `error` | `on error:`            | Triggered when an internal error occurs. |

Examples:

```ocean-dsl
on enter:
```

```ocean-dsl
on exit:
```

```ocean-dsl
on event confirm:
```

```ocean-dsl
on after 30s:
```

```ocean-dsl
on each 5s:
```

```ocean-dsl
on error:
```

Durations may use formats such as:

```text
30s
2m
1h
```

---

## 9. Handler Types

Every handler may specify one of four handler types:

- `logic`;
- `external`;
- `placeholder`;
- `empty`.

General syntax:

```ocean-dsl
on <trigger> [<arg>] : [logic|external|placeholder|empty]
```

If the handler type is omitted, it defaults to `logic`.

---

### 9.1 Logic

A logic handler contains inline Ocean expression-language logic, using the same
statement forms as `@expression` (`var` locals, block `if <cond> then … end`,
assignments) plus the FSM actions in Section 10.

Explicit form:

```ocean-dsl
on event calculate : logic
    var res Float
    res = Add(a, b)
    emit r = res
```

Equivalent implicit form (handler type defaults to `logic`):

```ocean-dsl
on event calculate:
    emit r = Add(a, b)
```

---

### 9.2 External

An external handler delegates implementation to an external source.

Syntax:

```ocean-dsl
on <trigger> : external
    source: "<path>"
    method: "<symbol>"
```

Example:

```ocean-dsl
on exit : external
    source: my-funcs.go
    method: MyFunc
```

Both `source` and `method` are required. They must appear on the two lines
immediately following the external handler header. The values may be written
with or without quotes (`source: my-funcs.go` or `source: "my-funcs.go"`).

---

### 9.3 Placeholder

A placeholder declares a handler intentionally left for future implementation.

```ocean-dsl
on each 5s : placeholder
```

A placeholder handler has no body.

---

### 9.4 Empty

An empty handler explicitly defines no behavior.

```ocean-dsl
on exit : empty
```

An empty handler has no body.

---

## 10. Supported Actions

Logic handlers may use FSM actions.

| Action           | Format                                             | Description                                                |
| ---------------- | -------------------------------------------------- | ---------------------------------------------------------- |
| `next`           | `next <StateName> [: out1=expr1, out2=expr2, ...]` | Transition to another state and end the current handler.   |
| `emit`           | `emit <output> = <expr>`                           | Assign a declared event output and emit it.                |
| `update`         | `update <target> = <expr>`                         | Assign a context field (in-memory FSM) or an entity field. |
| field assignment | `this.<field> = <expr>` / `ctx.<field> = <expr>`   | Assign an entity or context field directly.                |
| expression call  | `<ExprName>(args)` or `<alias>(args)`              | Invoke a used expression (by name or alias).               |
| database call    | `<Entity>.<queryOrCommand>(args)`                  | Invoke a database query or command on an entity.           |
| `return`         | `return`                                           | Exit the current handler early.                            |

`emit` and `update` use `=`, for example `emit r = add(a, b)` and
`update fsmCtx.mode = "Adding mode"`.

---

## 11. State Transitions

The `next` action transitions the FSM to another state.

Basic syntax:

```ocean-dsl
next <StateName>
```

Example:

```ocean-dsl
on event setMode:
    next Adding
```

A transition immediately ends the current handler.

The transition sequence is:

1. execute `on exit` of the current state;
2. update the FSM state;
3. execute `on enter` of the target state.

---

### 11.1 Returning Outputs During Transition

If an event defines outputs, `next` may return output values.

Example:

```ocean-dsl
event calculate in(a:Float & b:Float) out(r:Float)

state Adding
    on event calculate:
        next Adding : r=Add(a,b)
```

Multiple outputs:

```ocean-dsl
event compute in(x:Int) out(value:Int & status:String)

state NoOp
    on event compute:
        next Ready : value=0, status="initialized"
```

Conditional example:

```ocean-dsl
on event calculate:
    if b == 0 then
        next NoOp : value=0, status="div-by-zero"
    end
    next Dividing : value=Div(a,b), status="ok"
```

Rules:

- `next` always ends the current handler.
- `next` may be used in any logic handler, including event, enter, exit, error,
  after, and each handlers.
- Output assignments are allowed when the event defines outputs.
- Outputs not explicitly assigned receive their zero values.
- If the handler has no outputs, output assignments must not be provided.
- Non-event handlers, including `on enter` and `on exit`, cannot return output
  assignments.
- If the transition fails, the transition error is returned and outputs receive zero values.

---

## 12. `this`

In an Entity-Controlled FSM, `this` refers to the current entity.

It may be used to read entity fields and to assign them, both as plain
statements and inside a `next` transition:

```ocean-dsl
on event approve:
    this.remark = appReq.remark
    this.approvedAt = nowUTC()
    approver = User.findById(appReq.approvedBy)
    this.approvedBy = approver
    next Approved
```

```ocean-dsl
on event ship:
    next Shipped : this.updatedAt = nowUTC()
```

For entity-controlled FSMs, changes to the controlled entity may be persisted according to the FSM execution and database model.

For in-memory FSMs, state and working data are maintained through the associated context.

---

## 13. Expressions

FSMs may use reusable expressions.

Syntax:

```ocean-dsl
use expression <ExpressionName> [as <alias>]
```

Examples:

```ocean-dsl
use expression GenerateInvalidCalcResult
use expression CalcAdd as add
use expression CalcDiv as div
```

The alias is optional. When present, FSM logic invokes the expression through
the alias; otherwise it uses the expression name:

```ocean-dsl
emit r = add(a, b)
emit r = GenerateInvalidCalcResult()
```

Expressions allow reusable logic to remain separate from FSM state and transition definitions.

---

## 14. Context

In-memory FSMs use a context for temporary state and working data.

Syntax:

```ocean-dsl
use context <ContextName> as <alias>
```

Example:

```ocean-dsl
use context WizardContext as ctx
```

The alias may then be referenced from FSM logic:

```ocean-dsl
ValidateStep(ctx.step1) then next Step2
```

Context data is volatile and is not persisted as database state.

Every `use context` declaration requires the explicit alias, which is the name
used to access context fields in FSM logic.

---

## 15. Database

Entity-Controlled FSMs specify the database containing the controlled entity.

Syntax:

```ocean-dsl
database: <DatabaseName>
```

Example:

```ocean-dsl
database: OrderDB
```

The database is used to locate and persist the entity whose field is controlled by the FSM.

---

## 16. Execution Model

At runtime, an FSM:

1. receives an event and the current state;
2. selects the matching state handler;
3. executes the handler logic;
4. may perform actions such as:
   - transition using `next`;
   - emit outputs using `emit`;
   - update entity or context data;
   - call expressions;
   - call database commands;
5. completes execution or transitions to another state.

For Entity-Controlled FSMs:

- state is represented by the controlled entity field;
- entity state is persisted through the database.

For In-Memory FSMs:

- state is maintained in memory;
- context provides temporary working data;
- state is not persisted between runs.

---

## 17. Complete Entity-Controlled Example

A complete entity-controlled `@fsm` file
(`ocean-examples/0008-invoice-approval-system/60-fsm.ocn`):

```ocean-dsl
@fsm

InvoiceApprovalFSM controls Invoice.status
    key: id

    # ---------------------------------------------------
    # Events (id is implicit input)
    # ---------------------------------------------------
    event approve in(appReq:ApproveInvoiceReq)  out(appInvoice:*Invoice)
    event reject  in(rejReq:*RejectInvoiceReq)  out(rejInvoice:Invoice)

    # ---------------------------------------------------
    # States
    # ---------------------------------------------------
    state PendingApproval
        on event approve:
            this.remark = appReq.remark
            this.approvedAt = nowUTC()
            approver = User.findById(appReq.approvedBy)
            this.approvedBy = approver
            next Approved

        on event reject:
            this.remark = rejReq.reason
            this.rejectedAt = nowUTC()
            rejector = User.findById(rejReq.rejectedBy)
            this.rejectedBy = rejector
            next Rejected

    state Approved
        # nop

    state Rejected
        # nop
```

This example demonstrates:

- an Entity-Controlled FSM with `key:` and no explicit `database:`;
- states inferred from the controlled `Invoice.status` enum;
- events whose `id` input is implicit, with pointer input/output types;
- implicit `logic` handlers (`on event approve:`);
- direct `this.<field>` assignment and an entity query call (`User.findById(...)`);
- `next` transitions with no output assignment;
- states with only a comment body.

---

## 18. Complete In-Memory Example

A complete in-memory `@fsm` file
(`ocean-examples/0006-in-mem-fsm/06-fsm.ocn`):

```ocean-dsl
@fsm

CalcFSM in-memory
    states     : NoOp, Adding, Subtracting, Multiplying, Dividing
    default    : NoOp

    event getMode in(_) out(mode:String)
    event setOperation in(op:String) out(result:String)
    event calculate in(a:Float & b:Float) out(r:Float)
    event calculateWithValidation in(a:Float & b:Float) out(r:CalcResult)

    use context CalcFsmContext as fsmCtx

    use expression CalcAdd  as add
    use expression CalcSub  as sub
    use expression CalcMul  as mul
    use expression CalcDiv  as div
    use expression CalcWithValidationAdd  as addV
    use expression CalcWithValidationSub  as subV
    use expression CalcWithValidationMul  as mulV
    use expression CalcWithValidationDiv  as divV
    use expression GenerateInvalidCalcResult

    state Every
        on event setOperation : logic
            if op == "add" then
                next Adding : result = "ADD"
            end
            if op == "sub" then
                next Subtracting : result = "SUB"
            end
            if op == "mul" then
                next Multiplying : result = "MUL"
            end
            if op == "div" then
                next Dividing : result = "DIV"
            end
            #next NoOp : result="NOP"
            next NoOp

        on event getMode : logic
            emit mode = fsmCtx.mode

    state NoOp
        on enter : logic
            update fsmCtx.mode = "No-operation mode. Returns zero"
        on event calculate : logic
            emit r = 0
        on event calculateWithValidation : logic
            emit r = GenerateInvalidCalcResult()
        on exit : empty

    state Adding
        on enter:
            update fsmCtx.mode = "Adding mode."
        on event calculate:
            emit r = add(a, b)
        on event calculateWithValidation : logic
            emit r = addV(a, b)
        on exit : placeholder

    state Subtracting
        on enter:
            update fsmCtx.mode = "Subtracting mode"
        on event calculate:
            var res Float
            res = sub(a, b)
            emit r = res
        on event calculateWithValidation:
            var res CalcResult
            res = subV(a, b)
            emit r = res
        on exit : external
            source: my-funcs.go
            method: MyFunc

    state Multiplying
        on enter:
            update fsmCtx.mode = "Multiplying mode"
        on event calculate:
            emit r = CalcMul(a, b)
        on event calculateWithValidation:
            emit r = CalcWithValidationMul(a, b)

    state Dividing
        on enter:
            update fsmCtx.mode = "Dividing mode"
        on event calculate:
            var res Float
            res = CalcDiv(a, b)
            emit r = res
        on event calculateWithValidation:
            var res CalcResult
            res = CalcWithValidationDiv(a, b)
            emit r = res
```

This example demonstrates:

- an In-Memory FSM with explicit `states` and a `default`;
- multiple events with typed and `_` inputs/outputs;
- `use context ... as` and `use expression ... as` (aliased and un-aliased);
- the special `Every` state hosting cross-state event handlers;
- explicit (`: logic`) and implicit handlers;
- `update <ctxField> = ...`, `emit <output> = ...`, and `var` locals;
- block `if <cond> then … end` with `next … : <output>=<value>`;
- `empty`, `placeholder`, and `external` handlers;
- calling an expression by alias (`add(a, b)`) or by name (`CalcMul(a, b)`).

---

## 19. Internal Normalization

FSM state definitions may be normalized internally into a fixed runtime structure.

For example, the current implementation represents state handlers conceptually as:

```go
type FSMStateBlock struct {
    Name      string
    OnEnter   []string
    OnExit    []string
    OnError   []string
    OnInputs  map[string][]string
    OnAfter   map[string][]string
    OnEach    map[string][]string
}
```

Sections not explicitly defined in the DSL may be represented internally as empty collections.

This normalized representation is an implementation detail and is not part of the canonical Ocean DSL syntax.

---

## 20. Imports

An `@fsm` file may import reusable definitions from the Ocean Repository.

Examples:

```ocean-dsl
@import datatype O.domain.Order@1.0.0 as Order
```

```ocean-dsl
@import expression O.logic.CheckReady@1.1.0 as CheckReady
```

Imported definitions may then be used by the FSM according to their DSL type.

Every imported datatype or expression requires an `as <alias>` clause and is
referenced through that alias.

Reusable definitions may also be made available through the supported Ocean include mechanism.

---

## 21. Rules and Constraints

The following rules apply:

- An FSM file starts with `@fsm`.
- An FSM is either Entity-Controlled or In-Memory.
- Entity-Controlled FSMs use `controls <Datatype>.<field>`.
- Entity-Controlled FSMs require `key:`; `database:` is optional and is inferred from the entity when omitted.
- In-Memory FSMs use `in-memory`.
- In-Memory FSMs require a context, explicit states, and a default state.
- Every In-Memory FSM `use context` declaration requires an explicit alias.
- Events define typed inputs and outputs.
- `_` represents no input or output.
- Every handler may be `logic`, `external`, `placeholder`, or `empty`.
- Handler type defaults to `logic` when omitted.
- External handlers require `source:` and `method:`.
- External-handler `source:` and `method:` metadata immediately follows its
  header.
- Placeholder and empty handlers have no body.
- `next` transitions to another state and ends the current handler.
- `this` refers to the controlled entity in an Entity-Controlled FSM.
- Durations may be used with `after` and `each` triggers.
- Expressions may be used as reusable FSM logic.
- Imported datatypes and expressions require explicit aliases.
- Entity-Controlled state is persisted through the associated database.
- In-Memory state is volatile.

---

## 22. Related Knowledge

The `@fsm` DSL is related to:

- `dsl.datatype` — defines controlled entities, state enums, event input types, and event output types.
- `dsl.database` — provides persistence for Entity-Controlled FSMs.
- `dsl.context` — provides volatile state and working data for In-Memory FSMs.
- `dsl.expression` — defines reusable logic invoked by FSM handlers.
- `dsl.import` — defines the mechanism for importing selected definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
