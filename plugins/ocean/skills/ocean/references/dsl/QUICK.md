# DSL Quick Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/

One card per DSL page: what it covers, its section titles, and a short syntax block copied from the page. Use it to orient yourself, then open the full file (named in each card) for rules, options and validation. Cards are excerpts, not complete grammars.

## Ocean-Atlas Reference Overview

File: [reference-overview.md](reference-overview.md) · Status: Approved · Authority: Authoritative

Consolidated overview of Ocean DSL references, directives, foundational concepts, identifiers, and maintenance conventions.

Sections: Overview · Start Here · DSL Reference Catalog · DSL Directives · Concepts · Reusable References · Behavioral Composition · Presentation Composition · Deployment Model · File Structure · File Extensions · Case Sensitivity · Comments · Document Authority · Suggested Reading Paths · Canonical Inventory · Maintenance

## Ocean DSL

File: [overview.md](overview.md) · Status: Accepted · Authority: Normative

Entry point for the Ocean domain-specific language, its source-file rules, directives, section identifiers, and canonical references.

Sections: Overview · Case Sensitivity · Source Files · Comments · Canonical DSL References · Definitions Pending Migration · Directives · Reserved and Future Sections · Related Knowledge

Syntax (from "Source Files"):

```ocean-dsl
@datatype

PurchaseOrder
    id     : String
    status : OrderStatus
```

## Ocean DSL System Design Model

File: [description.md](description.md) · Status: Draft · Authority: Explanatory — not final: content may be incomplete or change.

Explains how Ocean DSL definitions combine data contracts, behavior, composition, interfaces, persistence, messaging, and deployable services into a system model.

Sections: Overview · Architectural Layers · Data and State · Logic and Behavior · System Boundaries · End-to-End Model · Illustrative Definitions · Design Benefits · Scope and Evolution · Related Knowledge

Syntax (from "Data Contract"):

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

## Info DSL Reference

File: [info.md](info.md) · Status: Approved · Authority: Normative

Defines the Ocean `@info` section for declaring project-level identity, version, and descriptive metadata for a system or application.

Sections: Overview · Core Principle · Syntax · Project-level Section · Fields · Name · Name Representation · Version · Version Meaning · Semantic Versioning · Description · Description Representation · Field Ordering · Cardinality · Scope and Identity · Documentation Generation · Generated Output · Complete Example · Minimal Example · Validation · Invalid Examples · Info vs Service Metadata · Info vs Repository Identity · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

```ocean-dsl
@info

name: <SystemName>
version: <Version>
[title: <Title>]
[subtitle: <Subtitle>]
[shortDescription: <ShortDescription>]
[description: <Description>]
```

## DSL Metadata Reference

File: [metadata.md](metadata.md) · Status: Accepted · Authority: Normative

Defines generic Ocean DSL metadata using item-level tags and perspectives and an optional file-level group.

Sections: Overview · Item-Level Metadata · Tags · Perspectives · Canonical Spellings · File-Level Group · Placement and Validation · Complete Example · Related Knowledge

Syntax (from "Item-Level Metadata"):

```ocean-dsl
@<section>

<ItemName>
    @tags         : <tag1>, <tag2>, ...
    @perspectives : <key1>:<value1>, <key2>:<value2>, ...
    <item-specific definitions>
```

## Datatype DSL Reference

File: [datatype.md](datatype.md) · Status: Accepted · Authority: Normative

Defines the Ocean `@datatype` DSL section, including datatypes, primitive and compound types, IDs, enums, errors, envelopes, interfaces, and datatype imports.

Sections: Overview · Datatype Definition · Primitive Datatypes · Compound Datatypes · Naming Conventions · ID Fields · Enums · Errors · Envelopes · Interfaces · Imports · Complete Example · Related Knowledge

Syntax (from "Syntax"):

```text
<DatatypeName>
    <fieldName> : [*]<FieldType>
    ...
```

## Context DSL Reference

