# Simple Vault Usage

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/examples/simple-vault-usage/

**Contents**

- [Overview](#overview)
- [Files](#files)
- [DSL source files](#dsl-source-files)
- [Walkthrough](#walkthrough)

Define typed secrets in several logical vaults and expose them through one service.

Source: https://atlas.ocean-lab.ai/examples/simple-vault-usage/

Level: medium · Estimated time: 10 minutes · Version: 1.0.0 · Onboarding example: no

Components: 1 services, 0 brokers, 0 databases

Tags: api, configuration, rest, secrets, security, vault

## Overview

This example shows how Ocean defines typed database, authentication, and payment secrets in focused vaults and composes them into a primary vault for service access. Its executable DSL and its Horizon and Voyage explanation are maintained together in this example directory.

## Files

DSL source comes first, then diagrams; the long Horizon and Voyage walkthroughs are at the end. You can stop after the source files.

| File | Section | Lines |
| --- | --- | --- |
| `api.ocn` | `@api` | 24 |
| `config.ocn` | `@config` | 21 |
| `datatype.ocn` | `@datatype` | 23 |
| `deploy.ocn` | `@deploy` | 23 |
| `expression.ocn` | `@expression` | 29 |
| `service.ocn` | `@service` | 41 |
| `vault1.ocn` | `@vault` | 16 |
| `vault2.ocn` | `@vault` | 27 |

## DSL source files

### `api.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - secret-api
#   - rest-api
# perspective:
#   feature: simple-vault-usage
#   service: secret-service
# @ocean-meta-end

@api

SecretApi style: rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    description = API exposing secret values from configured vaults
    basePath = /
    generateSwagger = true
    @tags: internal

    get /db-credentials   getDbCredentials() : DatabaseCredentials
    get /jwt-config       getJwtConfig()     : JwtConfig
    get /payment-config   getPaymentConfig() : PaymentApiConfig
```

### `config.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - configuration
# perspective:
#   feature: simple-vault-usage
#   service: secret-service
# @ocean-meta-end

@config

@import config O.log.config@1.0.0 as LogConfig
@import config O.vault.config.local@1.0.0 as LocalVaultConfig

AppConfig
    apiConfig : ApiConfig
    logConfig : LogConfig
    vaultConfig: LocalVaultConfig
    
ApiConfig
    port: Int (default=8080)
```

### `datatype.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - typed-secrets
#   - datatype
# perspective:
#   feature: simple-vault-usage
#   service: secret-service
# @ocean-meta-end

@datatype

DatabaseCredentials
    username: String
    password: String

JwtConfig
    secret: String
    issuer: String

PaymentApiConfig
    apiKey: String
    endpoint: String
```

### `deploy.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - deployment
# perspective:
#   feature: simple-vault-usage
#   service: secret-service
# @ocean-meta-end

@deploy

@import service P.vault.hashicorp.docker@1.0.0 as Hashicorp

Name: SecretApp-deploy

SecretDeploy
    service     SecretService
    replica     1
    export      9098:SecretService.api
    dependsOn   Vault

Vault
    service     Hashicorp
```

### `expression.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - secrets-management
#   - expression
# perspective:
#   feature: simple-vault-usage
#   service: secret-service
# @ocean-meta-end

@expression

GetDbCredentials
    input: _
    output: out:DatabaseCredentials
    logic
        out = MainVault.DbCredentials

GetJwtConfig
    input: _
    output: out:JwtConfig
    logic
        out = AuthVault.JwtCfg

GetPaymentConfig
    input: _
    output: out:PaymentApiConfig
    logic
        out = PaymentVault.PaymentCfg
```

### `service.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - secret-api
#   - service
# perspective:
#   feature: simple-vault-usage
#   service: secret-service
# @ocean-meta-end

@service

SecretService

    @perspectives: version:1.0.0, id:SVC-SECRET, features:read-secret;secret-api, summary:Provides secure secret management and retrieval
    @tags: internal

    use config AppConfig as cfg

    impl api SecretApi as api on cfg.apiConfig.port

    # -------------------------
    #   Vaults
    # -------------------------
    use vault MainVault
    use vault AuthVault
    use vault PaymentVault

    # -------------------------
    #   Expressions
    # -------------------------
    use expression GetDbCredentials
    use expression GetJwtConfig
    use expression GetPaymentConfig

    # -------------------------
    #   Connections
    # -------------------------
    connect api.getDbCredentials  -> GetDbCredentials
    connect api.getJwtConfig      -> GetJwtConfig
    connect api.getPaymentConfig  -> GetPaymentConfig
```

### `vault1.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - vault
# perspective:
#   feature: simple-vault-usage
# @ocean-meta-end

@vault

Vault AuthVault
    primary = false

    Secret JwtCfg
        type = JwtConfig
        description = JWT signing configuration
```

### `vault2.ocn`

```ocean-dsl
# @ocean-meta-start
# tags:
#   - vault
#   - main-vault
# perspective:
#   feature: simple-vault-usage
# @ocean-meta-end

@vault

Vault PaymentVault
    primary = false

    Secret PaymentCfg
        type = PaymentApiConfig
        description = Payment provider configuration

Vault MainVault
    primary = true
    includes = AuthVault, PaymentVault
    engine = hashicorp # local/hashicorp
    @tags: internal

    Secret DbCredentials
        type = DatabaseCredentials
        description = Database credentials
```

## Walkthrough

A long, step-by-step explanation (Horizon and Voyage) is in [Simple Vault Usage walkthrough](simple-vault-usage-walkthrough.md). Read it only if you need the reasoning behind the design.
