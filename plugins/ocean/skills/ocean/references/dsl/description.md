# Ocean DSL System Design Model

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/description/

**Contents**

- [1. Overview](#1-overview)
- [2. Architectural Layers](#2-architectural-layers)
- [3. Data and State](#3-data-and-state)
- [4. Logic and Behavior](#4-logic-and-behavior)
- [5. System Boundaries](#5-system-boundaries)
- [6. End-to-End Model](#6-end-to-end-model)
- [7. Illustrative Definitions](#7-illustrative-definitions)
- [8. Design Benefits](#8-design-benefits)
- [9. Scope and Evolution](#9-scope-and-evolution)
- [10. Related Knowledge](#10-related-knowledge)

Explains how Ocean DSL definitions combine data contracts, behavior, composition, interfaces, persistence, messaging, and deployable services into a system model.

Source: https://atlas.ocean-lab.ai/dsl/description/

Status: Draft · Authority: Explanatory — not final: content may be incomplete or change.

## 1. Overview

Ocean is a domain-specific language for modeling systems using readable,
declarative, typed, and code-generation-oriented definitions.

The model separates concerns that are commonly mixed in implementation code:

- data contracts;
- pure computations;
- stateful behavior;
- behavioral composition;
- public and asynchronous interfaces;
- persistence;
- runtime configuration and deployment.

This separation allows each definition to remain focused while still
participating in a connected system model.

---

## 2. Architectural Layers

The central behavioral composition model is:

```text
Service
  ├── public boundaries: APIs and brokers
  ├── behavior: components, FSMs, and expressions
  ├── state: databases and contexts
  └── contracts: datatypes
```

Behavior composes from smaller units into larger ones:

```text
Expression ─┐
            ├──> FSM ──> Component ──> Service
Context  ───┘                 │             │
                              │             ├──> API
Datatype ─────────────────────┴─────────────├──> Broker
                                            └──> Database
```

The diagram is conceptual. Individual DSL references define the normative
syntax and allowed connections.

---

## 3. Data and State

### 3.1 Datatypes

`@datatype` defines reusable, technology-independent data contracts.

Datatypes provide the shared type system for API requests and responses, broker
messages, database entities, expression inputs and outputs, FSM events, context
fields, and component ports.

Datatype definitions contain structure, not behavior or persistence rules.

### 3.2 Contexts

`@context` defines volatile service-local state used during a request, session,
or orchestration.

Context access is explicit. Context is not a replacement for a database and is
not persisted as durable entity state.

### 3.3 Databases

`@database` maps existing datatypes to persistent entities and adds
persistence-specific behavior such as:

- relationships;
- keys;
- indexes and constraints;
- encrypted fields;
- typed queries;
- typed commands.

Entity fields remain defined by the backing datatype rather than being
redeclared inside the database model.

---

## 4. Logic and Behavior

### 4.1 Expressions

`@expression` defines stateless, side-effect-free computations.

Expressions are suitable for:

- conditions and guards;
- validation;
- mapping and projection;
- calculations;
- reusable decision logic.

Expressions do not own state, access databases directly, or perform external
I/O.

### 4.2 Finite-State Machines

`@fsm` models behavior over time through states, events, handlers, and
transitions.

Ocean supports:

- Entity-Controlled FSMs, whose state is represented by a field of a persisted
  entity;
- In-Memory FSMs, whose temporary state and working data are held in a context.

FSM handlers may transition using `next`, emit outputs, update allowed state,
call expressions, and invoke database commands according to the FSM reference.

### 4.3 Components

`@component` composes FSMs, expressions, and nested components through typed
inputs, outputs, and connections.

Components are stateless composition units. State belongs to the FSMs or other
stateful elements they contain.

The component DSL is currently a designed capability whose exact mapper and
port-reference syntax may be refined during implementation.

---

## 5. System Boundaries

### 5.1 APIs

`@api` defines synchronous or interactive public service contracts.

API styles include REST, gRPC, GraphQL, WebSocket, and SOAP. Operations use
Ocean datatypes for typed inputs and outputs and may obtain environmental values
explicitly from execution context.

An API defines a contract; service composition determines how an operation is
fulfilled.

### 5.2 Brokers

`@broker` defines asynchronous messaging boundaries using brokers and typed
topics.

Topics support event, request, response, and request-response patterns, along
with delivery and topic-specific configuration. A topic is referenced by its
fully qualified name:

```text
<BrokerName>.<topicName>
```

Examples:

```text
OrderBroker.orderCreated
UserBroker.userRegistered
```

### 5.3 Services

`@service` is the intended deployable composition boundary. It connects public
interfaces, messaging, components, FSMs, expressions, contexts, databases, and
configuration.

The service reference has not yet been migrated into Ocean-Atlas. Until that
work is complete, this document treats the service layer as an architectural
concept rather than defining normative service syntax.

---

## 6. End-to-End Model

A typical request or event follows this conceptual path:

1. An API operation or broker topic receives typed data.
2. A service routes that data to a component, FSM, or expression.
3. Expressions validate or transform values without side effects.
4. FSMs apply state-dependent behavior and transitions.
5. Components coordinate multiple behavioral units through typed connections.
6. Database commands persist durable changes when required.
7. The service returns an API result or publishes a broker message.

The same datatype definitions provide compatible contracts across these
boundaries.

---

## 7. Illustrative Definitions

The following fragments show how separate DSL sections describe parts of one
order domain. They are illustrative rather than a complete deployable project.

### 7.1 Data Contract

```ocean-dsl
@datatype

enum OrderStatus
    created
    confirmed
    cancelled

Order
    id     : String
    status : OrderStatus
```

### 7.2 Public Interface

```ocean-dsl
@api

OrderAPI style:rest
    post /orders createOrder(req:CreateOrderRequest) : Order
    get /orders/{id} getOrder(id:String) : Order
```

### 7.3 Messaging Interface

```ocean-dsl
@broker

OrderBroker
    orderCreated   : Order as event
    orderConfirmed : Order as event
    orderCancelled : Order as event
```

### 7.4 Persistent State

```ocean-dsl
@database

Database OrderDB
    engine = postgres
    configType = DatabaseConfig

    Entity Order
        key(id)
        query findById(id:String) : Order
        command T updateOrder(order:Order) : Order
```

### 7.5 Lifecycle Behavior

```ocean-dsl
@fsm

OrderFSM controls Order.status
    key: id
    database: OrderDB
    event confirm in(_) out(_)
    event cancel in(_) out(_)

    state created
        on event confirm:
            next confirmed

        on event cancel:
            next cancelled

    state confirmed
        on enter:
            emit confirmed

    state cancelled
        on enter:
            emit cancelled
```

The service layer would connect `OrderAPI`, `OrderBroker`, `OrderDB`, and
`OrderFSM`. Its exact syntax belongs in the future canonical service reference.

---

## 8. Design Benefits

| Characteristic | Benefit |
| --- | --- |
| Typed contracts | Makes boundaries explicit and enables validation and generation. |
| Separation of state and logic | Keeps expressions pure and state ownership visible. |
| FSM-based behavior | Makes lifecycle rules deterministic, testable, and visualizable. |
| Explicit composition | Makes routing and transformations inspectable. |
| Interface independence | Allows the same domain model to support APIs and asynchronous messaging. |
| Technology-independent models | Allows generators to target different runtimes and infrastructure. |
| Modular definitions | Supports reuse, independent evolution, and focused validation. |

---

## 9. Scope and Evolution

Some parts of the Ocean DSL are accepted canonical references, while others are
still being migrated or remain designed capabilities.

Syntax described by a section-specific normative reference takes precedence
over examples or architectural explanations in this document.

Future evolution may add or refine concerns such as:

- authorization and policy;
- observability and metrics;
- testing and scenarios;
- deployment targets;
- additional generators and runtime integrations.

---

## 10. Related Knowledge

- [Ocean DSL](overview.md) — DSL entry point, source-file rules, and reference index.
- [Datatype](datatype.md) — shared data contracts and type system.
- [Context](context.md) — volatile service-local state.
- [Expression](expression.md) — stateless reusable logic.
- [FSM](fsm.md) — stateful behavior and transitions.
- [Component](component.md) — typed behavioral composition.
- [API](api.md) — public service contracts.
- [Broker](broker.md) — asynchronous messaging contracts.
- [Database](database.md) — persistence modeling.
