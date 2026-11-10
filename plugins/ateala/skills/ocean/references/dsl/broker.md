# Broker DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/broker/

**Contents**

- [1. Overview](#1-overview)
- [2. Syntax](#2-syntax)
- [3. Broker Definition](#3-broker-definition)
- [4. Topics](#4-topics)
- [5. Naming Conventions](#5-naming-conventions)
- [6. Messaging Patterns](#6-messaging-patterns)
- [7. Delivery Semantics](#7-delivery-semantics)
- [8. Topic Configuration](#8-topic-configuration)
- [9. Publishers](#9-publishers)
- [10. Topic References](#10-topic-references)
- [11. Complete Example](#11-complete-example)
- [12. Imports](#12-imports)
- [13. Rules and Constraints](#13-rules-and-constraints)
- [14. Related Knowledge](#14-related-knowledge)

Defines the Ocean `@broker` DSL section for message brokers, topics, message payloads, messaging patterns, delivery semantics, publishers, and topic configuration.

Source: https://atlas.ocean-lab.ai/dsl/broker/

Status: Accepted · Authority: Normative

## 1. Overview

The `@broker` section defines one or more message brokers that serve as communication hubs between system components.

Each broker may define multiple topics.

Each topic defines:

- a topic name;
- a datatype representing the message payload;
- a messaging pattern;
- optional messaging configuration.

A broker file:

- starts with `@broker`;
- must define at least one broker;
- may define multiple brokers sequentially;
- must not contain nested broker definitions;
- may contain comments throughout the file.

---

## 2. Syntax

A broker is defined by its name followed by its topics.

```ocean-dsl
@broker

<BrokerName>
    <topicName> : <Datatype> as <pattern> [with <config>]
```

Example:

```ocean-dsl
MyBroker
    engine = nats
    configType = MyBrokerConfig

    dayTip            : String                           as event
    question.day      : _-String                         as request-response with timeout:1s
    question.timedate : TimeDateRequest-TimeDateResponse as request-response with timeout:1s
```

Each topic is defined on one line.

---

## 3. Broker Definition

Each broker groups a set of related messaging topics.

General format:

```ocean-dsl
<BrokerName>
    @perspectives: <key>:<value>, ...
    engine = <engineName>
    configType = <ConfigRef>
    tags = <tag1>, <tag2>, ...

    <topicName> : <Datatype> as <pattern> [with <config>]
```

### 3.1 Broker Name

| Element      | Description                                  | Example       |
| ------------ | -------------------------------------------- | ------------- |
| `BrokerName` | Name of the broker containing related topics | `OrderBroker` |

A broker name follows the same naming convention as a datatype name.

It:

- begins with an uppercase letter;
- contains only letters, digits, and `_`.

Example:

```text
OrderBroker
PaymentBroker
AuditBroker
```

### 3.2 Broker-Level Attributes

Before its topics, a broker may declare attributes, one per line, indented under
the broker name:

```ocean-dsl
AnswerBroker
    @perspectives: version:0.1.0, lifestyle:stable
    engine = nats
    configType = AnswerBrokerConfig
    tags = primary, shared

    dayTip : String as event
```

| Attribute        | Required | Description                                                                   |
| ---------------- | -------- | ----------------------------------------------------------------------------- |
| `engine`         | Yes      | Messaging engine that backs the broker, such as `nats`.                       |
| `configType`     | Yes      | Datatype that supplies the broker's runtime configuration (see `dsl.config`). |
| `tags`           | No       | Comma-separated classification tags, such as `primary, shared`.               |
| `@perspectives:` | No       | Comma-separated `key:value` perspective metadata (see `dsl.perspective`).     |

---

## 4. Topics

A topic represents a messaging channel handled by a broker.

Syntax:

```ocean-dsl
<topicName> : <Datatype> as <pattern> [with <config>]
```

Examples:

```ocean-dsl
orderCreated : PurchaseOrder  as event
```

```ocean-dsl
getOrderDetails : OrderRequest as request with timeout:2s
```

Each topic is declared on one line.

### 4.1 Topic Definition

| Element     | Required | Description                             | Example         |
| ----------- | -------- | --------------------------------------- | --------------- |
| `TopicName` | Yes      | Name of the topic                       | `orderCreated`  |
| `Datatype`  | Yes      | Message payload type                    | `PurchaseOrder` |
| `Pattern`   | Yes      | Messaging pattern declared with `as`    | `event`         |
| `Config`    | No       | Comma-separated messaging configuration | `timeout:2s`    |

The topic datatype must resolve to a valid Ocean datatype, including:

- primitive;
- compound;
- user-defined;
- built-in.

---

## 5. Naming Conventions

Broker naming follows the same conventions as `@datatype`.

### 5.1 Broker Names

Broker names follow datatype naming rules.

Examples:

```text
OrderBroker
UserBroker
PaymentBroker
```

### 5.2 Topic Names

Simple topic names follow datatype field naming rules and use lowercase
camelCase.

Examples:

```text
orderCreated
userRegistered
invoicePaid
```

Hierarchical topic names may contain dot-separated segments. Each segment must
individually follow datatype field naming rules:

```text
question.day
question.timedate
```

---

## 6. Messaging Patterns

Each topic declares its messaging pattern using `as`.

Syntax:

```ocean-dsl
<topicName> : <Datatype> as <pattern>
```

Supported patterns are:

| Pattern            | Description                                              | Example                                                      |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------------------ |
| `event`            | Basic event broadcast using publish-subscribe            | `orderCreated : PurchaseOrder as event`                      |
| `request`          | Message expects a reply                                  | `getOrder : OrderRequest as request`                         |
| `response`         | Reply message sent to a requester                        | `orderDetails : OrderResponse as response`                   |
| `request-response` | Shorthand combining a request and corresponding response | `updateOrder : UpdateOrder-OrderUpdated as request-response` |

### 6.1 Event

The basic event.

```ocean-dsl
orderCreated : PurchaseOrder as event
```

---

### 6.2 Request

A request topic represents a message that expects a response.

```ocean-dsl
getOrderDetails : OrderRequest as request
```

Optional configuration may be added:

```ocean-dsl
getOrderDetails : OrderRequest as request with timeout:2s
```

---

### 6.3 Response

A response topic represents a reply sent to a requester.

```ocean-dsl
orderDetails : OrderResponse as response
```

---

### 6.4 Request-Response

A `request-response` topic is shorthand that combines a request and its corresponding response in one topic definition.

Syntax:

```ocean-dsl
<topicName> : <RequestDatatype>-<ResponseDatatype> as request-response
```

Example:

```ocean-dsl
updateOrder : UpdateOrder-OrderUpdated as request-response
```

It is semantically equivalent to defining the request and corresponding response separately, while grouping them into one definition for clarity and maintainability.

Configuration may also be applied:

```ocean-dsl
updateOrder : UpdateOrder-OrderUpdated as request-response with retries:3, timeout:500ms
```

---

## 7. Delivery Semantics

Broker messaging may use different delivery semantics.

Supported semantics include:

- `at-most-once`
- `at-least-once`
- `exactly-once`

The default delivery semantic is:

```text
at-most-once
```

### 7.1 At-Most-Once

A message is delivered at most once.

Duplicate delivery is avoided, but a message may be lost if delivery fails.

### 7.2 At-Least-Once

A message is delivered one or more times until successful delivery is achieved.

Consumers must account for possible duplicate messages.

### 7.3 Exactly-Once

A message is logically processed exactly once where supported by the selected broker technology and runtime implementation.

The concrete guarantees depend on the generated messaging technology.

---

## 8. Topic Configuration

A topic may define optional configuration using `with`.

Syntax:

```ocean-dsl
<topicName> : <Datatype> as <pattern> with <key>:<value>, <key>:<value>
```

Example:

```ocean-dsl
getOrderDetails : OrderRequest as request with timeout:2s
```

Multiple configuration values are comma-separated:

```ocean-dsl
updateOrder : UpdateOrder-OrderUpdated as request-response with retries:3, timeout:500ms
```

### 8.1 Configuration Options

| Key             | Description                        | Example                   |
| --------------- | ---------------------------------- | ------------------------- |
| `timeout`       | Timeout duration for requests      | `2s`, `500ms`             |
| `retries`       | Number of retry attempts           | `3`                       |
| `mode`          | Delivery mode                      | `Sync`, `Async`, `Stream` |
| `correlationId` | Strategy for tracking message flow | `uuid`, `traceId`         |

Additional configuration may depend on the messaging engine and generated technology.

---

## 9. Publishers

Ocean broker usage supports publisher behaviors including:

| Publisher      | Description                                                                      |
| -------------- | -------------------------------------------------------------------------------- |
| `pubOnce`      | Publishes a message only when explicitly invoked by another component.           |
| `pubRecurring` | Periodically invokes its handler and publishes messages according to a schedule. |

`pubOnce` and `pubRecurring` describe publisher behavior associated with broker topics.

Their concrete use and orchestration may be defined by the DSL construct that consumes or connects to the broker.

---

## 10. Topic References

Topics are referenced using their fully qualified name:

```text
<BrokerName>.<topicName>
```

Examples:

```text
OrderBroker.orderCreated
UserBroker.userRegistered
BillingBroker.invoicePaid
```

The broker name identifies the broker and the topic name identifies the specific messaging channel within it.

---

## 11. Complete Example

The following is a complete `@broker` file
(`ocean-examples/0003-answer-async/20-ans-broker.ocn`):

```ocean-dsl
@broker

AnswerBroker
    @perspectives: version:0.1.0, lifestyle:stable
    engine = nats
    configType = AnswerBrokerConfig
    tags = primary, shared

    dayTip            : String                           as event
    question.day      : _-String                         as request-response with timeout:1s
    question.timedate : TimeDateRequest-TimeDateResponse as request-response with timeout:1s
```

This example demonstrates:

- broker perspectives and version information;
- a concrete broker engine;
- broker configuration through `configType`;
- broker tags;
- an event topic;
- hierarchical topic names using dot notation;
- a request-response topic without a request payload using `_`;
- a request-response topic with explicit request and response datatypes;
- topic-specific timeout configuration.

---

## 12. Imports

A `@broker` file may import datatypes from the Ocean Repository.

Imported datatypes can be used as message payload types.

Example:

```ocean-dsl
@broker

@import datatype O.domain.payment.CardInfo@1.2.0 as CardInfo

PaymentEvents
    paymentCreated : CardInfo
```

Only datatypes may be introduced with `@import` in an `@broker` file. Every
import requires an `as <alias>` clause, and topic declarations use that alias as
the message payload type.

Brokers themselves are not imported through this mechanism.

Reusable definitions may also be made available through the supported Ocean include mechanism.

---

## 13. Rules and Constraints

The following rules apply:

- A broker file starts with `@broker`.
- Each file must define at least one broker.
- A file may define multiple brokers.
- Broker definitions must not be nested.
- Each topic is declared on one line.
- Broker names follow datatype naming conventions.
- Simple topic names follow datatype field naming conventions; every segment of
  a hierarchical topic name follows the same convention.
- Every topic must define a valid message datatype.
- Every topic declares a messaging pattern using `as`.
- A topic may optionally define configuration using `with`.
- Multiple configuration options are comma-separated.
- Topics are referenced using `<BrokerName>.<topicName>`.
- Datatypes may be imported from the Ocean Repository for use as message payloads.
- Datatype imports require explicit aliases.
- Brokers themselves cannot be imported through the datatype import mechanism.

---

## 14. Related Knowledge

The `@broker` DSL is related to:

- `dsl.datatype` — defines the datatype system used for topic message payloads.
- `dsl.import` — defines the mechanism for importing selected datatype definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
