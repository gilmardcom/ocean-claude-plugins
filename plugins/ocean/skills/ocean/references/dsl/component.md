# Component DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/dsl/component/

**Contents**

- [1. Overview](#1-overview)
- [2. When to Use a Component](#2-when-to-use-a-component)
- [3. General Structure](#3-general-structure)
- [4. Component Identity and Metadata](#4-component-identity-and-metadata)
- [5. Public Ports](#5-public-ports)
- [6. Behavioral Dependencies](#6-behavioral-dependencies)
- [7. Connections](#7-connections)
- [8. Lifecycle and State](#8-lifecycle-and-state)
- [9. Infrastructure Boundary](#9-infrastructure-boundary)
- [10. Nested Component Example](#10-nested-component-example)
- [11. Imports and Includes](#11-imports-and-includes)
- [12. Validation](#12-validation)
- [13. Generation and Documentation](#13-generation-and-documentation)
- [14. Complete Example](#14-complete-example)
- [15. Rules Summary](#15-rules-summary)
- [16. Related Knowledge](#16-related-knowledge)

Defines reusable, stateless behavioral components with typed ports, explicit dependencies, and typed internal connections.

Source: https://atlas.ocean-lab.ai/dsl/component/

Status: Accepted · Authority: Normative

## 1. Overview

The `@component` section defines reusable in-process business behavior.

A component sits between an expression and a service:

- an **expression** implements focused, stateless logic;
- a **component** composes expressions, FSMs, and other components into a
  reusable typed flow;
- a **service** provides the runtime boundary and connects behavior to APIs,
  brokers, databases, configuration, secrets, and deployment.

A component:

- exposes typed input and output ports;
- declares every behavioral dependency explicitly;
- connects its ports and dependencies through typed connections;
- may be reused by multiple services or nested inside other components;
- is compiled into its hosting service and is not deployed independently.

A component does not expose a network endpoint, select a port, own deployment
configuration, or directly consume runtime infrastructure. Those
responsibilities remain with the hosting service.

---

## 2. When to Use a Component

Use a component when behavior:

- consists of more than one expression or stateful element;
- represents a cohesive domain capability;
- should be reused by multiple services;
- benefits from an explicit, inspectable internal flow;
- does not need an independent process or transport boundary.

Examples include:

- order acceptance;
- checkout processing;
- invoice approval;
- shipment preparation;
- reusable fraud-screening or eligibility flows.

Do not use a component merely to rename one expression. Do not use a component
for behavior that must be independently deployed, scaled, secured, or exposed
over a transport; use a service or integration for that boundary.

---

## 3. General Structure

```ocean-dsl
@component

<ComponentName>
    input  : <inputName>:<InputType> [& ...]
    output : <outputName>:<OutputType> [& ...]

    use expression <ExpressionName> as <alias>
    use fsm <FSMName> as <alias>
    use component <ComponentName> as <alias>

    connect <source> -> <target>
    connect <source> -<mapper>-> <target>
```

A file may define multiple components. Each definition continues until the next
component name or the end of the file.

Example:

```ocean-dsl
@component

OrderAcceptance
    input  : submit:SubmitOrderRequest
    output : accepted:OrderAccepted & rejected:OrderRejected

    use expression ValidateOrder as validate
    use expression ToOrderAccepted as toAccepted
    use fsm OrderLifecycle as lifecycle

    connect submit        -> validate
    connect validate      -> lifecycle.submit
    connect lifecycle.accepted -toAccepted-> accepted
    connect lifecycle.rejected             -> rejected
```

---

## 4. Component Identity and Metadata

Component names follow Ocean type-name conventions: they begin with an
uppercase letter and contain only letters, digits, and `_`.

```text
OrderAcceptance
CheckoutFlow
ShipmentPreparation2
```

The common `@perspectives` declaration may follow the component name:

```ocean-dsl
OrderAcceptance
    @perspectives: version:1.0.0, lifestyle:stable
```

File metadata and comments follow the common Ocean metadata and comment rules.
Comments attached to a component, dependency, port, or connection remain
associated with that item during formatting.

---

## 5. Public Ports

Ports form the complete public contract of a component. A hosting service or
parent component can reference only these public ports; internal aliases and
connections are encapsulated.

### 5.1 Input Ports

An input port activates component behavior with a typed value:

```ocean-dsl
input : submit:SubmitOrderRequest
```

Multiple inputs are separated by `&`:

```ocean-dsl
input : submit:SubmitOrderRequest & cancel:CancelOrderRequest
```

Each activation creates an independent execution scope. Values produced while
handling that activation remain correlated with the same scope, including when
the component is used to fulfill a request-response API operation.

### 5.2 Output Ports

An output port emits a typed result or signal:

```ocean-dsl
output : accepted:OrderAccepted
```

Multiple outputs are separated by `&`:

```ocean-dsl
output : accepted:OrderAccepted & rejected:OrderRejected
```

One activation may emit one of several outputs, multiple outputs, or no output,
depending on its internal behavior. A hosting request-response flow must wire a
terminal output that satisfies the implemented API contract.

### 5.3 Port Rules

- Port names follow Ocean field-name conventions.
- Port names are unique across both input and output ports of one component.
- Port types must resolve to Ocean primitive, enum, or datatype definitions.
- Optional, collection, and map types follow the common Ocean type system.
- `_` is not a port type. A signal without business data uses an explicit
  signal datatype.
- A component declares at least one input or output port.

---

## 6. Behavioral Dependencies

Components declare their dependencies with `use`. Dependencies are resolved
statically and become part of the component's transitive generation closure.

### 6.1 Expressions

```ocean-dsl
use expression <ExpressionName> as <alias>
```

Example:

```ocean-dsl
use expression ValidateOrder as validate
use expression MapAcceptedOrder as mapAccepted
```

An expression alias is callable. As a connection target it receives the source
value; as a connection source it represents the expression result.

### 6.2 FSMs

```ocean-dsl
use fsm <FSMName> as <alias>
```

Example:

```ocean-dsl
use fsm OrderLifecycle as lifecycle
```

FSM events are referenced as `<alias>.<event>`. As a target the reference sends
the value to the event; as a source it represents that event's declared output.
State remains owned by the FSM, not by the component.

A component's own connections never draw an edge between two events of the
same FSM alias — each event is its own independent connection source or
target. The canonical idiom (§3, §10, §14) is nonetheless one "trigger" event
with no declared output (e.g. `submit`) driving the FSM, and one or more
"outcome" events with no declared input (e.g. `accepted`, `rejected`) reading
back whichever state it moved to. Every event of one `use fsm` alias is
therefore treated as reachable from every other event of that same alias:
once a component's internal flow reaches any event of an alias, every other
event of that alias that feeds one of the component's own connections becomes
reachable too, called with no incoming value. Calling an outcome event while
the FSM is not in the state that handles it is always well-defined — the FSM
contract requires every unimplemented state/event pair to return its declared
zero values, never an error — so this reflects an already-guaranteed safe
call, not a new failure mode.

### 6.3 Nested Components

```ocean-dsl
use component <ComponentName> as <alias>
```

Example:

```ocean-dsl
use component AuditTrail as audit
```

Nested ports are referenced as `<alias>.<port>`. A nested input is a connection
target and a nested output is a connection source.

### 6.4 Alias Rules

- Every component dependency requires an explicit alias.
- Aliases follow Ocean field-name conventions.
- Aliases are unique within the component.
- An alias must not conflict with a public port or another local symbol.
- Internal references use aliases consistently; original names do not remain
  alternate local references after aliasing.

Explicit aliases make nested flows stable when reusable definitions are renamed
or imported under repository aliases.

The parser enforces this for all three dependency kinds: `use expression`,
`use fsm`, and `use component` each require `as <alias>` and reject the line
otherwise. Alias *uniqueness* within a component, and conflicts with a public
port, are not yet validated (§12).

---

## 7. Connections

Connections declare component execution and data flow.

```ocean-dsl
connect <source> -> <target>
```

The source produces a value or signal. The target consumes it.

```ocean-dsl
connect submit             -> validate
connect validate           -> lifecycle.submit
connect lifecycle.accepted -> accepted
```

### 7.1 Valid Endpoints

| Reference | As source | As target |
| --- | --- | --- |
| Component input port | Activating input value | Invalid |
| Component output port | Invalid | Emitted public output |
| Expression alias | Expression result | Expression invocation |
| `<fsm>.<event>` | Event output | Event invocation |
| `<component>.<input>` | Invalid | Nested component activation |
| `<component>.<output>` | Nested component output | Invalid |

Direction is determined by the declared contract, not merely by where a name
appears. Reversing an input or output endpoint is a validation error.

### 7.2 Single-Value Flow

Each connection transports exactly one typed value. A callable used directly as
a target must therefore accept one compatible flow input, and a callable used
as a source must produce one flow output.

When behavior needs several related values, those values are represented by an
explicit datatype and assembled by an upstream expression. A component does
not implicitly zip several branches into positional arguments or infer when
all branches are complete. This rule keeps invocation order, correlation, and
completion deterministic.

Zero-input triggers and multi-input joins require an explicit behavior whose
contract defines their activation semantics; they are not inferred from an
ordinary connection.

### 7.3 Type Compatibility

Without a mapper, the source output type must be assignable to the target input
type under the Ocean type system.

```ocean-dsl
connect submit -> lifecycle.submit
```

No implicit object reshaping, field projection, parsing, or lossy numeric
conversion is performed.

Type identity is case-sensitive, like every other Ocean identifier: a
datatype, enum, imported alias, or port named `Order` is not the same type as
one named `order`. A connection whose source and target differ only in case
is a type mismatch, not a coincidental match.

### 7.4 Mappers

Use the common Ocean mapper form when source and target types differ:

```ocean-dsl
connect <source> -<mapper>-> <target>
```

Example:

```ocean-dsl
connect lifecycle.accepted -toAccepted-> accepted
```

The mapper must resolve to a used expression with exactly one flow input and
one flow output. Its input must accept the source type, and its output must be
assignable to the target type. A mapper transforms one connection; it does not
become an additional independently triggered flow.

### 7.5 Fan-out

A source may feed multiple targets:

```ocean-dsl
connect lifecycle.accepted -> accepted, audit.record
```

Writing separate connections is equivalent and is preferred when targets use
different mappers:

```ocean-dsl
connect lifecycle.accepted -toAccepted-> accepted
connect lifecycle.accepted -toAudit-> audit.record
```

Fan-out does not guarantee target execution order and is not transactional.
Every branch is validated independently.

### 7.6 Fan-in and Aggregation

Multiple sources connected to one internal target — an expression alias, an
FSM event, or a nested component's input — activate that target
independently. They are not implicitly joined, buffered, or aggregated.

Combining several results requires an explicit expression, FSM, nested
component, or service-level `aggregate` whose contract defines completion and
ordering behavior.

A public output port is different: it is the component's observable,
deterministic contract (§5.2), not an internal activation. At most one
connection may target a given output port. Two connections from different
sources both producing the same output leave a caller with no way to know
which one filled it for a given activation, and are rejected as ambiguous
competing producers, whether or not the sources are actually mutually
exclusive at runtime. Route both through an explicit upstream expression,
FSM, or nested component first, and connect its single result to the output.

### 7.6a Duplicate and Ambiguous Connections

The same `<source> -> <target>` edge declared more than once is redundant if
every declaration agrees — same mapper, or no mapper on any of them — and
ambiguous if they disagree. Fan-out (§7.5) and fan-in to an internal target
(above) are not affected: those are different edges, not repeats of the same
one.

```ocean-dsl
# redundant: the exact same edge twice
connect validate -> accepted
connect validate -> accepted

# ambiguous: which mapper applies to validate -> accepted?
connect validate -toA-> accepted
connect validate -toB-> accepted
```

### 7.7 Cycles

Recursive component use is invalid, whether direct or transitive. Plain
connection cycles are also invalid because they would create unbounded
execution. Repetition and state transitions belong in an FSM, where their
lifecycle is explicit.

---

## 8. Lifecycle and State

A component is not an independently running process. It is instantiated by a
hosting service or parent component.

For each `use component` declaration:

- the host receives a logically distinct component instance;
- component-local execution data is scoped to one activation;
- no mutable state is retained by the component itself;
- nested FSM state follows the FSM's declared lifecycle and persistence model;
- nested component instances follow the same rules recursively.

Two services using the same component definition do not share runtime state.
Generated implementations may optimize stateless objects, but such an
optimization must not change these semantics.

Concurrency, retries, transactions, and timeouts are not inferred at the
component boundary. They are defined by the hosting service, transport,
explicit FSM behavior, or other DSL contracts that own those concerns.

---

## 9. Infrastructure Boundary

A component may directly use:

- expressions;
- FSMs;
- other components.

A component does not directly declare:

- implemented or used APIs;
- brokers or topics;
- databases, queries, or commands;
- configuration or vaults;
- integrations;
- schedules;
- deployment or ports.

The hosting service owns these resources and connects them to component ports:

```ocean-dsl
@service

OrderService
    use config CommonServiceConfig as cfg
    impl api OrderApi as api on cfg.apiConfig.port
    use component OrderAcceptance as orders
    use broker OrderBroker as broker

    connect api.submitOrder -> orders.submit
    connect orders.accepted -> api.submitOrder
    connect orders.accepted -> broker.orders.accepted
```

This boundary keeps a component reusable across transports, deployment
environments, and infrastructure providers.

An FSM nested by a component may use persistence according to the FSM contract.
That transitive dependency must remain visible in validation, generation,
documentation, and architecture views; nesting must never hide operational
requirements from the hosting service.

---

## 10. Nested Component Example

```ocean-dsl
@component

AuditTrail
    input  : record:AuditRecord
    output : recorded:AuditRecorded

    use expression RecordAudit as store

    connect record -> store
    connect store  -> recorded

OrderAcceptance
    input  : submit:SubmitOrderRequest
    output : accepted:OrderAccepted & audited:AuditRecorded

    use expression ValidateOrder as validate
    use expression ToAuditRecord as toAudit
    use fsm OrderLifecycle as lifecycle
    use component AuditTrail as audit

    connect submit             -> validate
    connect validate           -> lifecycle.submit
    connect lifecycle.accepted -> accepted
    connect lifecycle.accepted -toAudit-> audit.record
    connect audit.recorded     -> audited
```

Only `submit`, `accepted`, and `audited` are visible to a host using
`OrderAcceptance`. The `validate`, `lifecycle`, and `audit` aliases are internal.

---

## 11. Imports and Includes

A component file may import:

- `datatype`;
- `expression`;
- `fsm`;
- `component`.

`context` is intentionally not a supported component dependency: a component
owns no state of its own (§8), so it has nothing to bind a context import to.
The parser's import allow-list for `@component` no longer accepts
`@import context`; state remains owned by explicitly used FSMs and their
contracts.

```ocean-dsl
@component

@import datatype O.domain.order.SubmitOrderRequest@1.0.0 as SubmitOrderRequest
@import expression O.domain.order.ValidateOrder@1.2.0 as ValidateOrder
@import fsm O.domain.order.OrderLifecycle@2.0.0 as OrderLifecycle
@import component O.common.audit.AuditTrail@1.0.0 as AuditTrail
```

Every repository import requires `as <alias>`. The import alias is the
definition name available to declarations in the file. Local `use` aliases
still define instance names:

```ocean-dsl
OrderAcceptance
    input : submit:SubmitOrderRequest

    use expression ValidateOrder as validate
    use fsm OrderLifecycle as lifecycle
    use component AuditTrail as audit
```

Repository-level `@include` is processed according to the common include
contract. It is distinct from nested component use: inclusion makes definitions
available to the file, while `use component` instantiates a component inside
another component.

---

## 12. Validation

Component validation must report as many independent issues as possible in one
validation pass.

The list below is the required target validation contract. The current
registry implements all of it: unique component names; resolution of
declared expressions, FSMs, and nested components; the presence of a
connection source and target; import kind/alias/resolution errors; port
rules (unique names, resolvable datatypes, at least one port, `_` rejected);
alias rules (required, unique, no conflict with a port); connection
reference resolution, including encapsulation — a nested component's
internal aliases are not reachable through its alias, only its public ports;
endpoint direction; source/target and mapper type compatibility (exact,
case-sensitive type-identity match — see §7.3 — not yet
List<T>/Optional<T>-aware structural compatibility); direct and transitive
component-use cycles; plain connection cycles (opaque to what happens inside
a used FSM or nested component — see §8); duplicate and ambiguous
connections, including competing producers for one output port (§7.6,
§7.6a); unused local aliases; and outputs no connection ever produces.

Validation includes:

- valid and unique component names;
- valid, unique public port names;
- at least one public input or output;
- resolvable port datatypes;
- resolvable expressions, FSMs, and nested components;
- required and unique local aliases;
- valid source and target references;
- correct endpoint direction;
- source/target type compatibility;
- mapper existence and mapper signature compatibility;
- duplicate or ambiguous connections;
- unreachable internal elements;
- public outputs that can never be produced;
- direct or transitive component-use cycles;
- invalid plain connection cycles;
- import kind, alias, and resolution errors;
- visibility violations that reference another component's internal elements.

Infrastructure declarations inside a component are parse or validation errors,
depending on the phase in which they can be identified.

---

## 13. Generation and Documentation

A component is generated as reusable in-process code and linked into every
hosting service that uses it. It does not receive its own executable,
container, health endpoint, configuration file, or deployment entry.

Generation:

- preserves the typed public port contract: one generated method per input
  port, taking that port's value and returning every declared output port
  (§5.2 — a Go-style multi-return where the branches this activation did not
  reach come back zero-valued, the same convention an FSM event already uses
  for its own declared outputs);
- includes transitive expression, FSM, and nested-component dependencies
  (`GetUsedExpressions`, `GetUsedFSMs`, `GetUsedComponents` — one shared,
  cycle-safe traversal, covered for self-cycles, multi-component cycles, and
  diamond-shaped sharing);
- creates one process-wide instance per component definition, reached by
  name, exactly like an FSM instance (§8's per-host isolation therefore holds
  automatically between services, which are separate processes; two `use
  component` aliases for the same definition within one service share that
  one instance — the same restriction FSMs already have);
- preserves activation correlation through the internal flow by compiling
  each connection into a direct call, threading one hop's result into the
  next;
- rejects unsupported constructs before emitting partial code: an
  unresolvable connection reference, or a connection graph a component-use or
  connection cycle check did not already reject but the FSM-bridging rule
  above would make cyclic, fails generation outright.

A hosting service reaches a component's input port exactly like it reaches an
FSM event or an expression — `connect api.op -> alias.port` compiles to a
direct call, matching every other dependency kind. Component port methods do
not take a context parameter, again matching FSM events; a future revision
may thread one through, mirroring how an implemented API method already
does.

Architecture documentation should show:

- the component's public inputs and outputs;
- its internal expression, FSM, and nested-component dependencies;
- typed and mapped connections;
- every hosting service;
- transitive state or persistence requirements.

In a high-level microservices view, a component belongs inside its hosting
service rather than appearing as an independently deployed node. A detailed
flow view may expand it into its internal graph.

---

## 14. Complete Example

```ocean-dsl
# @ocean-meta-start
# name: order-acceptance-component
# version: 1.0.0
# tags:
#   - order
#   - reusable-behavior
# perspective:
#   feature: order-management
#   boundedContext: ordering
#   service: shared
# @ocean-meta-end

@component

@import datatype O.domain.order.SubmitOrderRequest@1.0.0 as SubmitOrderRequest
@import datatype O.domain.order.OrderAccepted@1.0.0 as OrderAccepted
@import datatype O.domain.order.OrderRejected@1.0.0 as OrderRejected

OrderAcceptance
    @perspectives: version:1.0.0, lifestyle:stable

    input  : submit:SubmitOrderRequest
    output : accepted:OrderAccepted & rejected:OrderRejected

    use expression ValidateOrder as validate
    use expression ToOrderAccepted as toAccepted
    use expression ToOrderRejected as toRejected
    use fsm OrderLifecycle as lifecycle

    connect submit -> validate
    connect validate -> lifecycle.submit
    connect lifecycle.accepted -toAccepted-> accepted
    connect lifecycle.rejected -toRejected-> rejected
```

This component can be reused by an HTTP service, a broker-driven worker, or an
integration host without changing its business flow.

---

## 15. Rules Summary

- A component file starts with `@component`.
- A component is reusable in-process behavior, not a runtime boundary.
- Components expose only typed input and output ports.
- Components directly use expressions, FSMs, and nested components.
- Every dependency has a unique explicit alias.
- Connections use the common `connect` and `-mapper->` grammar.
- Every connection is directional and type-checked.
- Fan-out is allowed; implicit aggregation and ordering are not.
- A public output port has at most one producing connection; type identity
  is case-sensitive, and a redundant or conflicting edge is rejected.
- Components do not own mutable state; FSMs own state.
- Component instances and their stateful dependencies are isolated per host.
- Direct or transitive recursive component use is invalid.
- APIs, brokers, databases, configuration, vaults, schedules, integrations,
  ports, and deployments remain service-level concerns.
- A component is generated into its host and is never deployed independently.
- Internal aliases and connections are encapsulated from hosts.

---

## 16. Related Knowledge

- `dsl.datatype` — defines component port and dependency types.
- `dsl.expression` — defines stateless logic and connection mappers.
- `dsl.fsm` — defines stateful behavior composed by components.
- `dsl.service` — hosts components and connects them to runtime resources.
- `dsl.import` — imports repository definitions into a component file.
- `dsl.include` — makes reusable definitions available during composition.