File: [context.md](context.md) · Status: Accepted · Authority: Normative

Defines the Ocean `@context` DSL section for service-local volatile shared state used by service logic and orchestration.

Sections: Overview · Definition · Syntax · Context Fields · Naming Conventions · Purpose and Semantics · Use in `@service` · Imports · Rules and Constraints · Complete Example · Related Knowledge

Syntax (from "Syntax"):

```ocean-dsl
@context

<ContextName>
    <fieldName> : <FieldType> (default=<value>)
```

## Config DSL Reference

File: [config.md](config.md) · Status: Approved · Authority: Normative

Defines the Ocean `@config` section for declaring named, reusable, typed configuration schemas used by deployable services.

Sections: Overview · Core Principle · Syntax · Configuration Schema · Schema Names · Configuration Keys · Flat Keys · Value Types · Required Keys · Default Values · Literal Values · Nested Configuration · Nested Key Paths · Nested Defaults · Recursive Schema References · Complete Schema Example · Service Usage · Service-only Consumption · Configuration Aliases · Multiple Configurations · Runtime Values · Secrets · Resolution · Defaults and Value Precedence · Validation · Invalid Examples · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

```ocean-dsl
@config

<ConfigName>
    <key> : <Type>
    <key> : <Type> (default=<value>)
```

## Vault DSL Reference

File: [vault.md](vault.md) · Status: Approved · Authority: Normative

Defines the Ocean `@vault` section for modeling logical secret stores, typed secret schemas, vault composition, and physical secret-backend generation.

Sections: Overview · Core Principle · Syntax Overview · Vault File · Vault Declaration · Vault Names · Vault Attributes · Engine · Engine Responsibilities · Configuration Type · Primary Vaults · Logical and Physical Vaults · Included Vaults · Inclusion Semantics · Vault Inclusion Graph · Secret Declaration · Secret Names · Secret Attributes · Secret Types · Primitive Secrets · Structured Secrets · Optionality · Secret Descriptions · Perspectives and Tags · Secret References · Secret Usage · Secret Values · Physical Deployment · Shared Physical Backend Example · Complete Example · Datatype Resolution · Vault Import Restrictions · Validation · Invalid Examples · Security Properties · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "Syntax Overview"):

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

## Perspective DSL Reference

File: [perspective.md](perspective.md) · Status: Accepted · Authority: Normative

Canonical reference ID for Ocean perspective metadata. Perspectives are defined by the generic metadata model in `dsl.metadata`; this document identifies the concept and points to its normative definition.

Sections: Overview · Normative Definition · Minimal Example · Related Knowledge

Syntax (from "Minimal Example"):

```ocean-dsl
NotificationFlow
    @perspectives : channel:email, region:US
```

## Tag DSL Reference

File: [tag.md](tag.md) · Status: Accepted · Authority: Normative

Canonical reference ID for Ocean classification tags. Tags are defined by the generic metadata model in `dsl.metadata`; this document identifies the concept and points to its normative definition.

Sections: Overview · Normative Definition · Minimal Example · Related Knowledge

Syntax (from "Minimal Example"):

```ocean-dsl
NotificationFlow
    @tags : messaging, core, external
```

## API DSL Reference

File: [api.md](api.md) · Status: Accepted · Authority: Normative

Defines the technology-independent Ocean `@api` DSL section and its style-specific public API operations, payload representations, transport configuration, and context-derived inputs.

Sections: Overview · API Definition · REST · SOAP · gRPC · GraphQL · WebSocket · API Operation References · Imports · Rules and Constraints · Complete REST Example · Related Knowledge

Syntax (from "API Definition"):

```ocean-dsl
@api

<ApiName> [style:<style>]
    <style-specific definitions>
```

## Broker DSL Reference

File: [broker.md](broker.md) · Status: Accepted · Authority: Normative

Defines the Ocean `@broker` DSL section for message brokers, topics, message payloads, messaging patterns, delivery semantics, publishers, and topic configuration.

