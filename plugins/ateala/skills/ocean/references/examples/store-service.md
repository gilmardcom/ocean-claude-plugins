# Store Service

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/examples/store-service/

**Contents**

- [Overview](#overview)
- [Files](#files)
- [DSL source files](#dsl-source-files)
- [Walkthrough](#walkthrough)
- [Other files in the example (not inlined)](#other-files-in-the-example-not-inlined)

Provide a generic, TTL-based in-memory key-value store that other microservices can use to put, get, pop, check, and delete arbitrary JSON values.

Source: https://atlas.ocean-lab.ai/examples/store-service/

Level: easy · Estimated time: 10 minutes · Version: 1.0.0 · Onboarding example: no

Components: 1 services, 0 brokers, 1 databases

Tags: api, cache, in-memory, key-value, microservice, redis, rest, store, ttl

## Overview

This example shows how Ocean models a small, reusable, schema-free store: a
single entity backed by an in-memory Redis database whose full set of
operations — put, get, pop, exists, and delete, including TTL-based expiry and
an atomic pop — is generated automatically from one `@database` declaration,
without any hand-written queries or commands. Its executable DSL and its
Horizon and Voyage explanation are maintained together in this example
directory.

## Files

DSL source comes first, then diagrams; the long Horizon and Voyage walkthroughs are at the end. You can stop after the source files.

| File | Section | Lines |
| --- | --- | --- |
| `in-mem-db.ocn` | `@database` | 23 |
| `store-api.ocn` | `@api` | 29 |
| `store-config.ocn` | `@config` | 29 |
| `store-datatype.ocn` | `@datatype` | 22 |
| `store-deploy.ocn` | `@deploy` | 24 |
| `store-expressions.ocn` | `@expression` | 55 |
| `store-service.ocn` | `@service` | 52 |

## DSL source files

### `in-mem-db.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - key-value-store
#   - in-memory
#   - database
#   - redis
# perspective:
#   feature: store-service
#   service: store-service
# @ocean-meta-end

@database

Database StoreDb
	type = in-memory
    engine = redis
    configType = StoreDbConfig
    tags = in-memory

    Entity StoreItem

    # all PUT, GET, POP, EXISTS and DELETE endpoints are generated automatically
```

### `store-api.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - store-api
#   - rest-api
# perspective:
#   feature: store-service
#   service: store-service
# @ocean-meta-end

@api

StoreApi style: rest
  engine = gin
  configType = ApiConfig
  tags = internal
  version = 1.0.0
  description = Provides in-memory store for other microservices
  basePath = /
  generateSwagger = true

  # -------------------------
  # endpoints
  # -------------------------
  post    /store/put           putData(req:StoreRequest) : String
  get     /store/get/{key}     getData()                 : StoreItem
  get     /store/pop/{key}     popData()                 : StoreItem
  get     /store/exists/{key}  existsData()              : Boolean
  delete  /store/delete/{key}  deleteData()              : _
```

### `store-config.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - configuration
# perspective:
#   feature: store-service
#   service: store-service
# @ocean-meta-end

@config

@import config O.log.config@1.0.0 as LogConfig

StoreConfig
    apiConfig : ApiConfig
    dbConfig  : StoreDbConfig
    logConfig : LogConfig

ApiConfig
    port: Int (default=8080)

StoreDbConfig
  url: String (default=localhost:6379)
  password: String (default=)
  db: Int (default=0)
  retriesNo: Int (default=10)
  retriesDelaySec: Int (default=30)
  tls: Boolean (default=false)
  dialTimeoutSec: Int (default=5)
```

### `store-datatype.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - datatype
# perspective:
#   feature: store-service
#   service: store-service
# @ocean-meta-end

@datatype

StoreRequest
  key:    String
  value:  Json
  remark: *String
  ttlSeconds: Int

StoreItem
  key:    String
  value:  Json
  remark: *String
  expiresAt: *DateTime
```

### `store-deploy.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - deployment
#   - redis
# perspective:
#   feature: store-service
#   service: store-service
# @ocean-meta-end

@deploy

@import service P.db.redis.docker@1.0.0 as RedisInMemDB

Name: STR-deploy

StoreDeploy
    service     StoreService
    replica     1
    export      9093:StoreService.api
    dependsOn   InMemStorDatabase

InMemStorDatabase
    service     RedisInMemDB
```

### `store-expressions.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - expression
# perspective:
#   feature: store-service
#   service: store-service
# @ocean-meta-end

@expression

ConvertStoreRequestToStoreItem
    input:  r:StoreRequest
    output: i:*StoreItem
    logic
        i.key = r.key
        i.value = r.value
        i.remark = r.remark
        exp = TtlToExpire(r.ttlSeconds)
        i.expiresAt = &exp

PutData
    input:   req: StoreRequest
    output:  out: String
    logic
        var s *StoreItem
        s = ConvertStoreRequestToStoreItem(req)
        out = StoreItem.PutStoreItemWithIdAndTtl(req.key, s, req.ttlSeconds)

GetData
    input:   key: String
    output:  out: StoreItem
    logic
        i = StoreItem.GetStoreItem(key)
        out = *i

PopData
    input:   key: String
    output:  out: StoreItem
    logic
        i = StoreItem.PopStoreItem(key)
        out = *i

ExistsData
    input:   key: String
    output:  out: Boolean
    logic
        e = StoreItem.ExistsStoreItem(key)
        out = e

DeleteData
    input:   key: String
    output:  _
    logic
        StoreItem.DeleteStoreItem(key)
```

### `store-service.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - store-api
#   - service
# perspective:
#   feature: store-service
#   service: store-service
# @ocean-meta-end

@service

StoreService

    @perspectives: use:internal, microservice:generic
    @tags: kinaxis, sce
    use config StoreConfig as cfg
    impl api StoreApi as api on cfg.apiConfig.port
    use database StoreDb as db

    # -------------------------
    #   Config binding
    # -------------------------
    # connect cfg.DbConfig -> MyDb

    # -------------------------
    # Config binding
    # -------------------------
    connect cfg.dbConfig -> db

    # -------------------------
    #   Used Expressions
    # -------------------------
    use expression PutData
    use expression GetData
    use expression PopData
    use expression ExistsData
    use expression DeleteData

    # -------------------------
    #   Init
    # -------------------------
    # init MyInitExpr

    # -------------------------
    #   Connections
    # -------------------------
    connect api.putData  ->  PutData
    connect api.getData  ->  GetData
    connect api.popData  ->  PopData
    connect api.existsData  -> ExistsData
    connect api.deleteData  -> DeleteData
```

## Walkthrough

A long, step-by-step explanation (Horizon and Voyage) is in [Store Service walkthrough](store-service-walkthrough.md). Read it only if you need the reasoning behind the design.

## Other files in the example (not inlined)

Fixtures, scripts and supporting artifacts. See the example on the Atlas site for their purpose.

- `api/00_info.hurl` (fixture)

- `api/10_health.hurl` (fixture)

- `api/20_store_lifecycle.hurl` (fixture)

- `api/30_store_pop.hurl` (fixture)

- `api/40_store_not_found.hurl` (fixture)

- `api/50_store_validation.hurl` (fixture)

- `api/60_routing.hurl` (fixture)
