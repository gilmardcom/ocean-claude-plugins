# Vault DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content a0c57f8d). Canonical source: https://atlas.ocean-lab.ai/dsl/vault/

**Contents**

- [1. Overview](#1-overview)
- [2. Core Principle](#2-core-principle)
- [3. Syntax Overview](#3-syntax-overview)
- [4. Vault File](#4-vault-file)
- [5. Vault Declaration](#5-vault-declaration)
- [6. Vault Names](#6-vault-names)
- [7. Vault Attributes](#7-vault-attributes)
- [8. Engine](#8-engine)
- [9. Engine Responsibilities](#9-engine-responsibilities)
- [10. Configuration Type](#10-configuration-type)
- [11. Primary Vaults](#11-primary-vaults)
- [12. Logical and Physical Vaults](#12-logical-and-physical-vaults)
- [13. Included Vaults](#13-included-vaults)
- [14. Inclusion Semantics](#14-inclusion-semantics)
- [15. Vault Inclusion Graph](#15-vault-inclusion-graph)
- [16. Secret Declaration](#16-secret-declaration)
- [17. Secret Names](#17-secret-names)
- [18. Secret Attributes](#18-secret-attributes)
- [19. Secret Types](#19-secret-types)
- [20. Primitive Secrets](#20-primitive-secrets)
- [21. Structured Secrets](#21-structured-secrets)
- [22. Optionality](#22-optionality)
- [23. Secret Descriptions](#23-secret-descriptions)
- [24. Perspectives and Tags](#24-perspectives-and-tags)
- [25. Secret References](#25-secret-references)
- [26. Secret Usage](#26-secret-usage)
- [27. Secret Values](#27-secret-values)
- [28. Physical Deployment](#28-physical-deployment)
- [29. Shared Physical Backend Example](#29-shared-physical-backend-example)
- [30. Complete Example](#30-complete-example)
- [31. Datatype Resolution](#31-datatype-resolution)
- [32. Vault Import Restrictions](#32-vault-import-restrictions)
- [33. Validation](#33-validation)
- [34. Invalid Examples](#34-invalid-examples)
- [35. Security Properties](#35-security-properties)
- [36. Purpose](#36-purpose)
- [37. Rules and Constraints](#37-rules-and-constraints)
- [38. Related Knowledge](#38-related-knowledge)

Defines the Ocean `@vault` section for modeling logical secret stores, typed secret schemas, vault composition, and physical secret-backend generation.

Source: https://atlas.ocean-lab.ai/dsl/vault/

Status: Approved · Authority: Normative

## 1. Overview

The `@vault` section defines logical vaults and the typed secrets exposed by them.

A vault declaration describes:

- the logical identity of a secret store;
- the secret-management engine associated with it;
- the configuration schema required by that engine;
- whether it represents a primary physical backend;
- other logical vaults included by a primary vault;
- the names and types of available secrets;
- optional documentation and classification metadata.

The DSL contains secret **schemas and references**, not secret values.

Secret values remain environment-specific and are managed outside Ocean by deployment systems, platform administrators, or external secret-management services.

---

## 2. Core Principle

Ocean models the existence, type, ownership, and location of secrets without embedding their sensitive values in the DSL.

Conceptually:

```text
@vault definition
      │
      ├── logical vault identity
      ├── engine and config type
      ├── typed secret declarations
      └── optional physical-backend composition
                       │
                       ▼
              generated integration
                       │
                       ▼
          externally managed secret values
```

This lets Ocean validate secret usage and generate integration artifacts while preserving external control of secret material.

---

## 3. Syntax Overview

General structure:

```ocean-dsl
@vault

Vault <VaultName>
    primary = <Boolean>
    includes = <VaultName1>, <VaultName2>, ...
    engine = <EngineName>
    configType = <ConfigRef>
    @perspectives: <key1>:<value1>, <key2>:<value2>
    @tags: <tag1>, <tag2>

    Secret <SecretName>
        type = <TypeRef>
        description = <Description>
        @perspectives: <key1>:<value1>, <key2>:<value2>
        @tags: <tag1>, <tag2>
```

A file may define one or more `Vault` blocks.

---

## 4. Vault File

A vault file begins with:

```ocean-dsl
@vault
```

It then declares one or more named vaults:

```ocean-dsl
@vault

Vault OrderVault
    engine = hashicorp
    configType = VaultConfig

Vault InventoryVault
    engine = hashicorp
    configType = VaultConfig
```

Vault names must be unique within the resolved `@vault` section scope.

---

## 5. Vault Declaration

A vault declaration has the form:

```ocean-dsl
Vault <VaultName>
    <vault attributes>

    Secret <SecretName>
        <secret attributes>
```

Example:

```ocean-dsl
Vault OrderVault
    engine = hashicorp
    configType = VaultConfig

    Secret JwtSecret
        type = String
        description = Secret used for signing JWT tokens
```

A vault may contain one or more secret declarations.

---

## 6. Vault Names

A vault name identifies a logical secret store.

Examples:

```text
OrderVault
InventoryVault
MainVault
PaymentVault
```

Vault names follow the Ocean type-name convention and must be unique in their section scope.

The vault name forms the first part of every secret reference:

```text
<VaultName>.<SecretName>
```

---

## 7. Vault Attributes

A vault supports the following attributes:

| Attribute       | Required                       | Meaning                                                                        |
| --------------- | ------------------------------ | ------------------------------------------------------------------------------ |
| `primary`       | No                             | Whether the vault represents the primary physical backend; defaults to `false` |
| `includes`      | No                             | Logical vaults whose secrets are exposed through this primary vault            |
| `engine`        | For a primary/standalone vault | Secret-backend implementation used for the vault                               |
| `configType`    | No                             | Configuration schema for the selected engine                                   |
| `@perspectives` | No                             | Perspective metadata associated with the vault                                 |
| `@tags`         | No                             | Classification tags associated with the vault                                  |

A non-primary vault that is `includes`-d by a primary vault carries only
`primary = false` and its `Secret` declarations; it takes its engine and backend
configuration from the primary vault. `engine` is required on the primary vault
(or on a standalone vault that is not included by any primary).

`configType` is optional. When omitted, the engine configuration is supplied
another way — for example through the consuming service's `use config` binding
(see Section 10). The example project omits it.

Attributes must satisfy the validation rules of the vault and the selected engine.

---

## 8. Engine

The `engine` attribute selects the vault backend implementation.

Syntax:

```ocean-dsl
engine = <EngineName>
```

Example:

```ocean-dsl
engine = hashicorp
```

Engine implementations may include:

```text
local
service
hashicorp
aws
gcp
k8s
```

This list is illustrative rather than exhaustive. The available engines depend on the active Ocean toolchain and generation target.

The engine name must resolve to a supported vault engine.

---

## 9. Engine Responsibilities

A vault engine defines how Ocean integrates the logical vault with a secret-management backend.

Engine-specific behavior may include:

- validating the engine configuration;
- generating deployment resources;
- generating client or provider integration;
- mapping logical secret names to backend paths or identifiers;
- configuring authentication to the backend;
- determining whether a physical backend must be created;
- producing documentation or operational metadata.

The `@vault` model remains independent of one specific vendor. Engine-specific details belong to the engine implementation and its configuration schema.

---

## 10. Configuration Type

The optional `configType` attribute references a configuration schema for the
selected engine. It may be omitted when the engine configuration is supplied
another way — for example through the consuming service's `use config` binding,
where the service config includes a vault config schema.

Syntax:

```ocean-dsl
configType = <ConfigRef>
```

Example:

```ocean-dsl
configType = VaultConfig
```

The reference must resolve to a valid configuration schema.

The schema may describe settings such as:

- backend address;
- mount or project identifier;
- authentication configuration;
- namespace;
- region;
- engine-specific options.

The actual configuration values are supplied through the applicable service, deployment, or generation configuration mechanism.

---

## 11. Primary Vaults

The optional `primary` attribute indicates whether a logical vault should represent or generate a physical backend.

Syntax:

```ocean-dsl
primary = true
```

or:

```ocean-dsl
primary = false
```

The default is:

```text
false
```

A primary vault may include other logical vaults and expose their secrets through one physical backend integration.

---

## 12. Logical and Physical Vaults

Ocean distinguishes a logical vault definition from a physical secret-management backend.

```text
Logical vault
    └── organizes and types secrets in the Ocean model

Physical backend
    └── stores environment-specific secret values
```

Multiple logical vaults may share one physical backend.

A primary vault provides the composition boundary used to map those logical vaults to a generated or configured physical backend.

This preserves logical ownership without requiring a separate deployed backend for every logical vault.

---

## 13. Included Vaults

The `includes` attribute lists logical vaults whose secrets are exposed through a primary vault.

Syntax:

```ocean-dsl
includes = <VaultName1>, <VaultName2>, ...
```

Example:

```ocean-dsl
Vault MainVault
    primary = true
    includes = OrderVault, InventoryVault
    engine = hashicorp
    configType = VaultConfig
```

`includes` is valid only when `primary = true`.

Every included vault name must resolve to a declared logical vault.

---

## 14. Inclusion Semantics

Including a vault exposes its declared secrets through the primary physical-backend composition.

Conceptually:

```text
OrderVault ───────┐
                  │
InventoryVault ──┼── MainVault (primary) ── physical backend
                  │
MainVault secrets ┘
```

Inclusion does not copy secret values into the DSL.

Logical vault names remain part of the model, allowing Ocean tooling to preserve ownership and qualified references even when one physical backend serves multiple logical vaults.

---

## 15. Vault Inclusion Graph

Vault inclusion creates a dependency graph.

Every included vault must exist, and inclusion must be deterministic.

A primary vault must not include itself.

Direct or indirect inclusion cycles are invalid:

```text
VaultA includes VaultB
VaultB includes VaultA
```

```text
VaultA includes VaultB
VaultB includes VaultC
VaultC includes VaultA
```

Ocean must detect cycles before generation.

---

## 16. Secret Declaration

A secret declaration defines a named, typed sensitive value within a vault.

Syntax:

```ocean-dsl
Secret <SecretName>
    type = <TypeRef>
    description = <Description>
    @perspectives: <key1>:<value1>, <key2>:<value2>
    @tags: <tag1>, <tag2>
```

Example:

```ocean-dsl
Secret JwtSecret
    type = String
    description = Secret used for signing JWT tokens
```

The declaration defines the secret's schema. It does not contain its value.

---

## 17. Secret Names

A secret name identifies one sensitive value inside a vault.

Examples:

```text
JwtSecret
ApiKey
DbCredentials
SigningCertificate
```

Secret names follow the Ocean type-name convention.

They must be unique within their containing vault.

The same secret name may appear in another vault because the qualified identity includes the vault name.

---

## 18. Secret Attributes

A secret supports the following attributes:

| Attribute       | Required | Meaning                                                     |
| --------------- | -------- | ----------------------------------------------------------- |
| `type`          | Yes      | Primitive or datatype reference describing the secret value |
| `description`   | No       | Human-readable explanation of the secret's purpose          |
| `@perspectives` | No       | Perspective metadata associated with the secret             |
| `@tags`         | No       | Classification tags associated with the secret              |

Descriptions and metadata do not contain the secret value and do not change runtime behavior.

---

## 19. Secret Types

Every secret has a type.

Syntax:

```ocean-dsl
type = <TypeRef>
```

A secret type may be:

- a supported primitive type;
- an existing Ocean datatype.

Primitive example:

```ocean-dsl
Secret JwtSecret
    type = String
```

Structured example:

```ocean-dsl
Secret DbCredentials
    type = DatabaseCredentials
```

The referenced type must resolve unambiguously.

---

## 20. Primitive Secrets

A primitive secret contains one value of a supported primitive type.

Example:

```ocean-dsl
Secret ApiKey
    type = String
    description = API key for the inventory provider
```

Primitive secret types are useful for values such as:

- tokens;
- API keys;
- passwords;
- signing strings;
- identifiers.

Declaring a primitive type validates the shape of the resolved value but does not reduce the requirement to handle it as sensitive data.

---

## 21. Structured Secrets

A structured secret uses an Ocean datatype to define multiple related secret fields.

Vault declaration:

```ocean-dsl
Secret DbCredentials
    type = DatabaseCredentials
    description = Credentials used for database access
```

Referenced datatype, conceptually:

```ocean-dsl
@datatype

DatabaseCredentials
    username : String
    password : String
```

The external secret value must conform to the referenced datatype.

Structured secrets allow related sensitive fields to be validated and consumed as one typed value.

---

## 22. Optionality

Secret optionality is expressed through the secret's type.

The vault declaration does not define a separate `optional` attribute.

Conceptually:

```text
type = <required type>
```

or, where supported by the Ocean type system:

```text
type = <optional type>
```

The exact optional-type syntax is defined by `dsl.datatype` and the Ocean type system.

---

## 23. Secret Descriptions

The optional `description` attribute explains what a secret is used for.

Example:

```ocean-dsl
description = Secret used for signing JWT tokens
```

Descriptions are intended for:

- generated documentation;
- user interfaces;
- operator guidance;
- discovery and organization.

A description must not contain the secret value or other sensitive material.

Descriptions do not affect runtime behavior.

---

## 24. Perspectives and Tags

Vaults and secrets may declare optional perspective and tag metadata.

Example:

```ocean-dsl
@perspectives: security:critical, domain:orders
@tags: internal, secure
```

Secret-level example:

```ocean-dsl
@tags: database, credentials
```

This metadata may support:

- documentation;
- classification;
- discovery;
- filtering;
- generated user interfaces;
- governance processes.

Metadata does not replace access control and does not contain secret values.

---

## 25. Secret References

A secret is referenced through its qualified vault and secret names.

Canonical format:

```text
<VaultName>.<SecretName>
```

Examples:

```text
MainVault.DbCredentials
OrderVault.JwtSecret
InventoryVault.ApiKey
```

The vault name provides the logical secret scope. The secret name identifies the value within that scope.

A reference must resolve to exactly one declared secret.

---

## 26. Secret Usage

Qualified secret references may be used by DSL constructs that explicitly support secrets.

Potential consumers include:

- services;
- expressions with supported external implementations;
- components through supported integration points;
- generated infrastructure or configuration.

The receiving DSL reference defines the exact syntax, placement, and permitted use of a secret reference.

Declaring a vault does not make every secret automatically available to every model element. Availability may also depend on scope, generation target, and access policy.

---

## 27. Secret Values

Secret values must not be stored in an Ocean DSL file.

The following responsibilities remain external to the schema declaration:

- provisioning values;
- encrypting values at rest and in transit;
- access control;
- authentication to the secret backend;
- value rotation;
- auditing access;
- environment-specific overrides;
- revocation and incident response.

Values may be managed by platform administrators, deployment pipelines, or external secret-management systems.

---

## 28. Physical Deployment

Depending on the selected engine and generation target, Ocean may generate:

- a physical vault service;
- cloud secret-manager resources;
- Kubernetes secret integration;
- backend configuration;
- application client integration;
- deployment artifacts required to connect to an existing backend.

The exact generated output is engine-specific.

A vault declaration does not guarantee that Ocean creates a new backend. An engine may instead integrate with an existing externally managed service.

---

## 29. Shared Physical Backend Example

```ocean-dsl
@vault

Vault OrderVault
    engine = hashicorp
    configType = VaultConfig

    Secret JwtSecret
        type = String

Vault InventoryVault
    engine = hashicorp
    configType = VaultConfig

    Secret ApiKey
        type = String

Vault MainVault
    primary = true
    includes = OrderVault, InventoryVault
    engine = hashicorp
    configType = VaultConfig

    Secret DbCredentials
        type = DatabaseCredentials
```

During generation, `MainVault` may represent one physical backend while Ocean preserves the logical separation of `OrderVault`, `InventoryVault`, and their secrets.

---

## 30. Complete Example

The `@vault` files of `ocean-examples/0009-simple-vault-usage`
(`vault1.ocn` and `vault2.ocn`):

```ocean-dsl
@vault

Vault AuthVault
    primary = false

    Secret JwtCfg
        type = JwtConfig
        description = JWT signing configuration
```

```ocean-dsl
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

This example demonstrates:

- logical vaults split across two `@vault` files;
- non-primary vaults (`AuthVault`, `PaymentVault`) that declare only `primary = false`
  and their secrets — no `engine` or `configType`;
- a primary `MainVault` that declares `engine`, `includes` the two logical vaults,
  and has no `configType` (the engine config comes from the consuming service);
- structured secrets typed by `@datatype` datatypes (`JwtConfig`, `PaymentApiConfig`,
  `DatabaseCredentials`);
- unquoted `description` values and a `@tags` line;
- a trailing `#` comment on an attribute line.

The `SecretService` in the same project consumes these with
`use vault MainVault` / `use vault AuthVault` / `use vault PaymentVault`.

---

## 31. Datatype Resolution

The `@vault` section does not import secret declarations directly from the Ocean Repository or local Pre-baked registry.

All referenced datatypes must be available through the datatype model.

They may be:

- defined locally in the project;
- imported through the `@datatype` section's supported import mechanism;
- included through the `@datatype` section's supported inclusion mechanism.

The vault references the resolved datatype by its available DSL name.

---

## 32. Vault Import Restrictions

Secrets cannot be imported directly into `@vault` from reusable Ocean items.

Conceptually invalid:

```text
@vault
@import secret O.example.SharedSecret@1.0.0 as SharedSecret
```

The restriction ensures that secret ownership and logical vault placement remain explicit in the local vault model.

Reusable datatype schemas may still describe the shape of structured secrets without providing or importing secret values.

---

## 33. Validation

Vault validation includes:

- presence of the `@vault` section declaration;
- vault-name validity and uniqueness;
- `engine` attribute on primary and standalone vaults;
- resolution of the selected engine;
- resolution and compatibility of `configType` when it is present;
- Boolean validity of `primary`;
- use of `includes` only by a primary vault;
- resolution and uniqueness of included vaults;
- prevention of self-inclusion;
- vault-inclusion cycle detection;
- secret-name validity and uniqueness within a vault;
- required secret `type` attribute;
- secret-type resolution;
- metadata syntax;
- qualified secret-reference resolution;
- engine-specific semantic validation.

Validation must not require secret values to be present in the DSL.

---

## 34. Invalid Examples

### Missing engine

```ocean-dsl
Vault OrderVault
    configType = VaultConfig
```

Invalid because `engine` is required.

### Includes on a non-primary vault

```ocean-dsl
Vault MainVault
    includes = OrderVault
    engine = hashicorp
    configType = VaultConfig
```

Invalid because `includes` may be declared only when `primary = true`.

### Secret without a type

```ocean-dsl
Secret JwtSecret
    description = JWT signing secret
```

Invalid because every secret requires a type.

### Embedded secret value

```ocean-dsl
Secret JwtSecret
    type = String
    value = "actual-secret-value"
```

Invalid because the vault schema does not store secret values.

### Inclusion cycle

```text
MainVault includes SharedVault
SharedVault includes MainVault
```

Invalid because vault inclusion must be acyclic.

---

## 35. Security Properties

The vault model supports secure system generation by keeping several boundaries explicit:

- schemas are separated from values;
- logical ownership is expressed through vault names;
- references are qualified;
- backend selection is explicit;
- backend configuration is typed;
- secret shapes are validated;
- deployment integration is engine-controlled.

These model properties do not by themselves guarantee backend security.

Actual confidentiality and access control depend on the selected engine, deployment configuration, credentials, policies, infrastructure, and operating practices.

---

## 36. Purpose

`@vault` exists to:

- model secret requirements without embedding secret values;
- give secrets explicit names and types;
- organize secrets into logical ownership boundaries;
- allow structured secret values through Ocean datatypes;
- select a secret-management engine explicitly;
- validate the engine's configuration schema;
- support generation of secret-backend integration;
- map multiple logical vaults to a shared physical backend;
- support documentation, classification, and governance metadata;
- make secret references explicit throughout the Ocean model.

---

## 37. Rules and Constraints

The following rules apply:

- A vault file starts with `@vault`.
- A vault file may define multiple `Vault` blocks.
- Vault names are unique within the resolved vault scope.
- A primary or standalone vault requires an `engine`; a non-primary vault included by a primary may omit it and inherit the primary's engine.
- `configType` is optional; engine configuration may instead be supplied through the consuming service's configuration.
- The engine must be known and supported by the active toolchain.
- When present, the configuration reference must resolve to a compatible config schema.
- `primary` is optional and defaults to `false`.
- `includes` is optional and is allowed only when `primary = true`.
- Every included vault must resolve unambiguously.
- A vault cannot include itself.
- Direct and indirect vault-inclusion cycles are invalid.
- A vault may declare multiple secrets.
- Secret names are unique within their vault.
- Every secret requires a type.
- Secret types may be primitive or reference an existing Ocean datatype.
- Secret optionality is expressed through the type system.
- Secret descriptions are optional and do not affect runtime behavior.
- Vault and secret perspectives and tags are optional metadata.
- Secret references use `<VaultName>.<SecretName>`.
- Secret references must resolve unambiguously.
- Secret values must not appear in the DSL.
- Secret values are supplied and managed externally.
- Multiple logical vaults may share one physical backend.
- Physical generation and integration behavior are engine-specific.
- The `@vault` section does not import secrets directly from reusable Ocean items.
- Referenced datatypes must be resolved through the datatype model.

---

## 38. Related Knowledge

The `@vault` DSL is related to:

- `dsl.datatype` — defines primitive and structured types used by secret declarations.
- `dsl.config` — defines the typed configuration schemas referenced by vault engines.
- `dsl.service` — services may consume qualified secret references and host generated vault integration.
- `dsl.expression` — supported expression implementations may consume secrets through explicitly defined integration points.
- `dsl.perspective` — defines optional perspective metadata associated with vaults and secrets.
- `dsl.tag` — defines optional classification tags associated with vaults and secrets.

These semantic relationships are declared in the document metadata.