Sections: Overview · Syntax · Broker Definition · Topics · Naming Conventions · Messaging Patterns · Delivery Semantics · Topic Configuration · Publishers · Topic References · Complete Example · Imports · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

```ocean-dsl
@broker

<BrokerName>
    <topicName> : <Datatype> as <pattern> [with <config>]
```

## Database DSL Reference

File: [database.md](database.md) · Status: Accepted · Authority: Normative

Defines the Ocean `@database` DSL section, including databases, entities, relationships, keys, indexes, encryption, queries, and commands.

Sections: Overview · Syntax · Database Definition · Database Types · Entity Definition · Entity Relationships · Keys · Indexes and Constraints · Field Encryption · Queries and Commands · Complete Example · Imports and Datatype Resolution · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

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

## Integration DSL Reference

File: [integration.md](integration.md) · Status: Accepted · Authority: Normative

Defines the Ocean `@integration` section for composing reusable integration flows between APIs, brokers, databases, files, and other resources using implemented contracts, used dependencies, typed connections, mapping expressions, scheduling, and aggregation.

Sections: Overview · Core Principle · Syntax Overview · Integration Names · Direction · No Explicit Strategy · Ports · Use Declarations · Implementation Declarations · Connect · Connection Sources and Targets · Mappers · Multiple Mappings · Aggregation · Scheduling and Triggering · Authentication and Credentials · Configuration · Hosting Boundary · Complete Example: Incoming Webhook to Broker · Complete Example: Scheduled API Polling · Complete Example: Outgoing Event to External API · Complete Example: File Import · Imports · Includes · Validation · Invalid Examples · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "Syntax Overview"):

```ocean-dsl
@integration

<IntegrationName>
    direction: <in|out|in/out>

    use config <ConfigName> as <alias>
    use api <ApiName> as <alias>
    impl api <ApiName> as <alias> on <port-or-config-path>
    use broker <BrokerName> as <alias>
    impl broker <BrokerName> as <alias>
    use database <DatabaseName> [as <alias>]
    use expression <ExpressionName> [as <alias>]
    use function <FunctionName> [as <alias>]
    use file <FileName> [as <alias>]

    schedule <schedule-expression>

    init <expression-or-function-call>
    connect <source> -> <target>
    connect <source> -<mapper>-> <target>
    aggregate <aggregation>
```

## Expression DSL Reference

File: [expression.md](expression.md) · Status: Accepted · Authority: Normative

Defines the Ocean expression model for stateless, side-effect-free reusable logic, including explicit named expressions, implicit inline expressions, typed inputs and outputs, built-in functions, logic blocks, and external implementations.

Sections: Overview · Explicit Expressions · Expression Elements · Inputs · Outputs · Logic Expressions · External Expressions · Built-in Functions · Boolean Expressions · Mapping and Projection · Computation · Expression Language · Literals · Variables and Field Access · Boolean Operations · Comparisons · Arithmetic · Conditional Expressions · Enum Values · Expression Composition · Implicit Expressions · Explicit vs Implicit Expressions · Use in FSMs · Use in Components · Use in Services · Imports · Complete Example · Rules and Constraints · Related Knowledge

Syntax (from "Explicit Expressions"):

```ocean-dsl
@expression

<ExpressionName>
    authorize <actor-context-name>
        <authorization-policy>
    context : <name1:DataType1> & <name2:DataType2> & ...
    input  : <name1:DataType1> & <name2:DataType2> & ...
    output : <name1:DataType1> & <name2:DataType2> & ...

    include: <BuiltinRef>

    logic
        <expression logic>
```

## Authorization Policy DSL Reference

File: [authorization.md](authorization.md) · Status: Approved · Authority: Normative

Defines declarative authorization policies (authenticated, role, scope, subject, tenant, organization, anyOf, allOf) and how expressions and connections attach them.

Sections: Purpose · Policy Rules · Expression Authorization · Connection Authorization · Trusted Context · Related Knowledge

