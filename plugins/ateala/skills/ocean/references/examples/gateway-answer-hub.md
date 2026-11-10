# Gateway Answer Hub

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/examples/gateway-answer-hub/

**Contents**

- [Overview](#overview)
- [API composition](#api-composition)
- [Routing model](#routing-model)
- [Public boundary](#public-boundary)
- [Files](#files)
- [DSL source files](#dsl-source-files)
- [Diagrams](#diagrams)
- [Walkthrough](#walkthrough)

Expose one public API over independent answer providers, forwarding matching operations automatically and routing one explicitly through broker request-response.

Source: https://atlas.ocean-lab.ai/examples/gateway-answer-hub/

Level: medium · Estimated time: 15 minutes · Version: 1.0.0 · Onboarding example: no

Components: 4 services, 1 brokers, 0 databases

Tags: api, api-composition, api-facade, broker, dashboard, gateway, request-response, ui

## Overview

This example is the reference for Ocean's service-level gateway capability.
`AnswerGatewayService` exposes one public API, `AnswerGatewayApi`, while the
answers themselves are produced by independent provider services behind it.
Its executable DSL and its Horizon and Voyage explanation are maintained
together in this example directory.

## API composition

`AnswerGatewayApi` composes the operations of `TimeDateApi` and `CalendarApi`
with `include`, and declares one operation of its own, `getZoneTime`. The
gateway API owns the transport configuration: every operation is exposed under
its `/answers` base path.

| Public operation | Route | Source |
| --- | --- | --- |
| `getTime` | `GET /answers/time` | included from `TimeDateApi` |
| `getDate` | `GET /answers/date` | included from `TimeDateApi` |
| `getDay` | `GET /answers/day` | included from `CalendarApi` |
| `getDayTip` | `GET /answers/day-tip` | included from `CalendarApi` |
| `getZoneTime` | `GET /answers/zone-time` | declared locally |

## Routing model

`AnswerGatewayService` implements `AnswerGatewayApi` with
`impl gateway api ... including timeDate, calendar`. Public operations with a
matching, contract-compatible operation in an included API are forwarded
automatically:

- `getTime`, `getDate` → `TimeDateService`
- `getDay`, `getDayTip` → `CalendarService`

`getZoneTime` has no provider API counterpart. The gateway connects it
explicitly to the `answers.zoneTime` request-response topic of
`AnswerHubBroker`, which `ZoneTimeService` serves. Its responder expression
calls `GetZoneTimeString("Europe/Amsterdam")`.

```mermaid
flowchart LR
    UI[AnswerHubUi] -->|AnswerGatewayApi| GW[AnswerGatewayService]
    GW -->|automatic: getTime, getDate| TD[TimeDateService]
    GW -->|automatic: getDay, getDayTip| CAL[CalendarService]
    GW -->|request: answers.zoneTime| B[(AnswerHubBroker)]
    B -->|reply| Z[ZoneTimeService]
```

`architecture.mmd` and `broker-communication.mmd` contain the same topology
and the zone-time request-response sequence.

## Public boundary

`AnswerHubUi` uses only `AnswerGatewayApi`; it has no dependency on the
provider APIs. The deployment exports the gateway on port `9100` and the UI on
port `8085`. It also exports `TimeDateService` and `CalendarService` on ports
`9101` and `9102` so that they can be inspected directly; remove those exports
to make the gateway the only externally reachable API.

Gateway-wide policies such as authentication, authorization, CORS, rate
limiting, tracing, and error normalization are deliberately outside this
routing-focused example. They are a planned extension of the gateway model and
will be introduced separately.

## Files

DSL source comes first, then diagrams; the long Horizon and Voyage walkthroughs are at the end. You can stop after the source files.

| File | Section | Lines |
| --- | --- | --- |
| `00-gah-info.ocn` | `@info` | 32 |
| `10-gah-api.ocn` | `@api` | 42 |
| `20-gah-broker.ocn` | `@broker` | 20 |
| `30-gah-config.ocn` | `@config` | 30 |
| `40-gah-expression.ocn` | `@expression` | 42 |
| `50-gah-service-calendar.ocn` | `@service` | 23 |
| `50-gah-service-gateway.ocn` | `@service` | 28 |
| `50-gah-service-time-date.ocn` | `@service` | 23 |
| `50-gah-service-zone-time.ocn` | `@service` | 21 |
| `55-gah-dashboard.ocn` | `@dashboard` | 45 |
| `56-gah-ui.ocn` | `@ui` | 43 |
| `60-gah-deploy.ocn` | `@deploy` | 46 |

## DSL source files

### `00-gah-info.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - gateway
#   - api-facade
#   - broker-request-response
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
# @ocean-meta-end

@info

name: Gateway Answer Hub
version: 1.0.0
title: Gateway Answer Hub
subtitle: One public API for independent answer providers
shortDescription: A focused API gateway example with automatic API forwarding and an explicit broker-backed operation.

description:

Overview
Gateway Answer Hub exposes one public API for time, date, day, day tip, and zone time. It demonstrates the Ocean gateway feature without hiding how each operation is routed.

Automatic forwarding
The gateway includes the TimeDateApi and CalendarApi. Operations with the same name and compatible contracts are forwarded automatically to their owning services.

Explicit routing
Zone time is intentionally not exposed by an internal API. The gateway routes getZoneTime explicitly through a NATS request-response topic to ZoneTimeService.

UI
AnswerHubUi calls only AnswerGatewayApi. Each dashboard action independently invokes one public endpoint, so users see the gateway as the sole public boundary.
```

### `10-gah-api.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - rest-api
#   - gateway-contract
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
# @ocean-meta-end

@api

TimeDateApi style:rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    basePath = /time-date
    generateSwagger = true

    get /time getTime() : String
    get /date getDate() : String

CalendarApi style:rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    basePath = /calendar
    generateSwagger = true

    get /day getDay() : String
    get /day-tip getDayTip() : String

AnswerGatewayApi style:rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    basePath = /answers
    generateSwagger = true

    include TimeDateApi, CalendarApi

    get /zone-time getZoneTime() : String
```

### `20-gah-broker.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - broker
#   - nats
#   - request-response
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
# @ocean-meta-end

@broker

AnswerHubBroker
    @perspectives: version:1.0.0, lifestyle:stable
    engine = nats
    configType = AnswerHubBrokerConfig
    tags = primary, shared

    answers.zoneTime : _-String as request-response with timeout:2s
```

### `30-gah-config.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - configuration
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: shared
# @ocean-meta-end

@config

@import config O.broker.nats.config@1.0.0 as BrokerConfig
@import config O.log.config@1.0.0 as LogConfig

ApiConfig
    port : Int (default=8080)

CommonServiceConfig
    apiConfig    : ApiConfig
    brokerConfig : BrokerConfig
    logConfig    : LogConfig

AnswerHubBrokerConfig
    brokerConfig : BrokerConfig
    logConfig    : LogConfig

UiConfig
    port      : Int (default=8080)
    logConfig : LogConfig
```

### `40-gah-expression.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - expression
#   - answer-provider
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: shared
# @ocean-meta-end

@expression

GetTimeString
    input: _
    output: result:String
    logic
        result = nowTimeString()

GetDateString
    input: _
    output: result:String
    logic
        result = nowDateString()

GetDayString
    input: _
    output: result:String
    logic
        result = today()

GetDayTip
    input: _
    output: result:String
    logic
        result = tipOfTheDay()

GetAmsterdamTime
    input: _
    output: result:String
    logic
        result = GetZoneTimeString("Europe/Amsterdam")
```

### `50-gah-service-calendar.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - service
#   - provider
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: calendar-service
# @ocean-meta-end

@service

CalendarService
    @perspectives: version:1.0.0, lifestyle:stable

    use config CommonServiceConfig as svcCfg
    impl api CalendarApi as api on svcCfg.apiConfig.port
    use expression GetDayString as day
    use expression GetDayTip as tip

    connect api.getDay -> day
    connect api.getDayTip -> tip
```

### `50-gah-service-gateway.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - service
#   - gateway
#   - api-facade
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: answer-gateway-service
# @ocean-meta-end

@service

AnswerGatewayService
    @perspectives: version:1.0.0, lifestyle:stable

    use config CommonServiceConfig as gwCfg
    use api TimeDateApi as timeDate
    use api CalendarApi as calendar
    use broker AnswerHubBroker as broker

    impl gateway api AnswerGatewayApi as gw on gwCfg.apiConfig.port
        including timeDate, calendar

    # getTime, getDate, getDay, and getDayTip are automatically forwarded.
    # Zone time is intentionally broker-backed and therefore routed explicitly.
    connect gw.getZoneTime -> broker.answers.zoneTime
```

### `50-gah-service-time-date.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - service
#   - provider
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: time-date-service
# @ocean-meta-end

@service

TimeDateService
    @perspectives: version:1.0.0, lifestyle:stable

    use config CommonServiceConfig as svcCfg
    impl api TimeDateApi as api on svcCfg.apiConfig.port
    use expression GetTimeString as time
    use expression GetDateString as date

    connect api.getTime -> time
    connect api.getDate -> date
```

### `50-gah-service-zone-time.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - service
#   - broker-responder
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: zone-time-service
# @ocean-meta-end

@service

ZoneTimeService
    @perspectives: version:1.0.0, lifestyle:stable

    use config CommonServiceConfig as svcCfg
    use broker AnswerHubBroker as broker
    use expression GetAmsterdamTime as zoneTime

    connect broker.answers.zoneTime -> zoneTime
```

### `55-gah-dashboard.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - dashboard
#   - gateway
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: answer-hub-ui
# @ocean-meta-end

@dashboard

AnswerHubDashboard
    title: Gateway Answer Hub
    layout: SingleColumnLayout

    Widget GatewayGuide of type Text
        title = One public API, three provider services
        subtitle = API forwarding for time, date, day, and day tip; broker request-response for zone time
        content = <p>Each action calls <b>AnswerGatewayApi</b>. The gateway automatically forwards matching API operations and explicitly delegates zone time to a broker responder.</p>

    Widget TimeButton of type Button
        label = Current time

    Widget DateButton of type Button
        label = Current date

    Widget DayButton of type Button
        label = Current day

    Widget DayTipButton of type Button
        label = Day tip

    Widget ZoneTimeButton of type Button
        label = Amsterdam zone time

AboutDashboard
    title: About this example
    layout: SingleColumnLayout

    Widget AboutText of type Text
        title = Gateway routing patterns
        subtitle = Automatic forwarding and explicit routing in one service
        content = <p>This example demonstrates an Ocean service that implements a gateway API. Matching endpoints are forwarded to included APIs automatically. A distinct endpoint is explicitly connected to a broker request-response topic.</p>
```

### `56-gah-ui.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - ui
#   - gateway
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: answer-hub-ui
# @ocean-meta-end

@ui

AnswerHubUi
    @perspectives: version:1.0.0, lifestyle:stable

    framework: htmx
    styling: bootstrap
    template: go-html-template
    backend: go-gin
    static: static
    configType: UiConfig

    nav Answers dashboard = AnswerHubDashboard
    nav About dashboard = AboutDashboard

    header title = Gateway Answer Hub
    header subtitle = One public API for independent answer providers
    header align = center

    footer title = Built with Ocean-lab
    footer subtitle = Gateway feature example
    footer align = center

    use dashboard AnswerHubDashboard as answers
    use dashboard AboutDashboard as about
    use api AnswerGatewayApi as gateway

    connect answers.TimeButton.click -> gateway.getTime
    connect answers.DateButton.click -> gateway.getDate
    connect answers.DayButton.click -> gateway.getDay
    connect answers.DayTipButton.click -> gateway.getDayTip
    connect answers.ZoneTimeButton.click -> gateway.getZoneTime
```

### `60-gah-deploy.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - deployment
#   - gateway
# perspective:
#   feature: gateway-answer-hub
#   boundedContext: answer-hub
#   service: shared
# @ocean-meta-end

@deploy

Name: GatewayAnswerHubDeploy

@import service P.broker.nats.docker@1.0.0 as NATS

TimeDateDeploy
    service TimeDateService
    replica 1
    export 9101:TimeDateService.api

CalendarDeploy
    service CalendarService
    replica 1
    export 9102:CalendarService.api

ZoneTimeDeploy
    service ZoneTimeService
    replica 1
    dependsOn Broker

GatewayDeploy
    service AnswerGatewayService
    replica 1
    export 9100:AnswerGatewayService.gw
    dependsOn TimeDateDeploy, CalendarDeploy, ZoneTimeDeploy, Broker

Broker
    service NATS

AnswerHubUiDeploy
    service AnswerHubUi
    replica 1
    export 8085:AnswerHubUi.config
    dependsOn GatewayDeploy
```

## Diagrams

### `architecture.mmd`

```mermaid
flowchart LR
    UI[AnswerHubUi] -->|AnswerGatewayApi| GW[AnswerGatewayService]
    GW -->|automatic: getTime, getDate| TD[TimeDateService]
    GW -->|automatic: getDay, getDayTip| CAL[CalendarService]
    GW -->|request: answers.zoneTime| BROKER[(AnswerHubBroker)]
    BROKER -->|reply| ZONE[ZoneTimeService]
```

### `broker-communication.mmd`

```mermaid
sequenceDiagram
    participant UI as AnswerHubUi
    participant GW as AnswerGatewayService
    participant B as AnswerHubBroker
    participant Z as ZoneTimeService

    UI->>GW: GET /answers/zone-time
    GW->>B: request answers.zoneTime
    B->>Z: deliver request
    Z-->>B: timezone-aware time
    B-->>GW: reply
    GW-->>UI: zone time
```

## Walkthrough

A long, step-by-step explanation (Horizon and Voyage) is in [Gateway Answer Hub walkthrough](gateway-answer-hub-walkthrough.md). Read it only if you need the reasoning behind the design.
