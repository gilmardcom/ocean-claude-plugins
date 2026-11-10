# Database DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/database/

**Contents**

- [1. Overview](#1-overview)
- [2. Syntax](#2-syntax)
- [3. Database Definition](#3-database-definition)
- [4. Database Types](#4-database-types)
- [5. Entity Definition](#5-entity-definition)
- [6. Entity Relationships](#6-entity-relationships)
- [7. Keys](#7-keys)
- [8. Indexes and Constraints](#8-indexes-and-constraints)
- [9. Field Encryption](#9-field-encryption)
- [10. Queries and Commands](#10-queries-and-commands)
- [11. Complete Example](#11-complete-example)
- [12. Imports and Datatype Resolution](#12-imports-and-datatype-resolution)
- [13. Rules and Constraints](#13-rules-and-constraints)
- [14. Related Knowledge](#14-related-knowledge)

Defines the Ocean `@database` DSL section, including databases, entities, relationships, keys, indexes, encryption, queries, and commands.

Source: https://atlas.ocean-lab.ai/dsl/database/

Status: Accepted · Authority: Normative

## 1. Overview

The `@database` section defines one or more databases.

Each database contains entities that map to existing datatypes and may define:

- database type and engine;
- configuration;
- entity relationships;
- keys;
- indexes and constraints;
- encrypted fields;
- queries;
- commands.

Entity fields are defined in `@datatype`. The `@database` section adds persistence-specific information to those datatypes.

A database file starts with:

```ocean-dsl
@database
```

A file may define multiple `Database` blocks.

---

## 2. Syntax

```ocean-dsl
@database

Database <DatabaseName>
    type = <databaseType>
    engine = <engineName>
    configType = <ConfigRef>
    tags = <tag1>, <tag2>, ...
    encryptionKey = <KeySpec>

    Entity <EntityName>
        <fieldName> <linkType> <TargetEntity> [<modifiers>]
        key(<field1>, <field2>, ...)
        indexes: <indexDefinition>, <indexDefinition>, ...
        encrypt: <field1>, <field2>, ...
        query <queryName>(<parameters>) : <ReturnType>
        command [T|Transactional] <commandName>(<parameters>) : <ReturnType>
```

Only applicable attributes and entity definitions need to be declared.

---

## 3. Database Definition

A `Database` block defines a logical database and its persistence configuration.

Example:

```ocean-dsl
@database

Database TodoDB
    engine = postgres
    configType = DatabaseConfig
    encryptionKey = generate
    tags = primary

    Entity TodoItem
        key(id)
        indexes: unique(title), index(title), index(dueDate)
        encrypt: secret1, secret2

        # Queries
        query listTodoItems(offset:Int, limit:Int) : List<TodoItem>
        query findById(id: String) : TodoItem
        query findByTitle(title: String) : List<TodoItem>
        query findByTitleAndPriority(title: String, priority: Priority) : List<TodoItem>
        query findByTitleOrPriority(title: String, priority : Priority) : List<TodoItem>

        # Commands
        command Transactional createTodoItem(item:TodoItem) : TodoItem
        command T updateTodoItem(item:TodoItem) : TodoItem
        command deleteTodoItem(id:String) : _
```

### 3.1 Database Attributes

| Attribute       | Required    | Description                                                                 |
| --------------- | ----------- | --------------------------------------------------------------------------- |
| `type`          | No          | Logical database type. Defaults to `transactional`.                         |
| `engine`        | Yes         | Database engine, such as `postgres`, `mysql`, `mongodb`, or `neo4j`.        |
| `configType`    | Yes         | Reference to the configuration type used by the database.                   |
| `tags`          | No          | Metadata tags used for classification or processing.                        |
| `encryptionKey` | Conditional | Defines the encryption key source. Required when an entity uses `encrypt:`. |

---

## 4. Database Types

The optional `type` attribute identifies the logical type of database.

Supported values are:

- `transactional`
- `in-memory`
- `document`
- `graph`
- `timeseries`
- `analytical`

The default is:

```ocean-dsl
type = transactional
```

Example:

```ocean-dsl
Database MainDB
    type = transactional
    engine = postgres
    configType = DatabaseConfig
```

The `type` describes the logical storage model, while `engine` identifies the concrete database technology.

For `type = in-memory` databases, entities receive a standard generated set of CRUD operations regardless of the specific engine — see [10.4 Implicit CRUD for In-Memory Entities](#104-implicit-crud-for-in-memory-entities).

---

## 5. Entity Definition

An `Entity` maps an existing datatype into a database.

Syntax:

```ocean-dsl
Entity <EntityName>
```

The entity name must match an existing datatype name.

For example, given:

```ocean-dsl
@datatype

TodoItem
    id       : String
    title    : String
    dueDate  : Date
    secret1  : String
    secret2  : String
```

the corresponding database entity may be:

```ocean-dsl
Entity TodoItem
    key(id)
    indexes: unique(title), index(title), index(dueDate)
    encrypt: secret1, secret2
```

Fields must not be declared again inside the entity.

The datatype defines the data structure. The entity defines its persistence behavior.

---

## 6. Entity Relationships

Entities may define relationships to other entities.

### 6.1 Syntax

Each relationship is defined on one line:

```ocean-dsl
<fieldName> <linkType> <TargetEntity> [<modifiers>]
```

Example:

```ocean-dsl
buyer -m2o-> Party [index, not-null]
```

### 6.2 Link Types

| Type         | Syntax   | Description               |
| ------------ | -------- | ------------------------- |
| One-to-One   | `-o2o->` | One-to-one relationship   |
| One-to-Many  | `-o2m->` | One-to-many relationship  |
| Many-to-One  | `-m2o->` | Many-to-one relationship  |
| Many-to-Many | `-m2m->` | Many-to-many relationship |

Example:

```ocean-dsl
Entity PurchaseOrder
    buyer -m2o-> Party [index, not-null]
    item -m2o-> Item
```

### 6.3 Relationship Modifiers

| Modifier   | Description                                                 |
| ---------- | ----------------------------------------------------------- |
| `not-null` | The relationship must not be null.                          |
| `unique`   | The relationship must have a unique value where applicable. |
| `index`    | The relationship should be indexed.                         |

Multiple modifiers are declared inside brackets:

```ocean-dsl
buyer -m2o-> Party [index, not-null]
```

Entity references use ID-based mapping by default. No explicit `store` strategy is required.

---

## 7. Keys

The `key(...)` declaration defines the key of an entity.

Syntax:

```ocean-dsl
key(<field1>, <field2>, ...)
```

A single-field key:

```ocean-dsl
key(id)
```

A composite key:

```ocean-dsl
key(code, version)
```

The referenced fields must exist in the datatype backing the entity.

Example:

```ocean-dsl
Entity PurchaseOrder
    key(code, version)
```

---

## 8. Indexes and Constraints

Indexes and uniqueness constraints are declared using `indexes:`.

All index definitions for an entity are declared on one line.

### 8.1 Syntax

```ocean-dsl
indexes: <indexDefinition>, <indexDefinition>, ...
```

Example:

```ocean-dsl
indexes: unique(title), index(title), index(dueDate)
```

Supported definitions include:

| Definition                    | Description                                                           |
| ----------------------------- | --------------------------------------------------------------------- |
| `unique(field)`               | Defines a uniqueness constraint or unique index on a single field.    |
| `index(field)`                | Defines an index for a single field.                                  |
| `unique(field1, field2, ...)` | Defines a composite uniqueness constraint over the listed fields.     |
| `index(field1, field2, ...)`  | Defines a composite index over the listed fields, in the given order. |

Single-field example:

```ocean-dsl
Entity TodoItem
    indexes: unique(title), index(title), index(dueDate)
```

Composite example:

```ocean-dsl
Entity AuditAction
    indexes: index(actorId, occurredAt, id), index(tenantId, occurredAt, id)
```

```ocean-dsl
Entity AccessRequest
    indexes: unique(subjectId, requestId), index(subjectId), index(timestamp)
```

Field order in a composite definition is significant and is preserved in the generated index.

All referenced fields must exist in the datatype backing the entity.

---

## 9. Field Encryption

Entity fields may be stored in encrypted form.

Encryption consists of:

1. `encryptionKey` at database level;
2. `encrypt:` at entity level.

### 9.1 Encrypted Fields

All encrypted fields for an entity are declared on one line.

Syntax:

```ocean-dsl
encrypt: <field1>, <field2>, ...
```

Example:

```ocean-dsl
encrypt: secret1, secret2
```

The fields must exist in the datatype backing the entity.

---

### 9.2 Encryption Key

If any entity uses `encrypt:`, the database must define `encryptionKey`.

Example:

```ocean-dsl
Database TodoDB
    engine = postgres
    configType = DatabaseConfig
    encryptionKey = generate

    Entity TodoItem
        encrypt: secret1, secret2
```

The supported key specifications are:

```text
generate
env:<ENV_VAR_NAME>
file:<PATH>
vault:<SECRET_PATH>
vault:<SECRET_PATH>#<FIELD>
literal:<BASE64_32B_LITERAL>
```

| Key Specification              | Description                                 |
| ------------------------------ | ------------------------------------------- |
| `generate`                     | Generates an encryption key.                |
| `env:<ENV_VAR_NAME>`           | Reads the key from an environment variable. |
| `file:<PATH>`                  | Reads the key from a file.                  |
| `vault:<SECRET_PATH>`          | Reads the key from a vault secret.          |
| `vault:<SECRET_PATH>#<FIELD>`  | Reads a specific field from a vault secret. |
| `literal:<BASE64_32B_LITERAL>` | Uses a Base64-encoded 32-byte literal key.  |

Examples:

```ocean-dsl
encryptionKey = generate
```

```ocean-dsl
encryptionKey = env:DATABASE_ENCRYPTION_KEY
```

```ocean-dsl
encryptionKey = file:/run/secrets/database-key
```

```ocean-dsl
encryptionKey = vault:secret/database#encryptionKey
```

### 9.3 Rules

- `encryptionKey` is required when any entity uses `encrypt:`.
- Fields listed in `encrypt:` must exist in the backing datatype.
- Encryption affects persistence and does not change the logical datatype of the field.
- The concrete encryption implementation depends on the generated target technology.
- Secure external key sources should be preferred over embedding literal secrets directly in DSL files.

---

## 10. Queries and Commands

Entities may define inline queries and commands.

Each query or command is defined on exactly one line.

---

### 10.1 Queries

Queries define read operations.

Syntax:

```ocean-dsl
query <name>(<parameters>) : <ReturnType>
```

Examples:

```ocean-dsl
query listTodoItems(offset:Int, limit:Int) : List<TodoItem>
```

```ocean-dsl
query findById(id: String) : TodoItem
```

```ocean-dsl
query findByTitle(title: String) : List<TodoItem>
```

```ocean-dsl
query findByTitleAndPriority(title: String, priority: Priority) : List<TodoItem>
```

```ocean-dsl
query findByTitleOrPriority(title: String, priority : Priority) : List<TodoItem>
```

Parameters and return values use Ocean datatypes. They may also reference pointer
datatypes, both directly and inside collections:

```ocean-dsl
query findById(id: String) : *Invoice
query findByStatus(status: InvoiceStatus) : List<*Invoice>
```

---

### 10.2 Commands

Commands define operations that modify persistent state.

Syntax:

```ocean-dsl
command [T|Transactional] <name>(<parameters>) : <ReturnType>
```

Examples:

```ocean-dsl
command Transactional createTodoItem(item:TodoItem) : TodoItem
```

```ocean-dsl
command T updateTodoItem(item:TodoItem) : TodoItem
```

```ocean-dsl
command deleteTodoItem(id:String) : _
```

A command may optionally include a command modifier before its name.

Examples from the current DSL include:

- `Transactional`
- `T`

The `_` return type indicates that the command does not return a meaningful result value.

Command parameters and return values may reference pointer datatypes in the same
way as queries:

```ocean-dsl
command T updateInvoice(item: *Invoice) : *Invoice
```

---

### 10.3 Atomic Numeric Field Adjustments

A command whose name follows `increase<Field>` or `decrease<Field>` atomically
adjusts the corresponding numeric entity field by one.

For example, given an `AccessMatrix` datatype with an `Int` field named
`version`:

```ocean-dsl
Entity AccessMatrix
    command increaseVersion(id:String) : Int
    command decreaseVersion(id:String) : Int
```

`increaseVersion` resolves the `Version` suffix to the `version` field,
increments its stored value by one, and returns the resulting value.
`decreaseVersion` resolves the same field, decrements its stored value by one,
and returns the resulting value.

#### Validation Rules

- The command name must contain a field suffix after `increase` or `decrease`.
- The resolved field must exist in the datatype backing the entity.
- The resolved field must have a supported numeric type.
- The command must accept exactly one identifying parameter.
- The return type must match the adjusted field type.
- Pointer, collection, and non-numeric fields cannot be adjusted this way.

Invalid adjustment commands are rejected during model validation rather than
producing invalid generated code.

#### Atomicity and Transactions

The adjustment is atomic. Concurrent calls must each apply exactly one
adjustment and return the value resulting from that adjustment. Implementations
must not use an unsafe read-modify-write sequence that can lose concurrent
updates.

An explicit `T` or `Transactional` modifier is not required when the adjustment
is used alone. It may be declared when the command participates in a larger
workflow whose other database changes must commit or roll back together:

```ocean-dsl
command T increaseVersion(id:String) : Int
```

---

### 10.4 Implicit CRUD for In-Memory Entities

Every entity in a `type = in-memory` database automatically receives a
standard set of generated CRUD operations. These operations do not need to
be declared with `query` or `command` — they are generated per entity
regardless of the concrete in-memory engine (`redis` or any other in-memory
engine).

Generated operations:

| Signature                                                                     | Kind    | Description                                                                                      |
| ----------------------------------------------------------------------------- | ------- | ------------------------------------------------------------------------------------------------ |
| `Put<EntityName>(entity: <EntityName>) : String`                              | Command | Stores the entity, generating a new ID; returns the generated ID.                                |
| `Put<EntityName>WithId(id: String, entity: <EntityName>) : _`                 | Command | Stores the entity under a caller-supplied ID, overwriting any existing value.                    |
| `Put<EntityName>WithTtl(entity: <EntityName>, ttl: Int) : String`             | Command | Stores the entity with a generated ID and an expiration; returns the generated ID.               |
| `Put<EntityName>WithIdAndTtl(id: String, entity: <EntityName>, ttl: Int) : _` | Command | Stores the entity under a caller-supplied ID with an expiration, overwriting any existing value. |
| `Exists<EntityName>(id: String) : Boolean`                                    | Query   | Checks whether an entity exists for the given ID.                                                |
| `Get<EntityName>(id: String) : <EntityName>`                                  | Query   | Retrieves the entity by ID without removing it. Fails if no entity exists for the ID.            |
| `Delete<EntityName>(id: String) : _`                                          | Command | Removes the entity by ID. Idempotent.                                                            |
| `Pop<EntityName>(id: String) : <EntityName>`                                  | Command | Atomically retrieves and removes the entity by ID. Fails if no entity exists for the ID.         |

`<EntityName>` refers to the entity's datatype. Fields listed as `id` are
the entity's identifier, independent of any `key(...)` declared on the
entity. `Get<EntityName>` and `Pop<EntityName>` fail when no entity exists
for the given ID; `Delete<EntityName>` is idempotent and does not fail when
the ID does not exist.

An entity may still declare additional custom `query` and `command`
operations alongside the implicit generated set.

---

## 11. Complete Example

The following is a complete `@database` file
(`ocean-examples/0002-inv-mgt/20-inv-db.ocn`):

```ocean-dsl
@database

Database InvDb
    engine = postgres
    configType = InvMgtDatabaseConfig
    tags = primary

    # ---------------------------------------------------
    # InvItem Entity
    # ---------------------------------------------------
    Entity InvItem
        key(id)
        indexes: index(status), index(locationId), index(batchId), index(expiryDate)

        # Relations
        location -m2o-> Location

        # Queries
        query listInvItems(offset:Int, limit:Int) : List<InvItem>
        query findById(id: String) : InvItem
        query findByStatus(status: ItemStatus) : List<InvItem>
        query findByLocationId(locationId: String) : List<InvItem>
        query findByMaterialType(materialType: MaterialType) : List<InvItem>
        query findByBatchId(batchId: String) : List<InvItem>
        query findByExpiryDateBefore(expiryDate: DateTime) : List<InvItem>
        query findByLocationAndName(locationId: String, name: String) : List<InvItem>
        query findByLocationAndBatchId(locationId: String, batchId: String) : List<InvItem>

        # Commands
        command T createInvItem(item: InvItem) : InvItem
        command T updateInvItem(item: InvItem) : InvItem
        command deleteInvItem(id: String) : _

    # ---------------------------------------------------
    # Location Entity
    # ---------------------------------------------------
    Entity Location
        key(id)
        indexes: unique(name), index(site), index(warehouse)

        # Queries
        query listLocations(offset:Int, limit:Int) : List<Location>
        query findById(id: String) : Location
        query findByName(name: String): List<Location>
        query findBySite(site: String): List<Location>
        query findByWarehouse(warehouse: String): List<Location>

        # Commands
        command T createLocation(item: Location) : Location
        command T updateLocation(item: Location) : Location
        command deleteLocation(id: String) : _

    # ---------------------------------------------------
    # ItemHistoryRecord Entity
    # ---------------------------------------------------
    Entity ItemHistoryRecord
        key(id)
        indexes: index(itemId), index(eventType), index(timestamp)

        # Queries
        query findByItemId(itemId: String) : List<ItemHistoryRecord>
        query findByItemIdAndEventType(itemId: String, eventType: ItemEventType) : List<ItemHistoryRecord>
        query listAll(offset:Int, limit:Int) : List<ItemHistoryRecord>

        # Commands
        command T createItemHistoryRecord(record: ItemHistoryRecord) : ItemHistoryRecord
```

This example demonstrates:

- a database definition;
- a concrete database engine (`postgres`);
- configuration through `configType`;
- database tags;
- multiple entity mappings in one database;
- single-field keys;
- inline single-field indexes and a uniqueness constraint;
- a many-to-one entity relationship;
- inline queries, including multi-parameter finders;
- inline commands with the `T` (transactional) modifier;
- `_` as an empty result type.

For encryption (`encryptionKey` plus entity-level `encrypt:`), see
Section 9; for composite indexes and `type = in-memory` implicit CRUD, see
Sections 8 and 10.4.

---

## 12. Imports and Datatype Resolution

The `@database` section does not directly import entities from Ocean.

All entities must be backed by datatypes that are already available to the DSL context.

Datatypes may be:

- defined locally in `@datatype`;
- included through the supported include mechanism;
- imported through the `@datatype` import mechanism.

For example:

```ocean-dsl
@datatype

@import datatype O.domain.payment.CardInfo@1.2.0 as CardInfo

Payment
    card : CardInfo
```

The database may then define an entity for an available datatype where required.

This keeps datatype definition and reuse within `@datatype`, while `@database` remains responsible for persistence concerns.

### 12.1 Including a Published Database Module

A `@database` file may also consist solely of an `@include` that pulls a complete,
published database definition from the Ocean Repository:

```ocean-dsl
@database

@include O.rbac.database@1.0.0
```

The included module is resolved as if its `Database` blocks were declared inline.

---

## 13. Rules and Constraints

The following rules apply:

- A database file starts with `@database`.
- A file may contain multiple `Database` blocks.
- `engine` is required.
- `configType` is required.
- `type` is optional and defaults to `transactional`.
- `tags` is optional.
- `encryptionKey` is required when encrypted entity fields are declared.
- Each entity must map to an existing datatype.
- Fields must be declared in the corresponding datatype and must not be redeclared in the entity.
- Fields referenced by `key(...)`, `indexes:`, `encrypt:`, or relationships must exist in the corresponding datatype.
- Entity references use ID-based mapping by default.
- `indexes:` definitions are declared on one line.
- `encrypt:` definitions are declared on one line.
- Each relationship is declared on one line.
- Each query is declared on one line.
- Each command is declared on one line.
- `increase<Field>` and `decrease<Field>` commands must satisfy the atomic
  numeric field-adjustment rules in Section 10.3.
- Entities in a `type = in-memory` database receive a standard generated
  CRUD operation set per Section 10.4, in addition to any explicitly
  declared queries and commands.
- Entities cannot be imported directly through `@database`.

---

## 14. Related Knowledge

The `@database` DSL is related to:

- `dsl.datatype` — defines the datatypes used as database entities.
- `dsl.config` — defines configuration referenced through `configType`.
- `dsl.vault` — may provide encryption keys using the `vault:` key specification.
- `dsl.import` — defines the mechanism for importing selected definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