Syntax (from "Expression Authorization"):

```ocean-dsl
UpdateUser
    authorize actor
        authenticated
        anyOf
            subject equals user.id
            role "platform-admin"
    input: ctx:Context & user:User
    output: result:User
    logic
        result = User.updateUser(user)
```

## Schedule and Trigger DSL Reference

File: [schedule.md](schedule.md) · Status: Accepted · Authority: Normative

Defines reusable, technology-independent schedules and their use as triggers for executable Ocean DSL items.

Sections: Overview · Scope and Applicable Items · Named Schedule Definitions · Schedule Values · Timezones and Daylight Saving Time · Overlap Behavior · Using a Schedule · Complete Service Example · Validation Rules · Generator Responsibility · Related Knowledge

Syntax (from "Named Schedule Definitions"):

```ocean-dsl
schedule FastSync every 10s overlap skip
schedule DailyExport daily at 14:00:35 timezone Europe/Amsterdam overlap queue
schedule WeeklyCleanup weekly on Monday, Wednesday at 09:00 timezone UTC
schedule MonthlyBilling monthly on day 1 at 00:05 overlap parallel
```

## FSM DSL Reference

File: [fsm.md](fsm.md) · Status: Accepted · Authority: Normative

Defines the Ocean `@fsm` DSL section for modeling stateful domain behavior using entity-controlled or in-memory finite-state machines, states, events, triggers, transitions, actions, and reusable logic.

Sections: Overview · FSM Types · FSM Declaration · FSM Declaration Elements · Events · States · Special States · State Triggers · Handler Types · Supported Actions · State Transitions · `this` · Expressions · Context · Database · Execution Model · Complete Entity-Controlled Example · Complete In-Memory Example · Internal Normalization · Imports · Rules and Constraints · Related Knowledge

Syntax (from "Entity-Controlled FSM"):

```ocean-dsl
@fsm

OrderFSM controls Order.status
    key: id
    database: OrderDB
    event confirm in(_) out(_)

    state Created
        on event confirm : logic
            next Confirmed

    state Confirmed
        on enter : external
            source: "order_hooks.go"
            method: "NotifyConfirmed"
```

## Component DSL Reference

File: [component.md](component.md) · Status: Accepted · Authority: Normative

Defines reusable, stateless behavioral components with typed ports, explicit dependencies, and typed internal connections.

Sections: Overview · When to Use a Component · General Structure · Component Identity and Metadata · Public Ports · Behavioral Dependencies · Connections · Lifecycle and State · Infrastructure Boundary · Nested Component Example · Imports and Includes · Validation · Generation and Documentation · Complete Example · Rules Summary · Related Knowledge

Syntax (from "General Structure"):

```ocean-dsl
@component

<ComponentName>
    input  : <inputName>:<InputType> [& ...]
    output : <outputName>:<OutputType> [& ...]

    use expression <ExpressionName> as <alias>
    use fsm <FSMName> as <alias>
    use component <ComponentName> as <alias>

    connect <source> -> <target>
    connect <source> -<mapper>-> <target>
```

## Service DSL Reference

File: [service.md](service.md) · Status: Approved · Authority: Normative

Defines the Ocean `@service` section for composing domain behavior, external contracts, infrastructure, configuration, secrets, initialization, aggregation, and typed connections into deployable units.

Sections: Overview · Core Principle · Syntax Overview · Service Names · Ports · Use and Implementation Declarations · API Dependencies · API Implementations · Config and Context · Brokers and Databases · Expressions, FSMs, and Components · Functions and Vaults · Alias Rules · Initialization · Connections · Connection Compatibility · Mappers · Fan-out · Common Connection Directions · Broker Patterns and Binds · Aggregation · Imports · Includes · Complete Example · Deployment Boundary · Validation · Invalid Examples · Rules and Constraints · Related Knowledge

Syntax (from "Ports"):

