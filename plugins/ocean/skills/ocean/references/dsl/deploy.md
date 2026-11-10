# Deploy DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/deploy/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. File Syntax](#3-file-syntax)
- [4. File Name](#4-file-name)
- [5. Deployment Blocks](#5-deployment-blocks)
- [6. Deployable Target](#6-deployable-target)
- [7. Replica Count](#7-replica-count)
- [8. Configuration Bindings](#8-configuration-bindings)
- [9. Exported Ports](#9-exported-ports)
- [10. Environment Variables](#10-environment-variables)
- [11. Volumes](#11-volumes)
- [12. Brokers](#12-brokers)
- [13. Dependencies](#13-dependencies)
- [14. Dependency Graph](#14-dependency-graph)
- [15. Connections](#15-connections)
- [16. Connection Resolution](#16-connection-resolution)
- [17. Multiple Targets](#17-multiple-targets)
- [18. Imports](#18-imports)
- [19. Includes](#19-includes)
- [20. Groups](#20-groups)
- [21. Generation Units](#21-generation-units)
- [22. Per-deploy and Per-file Semantics](#22-per-deploy-and-per-file-semantics)
- [23. Complete Example](#23-complete-example)
- [24. Local and Imported Targets](#24-local-and-imported-targets)
- [25. Validation](#25-validation)
- [26. Invalid Examples](#26-invalid-examples)
- [27. Purpose](#27-purpose)
- [28. Rules and Constraints](#28-rules-and-constraints)
- [29. Related Knowledge](#29-related-knowledge)

Defines the Ocean `@deploy` section for materializing services, UIs, and brokers with replicas, configuration, exported ports, environment values, volumes, dependencies, brokers, and runtime connections.

Source: https://atlas.ocean-lab.ai/dsl/deploy/

Status: Approved · Authority: Normative

## 1. Overview

The `@deploy` section defines how Ocean deployable units are materialized for a runtime environment.

A deployment can select a service, UI, or broker and define:

- replica count;
- configuration bindings;
- exported ports;
- environment variables;
- volume mappings;
- broker bindings;
- dependencies on other deployment blocks;
- runtime connections.

Deployment definitions can drive generators for Docker Compose, Kubernetes, other orchestrators, or Ocean runtime tooling.

---

## 2. Core Principle

Logical system definitions and environment-specific materialization are separate concerns.

```text
Service / UI / Broker / Database / etc.
        │
        ▼
    @deploy block
        ├── replicas
        ├── config
        ├── ports
        ├── environment
        ├── volumes
        ├── dependencies
        └── runtime wiring
                │
                ▼
       deployment artifacts
```

The deploy model does not redefine the behavior of the selected unit. It specifies how that unit participates in an environment.

---

## 3. File Syntax

```ocean-dsl
@deploy [(group:<GroupName>)]

Name: <DeployFileName>
[@import service <RepositoryRef> as <Alias>]
[@import config  <RepositoryRef> as <Alias>]
[@import broker  <RepositoryRef> as <Alias>]

<DeployName>
    service <DeployableName>
    replica <Count>
    config <ConfigName> [as <alias>], ...
    export <HostPort>:<ListenerTarget>, ...
    env <Key>=<Value>, ...
    vol <Source>=<Target>[:<mode>], ...
    broker <BrokerName> [as <alias>]
    dependsOn <DeployName>, ...
    connect <source> -> <target>, ...
```

One file may contain multiple deployment blocks. The file-level `Name:` line and
the `@import` directives both precede the deployment blocks and may appear in
either order relative to each other.

---

## 4. File Name

Every deploy file requires a file-level `Name:` declaration before the deployment
blocks. It and any `@import` directives may appear in either order:

```ocean-dsl
@deploy

Name: TaskMgt-deploy

@import service P.db.postgres.docker@1.0.0 as PostgresqlDB
```

This name identifies the deployment definition file as a whole. It is distinct from the names of the deployment blocks contained in the file.

```text
Deploy file name
    ├── Deployment A
    ├── Deployment B
    └── Deployment C
```

Omitting or malformed `Name:` syntax is invalid.

---

## 5. Deployment Blocks

A deployment block starts with a valid Ocean identifier:

```ocean-dsl
OrderRuntime
    service OrderService
```

Deployment names must be unique across the resolved deploy registry.

Each block represents one independently addressable deployed unit within its deploy file.

---

## 6. Deployable Target

Every deployment block requires a service target declaration:

```ocean-dsl
service <DeployableName>
```

### 6.1 Service Target

Despite the keyword `service`, a service target may resolve to:

- an Ocean service;
- an Ocean UI;
- an Ocean broker;
- a supported imported deployable item.

Examples:

```ocean-dsl
service OrderService
```

```ocean-dsl
service AdminUI
```

```ocean-dsl
service EventBroker
```

---

## 7. Replica Count

```ocean-dsl
replica <Count>
```

Example:

```ocean-dsl
replica 3
```

The default is `1`.

The count must be an integer greater than or equal to `1`.

Replica semantics may depend on the generated orchestrator, particularly for stateful services or brokers.

---

## 8. Configuration Bindings

A deployment may bind one or more configuration schemas:

```ocean-dsl
config MainConfig, EnvironmentConfig as envCfg
```

Configuration aliases are optional.

Every referenced configuration must resolve locally or through a supported import. Effective local names must not conflict.

The deploy block binds environment-specific configuration to the selected deployable unit; it does not redefine the schema.

---

## 9. Exported Ports

The `export` declaration accepts a comma-separated list of host-to-listener
mappings:

```ocean-dsl
export 9091:TodoService.api, 9998:AuditService.AuditReadApi
```

Each entry has this form:

```text
<hostPort>:<ListenerTarget>
```

For a service target, `<ListenerTarget>` is a supported listener identity of the
deployed target, such as an API identity or a UI configuration listener:

```text
9091:TodoService.api
9998:AuditService.AuditReadApi
8081:TodoUi.config
```

Host ports must be unique within a deployment block. Generator-specific
interpretation must preserve deterministic and valid port mappings.

A deployment may publish more than one listener:

```ocean-dsl
export 18080:OrderService.publicApi, 19090:OrderService.adminApi
```

Each host port must be a decimal integer from `1` through `65535`. A direct
form such as `export 8080` is shorthand for `8080:8080`.

---

## 10. Environment Variables

Environment values use comma-separated key-value pairs:

```ocean-dsl
env MODE=prod, REGION=eu-west
```

Each entry is parsed at the first `=` character:

```text
<key>=<value>
```

Keys should be unique within a deployment block. Environment entries configure deployment output and do not alter the referenced Ocean config schema.

Sensitive values should be supplied through vault or platform secret mechanisms rather than embedded as plain environment values.

---

## 11. Volumes

Volume mappings use the `vol` keyword:

```ocean-dsl
vol data=/var/lib/app, config=./config
```

Each comma-separated entry has the form:

```text
<source>=<target>[:<mode>]
```

where `<source>` is a host path or named volume, `<target>` is the container
path, and the optional `<mode>` is a flag such as `ro`:

```ocean-dsl
vol ./secrets/firebase-service-account.json=/run/secrets/firebase-sa.json:ro
```

The precise host, target, named-volume, or platform interpretation is generator-specific.

The canonical keyword is `vol`, not `volume`.

---

## 12. Brokers

A deployment may bind brokers:

```ocean-dsl
broker OrderBroker
```

or:

```ocean-dsl
broker OrderBroker as events
```

Broker aliases are optional at deploy level. Multiple broker declarations are allowed.

Every broker must resolve locally or through a supported import.

---

## 13. Dependencies

Dependencies are declared by deployment-block name:

```ocean-dsl
dependsOn DatabaseRuntime, BrokerRuntime
```

Each dependency must name another deployment block in the same deploy file.

A deployment cannot depend on itself.

Dependencies communicate startup or orchestration ordering. They do not imply that one service may directly access another without an explicit system contract.

---

## 14. Dependency Graph

Deployment dependencies conceptually form a directed graph:

```text
DatabaseRuntime ──┐
                  ├── OrderRuntime ── UI Runtime
BrokerRuntime ────┘
```

The current implementation validates that dependencies exist in the same file and are not self-references.

Indirect cycle detection is not currently enforced. Authors must therefore keep the graph acyclic; future validation should reject cycles deterministically.

---

## 15. Connections

Deploy-level connections use the common connection grammar:

```ocean-dsl
connect <source> -> <target1>, <target2>, ...
```

Example:

```ocean-dsl
connect events.createOrder -> OrderRuntime.start
```

Current deploy endpoint format is:

```text
<alias>.<method-or-topic>
```

Each connection requires one source and at least one target.

---

## 16. Connection Resolution

Endpoint aliases may refer to:

- the deployed target;
- a broker bound by the block;
- a configuration binding where supported;
- an imported alias available to the deploy file.

Broker endpoints are checked against declared broker topics.

Unknown aliases, malformed endpoints, missing methods/topics, or unresolved broker topics are invalid.

The deploy registry currently validates structural endpoint resolution; deeper type and direction validation may depend on the referenced models and generator.

---

## 17. Multiple Targets

A connection may contain multiple targets:

```ocean-dsl
connect events.created -> OrderRuntime.created, AuditRuntime.record
```

Each target must be resolvable. The connection expresses deployment-time fan-out and must not be used to bypass service-level typed orchestration.

---

## 18. Imports

Deploy files support reusable deployment dependencies through imports.

The declared import set consists of:

- services;
- configurations;
- brokers.

Example:

```ocean-dsl
@deploy

@import service O.std.cache.CacheService@1.0.0 as CacheService
@import config O.std.cache.CacheConfig@1.0.0 as CacheConfig

Name: CacheDeployment
```

Compatible `P.` references may identify local Pre-baked items. Imported items are referenced through their aliases.

---

## 19. Includes

A deploy file may use `@include` to compose compatible deploy definitions according to the common inclusion rules.

Included content must remain compatible with the `@deploy` section. Inclusion must not create duplicate deployment names or duplicate deployment groups.

---

## 20. Groups

A deploy file may declare a group in its section header:

```ocean-dsl
@deploy (group:backend-prod)
```

The group is file-level metadata inherited by deployment blocks unless refined by supported metadata rules.

Each deploy file must have a unique effective group across the model. An omitted group is normalized to the `default` group and also participates in uniqueness validation.

---

## 21. Generation Units

Each deploy file group represents an independently generated deployment unit.

Possible generated layouts include:

```text
out/deploy/<group>/docker-compose.yml
```

or:

```text
out/deploy/docker-compose-<group>.yml
```

The exact path and artifact format are generator-specific. Group uniqueness guarantees that output ownership remains deterministic.

---

## 22. Per-deploy and Per-file Semantics

The model distinguishes:

```text
Deploy file
    ├── Name
    ├── Group
    └── one or more deployment blocks
            ├── target
            ├── replicas
            ├── config
            └── runtime bindings
```

Deployment artifacts may be organized per file/group while containing multiple deployment blocks.

This is more precise than assuming every block must always generate a separate orchestrator file.

---

## 23. Complete Example

A complete `@deploy` file
(`ocean-examples/0003-answer-async/60-ans-deploy.ocn`):

```ocean-dsl
@deploy

Name: AnswerService-PRD-deploy

@import service P.broker.nats.docker@1.0.0 as NATS

AnswerDeploy
    service     AnswerService
    replica     1
    export      9093:AnswerService.api
    dependsOn   Broker

DayTipDeploy
    service     DayTipService
    replica     1
    export      9094:DayTipService.api
    dependsOn   Broker

TimeDateDeploy
    service     TimeDateService
    replica     1
    export      9095:TimeDateService.api
    dependsOn   Broker

DayDeploy
    service     DayService
    replica     1
    export      9096:DayService.api
    dependsOn   Broker

AnswerBrokerDeploy
    service     AnswerBroker
    replica     1
    dependsOn   Broker

Broker
    service NATS

AnswerUiDeploy
    service     AnswerUi
    replica     1
    export      8083:AnswerUi.config
    dependsOn   AnswerDeploy
```

This example demonstrates:

- a file-level `Name:` followed by an `@import service` from a `P.` Pre-baked reference;
- several service deployment blocks, a broker deployment block, and a UI deployment block;
- an imported broker image (`NATS`) materialized as its own `Broker` block that the others `dependsOn`;
- `export` using the `<hostPort>:<DeployTarget>.<apiName>` form, and `.config` for the UI;
- a block with no `export` (`AnswerBrokerDeploy`).

For `config`, `env`, `vol`, `broker`, `connect`, and the `(group:<name>)`
section header, see Sections 8, 10, 11, 12, 15, and 20.

---

## 24. Local and Imported Targets

A deployment target may be defined locally:

```ocean-dsl
OrderRuntime
    service OrderService
```

or supplied through a supported import:

```ocean-dsl
@import service O.std.cache.CacheService@1.0.0 as CacheService

CacheRuntime
    service CacheService
```

Both forms use the same deployment-block semantics after resolution.

---

## 25. Validation

Validation includes:

- valid `@deploy` header;
- required file-level `Name:` declaration;
- valid and unique deployment names;
- one required, resolvable service target declaration;
- replica count of at least one;
- resolvable configuration bindings;
- resolvable broker bindings;
- comma-separated `hostPort:listenerTarget` export entries;
- unique host ports and listener-target resolution for exports;
- valid environment and volume key-value entries;
- connection source and target presence;
- `alias.method` endpoint structure;
- known connection aliases;
- broker-topic resolution;
- valid import aliases;
- dependency existence within the file;
- prevention of self-dependency;
- unique effective deploy-file group.

Indirect dependency-cycle detection is a future validation requirement and must not be assumed to exist today.

---

## 26. Invalid Examples

Missing file name:

```ocean-dsl
@deploy

OrderRuntime
    service OrderService
```

Invalid replica count:

```ocean-dsl
replica 0
```

Wrong volume keyword:

```ocean-dsl
volume data=/var/lib/app
```

Use `vol`.

Unknown dependency:

```ocean-dsl
dependsOn MissingRuntime
```

Malformed connection endpoint:

```ocean-dsl
connect invalid -> events.created
```

Endpoints require `alias.method-or-topic` form.

---

## 27. Purpose

`@deploy` exists to:

- separate deployment materialization from logical service behavior;
- deploy services, UIs, and brokers;
- define replicas and dependencies;
- bind runtime configuration;
- expose ports;
- supply environment and volume mappings;
- connect deployment-level endpoints;
- generate orchestrator artifacts deterministically;
- support separate environment or group outputs;
- deploy local or reusable Ocean definitions through one model.

---

## 28. Rules and Constraints

- A deploy file starts with `@deploy`.
- A file-level `Name:` declaration is required.
- A file may contain multiple deployment blocks.
- Deployment names are valid Ocean identifiers and globally unique in the deploy registry.
- Every deployment block declares one `service <DeployableName>` target.
- A service target may resolve to a service, UI, broker, or supported imported deployable.
- `replica` defaults to `1` and must be at least `1`.
- Multiple configs may be declared in one comma-separated `config` line.
- Config aliases are optional.
- `export` accepts comma-separated `hostPort:listenerTarget` mappings; host ports are unique within the block.
- Multiple environment values, volumes, brokers, dependencies, and connections are supported.
- The canonical volume keyword is `vol`.
- Broker aliases are optional at deploy level.
- Dependencies refer to deployment-block names in the same file.
- A deployment cannot depend on itself.
- Authors must keep dependency graphs acyclic even though indirect cycle validation is not yet implemented.
- Connection endpoints use `alias.method-or-topic`.
- Connections require one source and at least one target.
- Broker topics referenced by connections must exist.
- Deploy-file groups are unique; omitted groups count as `default`.
- Imports support services, configurations, and brokers.
- Imported items are referenced through their aliases.
- Output naming and orchestrator format are generator-specific.

---

## 29. Related Knowledge

- `dsl.service` — defines deployable backend orchestration units.
- `dsl.integration` — defines reusable integration behavior hosted by a service.
- `dsl.ui` — defines deployable presentation services.
- `dsl.broker` — defines deployable messaging infrastructure and connection topics.
- `dsl.config` — defines schemas bound to deployment environments.
- `dsl.import` — provides reusable services, configs, and brokers.
- `dsl.include` — composes compatible deploy definitions.

These relationships are declared in the metadata.
