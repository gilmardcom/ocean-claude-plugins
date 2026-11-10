# Audit Service

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/examples/audit-service/

**Contents**

- [Overview](#overview)
- [Files](#files)
- [DSL source files](#dsl-source-files)
- [Walkthrough](#walkthrough)
- [Other files in the example (not inlined)](#other-files-in-the-example-not-inlined)

Record immutable audit actions and list them by actor or tenant, with idempotent recording and offset/limit pagination backed by Postgres.

Source: https://atlas.ocean-lab.ai/examples/audit-service/

Level: medium · Estimated time: 15 minutes · Version: 1.0.0 · Onboarding example: no

Components: 1 services, 0 brokers, 1 databases

Tags: api, append-only, audit, audit-trail, compliance, idempotency, logging, multi-tenant, pagination, postgres, rest, security

## Overview

This example shows how Ocean models an append-only audit trail: a single,
richly structured, immutable action record; a Postgres entity with explicit
indexes and a hand-written idempotent insert command; and REST endpoints
wired directly onto generated database queries and commands, with no
expression layer between them. Its executable DSL and its Horizon and Voyage
explanation are maintained together in this example directory.

## Files

DSL source comes first, then diagrams; the long Horizon and Voyage walkthroughs are at the end. You can stop after the source files.

| File | Section | Lines |
| --- | --- | --- |
| `api/audit-api.ocn` | `@api` | 63 |
| `audit-cfg.ocn` | `@config` | 22 |
| `audit-db.ocn` | `@database` | 56 |
| `audit-deploy.ocn` | `@deploy` | 25 |
| `audit-dt.ocn` | `@datatype` | 107 |
| `audit-service.ocn` | `@service` | 26 |

## DSL source files

### `api/audit-api.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - audit-api
#   - rest-api
# perspective:
#   feature: audit-service
#   service: audit-service
# @ocean-meta-end

@api

AuditApi style: rest
    engine = gin
    configType = ApiConfig
    tags = internal
    version = 1.0.0
    description = Internal API for recording and querying immutable audit actions.
    basePath = /
    generateSwagger = true


  # ----------------------------------------------------------------------------
  # Record audit action
  # ----------------------------------------------------------------------------

  # Records one completed action attempt.
  #
  # The caller generates the ACT-ULID before submitting the action.
  # Audit records are immutable after they have been accepted.
  #
  # Submitting an existing id with identical content is treated as an idempotent
  # retry. Submitting the same id with different content must be rejected.
  post /audit/actions  recordAuditAction(action:AuditAction) : AuditAction


  # ----------------------------------------------------------------------------
  # Single audit action
  # ----------------------------------------------------------------------------

  # Retrieves one audit action by its ACT-ULID.
  get /audit/actions/{id}  getAuditActionById(id:String) : AuditAction


  # ----------------------------------------------------------------------------
  # Actor activity
  # ----------------------------------------------------------------------------

  # Lists audit actions initiated by an actor.
  #
  # Actor IDs are globally unique, so tenant scope is not required.
  # Results are ordered by occurredAt descending, with id as the tie-breaker.
  get /audit/actors/{actorId}/{offset}/{limit}  listAuditByActor(actorId:String, offset:Int, limit:Int) : List<AuditAction>


  # ----------------------------------------------------------------------------
  # Tenant activity
  # ----------------------------------------------------------------------------

  # Lists audit actions affecting a tenant.
  #
  # Results are ordered by occurredAt descending, with id as the tie-breaker.
  get /audit/tenants/{tenantId}/{offset}/{limit}  listAuditByTenant(tenantId:String, offset:Int, limit:Int) : List<AuditAction>
```

### `audit-cfg.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - configuration
# perspective:
#   feature: audit-service
#   service: audit-service
# @ocean-meta-end

@config

@import config O.database.postgres.config@1.0.0 as DatabaseConfig
@import config O.log.config@1.0.0 as LogConfig


AuditServiceConfig
    apiConfig : ApiConfig
    dbConfig  : DatabaseConfig
    logConfig : LogConfig
    
ApiConfig
    port: Int (default=8080)
```

### `audit-db.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - audit-trail
#   - database
# perspective:
#   feature: audit-service
#   service: audit-service
# @ocean-meta-end

@database

Database AuditDb
    engine = postgres
    configType = DatabaseConfig
    tags = audit

    Entity AuditAction
        # --------------------------------------------------------------------
        # Primary key
        # --------------------------------------------------------------------

        # Guarantees that every audit action is uniquely identified.
        # It also supports idempotent recording using the caller-generated ID.
        key(id)

        # Supports listing an actor's actions ordered by occurredAt and id.
        # Supports listing a tenant's actions ordered by occurredAt and id.
        indexes: index(actorId, occurredAt, id), index(tenantId, occurredAt, id)


        # --------------------------------------------------------------------
        # Queries
        # --------------------------------------------------------------------

        # Retrieves one audit action by its ACT-ULID.
        query findAuditActionById(id:String) : AuditAction

        # Lists actions initiated by a globally unique actor.
        # Results are ordered by occurredAt descending, with id as tie-breaker.
        query listAuditActionsByActor(actorId:String, offset:Int, limit:Int) : List<AuditAction>

        # Lists actions affecting a tenant.
        # Results are ordered by occurredAt descending, with id as tie-breaker.
        query listAuditActionsByTenant(tenantId:String, offset:Int, limit:Int) : List<AuditAction>


        # --------------------------------------------------------------------
        # Commands — append-only
        # --------------------------------------------------------------------

        # Inserts an immutable audit action.
        #
        # An existing id with identical content is an idempotent retry.
        # An existing id with different content must be rejected.
        command T recordAuditAction(action:AuditAction) : AuditAction
```

### `audit-deploy.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - deployment
# perspective:
#   feature: audit-service
#   service: audit-service
# @ocean-meta-end

@deploy

Name: Audit-deploy

@import service P.db.postgres.docker@1.0.0 as PostgresqlDB


AuditDeploy
    service     AuditService
    replica     2
    export      9092:AuditService.api
    dependsOn   AuditDatabase


AuditDatabase
    service     PostgresqlDB
```

### `audit-dt.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - audit-trail
#   - datatype
# perspective:
#   feature: audit-service
#   service: audit-service
# @ocean-meta-end

@datatype

AuditAction
    # Unique audit record identifier.
    # Generated by the Audit service.
    id           pattern ACT-ULID

    # UTC timestamp indicating when the action occurred.
    occurredAt : DateTime

    # -------------------------
    # Tenant
    # -------------------------

    # Tenant affected by the action.
    # Nil for platform-wide or pre-authentication actions.
    tenantId    : *String

    # -------------------------
    # Actor — who initiated it
    # -------------------------

    # Actor identifier.
    # May be nil for system or anonymous actors.
    actorId     : *String

    # Actor classification.
    # Conventional values: user, service, system, anonymous.
    actorType   : String

    # Snapshot of the actor's role when the action occurred.
    actorRole   : *String

    # -------------------------
    # Action — what was attempted
    # -------------------------

    # Stable action name using the <Resource>.<Operation> convention.
    # Examples: User.Login, Project.Create, Organization.MemberRemove.
    action       : String

    # Optional action classification.
    # Examples: authentication, authorization, security, administration.
    actionType   : *String

    # Channel through which the action originated.
    # Conventional values: ui, api, broker, scheduler.
    actionSource : String

    # -------------------------
    # Primary target
    # -------------------------

    # Type of the primary resource affected by the action.
    # Examples: User, Organization, Project.
    targetType  : *String

    # Identifier of the primary affected resource.
    # targetType and targetId should normally be provided together.
    targetId    : *String

    # -------------------------
    # Request context
    # -------------------------

    # Identifier used to correlate the action with its originating request.
    requestId   : *String

    # Client IP address, when supplied by a trusted gateway or service.
    ipAddress   : *String

    # Client user agent, when available.
    userAgent   : *String

    # -------------------------
    # Outcome
    # -------------------------

    # Outcome of the attempted action.
    # Conventional values: success, failure, denied.
    result       : String

    # Stable machine-readable error code.
    # Normally nil when result is success.
    errorCode    : *String

    # Short, scrubbed error description.
    # Must not contain secrets, stack traces, or sensitive payloads.
    errorMessage : *String

    # -------------------------
    # Additional context
    # -------------------------

    # Small key-value context that is not represented by a typed field.
    # Must be empty when unused and must not contain secrets or full objects.
    metadata    : Map<String,String>
```

### `audit-service.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - audit-api
#   - service
# perspective:
#   feature: audit-service
#   service: audit-service
# @ocean-meta-end

@service

AuditService

    @perspectives: version:0.1.0, lifestyle:stable

    use config AuditServiceConfig as myCfg
    impl api AuditApi as api on myCfg.apiConfig.port
    use database AuditDb as auditDb

    connect myCfg.dbConfig -> auditDb

    connect api.recordAuditAction -> AuditAction.recordAuditAction
    connect api.getAuditActionById -> AuditAction.findAuditActionById
    connect api.listAuditByActor -> AuditAction.listAuditActionsByActor
    connect api.listAuditByTenant -> AuditAction.listAuditActionsByTenant
```

## Walkthrough

A long, step-by-step explanation (Horizon and Voyage) is in [Audit Service walkthrough](audit-service-walkthrough.md). Read it only if you need the reasoning behind the design.

## Other files in the example (not inlined)

Fixtures, scripts and supporting artifacts. See the example on the Atlas site for their purpose.

- `api/00_info.hurl` (fixture)

- `api/01_health.hurl` (fixture)

- `api/02_options_preflight.hurl` (fixture)

- `api/03_swagger.hurl` (fixture)

- `api/10_record_action_happy_path.hurl` (fixture)

- `api/11_record_action_defaults.hurl` (fixture)

- `api/12_record_action_client_id_ignored.hurl` (fixture)

- `api/13_record_action_invalid_json.hurl` (fixture)

- `api/20_get_action_not_found.hurl` (fixture)

- `api/21_list_by_actor.hurl` (fixture)

- `api/22_list_by_tenant.hurl` (fixture)

- `api/23_list_invalid_pagination_params.hurl` (fixture)