```ocean-dsl
use config TodoServiceConfig as myCfg
impl api TodoApi as api on myCfg.apiConfig.port
```

## Dashboard DSL Reference

File: [dashboard.md](dashboard.md) · Status: Approved · Authority: Normative

Defines the Ocean `@dashboard` section for composing widgets into reusable, backend-agnostic pages or screens.

Sections: Overview · Core Principle · Syntax · Dashboard Definition · Dashboard Names · Title and Subtitle · Layout · Widgets · Widget Properties · Metadata · Multiple Dashboards · Usage in UI · Backend Independence · Complete Example · Imports · Validation · Invalid Examples · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

```ocean-dsl
@dashboard

<DashboardName>
    title: <Title>
    subtitle: <Subtitle>
    layout: <LayoutName>

    Widget <WidgetName> of type <WidgetType>
        <scalarProperty> = <value>
        <listProperty>:
            - <value>
        <mapProperty>:
            <key>: <value>
```

## Widget DSL Reference

File: [widget.md](widget.md) · Status: Approved · Authority: Normative

Defines widget declarations inside Ocean dashboards, including names, predefined types, scalar/list/map properties, datatype binding, and generated functions.

Sections: Overview · Syntax · Dashboard Scope · Widget Identity · Property Shapes · Datatype Binding · Predefined Functions · Defaults · Complete Example · Validation · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

```ocean-dsl
Widget <WidgetName> of type <WidgetType>
    <scalarProperty> = <value>
    <listProperty>:
        - <value>
    <mapProperty>:
        <key>: <value>
```

## Predefined Widget Types

File: [widget-predefined-types.md](widget-predefined-types.md) · Status: Approved · Authority: Normative

Catalogs the predefined Ocean widget types and their implemented properties, defaults, and generated functions.

Sections: Overview · Separator · Table · Form · Button · Chart · Text · Validation · Related Knowledge

Syntax (from "Separator"):

```ocean-dsl
Widget OrderDivider of type Separator
    label = "Order Actions"
    style = dashed
    size = lg
    margin = my-4
```

## UI DSL Reference

File: [ui.md](ui.md) · Status: Approved · Authority: Normative

Defines the Ocean `@ui` section for composing dashboards into deployable user-interface services and connecting widget functions to APIs and brokers.

Sections: Overview · Core Principle · Syntax Overview · UI Names · Technology Properties · Framework · Styling · Template · Backend · Static Path · Configuration Type · Navigation · Navigation Examples · Header · Footer · Using Dashboards · Using APIs · Using Brokers · Connections · Widget-to-backend Flow · Backend-to-widget Flow · Mappers and Fan-out · Type Compatibility · Imports · Includes · Complete Example · Deployment Boundary · Validation · Invalid Examples · Rules and Constraints · Related Knowledge

Syntax (from "Navigation Examples"):

```ocean-dsl
nav View dashboard = TodoViewDashboard
nav Company link = https://company.com
nav Schema file = static/schema.pdf
```

## Ocean UI Inline Tab Metadata

File: [tab-metadata.md](tab-metadata.md) · Status: Accepted · Authority: Normative for Ocean UI; non-semantic to the Ocean DSL engine

Define the optional inline metadata block Ocean UI reads from DSL bundle tabs to organize tabs by tags and semantic perspectives.

Sections: Overview · Canonical Block Format · Fields · Editing Metadata · Bundle Grouping · Invalid and Malformed Blocks · Examples · Non-Goals · Related Knowledge

Syntax (from "Canonical Block Format"):

```ocean-dsl
# @ocean-meta-start
# tags:
#   - authentication
#   - security
# perspective:
#   feature: authentication
#   boundedContext: identity
#   phase: mvp
#   status: stable
# @ocean-meta-end

@service

UserService
    ...
```

## Deploy DSL Reference

File: [deploy.md](deploy.md) · Status: Approved · Authority: Normative

Defines the Ocean `@deploy` section for materializing services, UIs, and brokers with replicas, configuration, exported ports, environment values, volumes, dependencies, brokers, and runtime connections.

