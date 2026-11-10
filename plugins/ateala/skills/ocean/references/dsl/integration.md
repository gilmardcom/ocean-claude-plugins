# Integration DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/integration/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax Overview](#3-syntax-overview)
- [4. Integration Names](#4-integration-names)
- [5. Direction](#5-direction)
- [6. No Explicit Strategy](#6-no-explicit-strategy)
- [7. Ports](#7-ports)
- [8. Use Declarations](#8-use-declarations)
- [9. Implementation Declarations](#9-implementation-declarations)
- [10. Connect](#10-connect)
- [11. Connection Sources and Targets](#11-connection-sources-and-targets)
- [12. Mappers](#12-mappers)
- [13. Multiple Mappings](#13-multiple-mappings)
- [14. Aggregation](#14-aggregation)
- [15. Scheduling and Triggering](#15-scheduling-and-triggering)
- [16. Authentication and Credentials](#16-authentication-and-credentials)
- [17. Configuration](#17-configuration)
- [18. Hosting Boundary](#18-hosting-boundary)
- [19. Complete Example: Incoming Webhook to Broker](#19-complete-example-incoming-webhook-to-broker)
- [20. Complete Example: Scheduled API Polling](#20-complete-example-scheduled-api-polling)
- [21. Complete Example: Outgoing Event to External API](#21-complete-example-outgoing-event-to-external-api)
- [22. Complete Example: File Import](#22-complete-example-file-import)
- [23. Imports](#23-imports)
- [24. Includes](#24-includes)
- [25. Validation](#25-validation)
- [26. Invalid Examples](#26-invalid-examples)
- [27. Purpose](#27-purpose)
- [28. Rules and Constraints](#28-rules-and-constraints)
- [29. Related Knowledge](#29-related-knowledge)

Defines the Ocean `@integration` section for composing reusable integration flows between APIs, brokers, databases, files, and other resources using implemented contracts, used dependencies, typed connections, mapping expressions, scheduling, and aggregation.

Source: https://atlas.ocean-lab.ai/dsl/integration/

Status: Accepted · Authority: Normative

## 1. Overview

The `@integration` section defines reusable integration-oriented Ocean modules.

An integration connects external or internal communication contracts and resources through typed, directional flows. It can receive data by implementing supported contracts, consume dependencies through `use` declarations, transform data through mappers, and route values using `connect` and `aggregate`.

An integration is not tied to one protocol such as REST, SOAP, gRPC, NATS, Kafka, FTP, or SFTP. Those details belong to the referenced DSL definitions, such as `@api`, `@broker`, `@database`, or future resource sections.

An integration is not independently deployable. A `@service` hosts it with
`impl integration <IntegrationName> as <alias>`; the hosting service supplies
the process, runtime configuration, infrastructure bindings, and `@deploy`
target.

A file begins with:

```ocean-dsl
@integration
```

and may define one or more integrations.

---

## 2. Core Principle

An integration owns communication orchestration, not transport definitions.

Conceptually:

```text
Implemented contracts
        │
        ▼
   Integration module
        │
        ├── connect
        ├── aggregate
        ├── map
        └── schedule / trigger
        │
        ▼
Used contracts and resources
        │
        ▼
Hosting service → @deploy
```

The integration describes the logical flow between contracts and resources. The actual protocol, payload representation, broker engine, database engine, endpoint path, topic behavior, authentication contract, and runtime configuration are defined by the referenced DSL sections.

---

## 3. Syntax Overview

```ocean-dsl
@integration

<IntegrationName>
    direction: <in|out|in/out>

    use config <ConfigName> as <alias>
    use api <ApiName> as <alias>
    impl api <ApiName> as <alias> on <port-or-config-path>
    use broker <BrokerName> as <alias>
    impl broker <BrokerName> as <alias>
    use database <DatabaseName> [as <alias>]
    use expression <ExpressionName> [as <alias>]
    use function <FunctionName> [as <alias>]
    use file <FileName> [as <alias>]

    schedule <schedule-expression>

    init <expression-or-function-call>
    connect <source> -> <target>
    connect <source> -<mapper>-> <target>
    aggregate <aggregation>
```

A file may define multiple integrations.

---

## 4. Integration Names

Integration names follow the Ocean type-name convention: **CamelCase**.

Examples:

```text
SapOrderIntegration
StripePaymentIntegration
PartnerFileImport
CustomerSyncIntegration
```

An integration name must be unique within the resolved integration scope.

---

## 5. Direction

Every integration declares its direction:

```ocean-dsl
direction: in
```

Supported values:

| Direction | Meaning                                                                   |
| --------- | ------------------------------------------------------------------------- |
| `in`      | The integration receives or imports data into the Ocean-generated system. |
| `out`     | The integration sends or exports data from the Ocean-generated system.    |
| `in/out`  | The integration supports both incoming and outgoing flows.                |

Direction is defined relative to the Ocean-generated system, not relative to a specific protocol call.

Examples:

```ocean-dsl
direction: in
```

```ocean-dsl
direction: out
```

```ocean-dsl
direction: in/out
```

---

## 6. No Explicit Strategy

The integration DSL does not declare `push` or `pull` as a required field.

Push or pull behavior is derived from the integration structure.

Examples:

| Integration Shape          | Derived Behavior                         |
| -------------------------- | ---------------------------------------- |
| `impl api`                 | Incoming push                            |
| `impl broker` subscription | Incoming push                            |
| `schedule` + `use api`     | Incoming pull or scheduled outbound flow |
| `use broker` as source     | Event-driven flow                        |
| `use file` with schedule   | File polling or scheduled file import    |

This avoids redundant or contradictory declarations.

Ocean validation and generation may report the detected communication mode as derived metadata.

---

## 7. Ports

Every implemented API selects its listening port with `on`:

```ocean-dsl
impl api PartnerWebhookApi as webhook on 8765
```

The target may be a literal port or a configuration path resolving to a port:

```ocean-dsl
impl api PartnerWebhookApi as webhook on cfg.webhookPort
```

`use config` must introduce the alias used by a configuration path. The `on`
clause is required for every `impl api`; `use api` does not bind a listening
port.

---

## 8. Use Declarations

`use` declares a dependency consumed by the integration.

Supported use declarations may include:

```ocean-dsl
use config IntegrationConfig as cfg
use api PartnerApi as partner
use broker OrderBroker as orders
use database OrderDatabase as db
use expression MapPartnerOrder
use function NormalizeStatus as normalize
use file PartnerOrderFile as myFile
```

A used item may be internal or external. The integration does not need to declare this explicitly. Such classification is derived from the referenced definition, metadata, or deployment context.

Aliases are required for `use config`, `use api`, `impl api`, `use broker`, and
`impl broker`. They are optional for the other supported `use` declarations.
All effective local names must be unique within an integration.

---

## 9. Implementation Declarations

`impl` declares a contract fulfilled by the integration.

Supported implementation declarations may include:

```ocean-dsl
impl api PartnerApi as partner on cfg.partnerPort
```

```ocean-dsl
impl broker PartnerIncomingBroker as partnerEvents
```

An implementation declaration is only valid for contracts or resources that can receive input.

For example:

- an API can be implemented;
- a broker subscription or supported topic contract may be implemented;
- a database is usually used, not implemented;
- a file resource may be implemented only if the file DSL defines a valid receive/import surface.

Invalid implementation targets must produce validation errors.

---

## 10. Connect

`connect` defines one directional flow between two endpoints.

Syntax:

```ocean-dsl
connect <source> -> <target>
    authorize context.<name>
        <authorization-policy>
```

With mapper:

```ocean-dsl
connect <source> -<mapper>-> <target>
```

Example:

```ocean-dsl
connect webhook.receiveOrder -MapPartnerOrder-> orders.orderCreated
```

### Notes

- One `connect` represents one direction only.
- For request-response interactions, request and response flows may be represented as separate directional connections when both directions require explicit orchestration or mapping.
- `bind <destination> <- context.<name>` remains a `@service`-only target
  parameter feature.
- An integration may declare `authorize context.<name>` beneath a connection
  when its hosting runtime supplies verified Ocean context. The policy grammar
  is defined by [`dsl.authorization`](authorization.md); integrations that are
  schedule-, broker-, or file-driven ordinarily have no trusted actor and
  therefore cannot satisfy such a policy.

---

## 11. Connection Sources and Targets

Connection endpoints use a declared alias or, where no alias is declared, the
original name introduced by an `impl` or `use` declaration.

Examples:

```ocean-dsl
connect webhook.receiveOrder -> mapperInput
connect webhook.receiveOrder -MapPartnerOrder-> orders.orderCreated
connect orders.orderApproved -MapOrderApproval-> partner.updateOrder
connect partner.getOrder -MapPartnerOrderResponse-> orders.orderReceived
```

Endpoint semantics are defined by the referenced DSL item.

Examples:

- API operations come from `@api`.
- Broker topics come from `@broker`.
- Database commands and queries come from `@database` Entities.
- File operations come from the file resource definition (WIP).
- Expressions and functions provide callable mapping or processing behavior.

---

## 12. Mappers

A connection may declare a mapper when source and target types are not directly compatible.

```ocean-dsl
connect webhook.receiveOrder -MapPartnerOrder-> orders.orderCreated
```

The mapper must resolve to a compatible expression or function.

Conceptually:

```text
source output type
        │
        ▼
mapper input type
        │
        ▼
mapper output type
        │
        ▼
target input type
```

Without a mapper, source output and target input must be compatible under the Ocean type system.

---

## 13. Multiple Mappings

An integration may define multiple mappings by declaring multiple `connect` statements.

Example:

```ocean-dsl
connect webhook.orderCreated -MapOrderCreated-> orders.orderCreated
connect webhook.orderUpdated -MapOrderUpdated-> orders.orderUpdated
connect webhook.orderCancelled -MapOrderCancelled-> orders.orderCancelled
```

This avoids special cases for APIs or brokers with one method/topic versus many methods/topics.

Each mapping is explicit, directional, and independently validated.

---

## 14. Aggregation

`aggregate` declares an integration-level aggregation using the common Ocean aggregation grammar.

Example:

```ocean-dsl
aggregate webhook.getPartnerOrder ->
    partner.getOrder |
    db.findCustomer |
    orders.orderImported
```

Aggregation is used when an integration flow combines multiple calls, events, resources, or intermediate results.

Aggregation follows the same conceptual model as service aggregation. It is distinct from simple point-to-point `connect`.

---

## 15. Scheduling and Triggering

Integrations may use the [shared Ocean scheduling model](schedule.md) where supported.

Example concepts:

```ocean-dsl
schedule every 10s
```

```ocean-dsl
schedule daily at 14:00:35
```

```ocean-dsl
schedule every Monday at 09:00
```

[Scheduling](schedule.md) is a shared DSL capability, not integration-specific syntax.

In integrations, schedules are commonly used for:

- polling an external API;
- reading files from FTP or SFTP;
- importing data on a recurring basis;
- exporting snapshots or reports;
- periodically publishing messages.

The schedule describes logical behavior. Runtime realization belongs to the generator and deployment target.

---

## 16. Authentication and Credentials

Authentication requirements are not declared directly in `@integration`.

They belong to the referenced resource definitions.

Examples:

- an API may declare its required auth contract;
- a broker may declare its authentication contract;
- a database may declare its credential contract;
- a file resource may declare its access credential contract.

The integration simply uses or implements those resources.

Actual secret values are supplied by deployment/runtime configuration, not by the integration DSL.

---

## 17. Configuration

Integrations may use configuration schemas when they require runtime settings.

```ocean-dsl
use config PartnerIntegrationConfig as cfg
```

Configuration schemas define shape only. Runtime values are supplied by deployment or configuration providers.

Configuration may be used for:

- ports;
- external base URLs;
- feature toggles;
- polling intervals where schedule values are configurable;
- non-secret runtime settings.

Sensitive values should be modeled through authentication/credential contracts and supplied through deployment-supported secret mechanisms.

---

## 18. Hosting Boundary

`@integration` defines logical integration behavior.

An integration must be hosted by a [`@service`](./service.md). The hosting
service is the sole deployable runtime boundary and may host one or more
integrations. It provides the process lifecycle, configuration, infrastructure
bindings, and deployment identity; the integration provides reusable flow and
orchestration behavior.

An integration may be reused by multiple services. Each hosting declaration has
its own required local alias, so generation and diagnostics can identify the
specific hosted instance.

Conceptually:

```text
@integration → reusable integration behavior
@service     → runtime host and API provider
@deploy      → service environment and materialization
```

---

## 19. Complete Example: Incoming Webhook to Broker

```ocean-dsl
@integration

PartnerOrderWebhookIntegration
    @perspectives: version:0.1.0, lifestyle:stable
    tags = external, partner, order

    direction: in

    use config PartnerWebhookConfig as cfg
    impl api PartnerOrderWebhookApi as webhook on 8765
    use broker OrderBroker as orders
    use expression MapPartnerOrderCreated
    use expression MapPartnerOrderCancelled

    connect webhook.orderCreated -MapPartnerOrderCreated-> orders.orderCreated
    connect webhook.orderCancelled -MapPartnerOrderCancelled-> orders.orderCancelled
```

This example represents:

- an incoming integration;
- an implemented API;
- a used broker;
- mapper expressions;
- two directional flows.

It becomes runnable only when hosted by a service:

```ocean-dsl
@service

PartnerWebhookService
    use config PartnerWebhookConfig as cfg
    use broker OrderBroker as orders
    impl integration PartnerOrderWebhookIntegration as webhookFlow
```

---

## 20. Complete Example: Scheduled API Polling

```ocean-dsl
@integration

PartnerOrderPollingIntegration
    @perspectives: version:0.1.0, lifestyle:stable
    tags = external, partner, polling

    direction: in

    use config PartnerPollingConfig as cfg
    use api PartnerOrderApi as partner
    use broker OrderBroker as orders
    use expression MapPartnerOrder

    schedule every 5m

    connect partner.listChangedOrders -MapPartnerOrder-> orders.orderChanged
```

This example represents a scheduled integration that imports data from an external API and publishes mapped events to an internal broker.

The pull behavior is derived from the use of a schedule and an outbound API dependency.

---

## 21. Complete Example: Outgoing Event to External API

```ocean-dsl
@integration

OrderExportIntegration
    @perspectives: version:0.1.0, lifestyle:stable
    tags = external, export, order

    direction: out

    use broker OrderBroker as orders
    use api PartnerOrderApi as partner
    use expression MapOrderExport

    connect orders.orderApproved -MapOrderExport-> partner.createOrder
```

This example represents an outgoing integration triggered by an internal broker topic and sent to an external API.

---

## 22. Complete Example: File Import

```ocean-dsl
@integration

PartnerOrderFileImport
    @perspectives: version:0.1.0, lifestyle:stable
    tags = external, file-import, order

    direction: in

    use file PartnerOrderFile as file
    use broker OrderBroker as orders
    use expression ParsePartnerOrderFile

    schedule daily at 02:00:00

    connect file.read -ParsePartnerOrderFile-> orders.orderImported
```

This example depends on a reusable file resource definition.

The integration does not define SFTP, FTP, path, format, or credentials directly. Those belong to the file resource definition.

---

## 23. Imports

An integration file may import reusable integration dependencies using the
common `@import` grammar, chiefly Pre-baked (`P.`) items. Imported definitions
are addressed through their declared aliases.

---

## 24. Includes

An integration file may include compatible `@integration` definitions through the common `@include` mechanism.

Included and local integrations form one logical integration section and are validated together.

Cross-section inclusion is invalid unless explicitly supported by the common include rules.

---

## 25. Validation

Validation includes:

- valid `@integration` header;
- valid and unique integration names;
- valid direction value;
- resolution of every used and implemented item;
- required aliases where applicable;
- alias uniqueness within integration scope;
- required port binding for implemented APIs;
- port and configuration-path resolution;
- implementation target capability validation;
- connection endpoint resolution;
- connection direction validation;
- source and target type compatibility;
- mapper resolution and signature compatibility;
- broker pattern compatibility;
- API operation compatibility;
- schedule syntax validation where scheduling is used;
- aggregation validation;
- import and include validation;
- section-specific metadata validation.

Failures must identify the integration and offending declaration.

---

## 26. Invalid Examples

### Missing Direction

```ocean-dsl
PartnerIntegration
    use api PartnerApi
```

Invalid because every integration must declare `direction`.

---

### Implementing a Non-receivable Resource

```ocean-dsl
PartnerIntegration
    direction: in
    impl database OrderDatabase as db
```

Invalid unless the database DSL explicitly defines an implementable receive surface.

---

### Missing Port for Implemented API

```ocean-dsl
PartnerIntegration
    direction: in
    impl api PartnerWebhookApi as webhook
```

Invalid because implemented APIs require a port binding.

---

### Incompatible Connection Without Mapper

```ocean-dsl
connect webhook.receiveOrder -> orders.orderCreated
```

Invalid when the API operation output type is not compatible with the broker topic datatype and no mapper is supplied.

---

### Unknown Endpoint

```ocean-dsl
connect webhook.missingMethod -> orders.orderCreated
```

Invalid when `missingMethod` does not exist on the implemented API.

---

## 27. Purpose

`@integration` exists to:

- define integration flows in a protocol-independent way;
- reuse existing Ocean contracts and resources;
- separate integration logic from transport configuration;
- support incoming, outgoing, and bidirectional integration scenarios;
- connect implemented contracts to used dependencies;
- route and transform data through typed connections;
- support scheduled integration behavior;
- keep reusable integration behavior separate from its service host and deployment;
- enable generators to produce listeners, pollers, clients, subscribers, publishers, and mapper wiring from one consistent model.

---

## 28. Rules and Constraints

- An integration file starts with `@integration`.
- A file may define multiple integrations.
- Each integration must have a unique name.
- Each integration must declare `direction: in`, `direction: out`, or `direction: in/out`.
- `push` and `pull` are not declared directly; they are derived.
- `impl` is only valid for receivable or exposable contracts.
- `use` declares dependencies consumed by the integration.
- `impl api` requires a port binding.
- Aliases must be unique within an integration.
- `connect` is directional.
- One `connect` represents one directional flow.
- Type-incompatible connections require a valid mapper.
- Mappers must resolve to compatible expressions or functions.
- Multiple mappings are modeled as multiple `connect` statements.
- `aggregate` follows the common Ocean aggregation grammar.
- Scheduling uses the shared Ocean scheduling model.
- Authentication belongs to referenced resource definitions, not to `@integration`.
- Runtime credential values belong to deployment or secret providers.
- A service hosts every integration; integrations are never deployed directly.
- Metadata such as tags and perspectives may be used for documentation and classification but must not change integration behavior.

---

## 29. Related Knowledge

The `@integration` DSL is related to:

- `dsl.api` — defines API contracts that integrations may implement or use.
- `dsl.broker` — defines broker topics and messaging patterns used by integrations.
- `dsl.database` — defines database resources, queries, commands, and entities.
- `dsl.expression` — defines stateless mappers and transformation logic.
- `dsl.config` — defines typed runtime configuration schemas.
- `dsl.datatype` — defines data contracts used by APIs, brokers, expressions, and mappings.
- `dsl.service` — hosts integrations and defines the deployable runtime boundary.
- `dsl.deploy` — materializes the hosting service for a runtime environment.
- `dsl.import` and `dsl.include` — define reuse mechanisms.

These relationships are declared in the metadata.
