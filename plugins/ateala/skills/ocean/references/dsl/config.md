# Config DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/config/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax](#3-syntax)
- [4. Configuration Schema](#4-configuration-schema)
- [5. Schema Names](#5-schema-names)
- [6. Configuration Keys](#6-configuration-keys)
- [7. Flat Keys](#7-flat-keys)
- [8. Value Types](#8-value-types)
- [9. Required Keys](#9-required-keys)
- [10. Default Values](#10-default-values)
- [11. Literal Values](#11-literal-values)
- [12. Nested Configuration](#12-nested-configuration)
- [13. Nested Key Paths](#13-nested-key-paths)
- [14. Nested Defaults](#14-nested-defaults)
- [15. Recursive Schema References](#15-recursive-schema-references)
- [16. Complete Schema Example](#16-complete-schema-example)
- [17. Service Usage](#17-service-usage)
- [18. Service-only Consumption](#18-service-only-consumption)
- [19. Configuration Aliases](#19-configuration-aliases)
- [20. Multiple Configurations](#20-multiple-configurations)
- [21. Runtime Values](#21-runtime-values)
- [22. Secrets](#22-secrets)
- [23. Resolution](#23-resolution)
- [24. Defaults and Value Precedence](#24-defaults-and-value-precedence)
- [25. Validation](#25-validation)
- [26. Invalid Examples](#26-invalid-examples)
- [27. Purpose](#27-purpose)
- [28. Rules and Constraints](#28-rules-and-constraints)
- [29. Related Knowledge](#29-related-knowledge)

Defines the Ocean `@config` section for declaring named, reusable, typed configuration schemas used by deployable services.

Source: https://atlas.ocean-lab.ai/dsl/config/

Status: Approved · Authority: Normative

## 1. Overview

The `@config` section defines named, reusable configuration schemas.

A configuration schema describes the runtime settings required by a deployable Ocean service, such as:

- database connection settings;
- server options;
- feature toggles;
- integration endpoints;
- credentials and secret values;
- nested groups of related settings.

Each schema contains typed keys. A key may be required, provide a default value, or reference another configuration schema to create a nested structure.

`@config` defines the expected shape of configuration. It does not itself provide environment-specific runtime values.

---

## 2. Core Principle

Runtime configuration should be explicit, typed, reusable, and validated against a declared schema.

Conceptually:

```text
@config schema
      │
      ├── typed keys
      ├── optional defaults
      └── nested config schemas
               │
               ▼
          @service use
               │
               ▼
     runtime configuration values
               │
               ▼
       validated service config
```

This separates the definition of configuration requirements from the values supplied for a particular deployment.

---

## 3. Syntax

General structure:

```ocean-dsl
@config

<ConfigName>
    <key> : <Type>
    <key> : <Type> (default=<value>)
```

A file may define one or more configuration schemas.

Example:

```ocean-dsl
@config

ServerConfig
    host : String (default="localhost")
    port : Int (default=8080)
```

### 3.1 Imported Configuration Schemas

A `@config` file may import reusable configuration schemas from the Ocean
Repository and reference them by their local alias, exactly like a locally
declared schema:

```ocean-dsl
@config

@import config O.database.postgres.config@1.0.0 as DatabaseConfig
@import config O.log.config@1.0.0 as LogConfig

InvoiceConfig
    apiConfig : ApiConfig
    dbConfig  : DatabaseConfig
    logConfig : LogConfig

ApiConfig
    port : Int (default=8080)
```

`@import` directives appear immediately after `@config`, before any local schema
definitions. Each import requires an `as <Alias>` clause; the alias is then used
as a nested key type. Only configuration schemas may be introduced with
`@import` in a `@config` file. See `dsl.import` for the reference syntax.

---

## 4. Configuration Schema

A configuration schema consists of:

- a schema name;
- one or more named keys;
- a type for each key;
- an optional default value for each key.

Example:

```ocean-dsl
PostgresConfig
    url      : String (default="jdbc:postgresql://localhost:5432/mydb")
    user     : String (default="admin")
    password : String
```

`PostgresConfig` is the schema name. `url`, `user`, and `password` are configuration keys.

---

## 5. Schema Names

A configuration schema name follows the Ocean datatype naming convention.

Examples:

```text
PostgresConfig
ServerConfig
FeatureConfig
PaymentProviderConfig
```

Schema names must be unique within the resolved `@config` section scope.

A schema name is also a valid type reference for nested configuration.

---

## 6. Configuration Keys

Each configuration key has a name and a type.

Syntax:

```ocean-dsl
<key> : <Type>
```

Example:

```ocean-dsl
password : String
```

Key names follow the Ocean field naming convention.

Examples:

```text
host
port
timeoutSec
featureEnabled
```

Key names must be unique within their configuration schema.

---

## 7. Flat Keys

Configuration keys are simple field names.

Dot-separated key names are not allowed.

Invalid:

```ocean-dsl
auth.token.ttl : Int
```

Nested structures must be modeled through another named configuration schema:

```ocean-dsl
AuthConfig
    token : TokenConfig

TokenConfig
    ttl : Int
```

This keeps nesting explicit and type-aware.

---

## 8. Value Types

A configuration key may use a supported scalar configuration type or another configuration schema.

The core scalar types are:

```text
String
Int
Boolean
```

Examples:

```ocean-dsl
host    : String
port    : Int
enabled : Boolean
```

A named `@config` schema may be used as a nested type:

```ocean-dsl
pool : PoolConfig
```

The active Ocean type system determines whether additional scalar types are supported.

---

## 9. Required Keys

A key without a default value is required.

Example:

```ocean-dsl
password : String
```

The deployment or runtime configuration provider must supply a value for the key before the service can use a valid configuration instance.

Required-key validation must identify:

- the service configuration alias;
- the schema;
- the missing key path.

---

## 10. Default Values

A key may define a default value.

Syntax:

```ocean-dsl
<key> : <Type> (default=<value>)
```

Examples:

```ocean-dsl
host    : String  (default="localhost")
port    : Int     (default=8080)
enabled : Boolean (default=true)
```

Default values are optional.

When no runtime value is supplied for a key with a default, the default value is used.

A default value must be compatible with the declared key type.

---

## 11. Literal Values

Default values are literals.

Supported literal forms include:

- string literals;
- integer literals;
- Boolean literals;
- literal nested configuration values where supported by the grammar.

Strings containing spaces, punctuation, URLs, or other special characters should be quoted.

Example:

```ocean-dsl
url : String (default="jdbc:postgresql://localhost:5432/mydb")
```

Quoting plain string defaults consistently is recommended to avoid ambiguity.

---

## 12. Nested Configuration

A configuration key may reference another configuration schema.

Example:

```ocean-dsl
PostgresConfig
    url  : String
    pool : PoolConfig

PoolConfig
    size       : Int (default=10)
    timeoutSec : Int (default=30)
```

Here, `pool` is a nested configuration object whose structure is defined by `PoolConfig`.

Nested schemas allow configuration models to remain modular and reusable.

---

## 13. Nested Key Paths

Although dot-separated names are not valid key declarations, nested values may be described by a path when reporting or resolving configuration.

Given:

```ocean-dsl
PostgresConfig
    pool : PoolConfig

PoolConfig
    timeoutSec : Int
```

the logical runtime path may be represented as:

```text
pool.timeoutSec
```

This path is derived from schema nesting. It is not declared as a single dotted key in the DSL.

---

## 14. Nested Defaults

A nested configuration key may use a literal configuration value where supported.

Conceptually:

```ocean-dsl
pool : PoolConfig (default=<literal-config>)
```

The literal must conform to the referenced configuration schema.

If a nested configuration value is supplied only partially, default application and required-key validation occur recursively:

```text
supplied nested values
        +
nested schema defaults
        ↓
validate remaining required keys
```

The exact literal syntax for nested configuration values is defined by the Ocean grammar and should not be inferred from this conceptual form.

---

## 15. Recursive Schema References

Nested configuration schemas form a dependency graph.

For example:

```text
PostgresConfig
    └── PoolConfig
```

Schema references must resolve unambiguously.

A direct or indirect nesting cycle cannot produce a finite configuration value and is therefore invalid unless a future Ocean type rule explicitly introduces bounded or optional recursive configuration.

Examples of invalid cycles:

```text
ConfigA → ConfigA
```

```text
ConfigA → ConfigB → ConfigA
```

---

## 16. Complete Schema Example

The following is a complete `@config` file
(`ocean-examples/0003-answer-async/30-ans-config.ocn`):

```ocean-dsl
@config

@import config O.log.config@1.0.0 as LogConfig
@import config O.broker.nats.config@1.0.0 as BrokerConfig

CommonServiceConfig
    apiConfig    : ApiConfig
    logConfig    : LogConfig
    brokerConfig : BrokerConfig

DayTipServiceConfig
    apiConfig    : ApiConfig
    logConfig    : LogConfig
    brokerConfig : BrokerConfig

ApiConfig
    port : Int (default=8080)

AnswerBrokerConfig
    brokerConfig : BrokerConfig
    logConfig    : LogConfig

UiConfig
    port: Int (default=8080)
    logConfig : LogConfig
```

This example defines:

- two imported configuration schemas (`LogConfig`, `BrokerConfig`);
- several local schemas, each composed from scalar keys and nested schema references;
- an `ApiConfig` scalar schema with a defaulted `port` key;
- nested configuration through named schema references rather than dotted keys;
- reuse of the same imported and local schemas across multiple service configs.

---

## 17. Service Usage

Deployable services consume configuration schemas through `use config` declarations.

General syntax:

```ocean-dsl
use config <ConfigName> as <alias>
```

Example:

```ocean-dsl
@service

OrderService
    use config PostgresConfig as dbConfig
    use api OrderAPI
    use fsm OrderFSM as orderFsm
```

The alias identifies the configuration instance within the service scope.

The detailed placement and service-level usage rules are defined by `dsl.service`.

---

## 18. Service-only Consumption

Only `@service` definitions may declare use of a configuration schema.

This ties runtime configuration explicitly to a deployable unit.

Components, FSMs, and expressions should not independently bind runtime configuration schemas. Their behavior should remain portable and receive required values through their typed contracts or through service-supported integration mechanisms.

Conceptually:

```text
Runtime environment
        │
        ▼
Service configuration alias
        │
        ▼
Service orchestration
        │
        ▼
Typed values passed to domain behavior
```

---

## 19. Configuration Aliases

Every `use config` declaration requires an explicit alias.

Example:

```ocean-dsl
use config PostgresConfig as dbConfig
```

The schema identity is:

```text
PostgresConfig
```

The local service identity is:

```text
dbConfig
```

Aliases must be unique within the service scope.

An alias allows the same schema to be used for more than one distinct configuration instance where the service grammar permits it.

---

## 20. Multiple Configurations

A service may use multiple configuration schemas.

Example:

```ocean-dsl
@service

OrderService
    use config PostgresConfig as database
    use config ServerConfig as server
    use config FeatureConfig as features
```

Each declaration has its own alias and validation context.

The configurations are assembled for the service during generation or runtime according to the active target and configuration provider.

---

## 21. Runtime Values

`@config` declares a schema, not a deployment-specific value source.

Runtime values may originate from mechanisms such as:

- environment variables;
- configuration files;
- deployment platforms;
- secret stores;
- command-line or runtime parameters;
- generated target-specific configuration.

The concrete binding between a schema key and a value source is target- or deployment-specific and is outside the scope of this DSL reference.

Regardless of the source, values must be validated against the declared schema.

---

## 22. Secrets

A configuration schema may contain keys whose runtime values are sensitive.

Example:

```ocean-dsl
password : String
```

The schema defines the key and its type, but does not by itself make the value secret or prescribe a secret-management provider.

Sensitive values should not be embedded as defaults.

Secret classification, secure storage, injection, masking, and rotation are deployment and configuration-provider concerns unless Ocean introduces explicit secret metadata in a separate contract.

---

## 23. Resolution

Before a service can use a configuration schema, Ocean must resolve:

- the schema name;
- all nested schema references;
- the local service alias;
- the supported scalar types;
- every supplied runtime value where runtime validation is performed.

Conceptually:

```text
use config declaration
          ↓
resolve configuration schema
          ↓
resolve nested schemas
          ↓
build configuration shape
          ↓
merge supplied values and defaults
          ↓
validate required keys and types
```

An unresolved or ambiguous schema reference is invalid.

---

## 24. Defaults and Value Precedence

When a runtime configuration value is supplied, it takes precedence over the schema default for that key.

Conceptually:

```text
supplied runtime value
          ↓ if absent
schema default
          ↓ if absent
required-key error
```

Values from multiple external providers may require additional precedence rules. Those rules belong to the active configuration provider or target contract.

`@config` itself establishes only that an explicitly supplied value overrides its declared default.

---

## 25. Validation

Configuration validation includes:

- schema-name validity;
- schema-name uniqueness;
- key-name validity;
- key-name uniqueness within a schema;
- supported scalar types;
- nested schema resolution;
- recursive schema-cycle detection;
- default-value type compatibility;
- service-only `use config` placement;
- required service aliases;
- alias uniqueness within a service;
- runtime value type compatibility;
- required-key satisfaction after defaults are applied;
- recursive validation of nested configuration values.

Validation should identify the full logical key path for errors inside nested configuration.

---

## 26. Invalid Examples

### Dot-separated declaration

```ocean-dsl
AuthConfig
    auth.token.ttl : Int
```

Invalid because configuration keys must be flat field names.

### Default type mismatch

```ocean-dsl
ServerConfig
    port : Int (default="8080")
```

Invalid when the quoted value is interpreted as a `String` rather than an `Int`.

### Missing service alias

```ocean-dsl
use config PostgresConfig
```

Invalid because every service configuration use requires an explicit alias.

### Use outside a service

```ocean-dsl
@component

OrderFlow
    use config PostgresConfig as database
```

Invalid because configuration schemas may be consumed only by services.

### Unknown nested schema

```ocean-dsl
PostgresConfig
    pool : MissingPoolConfig
```

Invalid when `MissingPoolConfig` cannot be resolved.

---

## 27. Purpose

`@config` exists to:

- make runtime configuration requirements explicit;
- provide typed validation of supplied values;
- distinguish required keys from keys with defaults;
- model nested configuration without dotted key declarations;
- reuse configuration schemas across services;
- keep configuration attached to deployable service boundaries;
- allow generation and runtime tooling to understand configuration shape;
- separate domain behavior from environment-specific value sources.

---

## 28. Rules and Constraints

The following rules apply:

- A configuration file starts with `@config`.
- A file may define one or more named configuration schemas.
- A file may import reusable configuration schemas with `@import config <RepositoryRef> as <Alias>`, declared before any local schema.
- Schema names follow Ocean datatype naming conventions.
- Schema names are unique within the resolved config scope.
- Key names follow Ocean field naming conventions.
- Key names are unique within their schema.
- Dot-separated key declarations are not allowed.
- A key has a supported scalar type or references another configuration schema.
- Core scalar configuration types include `String`, `Int`, and `Boolean`.
- A key without a default is required.
- Default values are optional literals.
- A default value must match the declared key type.
- Nested configuration is modeled through named schema references.
- Nested schema references must resolve unambiguously.
- Direct and indirect nested-schema cycles are invalid.
- Only services may consume configuration schemas.
- Services consume schemas using `use config <ConfigName> as <alias>`.
- Every service configuration use requires an alias.
- Configuration aliases are unique within their service scope.
- A service may use multiple configurations.
- Runtime values override schema defaults.
- Missing required keys are invalid after defaults are applied.
- Runtime configuration values are validated recursively.
- Sensitive values should not be embedded as defaults.

---

## 29. Related Knowledge

The `@config` DSL is related to:

- `dsl.datatype` — provides the naming and type-system conventions used by configuration schemas and keys.
- `dsl.service` — defines deployable units that consume configuration schemas through explicit aliases.
- `dsl.import` — resolves reusable configuration schemas referenced with `@import config ... as <Alias>`.
- `dsl.include` — may compose compatible configuration sections where supported.

These semantic relationships are declared in the document metadata.