Sections: Overview · Core Principle · File Syntax · File Name · Deployment Blocks · Deployable Target · Replica Count · Configuration Bindings · Exported Ports · Environment Variables · Volumes · Brokers · Dependencies · Dependency Graph · Connections · Connection Resolution · Multiple Targets · Imports · Includes · Groups · Generation Units · Per-deploy and Per-file Semantics · Complete Example · Local and Imported Targets · Validation · Invalid Examples · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "File Syntax"):

```ocean-dsl
@deploy [(group:<GroupName>)]

Name: <DeployFileName>
[@import service <RepositoryRef> as <Alias>]
[@import config  <RepositoryRef> as <Alias>]
[@import broker  <RepositoryRef> as <Alias>]

<DeployName>
    service <DeployableName>
    replica <Count>
    config <ConfigName> [as <alias>], ...
    export <HostPort>:<ListenerTarget>, ...
    env <Key>=<Value>, ...
    vol <Source>=<Target>[:<mode>], ...
    broker <BrokerName> [as <alias>]
    dependsOn <DeployName>, ...
    connect <source> -> <target>, ...
```

## Import DSL Reference

File: [import.md](import.md) · Status: Draft · Authority: Normative — not final: content may be incomplete or change.

Defines the Ocean `@import` directive for resolving a selected reusable definition and exposing it through a local alias.

Sections: Overview · Syntax · Examples · Semantics · Aliases · Type Compatibility · Version Resolution · Validation · Import vs Include · Related Knowledge

Syntax (from "Examples"):

```ocean-dsl
@import fsm O.auth.loginFlow@1.2.0 as login
@import expression O.std.IsEmail@latest as isEmail
@import component O.billing.InvoiceLogic@2.0.0 as invoice
```

## Include DSL Reference

File: [include.md](include.md) · Status: Draft · Authority: Normative — not final: content may be incomplete or change.

Defines the Ocean `@include` directive for statically composing same-section Ocean DSL definitions from versioned Ocean Repository or local Pre-baked references.

Sections: Overview · Core Principle · Syntax · Placement · Static Inclusion · Section-Type Compatibility · Supported Section Types · Ocean Repository References · Local Pre-baked References · Version Resolution · Transitive Inclusion · Cyclic Inclusion · Name Scope and Uniqueness · Reference Resolution · Validation · Include vs Import · Complete Example · Invalid Examples · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

```text
O.<namespace>.<name>@<version>
P.<namespace>.<name>@<version>
```

## Asset DSL Reference

File: [asset.md](asset.md) · Status: WIP · Authority: Normative — not final: content may be incomplete or change.

Defines the `@asset` directive for attaching external files to DSL elements. Assets are copied verbatim into generated projects and are not interpreted or transformed.

Syntax (from "Versioned asset"):

```ocean-dsl
@expression

@asset "firebase-auth@1.1.0"

IsAuthenticated
    input  : token:String
    output : result:Boolean

    external
        source: "firebase-auth.go"
        method: "ValidateToken"
```

## Artifact Directive Reference

File: [artifact.md](artifact.md) · Status: Approved · Authority: Normative

Defines the Ocean `@artifact` directive for resolving, interpreting, transforming, and materializing external semantic files as part of a generated project.

Sections: Overview · Core Principle · Directive, Not Section · Syntax · Artifact Type · Artifact Reference · Local File References · Named and Versioned References · Scoped Ownership · Valid Scopes · Resolution · Type-aware Interpretation · Transformation · Materialization · Generation Order · Output Ownership and Conflicts · Artifact vs Asset · Go Artifact Example · Local Artifact Example · Validation · Failure Behavior · Determinism and Safety · Purpose · Rules and Constraints · Related Knowledge

Syntax (from "Syntax"):

```text
<artifact-type> ::= <identifier>

<artifact-ref>  ::= <path> | <name>[@<version>]
```
