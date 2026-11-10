# Service DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/service/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax Overview](#3-syntax-overview)
- [4. Service Names](#4-service-names)
- [5. Ports](#5-ports)
- [6. Use and Implementation Declarations](#6-use-and-implementation-declarations)
- [7. API Dependencies](#7-api-dependencies)
- [8. API Implementations](#8-api-implementations)
- [9. Config and Context](#9-config-and-context)
- [10. Brokers and Databases](#10-brokers-and-databases)
- [11. Expressions, FSMs, and Components](#11-expressions-fsms-and-components)
- [12. Functions and Vaults](#12-functions-and-vaults)
- [13. Alias Rules](#13-alias-rules)
- [14. Initialization](#14-initialization)
- [15. Connections](#15-connections)
- [16. Connection Compatibility](#16-connection-compatibility)
- [17. Mappers](#17-mappers)
- [18. Fan-out](#18-fan-out)
- [19. Common Connection Directions](#19-common-connection-directions)
- [20. Broker Patterns and Binds](#20-broker-patterns-and-binds)
- [21. Aggregation](#21-aggregation)
- [22. Imports](#22-imports)
- [23. Includes](#23-includes)
- [24. Complete Example](#24-complete-example)
- [25. Deployment Boundary](#25-deployment-boundary)
- [26. Validation](#26-validation)
- [27. Invalid Examples](#27-invalid-examples)
- [28. Rules and Constraints](#28-rules-and-constraints)
- [29. Related Knowledge](#29-related-knowledge)

Defines the Ocean `@service` section for composing domain behavior, external contracts, infrastructure, configuration, secrets, initialization, aggregation, and typed connections into deployable units.

Source: https://atlas.ocean-lab.ai/dsl/service/

Status: Approved · Authority: Normative

## 1. Overview

The `@service` section defines deployable Ocean units.

A service composes domain behavior and connects it to system interfaces and infrastructure. It can implement or consume APIs, host reusable integrations, use brokers and databases, orchestrate components and FSMs, use expressions and functions, bind configuration and context, expose vault-backed secrets, initialize behavior, aggregate results, and declare typed connections.

The service is the bridge between abstract domain behavior and runnable system integration.

---

## 2. Core Principle

A service owns orchestration and deployment-facing integration—not the internal state of domain behavior.

```text
APIs ─────────┐
Integrations ─┤
Brokers ──────┤
Databases ────┤
Config ───────┼── Service ── Components / FSMs / Expressions
Context ──────┤
Vaults ───────┤
Functions ────┘
```

Components own composition, FSMs own behavioral state, and expressions own stateless logic. The service connects these capabilities into a deployable boundary.

---

## 3. Syntax Overview

```ocean-dsl
@service

<ServiceName>
    @perspectives: <key>:<value>, ...
    @tags: <tag1>, <tag2>, ...

    [port: <port1>, <port2>, ...]

    use config <ConfigName> as <alias>
    use context <ContextName> as <alias>
    use api <ApiName> [as <alias>]
    impl api <ApiName> [as <alias>] on <port-or-config-path>
    impl gateway api <ApiName> [as <alias>] on <port-or-config-path>
        including <used-api-alias>, ...
    authenticate actor using <expression-or-alias> resolve <api-alias>.<resolver>
    impl integration <IntegrationName> as <alias>
    use broker <BrokerName> as <alias>
    use database <DatabaseName> [as <alias>]
    use expression <ExpressionName> [as <alias>]
    use fsm <FSMName> as <alias>
    use component <ComponentName> [as <alias>]
    use function <FunctionType> [as <alias>]
    use vault <VaultName> [as <alias>]

    init <expression-or-function-call>
    aggregate <aggregation>
    connect <source> -> <target>
```

A file may define multiple services.

An optional `@perspectives:` line (and, less commonly, `@tags:`) may follow the
service name to attach comma-separated `key:value` metadata — for example
`@perspectives: version:1.0.0, lifestyle:stable`. See `dsl.perspective` and
`dsl.tag`.

---

## 4. Service Names

Service names follow the Ocean type-name convention and must be unique within the resolved service scope.

Examples:

```text
OrderService
InventoryService
PaymentOrchestrator
```

A service name identifies the deployable model unit; target-specific deployment names may be derived separately.

---

## 5. Ports

An implemented API selects its port with `on`. The port may be written inline:

```ocean-dsl
impl api TodoApi as api on 8080
```

or resolved from a configuration path — the alias of a `use config` binding
followed by the path to the port value:

```ocean-dsl
use config TodoServiceConfig as myCfg
impl api TodoApi as api on myCfg.apiConfig.port
```

A service may also declare symbolic ports as a comma-separated list and bind an
API to one of them:

```ocean-dsl
port: rest, admin

impl api PublicAPI as publicApi on rest
```

Port symbols and configuration paths must resolve unambiguously.

---

## 6. Use and Implementation Declarations

Services bring model elements into local scope through injection declarations.

`use` declares a dependency consumed by the service.

`impl api` declares an API contract fulfilled by the service.

`impl gateway api` declares an API contract fulfilled by the service as a
gateway façade. It exposes one public API while routing its operations to used
APIs automatically where their contracts match, and to explicitly connected
expressions, broker topics, or other supported targets where they do not.

`impl integration` declares a reusable integration module hosted by the
service. The service remains the runtime and deployment boundary.

```text
use api  → service calls or depends on an API
impl api → service exposes and fulfills an API
```

Aliases provide the local identity used in connections and calls.

---

## 7. API Dependencies

```ocean-dsl
use api PartnerAPI
```

or:

```ocean-dsl
use api PartnerAPI as partner
```

A used API represents an outbound dependency. Its operations may be referenced through the local alias or available name according to alias-resolution rules.

`use api` does not bind a listening port.

---

## 8. API Implementations

```ocean-dsl
impl api OrderAPI as api on myCfg.apiConfig.port
```

`impl api` declares that the service fulfills the named API contract. The `on <target>` clause is required.

The target may be:

- a literal numeric port, such as `8080`;
- a valid configuration path resolving the port value, such as `myCfg.apiConfig.port` (the common form);
- a port declared by `port:`.

An implemented API operation can serve as a connection source for incoming requests and as a target for corresponding output behavior, subject to the API contract.

### 8.1 Gateway Implementations

A gateway is still a service: it owns a deployable process, a public API
contract, configuration, and ordinary service dependencies. Its special
behavior is bulk forwarding from the public façade API to compatible used APIs.

```ocean-dsl
@service

GatewayService
    use config OceanGatewayConfig as gwCfg

    use api BundleApi as bundle
    use api EntitlementApi as entitlement
    use api IdentityApi as identity
    use broker OceanBroker as broker
    use expression EngineValidate

    impl gateway api OceanApi as api on gwCfg.apiConfig.port
        including bundle, entitlement, identity

    # Explicit connections override automatic API forwarding.
    connect api.validate -> EngineValidate
    connect api.login    -> broker.auth.login
```

The public API (`OceanApi` above) is the contract exposed by the gateway. Every
name in `including` is the local alias of a `use api` declaration. An included
API supplies automatic targets for public operations with the same operation
name and compatible input, output, and error contracts. The public route and
documentation remain those of the gateway's implemented API; target APIs are
not exposed directly through this declaration.

An explicit `connect` whose source is an operation of the gateway API takes
precedence over a matching automatic forwarding target. This permits a gateway
to replace a default forwarding route with an expression, broker topic,
aggregate, or another supported target without duplicating every unrelated
route.

The initial gateway model permits one `impl gateway api` declaration per
service. A public façade API can still compose operations from any number of
included APIs and explicit targets.

Gateway validation requires that:

- each `including` name resolves to a distinct used API alias;
- every automatic match is unique and contract-compatible;
- explicit gateway connections are compatible with their public operation;
- every public operation resolves either through an explicit connection or one
  included API;
- a service does not implement the same public API both as a regular API and
  as a gateway.

Ambiguous automatic targets, unresolved public operations, duplicate public
routes, and incompatible overrides are validation errors.

#### Gateway authentication

A gateway may declare one authentication pipeline that establishes the trusted
`actor` request context from an external bearer token:

```ocean-dsl
GatewayService
    use config GatewayConfig as cfg
    use api UserServiceApi as user
    use expression FirebaseVerifyToken as auth

    impl gateway api GatewayApi as gw on cfg.apiConfig.port
        including user

    authenticate actor using auth resolve user.resolveAuthenticatedActor
```

`authenticate` is valid only in a service with `impl gateway api`. The verifier
is a used expression that accepts `(Context, String)` and returns an identity
with `providerConnectionId:String` and `uid:String`. The resolver is a used,
internal API operation with inputs `(providerConnectionId:String, uid:String)`
and an output containing `subjectId:String`.

The verifier reference may be either the local alias declared with
`use expression … as <alias>` or the original expression name. Prefer the
alias when one is declared; for example, `using auth` refers to
`use expression FirebaseVerifyToken as auth`.

For a valid `Authorization: Bearer <token>` request, generated gateway code
verifies the token, resolves the application-local actor, and establishes
`OceanContext.Actor`. `subjectType`, `tenantId`, `organizationIds`, `roles`,
and `scopes` are also copied when the resolver returns compatible fields.
Requests with no bearer token remain anonymous; invalid or unresolved tokens
receive `401 Unauthorized`. Downstream endpoints still declare
`@from-context:actor` when they require an authenticated actor.

The verifier is expressed as a dependency rather than a Firebase-specific DSL
keyword, allowing another provider implementation later without changing the
gateway contract.

Gateway-wide concerns belong at the gateway boundary. Gateway policy
declarations for CORS, authentication, authorization, request limits, rate
limits, tracing, resilience, and error normalization are a planned extension
to this model; they are not part of the initial gateway DSL capability and are
not inferred from included APIs. Domain-level authorization may still be
enforced by downstream services.

### 8.2 Hosted Integrations

```ocean-dsl
impl integration PurchaseOrderIntegration as orchestration
```

The alias is required and identifies the hosted integration instance within the
service. A service may host one or more integrations, including a service whose
only behavior is hosted integrations.

Integrations contribute their flow behavior, implemented contracts, broker
interactions, aggregations, and schedules to the hosting service. The service
supplies the deployable process, runtime configuration, and infrastructure
bindings. An integration cannot be deployed directly.

---

## 9. Config and Context

Configuration and context declarations require explicit aliases:

```ocean-dsl
use config AppConfig as cfg
use context RequestContext as ctx
```

Configuration supplies typed runtime settings to the deployable service. Context supplies supported execution or request-scoped information.

Aliases must be unique within the service.

---

## 10. Brokers and Databases

Broker aliases are required:

```ocean-dsl
use broker OrderBroker as broker
```

Database aliases are optional:

```ocean-dsl
use database PurchaseOrder as orders
use database Party
```

Brokers expose typed topics or messaging patterns. Databases expose typed queries, commands, or entities according to their own DSL contracts.

---

## 11. Expressions, FSMs, and Components

```ocean-dsl
use expression OrderInit
use expression SystemInit as systemInit
use fsm OrderFSM as orderFsm
use component OrderFlow as orderFlow
```

FSM aliases are required. Expression and component aliases are optional.

Expressions provide stateless logic, FSMs provide stateful behavior, and components provide stateless behavioral composition.

---

## 12. Functions and Vaults

```ocean-dsl
use function HashPassword as hash
use vault MainVault as secrets
```

Function and vault aliases are optional.

A `use function` may also bring in a generic function *type*, aliased to a local
name the service then wires through connections:

```ocean-dsl
use function Supplier<String> as GetDay
use function Func<TimeDateRequest, TimeDateResponse> as GetTimeDate
```

`Supplier<T>` is a no-argument producer of `T`; `Func<In, Out>` is a one-argument
function. The alias can be used as a connection source or passed as a named
argument to an expression (see Section 15).

Functions expose supported callable behavior. Vault use makes the vault's typed secret declarations available to the service according to the vault and generator contracts; secret values remain externally managed.

---

## 13. Alias Rules

Explicit aliases are required for:

- `use config`;
- `use context`;
- `use broker`;
- `use fsm`.
- `impl integration`.

Aliases are optional for:

- `use api`;
- `impl api`;
- `use database`;
- `use expression`;
- `use component`;
- `use function`;
- `use vault`.

All effective local names must be unique. When an alias is declared, service references should use that local identity consistently.

---

## 14. Initialization

`init` declares initialization behavior:

```ocean-dsl
init systemInit(config:cfg)
init OrderInit
```

Initialization entries preserve declaration order.

They must resolve to expressions or other supported callable elements available in the service scope, and supplied arguments must satisfy the target signature.

Initialization occurs as part of service startup according to the target runtime contract.

---

## 15. Connections

Connections route typed values or signals between service elements.

```ocean-dsl
connect <source> -> <target>
    authorize context.<name>
        <authorization-policy>
```

Sources and targets use injected aliases and their exposed operations, ports,
topics, commands, queries, inputs, or outputs.

Common target forms seen in the examples:

```ocean-dsl
# API operation to an expression (by alias or by name)
connect api.getInfo    -> info
connect api.submitInvoice -> SubmitInvoice

# API operation straight to a database query or command
connect api.getItem2   -> TodoItem.findById
connect api.deleteUser -> User.deleteUser

# API operation to an FSM event (via the fsm alias)
connect api.approveInvoice -> fsm.approve

# Broker topic to an expression
connect svcBroker.question.day -> GetDayString

# Binding a config sub-object to a used database
connect myCfg.dbConfig -> appDb
```

A connection target may also be an expression *call* with named arguments, where
each argument is bound to a used alias:

```ocean-dsl
connect svcBroker.dayTip  -> HandleDayTip(ctx:svcCtx)
connect api.getTimeAnswer -> HandleTimeQuestion(ctx:svcCtx, getday:GetDay, getTimeDate:GetTimeDate)
```

### 15.1 Trusted request-context bindings

For a service connection that begins at an API operation, an indented `bind`
line may supply a target logic parameter from verified Ocean request context:

```ocean-dsl
connect api.softDeleteUser -> User.softDeleteUser
    bind deletedBy <- context.actor
```

`bind <destination> <- context.<name>` belongs to the immediately preceding
`connect` declaration and must be more deeply indented than that declaration.
`destination` names a parameter on the connection target; `name` names a
standard item from the verified `Ocean-Context` described by `dsl.api`.

Bindings are typed and validated after the API operation and target signatures
are resolved. The generator applies the conservative context projection rules
defined by `dsl.api`; it does not infer a binding from parameter names.

The short form is intentionally limited to one trusted context item and one
target parameter. The reserved future form `fields(...)` can explicitly select
additional safe fields when that feature is implemented.

### 15.2 Connection authorization

A connection beginning at an implemented API endpoint may attach an
authorization policy evaluated before its target is invoked:

```ocean-dsl
connect api.findUserById -> User.findUserById
    authorize context.actor
        authenticated
        anyOf
            subject equals id
            role "platform-admin"
```

`context.<name>` is a trusted request-context item and must be declared by the
source API endpoint with `@from-context:<name>`. The policy rules, grouping,
reference validation, and HTTP `401`/`403` behavior are defined canonically in
[`dsl.authorization`](authorization.md). `bind` and `authorize` may coexist
beneath one connection.

---

## 16. Connection Compatibility

Every connection is type-checked.

Without a mapper, the source output and target input must be compatible under the Ocean type system.

Conceptually:

```text
source output type ── compatible with ── target input type
```

Unresolved endpoints, invalid direction, incompatible types, or ambiguous aliases are validation errors.

---

## 17. Mappers

A connection may declare a mapper when source and target types require explicit transformation.

Conceptual form:

```ocean-dsl
connect <source> -<mapper>-> <target>
```

Example:

```ocean-dsl
connect api.createOrder -MapOrderRequest-> orderFlow.orderRequest
```

The mapper must resolve to a compatible expression or function. Exact mapper parsing and invocation must follow the common connection grammar implemented by Ocean.

---

## 18. Fan-out

A source may connect to multiple targets where supported:

```ocean-dsl
connect orderFlow.orderCreated -> broker.orderCreated, orders.create
```

Each target is validated independently. Fan-out does not relax type compatibility or endpoint-direction rules.

Current binding modifiers may restrict a connection line to one bind even when it contains multiple targets.

---

## 19. Common Connection Directions

Supported service flows include:

| Source                               | Target                              | Meaning                                           |
| ------------------------------------ | ----------------------------------- | ------------------------------------------------- |
| API input                            | Component, FSM, or expression input | Route an incoming operation to behavior           |
| API input                            | Database query or command           | Serve an operation directly from the entity store |
| API input                            | FSM event (via the fsm alias)       | Drive a state transition from a request           |
| Component or FSM output              | API output                          | Produce an API response                           |
| Broker topic                         | Component, FSM, or expression input | Consume a message                                 |
| Component, FSM, or expression output | Broker topic                        | Publish a message                                 |
| Component or FSM output              | Database command                    | Persist or mutate data                            |
| Database query result                | Component or FSM input              | Route retrieved data                              |
| Component output                     | Component input                     | Chain composed behavior                           |

The referenced element contracts determine the precise valid directions and types.

---

## 20. Broker Patterns and Binds

Broker connections may carry a pattern-specific bind such as repetition or
timeout. The bind is appended with `&`, and durations use the `<value>:<unit>`
form:

```ocean-dsl
connect api.method   -> broker.topic
connect GetDayTip    -> svcBroker.dayTip & each(10:s)
connect GetTimeDate  -> svcBroker.question.timedate & timeout(3:s)
```

Publish targets must use publish-capable topics; subscription sources must use subscribe-capable topics; request-response flows must use compatible request-response topics.

Only one bind per connection line is currently supported. (A `,` in a connection
separates multiple fan-out targets — see Section 18 — and is distinct from the
`&` bind.)

---

## 21. Aggregation

`aggregate` combines values from several sources into one typed result, then
optionally delivers that result to one or more targets. It is distinct from a
point-to-point `connect`.

```ocean-dsl
aggregate <name>
    trigger <implemented-api-call>
    schedule <named-or-inline-schedule>

    result: <type>

    collect trigger [using <expression>]
    collect <source> [using <expression>]

    deliver <target> [using <expression>]

    respond [using <expression>]
    timeout: <duration>
```

An aggregate has exactly one activation: either `trigger` or `schedule`. A
`trigger` is one implemented API call. `collect trigger` adds that API request
payload to the result.

`result` is required. For `List<T>`, collected values must resolve to `T` and
are retained in `collect` declaration order. For a record result, collected
values populate its fields in declaration order. Collectors may execute
concurrently; deliveries execute in declaration order. Mappers are optional
but must be type-compatible.

`respond` applies only to a non-`Void` API trigger. Without a mapper it returns
the result directly; with a mapper it converts the result to the API response
type. Scheduled aggregates do not respond.

Example: `ingress.receivePurchaseOrder` is the implemented API through which
PO-1 enters the service. `po2.generatePurchaseOrder` is a used API that
supplies PO-2, and `orders.purchaseOrder.po3` is a broker request-response
source for PO-3. The aggregate collects the three orders, delivers their list
to Warehouse, then publishes a mapped shipment request.

```ocean-dsl
aggregate ProcessPurchaseOrders
    trigger ingress.receivePurchaseOrder

    result: List<PurchaseOrder>

    collect trigger
    collect po2.generatePurchaseOrder
    collect orders.purchaseOrder.po3

    deliver warehouse.receivePurchaseOrders
    deliver orders.shipment.request using PurchaseOrdersToShipmentRequest

    respond using PurchaseOrdersToAcknowledgement
```

### Concise one-line form

The one-line form remains available when an API call composes and returns one
result without explicit deliveries:

```ocean-dsl
aggregate api.getTimeStringAnswer -TimeAnswerToString-> day.getDay | td.getDate using PrettyDate | td.getTime using PrettyTime | tip.getDayTip using PrettyTip - timeout 2s
```

Its equivalent named aggregate is:

```ocean-dsl
aggregate GetTimeStringAnswer
    trigger api.getTimeStringAnswer
    result: TimeAnswer

    collect day.getDay
    collect td.getDate using PrettyDate
    collect td.getTime using PrettyTime
    collect tip.getDayTip using PrettyTip

    timeout: 2s
    respond using TimeAnswerToString
```

In the one-line form, the left-side API call maps to `trigger`, each pipe
target maps to `collect`, per-target mappers remain collect mappers, and the
left-side mapper maps to `respond using`.

---

## 22. Imports

The service section supports imports of:

- `api`;
- `broker`;
- `expression`;
- `fsm`;
- `component`;
- `integration`;
- `config`.

Example:

```ocean-dsl
@service

@import api O.std.order.OrderAPI@1.0.0 as OrderAPI
@import fsm O.std.order.OrderFSM@1.1.0 as OrderFSM
@import expression O.std.validation.Validator@2.0.0 as Validator
@import component O.core.AuditTrail@3.0.0 as AuditTrail
@import integration O.partner.orders.OrderSync@1.0.0 as OrderSync
```

The same import form may use `P.` references for local Pre-baked items where an item of the imported type is available.

Imported definitions are referenced through their declared import aliases. An
imported integration is hosted exactly like a local one:

```ocean-dsl
impl integration OrderSync as orderSync
```

---

## 23. Includes

A service file may include compatible `@service` definitions through the common `@include` mechanism.

Included and local services form one logical service section and are validated together. Cross-section inclusion is invalid.

The exact reference, placement, transitivity, and cycle rules are defined by `dsl.include`.

---

## 24. Complete Example

A complete `@service` file
(`ocean-examples/0001-task-manager/app-todo-service.ocn`):

```ocean-dsl
@service

TodoService

    @perspectives: version:0.1.0, lifestyle:stable

    use config TodoServiceConfig as myCfg
    impl api TodoApi as api on myCfg.apiConfig.port
    use database TodoDB as appDb
    use expression GetInfo as info
    use expression GetItem as get
    use expression GetAll as all
    use expression CreateItem as crt
    use expression AdjustItem as adj
    use expression DeleteItem as dlt

    connect myCfg.dbConfig -> appDb

    connect api.getInfo -> info
    connect api.getItem -> get
    connect api.getItem2 -> TodoItem.findById
    connect api.findItemByTitle -> TodoItem.findByTitle
    connect api.findItemByTitleAndPriority -> TodoItem.findByTitleAndPriority
    connect api.findItemByTitleOrPriority -> TodoItem.findByTitleOrPriority
    connect api.listItems -> all
    connect api.createItem -> crt
    connect api.adjustItem -> adj
    connect api.deleteItem -> dlt
```

This example demonstrates:

- a `@perspectives:` metadata line on the service;
- `use config` with an alias and `impl api ... on <configPath>`;
- `use database` and binding its config sub-object with `connect myCfg.dbConfig -> appDb`;
- aliased `use expression` declarations;
- connections from API operations to expressions (by alias) and straight to
  database queries/commands (`TodoItem.findById`).

For brokers, context, `use function` types, expression-call targets, and `&`
bind modifiers, see `ocean-examples/0003-answer-async/50-ans-service.ocn` and
Sections 12, 15, and 20.

---

## 25. Deployment Boundary

A service is deployable, but deployment policy is defined separately.

`@service` defines what the unit contains and how it behaves. `@deploy` determines how a service is materialized for an environment or target.

```text
@service → logical deployable unit
@deploy  → environment and target materialization
```

Generators may derive code, configuration, infrastructure integration, and packaging from the combined model.

---

## 26. Validation

Validation includes:

- valid and unique service names;
- resolution of every used or implemented item;
- required aliases and alias uniqueness;
- required port binding for every implemented API;
- port and configuration-path resolution;
- API dependency versus fulfillment semantics;
- initialization target and argument validation;
- connection endpoint and direction validation;
- source/target type compatibility;
- mapper resolution and signature compatibility;
- broker-pattern compatibility;
- one-bind-per-connection limitation;
- aggregation validation;
- import and include validation;
- section-specific metadata validation.

Failures must identify the service and offending declaration.

---

## 27. Invalid Examples

Missing implemented-API port:

```ocean-dsl
impl api PublicAPI as api
```

Missing required alias:

```ocean-dsl
use config AppConfig
```

Abbreviated keywords:

```ocean-dsl
use expr Validator
use comp AuditTrail
```

These are invalid; use `use expression` and `use component`.

Type-incompatible connection:

```ocean-dsl
connect api.createOrder -> orderFlow.integerInput
```

Invalid when the source and target types are incompatible and no valid mapper is supplied.

---

## 28. Rules and Constraints

- A service file starts with `@service`.
- A file may define multiple services.
- Services are deployable orchestration boundaries.
- `use api` declares an API dependency.
- `impl api` declares API fulfillment and requires `on <target>`, commonly a configuration path such as `myCfg.apiConfig.port`.
- `impl integration <IntegrationName> as <alias>` hosts reusable orchestration; its alias is required and unique within the service.
- Hosted integrations contribute behavior to the service but are not independent deploy targets.
- An implemented-API port may be a literal numeric port, a declared symbol, or a resolvable configuration path.
- An optional `@perspectives:` (and `@tags:`) line may follow the service name.
- `use function` may bring in a named function or a function type (`Supplier<T>`, `Func<In, Out>`); a function type requires an alias.
- A connection bind modifier is appended with `&` (e.g. `& timeout(3:s)`, `& each(10:s)`); `,` separates fan-out targets.
- A connection target may be an expression call with named arguments bound to aliases.
- Config, context, broker, and FSM uses require aliases.
- Database, expression, component, function, vault, and API aliases are optional.
- Effective local names are unique within a service.
- Initialization order is preserved.
- Connections use resolvable local endpoints.
- Connections are directional and typed.
- Incompatible types require a valid explicit mapper.
- A source may target multiple endpoints where supported.
- Only one bind per connection line is currently supported.
- Broker endpoints must match their declared messaging patterns.
- Aggregations follow the common aggregation grammar.
- Supported service imports are API, broker, expression, FSM, component, and config.
- Imported definitions are referenced through their import aliases.
- Same-section composition follows `dsl.include`.
- Deployment policy belongs to `dsl.deploy`.

---

## 29. Related Knowledge

- `concept.control-structure` — explains the service's role in Ocean behavioral composition.
- `dsl.api` — defines consumed and implemented API contracts.
- `dsl.broker` — defines messaging topics and patterns.
- `dsl.database` — defines persistence commands, queries, and entities.
- `dsl.expression` — defines stateless logic and mappers.
- `dsl.fsm` — defines stateful behavior.
- `dsl.component` — defines stateless behavioral composition.
- `dsl.config` — defines typed runtime configuration.
- `dsl.context` — defines supported contextual data.
- `dsl.vault` — defines typed secrets used by services.
- `dsl.import` and `dsl.include` — define reuse mechanisms.
- `dsl.deploy` — defines deployment materialization.

These relationships are declared in the metadata.
