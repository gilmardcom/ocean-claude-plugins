# DSL Reference Index

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/

One file per DSL section or directive. Open only the files for the constructs you are using. Start with `overview.md` for source-file rules.

## Overview

| Reference | Status | Summary |
| --- | --- | --- |
| [Ocean DSL](overview.md) | Accepted | Entry point for the Ocean domain-specific language, its source-file rules, directives, section identifiers, and canonical references. |

## Foundations

| Reference | Status | Summary |
| --- | --- | --- |
| [Ocean DSL System Design Model](description.md) | Draft | Explains how Ocean DSL definitions combine data contracts, behavior, composition, interfaces, persistence, messaging, and deployable services into a system model. |
| [Info DSL Reference](info.md) | Approved | Defines the Ocean `@info` section for declaring project-level identity, version, and descriptive metadata for a system or application. |
| [DSL Metadata Reference](metadata.md) | Accepted | Defines generic Ocean DSL metadata using item-level tags and perspectives and an optional file-level group. |
| [Datatype DSL Reference](datatype.md) | Accepted | Defines the Ocean `@datatype` DSL section, including datatypes, primitive and compound types, IDs, enums, errors, envelopes, interfaces, and datatype imports. |
| [Context DSL Reference](context.md) | Accepted | Defines the Ocean `@context` DSL section for service-local volatile shared state used by service logic and orchestration. |
| [Config DSL Reference](config.md) | Approved | Defines the Ocean `@config` section for declaring named, reusable, typed configuration schemas used by deployable services. |
| [Vault DSL Reference](vault.md) | Approved | Defines the Ocean `@vault` section for modeling logical secret stores, typed secret schemas, vault composition, and physical secret-backend generation. |
| [Perspective DSL Reference](perspective.md) | Accepted | Canonical reference ID for Ocean perspective metadata. Perspectives are defined by the generic metadata model in `dsl.metadata`; this document identifies the concept and points to its normative definition. |
| [Tag DSL Reference](tag.md) | Accepted | Canonical reference ID for Ocean classification tags. Tags are defined by the generic metadata model in `dsl.metadata`; this document identifies the concept and points to its normative definition. |

## Interfaces and Infrastructure

| Reference | Status | Summary |
| --- | --- | --- |
| [API DSL Reference](api.md) | Accepted | Defines the technology-independent Ocean `@api` DSL section and its style-specific public API operations, payload representations, transport configuration, and context-derived inputs. |
| [Broker DSL Reference](broker.md) | Accepted | Defines the Ocean `@broker` DSL section for message brokers, topics, message payloads, messaging patterns, delivery semantics, publishers, and topic configuration. |
| [Database DSL Reference](database.md) | Accepted | Defines the Ocean `@database` DSL section, including databases, entities, relationships, keys, indexes, encryption, queries, and commands. |
| [Integration DSL Reference](integration.md) | Accepted | Defines the Ocean `@integration` section for composing reusable integration flows between APIs, brokers, databases, files, and other resources using implemented contracts, used dependencies, typed connections, mapping expressions, scheduling, and aggregation. |

## Behavior and Composition

| Reference | Status | Summary |
| --- | --- | --- |
| [Expression DSL Reference](expression.md) | Accepted | Defines the Ocean expression model for stateless, side-effect-free reusable logic, including explicit named expressions, implicit inline expressions, typed inputs and outputs, built-in functions, logic blocks, and external implementations. |
| [Authorization Policy DSL Reference](authorization.md) | Approved | Defines declarative authorization policies (authenticated, role, scope, subject, tenant, organization, anyOf, allOf) and how expressions and connections attach them. |
| [Schedule and Trigger DSL Reference](schedule.md) | Accepted | Defines reusable, technology-independent schedules and their use as triggers for executable Ocean DSL items. |
| [FSM DSL Reference](fsm.md) | Accepted | Defines the Ocean `@fsm` DSL section for modeling stateful domain behavior using entity-controlled or in-memory finite-state machines, states, events, triggers, transitions, actions, and reusable logic. |
| [Component DSL Reference](component.md) | Accepted | Defines reusable, stateless behavioral components with typed ports, explicit dependencies, and typed internal connections. |
| [Service DSL Reference](service.md) | Approved | Defines the Ocean `@service` section for composing domain behavior, external contracts, infrastructure, configuration, secrets, initialization, aggregation, and typed connections into deployable units. |

## User Interfaces

| Reference | Status | Summary |
| --- | --- | --- |
| [Dashboard DSL Reference](dashboard.md) | Approved | Defines the Ocean `@dashboard` section for composing widgets into reusable, backend-agnostic pages or screens. |
| [Widget DSL Reference](widget.md) | Approved | Defines widget declarations inside Ocean dashboards, including names, predefined types, scalar/list/map properties, datatype binding, and generated functions. |
| [Predefined Widget Types](widget-predefined-types.md) | Approved | Catalogs the predefined Ocean widget types and their implemented properties, defaults, and generated functions. |
| [UI DSL Reference](ui.md) | Approved | Defines the Ocean `@ui` section for composing dashboards into deployable user-interface services and connecting widget functions to APIs and brokers. |
| [Ocean UI Inline Tab Metadata](tab-metadata.md) | Accepted | Define the optional inline metadata block Ocean UI reads from DSL bundle tabs to organize tabs by tags and semantic perspectives. |

## Deployment

| Reference | Status | Summary |
| --- | --- | --- |
| [Deploy DSL Reference](deploy.md) | Approved | Defines the Ocean `@deploy` section for materializing services, UIs, and brokers with replicas, configuration, exported ports, environment values, volumes, dependencies, brokers, and runtime connections. |

## Directives

| Reference | Status | Summary |
| --- | --- | --- |
| [Import DSL Reference](import.md) | Draft | Defines the Ocean `@import` directive for resolving a selected reusable definition and exposing it through a local alias. |
| [Include DSL Reference](include.md) | Draft | Defines the Ocean `@include` directive for statically composing same-section Ocean DSL definitions from versioned Ocean Repository or local Pre-baked references. |
| [Asset DSL Reference](asset.md) | WIP | Defines the `@asset` directive for attaching external files to DSL elements. Assets are copied verbatim into generated projects and are not interpreted or transformed. |
| [Artifact Directive Reference](artifact.md) | Approved | Defines the Ocean `@artifact` directive for resolving, interpreting, transforming, and materializing external semantic files as part of a generated project. |

## Other

| Reference | Status | Summary |
| --- | --- | --- |
| [Ocean-Atlas Reference Overview](reference-overview.md) | Approved | Consolidated overview of Ocean DSL references, directives, foundational concepts, identifiers, and maintenance conventions. |
