# API DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/api/

**Contents**

- [1. Overview](#1-overview)
- [2. API Definition](#2-api-definition)
- [3. REST](#3-rest)
- [4. SOAP](#4-soap)
- [5. gRPC](#5-grpc)
- [6. GraphQL](#6-graphql)
- [7. WebSocket](#7-websocket)
- [8. API Operation References](#8-api-operation-references)
- [9. Imports](#9-imports)
- [10. Rules and Constraints](#10-rules-and-constraints)
- [11. Complete REST Example](#11-complete-rest-example)
- [12. Related Knowledge](#12-related-knowledge)

Defines the technology-independent Ocean `@api` DSL section and its style-specific public API operations, payload representations, transport configuration, and context-derived inputs.

Source: https://atlas.ocean-lab.ai/dsl/api/

Status: Accepted · Authority: Normative

## 1. Overview

The `@api` section defines the public interface of a service.

It describes how external clients interact with a service through well-defined operations using typed request and response objects.

The Ocean API model is technology-independent. An API declares a `style` that determines the syntax and interaction model of its operations.

Supported styles include:

- REST — default; generated;
- SOAP — generated;
- gRPC — generated;
- GraphQL — planned; section 6 outlines the syntax only;
- WebSocket — planned; section 7 outlines the syntax only.

A file begins with:

```ocean-dsl
@api
```

and may define one or more APIs.

---

## 2. API Definition

General syntax:

```ocean-dsl
@api

<ApiName> [style:<style>]
    <style-specific definitions>
```

If no style is specified, REST is used by default.

Example:

```ocean-dsl
@api

OrderAPI
    get /orders/{id} getOrder(id:String) : Order
```

Equivalent explicit style:

```ocean-dsl
@api

OrderAPI style:rest
    get /orders/{id} getOrder(id:String) : Order
```

The syntax of individual operations depends on the API style.

### 2.1 API-Level Attributes

Before its operations, an API may declare attributes as `key = value` lines,
one per line, indented under the API name:

```ocean-dsl
TodoApi style: rest
    tags            = external
    version         = 1.0.0
    description     = ToDo API Description
```

| Attribute     | Required | Description                                                          |
| ------------- | -------- | -------------------------------------------------------------------- |
| `version`     | No       | API version string, typically semantic (`1.0.0`).                    |
| `description` | No       | Human-readable API description, surfaced in generated documentation. |
| `tags`        | No       | Comma-separated classification tags, such as `external`, `internal`. |

Each style declares its own transport attributes. For example, REST defines
`engine`, `configType`, `basePath`, and `generateSwagger`; SOAP defines its
own `engine`, `configType`, `namespace`, `endpointPath`, and `soapVersion`;
gRPC defines its own `engine`, `configType`, `protoPackage`, and `reflection`.

Whitespace around `=` is not significant, and `style:` may be written with or
without a space after the colon (`style:rest` or `style: rest`).

---

### 2.2 API Composition

An API may compose the public operations of one or more other APIs. Composition
is a contract-level capability: it reuses operation definitions without
repeating them in the composing API. It does not declare runtime routing; a
`@service` gateway declares how the public API is implemented.

```ocean-dsl
AnswerGatewayApi style:rest
    engine          = gin
    configType      = ApiConfig
    basePath        = /answers
    generateSwagger = true

    include TimeDateApi, CalendarApi

    get /zone-time getZoneTime() : String
```

`include` accepts one or more comma-separated API names. Inside an `@api`
definition, it always means API composition. It is distinct from the existing
repository-level `@include` directive. Included APIs may
include other APIs, so a composed API exposes the transitive effective set of
operations. Every included API must use the same API style as the composing
API.

The composing API owns its transport-level configuration. Included APIs
contribute operations, not transport attributes. For REST, an included operation
keeps its method and relative path, and the composing API's `basePath` is
applied when it is exposed. For SOAP, included operations are exposed through
the composing API's namespace, endpoint path, SOAP version, and WSDL contract.
For gRPC, included operations become methods of the composing API's protobuf
service, package, and `.proto` contract.

#### Overrides

A composing API may declare an operation with the same method name as an
included operation. The local declaration overrides the inherited operation:

```ocean-dsl
PublicOrdersApi style:rest
    basePath = /public/orders
    include InternalOrdersApi

    # Exposes the inherited getOrder contract at a public route.
    get /{id} getOrder(id:String) : Order
```

An override must keep the inherited operation contract compatible: the input
names and types, output type, and context-derived inputs must match. It may
change transport-specific presentation and documentation. For REST, this
includes the HTTP method, path, payload representations, and endpoint
configuration.

If two included APIs provide the same operation name, the composition is
ambiguous and invalid unless the composing API declares one compatible local
override. API inclusion cycles are invalid. An API's final effective operation
set must contain unique operation names.

Existing APIs without `include` keep their current behavior unchanged.

---

## 3. REST

REST is the default API style.

REST declares the following transport attributes before its endpoints:

| Attribute         | Required | Description                                                                        |
| ----------------- | -------- | ---------------------------------------------------------------------------------- |
| `engine`          | Yes      | Transport/framework that serves the REST API, such as `gin`.                       |
| `configType`      | Yes      | Datatype that supplies the API's runtime configuration (see `dsl.config`).         |
| `basePath`        | No       | Path prefix prepended to every operation path. Defaults to `/`.                    |
| `generateSwagger` | No       | Whether an OpenAPI/Swagger document is generated. Defaults to `false`.             |

### 3.1 Syntax

```ocean-dsl
<ApiName> [style:rest]
    <method> <path> <MethodName>(<inputName>:<InputType> [in-query],...) : <OutputType> [@request:<payload-type>] [@response:<payload-type>] [@from-context:<item1>,<item2>,...] [with <config>]
```

An endpoint may be written on one line or, for readability, with its input
parameters on separate indented lines.

Example:

```ocean-dsl
TodoApi style: rest
    engine = gin
    configType = ApiConfig
    tags = external
    version = 1.0.0
    description = ToDo API Description
    basePath = /
    generateSwagger = true

    get    /todo/info         getInfo(_) : String
    get    /todo/items/{offset}/{limit} listItems(offset:Int, limit:Int) : List<TodoItem>
    get    /todo/item/{id}    getItem(_) : TodoItem
    get    /todo/item2/{id}   getItem2(_) : TodoItem
    get    /todo/title/{title} findItemByTitle(title:String) : List<TodoItem>
    get    /todo/title-and-priority/{title}/{priority} findItemByTitleAndPriority(title:String, priority:Priority) : List<TodoItem>
    get    /todo/title-or-priority/{title}/{priority} findItemByTitleOrPriority(title:String, priority:Priority) : List<TodoItem>
    post   /todo/item         createItem(item:TodoItem) : TodoItem
    put    /todo/item         adjustItem(item:TodoItem) : TodoItem
    delete /todo/item/{id}    deleteItem(_) : _

```

All `id` path parameters are `String` by default when no explicit type is declared.

---

### 3.2 REST Endpoint Elements

| Element         | Description                                                                                       |
| --------------- | ------------------------------------------------------------------------------------------------- |
| `method`        | HTTP method such as `get`, `post`, `put`, `patch`, or `delete`.                                   |
| `path`          | URL path. Supports path parameters such as `{id}` or typed placeholders where supported.          |
| `MethodName`    | Unique operation identifier used to reference the API operation.                                  |
| `InputType`     | Datatype used as request input. `_` indicates no input.                                           |
| `in-query`      | Optional explicit binding of an input parameter to the URL query string.                           |
| `OutputType`    | Datatype returned by the operation. `_` indicates no output.                                      |
| `@request`      | Optional request payload representation.                                                          |
| `@response`     | Optional response payload representation.                                                         |
| `@from-context` | Optional values provided implicitly from the execution context.                                   |
| `with`          | Optional comma-separated `key:value` endpoint configuration, such as `auth:required, version:v1`. |

HTTP methods are written in lowercase.

Paths are written in lowercase. Path placeholders may be untyped (`{id}`) or
typed (`{id:String}`); an untyped placeholder defaults to `String`.

Method names follow Ocean field naming conventions: they begin with a lowercase
letter and contain only letters, digits, and `_`.

Input and output type names follow Ocean datatype naming conventions: they begin
with an uppercase letter and contain only letters, digits, and `_`.

---

### 3.3 Path Parameters

REST paths may contain parameters.

Example:

```ocean-dsl
get /orders/{id} getOrder(id:String) : Order
```

Every path parameter must also appear in the method input list, and its declared
type there applies to the path placeholder.

Path binding is implicit: an input whose name matches a placeholder in the path
is a path parameter. Do not add `in-query` to a path parameter.

### 3.4 Query Parameters

REST endpoints may bind an input parameter to the URL query string by adding
`in-query` after its type:

```ocean-dsl
get /orders searchOrders(
    status : *OrderStatus in-query,
    cursor : *String      in-query,
    limit  : *Int         in-query,
) : OrderPage
```

Path and query parameters may be combined in the same endpoint. Parameters
matching a path placeholder bind to the path; only the remaining filter and
pagination parameters use `in-query`:

```ocean-dsl
get /tenants/{tenantId}/audit/actions listAuditActions(
    tenantId : String,
    actorId  : *String in-query,
    action   : *String in-query,
    cursor   : *String in-query,
    limit    : *Int    in-query,
) : AuditActionPage
```

`in-query` is explicit by design. On `GET` and `DELETE` endpoints, every input
declared in the signature must therefore be either a path parameter or be
marked `in-query`. This prevents an input from silently acquiring an unintended
HTTP binding.

Query parameters are also allowed on `POST`, `PUT`, and `PATCH` endpoints when
an operation has query-based options in addition to its request payload. Inputs
without `in-query` retain their existing request-body behavior for those methods.

The parameter name is the query-string key. An optional Ocean type, such as
`*String`, `*Int`, or `*DateTime`, represents an optional query parameter; a
non-optional type represents a required query parameter.

The path must not contain a literal query string. Write `/orders`, not
`/orders?status=...`; all query parameters belong in the typed method signature.

#### Query Parameter Rules

- `in-query` is valid only for REST endpoint inputs.
- A query parameter must not also be a path parameter.
- A parameter may be bound at most once.
- Context-derived inputs declared with `@from-context` are not query parameters.
- Query parameters currently use scalar Ocean types (`String`, `Int`, `Float`,
  `Double`, `Boolean`, `Date`, `Time`, `DateTime`, and `Timestamp`) or enums.
  Collection and object query encodings require an explicit future contract.
- `GET` and `DELETE` endpoints still cannot declare `@request`; `in-query` does
  not create a request body.
- Existing REST definitions remain unchanged: path placeholders continue to
  define path parameters, and request-body inputs retain their current behavior.

#### Audit Filter Example

```ocean-dsl
AuditApi style:rest
    engine     = gin
    configType = ApiConfig
    basePath   = /audit

    get /actions listAuditActions(
        tenantId      : *String   in-query,
        actorId       : *String   in-query,
        sourceService : *String   in-query,
        targetType    : *String   in-query,
        targetId      : *String   in-query,
        action        : *String   in-query,
        result        : *String   in-query,
        from          : *DateTime in-query,
        to            : *DateTime in-query,
        cursor        : *String   in-query,
        limit         : *Int      in-query,
    ) : AuditActionPage
```

---

### 3.5 Payload Representations

REST endpoints may explicitly define request and response payload representations.

Supported representations are:

| Alias       | Canonical Media Type                | Request | Response |
| ----------- | ----------------------------------- | ------- | -------- |
| `json`      | `application/json`                  | Yes     | Yes      |
| `xml`       | `application/xml`                   | Yes     | Yes      |
| `multipart` | `multipart/form-data`               | Yes     | No       |
| `form`      | `application/x-www-form-urlencoded` | Yes     | No       |

Both aliases and canonical media types are accepted.

Examples:

```ocean-dsl
post /orders createOrder(req:OrderRequest) : OrderResponse
```

```ocean-dsl
post /orders createOrder(req:OrderRequest) : OrderResponse @request:application/json
```

```ocean-dsl
post /orders createOrder(req:OrderRequest) : OrderResponse @request:json @response:xml
```

```ocean-dsl
put /orders updateOrder(req:OrderRequest) : OrderResponse @request:application/xml @response:application/xml
```

```ocean-dsl
post /upload upload(file:UploadRequest) : UploadResult @request:multipart
```

```ocean-dsl
post /login login(req:LoginRequest) : LoginResponse @request:form
```

#### Payload Representation Rules

- Request and response representations are configured independently.
- `GET` and `DELETE` endpoints cannot declare `@request`.
- `POST`, `PUT`, and `PATCH` default to `application/json` when `@request` is omitted.
- Responses default to `application/json` when `@response` is omitted.
- Payload aliases are normalized to their canonical media types.

---

### 3.6 Empty Input and Output

Use `_` when an endpoint has no input or no output.

No input — either `(_)` or an empty parameter list `()`:

```ocean-dsl
delete /orders/{id} deleteOrder(_) : _
```

```ocean-dsl
get /orders getAllOrders() : List<Order>
```

No output:

```ocean-dsl
put /orders/{id} updateOrder(req:UpdateOrderRequest) : _
```

---

### 3.7 CORS Configuration

An API may define Cross-Origin Resource Sharing configuration using `cors:`.

CORS controls browser access to an API from different origins, such as a UI application hosted on another domain or port.

Syntax:

```ocean-dsl
<ApiName> style:rest
    cors:
        allowOrigins: <origin>[, <origin>, ...]
        allowMethods: <method>[, <method>, ...]
        allowHeaders: <header>[, <header>, ...]
        allowCredentials: <true|false>
```

#### CORS Fields

| Field              | Required | Description                                                          |
| ------------------ | -------- | -------------------------------------------------------------------- |
| `allowOrigins`     | Yes      | Allowed origins including scheme, host, and optional port.           |
| `allowMethods`     | No       | Allowed HTTP methods. Defaults to `GET, POST, PUT, DELETE, OPTIONS`. |
| `allowHeaders`     | No       | Allowed request headers. Defaults to `Content-Type, Authorization`.  |
| `allowCredentials` | No       | Whether browser credentials are allowed. Defaults to `false`.        |

#### CORS Rules

- CORS applies at API level rather than per endpoint.
- When `allowCredentials` is `true`, `allowOrigins` must not contain `*`.
- REST APIs should allow `OPTIONS` when browser preflight requests are required.
- If CORS configuration is omitted, no CORS headers are generated.

Example:

```ocean-dsl
@api

GatewayAPI style:rest
    basePath = /

    cors:
        allowOrigins: http://localhost:5173, https://ui.ocean-lab.ai
        allowMethods: GET, POST, PUT, DELETE, OPTIONS
        allowHeaders: Content-Type, Authorization
        allowCredentials: true

    post /auth/login  login(req:LoginRequest) : LoginResponse
    post /auth/logout logout(_) : _
```

---

### 3.8 Context-Derived Inputs

REST endpoints may declare values supplied implicitly by the execution context using `@from-context`.

Syntax:

```ocean-dsl
<method> <path> <MethodName>(<inputs>) : <OutputType> @from-context:<item1>,<item2>,...
```

Example:

```ocean-dsl
# Session token and authenticated actor
post /auth/session getSession(input:SessionInput) : *SessionResult
delete /users/{id} softDeleteUser(id:String) : _ @from-context:actor
```

Context-derived items are not part of the explicit API input signature.

They represent environmental or execution information such as an authenticated
actor, a session token, request identity, or locale. They are not domain input
supplied in an HTTP request body, path, or query parameter.

#### Ocean Request Context

For calls from an Ocean gateway to an Ocean backend service, generated
transports use the `Ocean-Context` HTTP field. It carries a compact, signed,
versioned internal envelope. `Ocean-Context` is an Ocean private-use field; new
definitions must not use an `X-` prefix.

The field is sent only on trusted service-to-service calls. A backend verifies
the envelope before making its values available in the execution context. An
external caller cannot establish, replace, or augment this context by sending
an `Ocean-Context` field.

Version 1 has the following logical payload. Fields are included only when
known; empty placeholder values are not emitted.

```json
{
  "v": 1,
  "requestId": "req_...",
  "actor": {
    "subjectId": "USR-...",
    "subjectType": "user",
    "tenantId": "TEN-...",
    "organizationIds": ["ORG-..."],
    "roles": ["admin"],
    "scopes": ["users:write"]
  },
  "locale": "en-NL",
  "idempotencyKey": "optional",
  "client": { "id": "mec-ui", "version": "1.2.0" },
  "deadline": "2026-10-20T15:30:00Z"
}
```

The envelope deliberately excludes full domain entities, request bodies,
cookies, arbitrary incoming headers, and raw bearer tokens. Distributed tracing
uses the standard `traceparent` and optional `tracestate` HTTP fields instead.
Raw authorization tokens are propagated only when an endpoint explicitly
requires them.

`actor` is the standard authenticated principal. It is intentionally generic:
the subject can be a user, service account, or future agent. The gateway is
responsible for authentication, authorization, and establishing this trusted
value. Backend services consume it but do not trust a caller-supplied actor.

#### Context bindings in service connections

An API declares the environmental information it requires; a service
connection declares how that information fulfils a target logic parameter.
This keeps domain-specific command parameters out of a public API contract.

```ocean-dsl
delete /users/{id} softDeleteUser(id:String) : _ @from-context:actor

connect api.softDeleteUser -> User.softDeleteUser
    bind deletedBy <- context.actor
```

The source type of `context.actor` is the standard `OceanActor` context type.
The destination type is taken from the target parameter—in this example,
`deletedBy:*User`. The generator creates the destination value using a
conservative structural projection:

- The target parameter must exist and have a supported destination type.
- Fields must have compatible names and types to be copied.
- The standard `OceanActor.subjectId` field is an explicit alias for an entity
  identity field named `id`.
- For an entity destination, every required identity/key field must be
  populated; otherwise model generation fails.
- Only identity and tenant/organization-scope fields are copied by default.
- No implicit type coercion, nested-object conversion, or copying of roles,
  scopes, permissions, profile data, or other non-identity claims occurs.

Thus an actor with `subjectId = "USR-1"` can create the lightweight reference
`&User{Id: "USR-1"}` required by a `deletedBy:*User` command parameter,
without transmitting a full `User` record between services.

The short binding form is intentionally conservative. A future explicit-field
extension may select additional safe fields without changing the meaning of an
existing binding:

```ocean-dsl
bind deletedBy <- context.actor fields(id, organizationId)
```

That extension is reserved syntax and is not inferred from future same-named
fields. Adding a field to either datatype must not silently broaden an existing
binding.

The transport layer extracts verified values and makes them available to the
service execution context before invoking service logic. Validation is enforced
by generated code and runtime infrastructure.

---

## 4. SOAP

SOAP is an operation-oriented API style. A SOAP API generates a SOAP server,
typed client, and WSDL contract from the same Ocean definition.

### 4.1 Syntax

```ocean-dsl
<ApiName> style:soap
    engine       = soap
    configType   = <ConfigType>
    version      = <version>
    namespace    = <absolute-URI>
    endpointPath = <absolute-path>
    soapVersion  = <1.1|1.2>

    operation <OperationName>(<inputName>:<InputType>, ...) : <OutputType>
        [action = <absolute-URI>]
        [fault <FaultName> : <FaultType>]
```

Example:

```ocean-dsl
@api

OrderSoapApi style:soap
    engine       = soap
    configType   = ApiConfig
    version      = 1.0.0
    namespace    = https://example.com/orders/v1
    endpointPath = /orders
    soapVersion  = 1.2

    operation submitOrder(
        request : SubmitOrderRequest
    ) : SubmitOrderResponse
        action = https://example.com/orders/v1/submitOrder
        fault invalidOrder    : InvalidOrderFault
        fault processingError : OrderProcessingFault

    operation getOrder(
        request : GetOrderRequest
    ) : GetOrderResponse

    operation notifyOrder(
        notification : OrderNotification
    ) : _
```

### 4.2 SOAP Attributes

| Attribute      | Required | Description                                                                                          |
| -------------- | -------- | ---------------------------------------------------------------------------------------------------- |
| `engine`       | Yes      | SOAP runtime/code-generation engine. Initial support accepts `soap`.                                |
| `configType`   | Yes      | Datatype that supplies the API's runtime configuration, including its listening port.               |
| `namespace`    | Yes      | Absolute XML namespace URI used as the WSDL target namespace and for generated XML contract names.  |
| `endpointPath` | Yes      | Absolute HTTP path at which the generated SOAP endpoint is served.                                  |
| `soapVersion`  | Yes      | SOAP protocol version: `1.1` or `1.2`.                                                              |
| `version`      | No       | Human-facing API version. It does not replace the SOAP protocol version.                            |

Ocean initially uses the interoperable document-literal-wrapped SOAP binding.
It is intentionally fixed rather than exposed as a `binding` attribute until
another binding is supported.

### 4.3 Operations and Faults

Each `operation` maps to a WSDL operation. Its named inputs form the generated
document-literal wrapper for the SOAP request body; its output type forms the
SOAP response body. An operation may use `()` or `(_)` for no input, and `_` for
no response (a one-way operation).

`action` is optional. When omitted, Ocean derives it as
`<namespace>/<OperationName>`. The generated transport applies that action in
the form required by the selected SOAP version.

Each `fault <FaultName> : <FaultType>` line adds a typed SOAP/WSDL fault. Fault
names must be unique within an operation, and fault types must resolve to
structured Ocean datatypes.

WSDL service, port type, binding, and port names are derived from the API name.
They are not declared separately unless a future interoperability requirement
needs explicit overrides. WSDL generation is inherent to a SOAP API and cannot
be disabled.

A generated SOAP server also serves `/health`, `/health/live`, and
`/health/ready` on its port, for container healthchecks and other probes.

#### SOAP Rules

- Operation names must be unique in the effective SOAP API.
- Input, output, and fault types must resolve.
- `namespace` must be an absolute URI and `endpointPath` must be an absolute
  HTTP path.
- `soapVersion` must be exactly `1.1` or `1.2`.
- `engine` must be a supported SOAP engine; initially, `soap` is the only value.
- SOAP API composition uses `include` exactly as other API styles do; every
  included API must also use `style:soap`.
- SOAP headers, WS-Security, MTOM attachments, and custom XML element mappings
  are future explicit extensions; they are not inferred from operation inputs.

---

## 5. gRPC

gRPC is an operation-oriented API style. A gRPC API generates a Protocol
Buffers contract (`.proto`), a typed server, and a typed client from the same
Ocean definition.

### 5.1 Syntax

```ocean-dsl
<ApiName> style:grpc
    engine       = connect
    configType   = <ConfigType>
    version      = <version>
    protoPackage = <protobuf-package>
    reflection   = <true|false>

    rpc <MethodName>(<inputName>:<InputType>, ...) : <OutputType>
```

Streaming operations mark the streamed side with `stream`:

```ocean-dsl
    rpc <MethodName>(<inputName>:<InputType>) : stream <OutputType>          # server streaming
    rpc <MethodName>(stream <inputName>:<InputType>) : <OutputType>          # client streaming
    rpc <MethodName>(stream <inputName>:<InputType>) : stream <OutputType>   # bidirectional streaming
```

An operation may be written on one line or, for readability, with its input
parameters on separate indented lines.

Example:

```ocean-dsl
@api

OrderGrpcApi style:grpc
    engine       = connect
    configType   = ApiConfig
    version      = 1.0.0
    protoPackage = example.orders.v1
    reflection   = true

    # A structured input and output are used directly as protobuf messages.
    rpc submitOrder(order:SubmitOrderRequest) : SubmitOrderResponse

    # Scalar inputs and outputs are wrapped in generated messages.
    rpc getOrderStatus(id:String) : String

    # Several inputs form one generated request message.
    rpc cancelOrder(
        id     : String,
        reason : String
    ) : _

    # No input.
    rpc listOpenOrders(_) : List<Order>

    # Server streaming: one request, many responses.
    rpc watchOrder(id:String) : stream OrderEvent
```

### 5.2 gRPC Attributes

| Attribute      | Required | Description                                                                                                                                                                                   |
| -------------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `engine`       | Yes      | gRPC runtime/code-generation engine. Initial support accepts `connect`, which serves gRPC, gRPC-Web, and the Connect protocol.                                                                |
| `configType`   | Yes      | Datatype that supplies the API's runtime configuration, including its listening port.                                                                                                         |
| `protoPackage` | Yes      | Protobuf package: lowercase dot-separated identifiers, conventionally ending in a version (`example.orders.v1`). Named `protoPackage` to keep it distinct from Go or other language packages. |
| `reflection`   | No       | Whether gRPC server reflection is served, so tools can discover the API at runtime. Defaults to `true`.                                                                                       |
| `version`      | No       | Human-facing API version. It does not replace the `protoPackage` version.                                                                                                                     |

Ocean derives the protobuf service from the API: the service is named after
the API, and each `rpc` becomes a method of that service. Service, method, and
message names are not declared separately.

The endpoint is not declared: gRPC serves each method at
`/<protoPackage>.<ApiName>/<RpcName>` on the API's configured port.

#### Served Endpoints

Besides the protobuf methods, a generated gRPC server serves plain HTTP
endpoints on the same port:

| Path                     | Purpose                                                         |
| ------------------------ | --------------------------------------------------------------- |
| `/docs`                  | Interactive documentation with a "try it" form per operation.   |
| `/docs/<api-name>.proto` | The API's `.proto` contract.                                    |
| `/health`, `/health/live`, `/health/ready` | Liveness and readiness probes.                |

The protobuf methods are also reachable as Connect JSON (`POST` of a JSON
message to the method path), which the documentation page uses. Native gRPC
uses HTTP/2, served in cleartext so services inside a deployment need no TLS.

#### Generated Contract

Generation of the `.proto` contract is inherent to a gRPC API and cannot be
disabled.

- The contract uses `syntax = "proto3"`.
- The file is named after the API in kebab case, dropping an `Api` suffix, as
  for a WSDL: `OrderGrpcApi` produces `order-grpc.proto`.
- The project's `proto/` folder is the build's protobuf module and holds one
  contract per API. Each implementing service also gets a copy beside its
  source, and the build publishes it with the service's other API contracts.
- The build compiles `proto/` into Go code under `gen/`, in the project's
  module path. It needs only the Go toolchain.
- Its `package` is the API's `protoPackage`.
- Its `go_package` is derived from the generated project's Go module path and
  the API. It cannot be declared in the DSL; an explicit override is a future
  extension.

### 5.3 Operations

Each `rpc` maps to a protobuf service method.

`MethodName` follows Ocean field naming conventions (it begins with a lowercase
letter). The generated protobuf method name is the same name with its first
letter uppercased: `submitOrder` becomes `SubmitOrder`.

`_` denotes no input or no output. Operations are unary unless `stream` is
declared.

#### Request and Response Messages

A protobuf method takes exactly one request message and returns exactly one
response message. Ocean maps declared inputs and outputs to messages as
follows:

| Declaration                                                    | Protobuf message                                                                  |
| -------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| No input: `()` or `(_)`                                        | `google.protobuf.Empty`                                                           |
| One input whose type is a structured datatype                  | That datatype's message. The input name only names the parameter.                 |
| Any other input: scalars, collections, or several inputs       | Generated `<RpcName>Request`, with one field per input in declaration order.      |
| Output `_`                                                     | `google.protobuf.Empty`                                                           |
| Output whose type is a structured datatype                     | That datatype's message.                                                          |
| Any other output: a scalar or a collection                     | Generated `<RpcName>Response` with a single field named `value`.                  |

Service implementations and `@service` `connect` statements see the operation
in its Ocean form: inputs and output are the declared types, regardless of the
wrapper messages used on the wire.

#### Streaming

`stream` before an input marks a client stream; `stream` before the output type
marks a server stream; both mark a bidirectional stream. A streamed input must
be the operation's only input, and the streamed message is that input's type.
A streamed output cannot be `_`.

Unary operations are supported first. The `stream` forms are part of the
syntax, and validation reports them as unsupported until streaming is
implemented.

### 5.4 Type Mapping

Datatypes used in gRPC operations are generated as protobuf messages.

| Ocean type                    | Protobuf type                                                           |
| ----------------------------- | ----------------------------------------------------------------------- |
| `String`                      | `string`                                                                |
| `Int`                         | `int64`                                                                 |
| `Float`                       | `float`                                                                 |
| `Double`                      | `double`                                                                |
| `Boolean`                     | `bool`                                                                  |
| `Timestamp`                   | `google.protobuf.Timestamp`                                             |
| `Date`, `Time`, `DateTime`    | `string` in ISO-8601 form; these carry no timezone, so no `Timestamp`.  |
| `Json`                        | `google.protobuf.Value`                                                 |
| `List<T>`, `Set<T>`           | `repeated T`                                                            |
| `Map<K, V>`                   | `map<K, V>`; `K` must be `String`, `Int`, or `Boolean`.                 |
| `*T` (optional)               | `optional T`                                                            |
| enum                          | `enum`, with a generated zero value `<ENUM>_UNSPECIFIED`.               |
| datatype                      | `message`                                                               |

A `Timestamp` field of a datatype is held by the generated Go struct as
milliseconds since the epoch; on the wire it is a `google.protobuf.Timestamp`.

`Int` maps to `int64` deliberately: it matches the 64-bit `int` that Ocean
generates for Go, and it cannot overflow on the wire. A narrower width is not
selectable.

Fields are numbered from `1` in declaration order. Appending fields is
wire-compatible; reordering, removing, or retyping a field changes the wire
contract. A field's number depends on its position, never on its name, so
renaming a field does not change the wire contract. Explicit field numbers are
a future extension.

#### Naming

- Message and enum names are the Ocean datatype and enum names.
- Field names are generated from Ocean field names in lower snake case
  (`currentOrder` becomes `current_order`). JSON payloads keep the names used
  in the DSL: where protobuf's default JSON name would differ from the Ocean
  name (`userID` would become `userId`), the field carries an explicit
  `json_name`.
- Enum values are generated as `<ENUM>_<VALUE>` in upper snake case
  (`Priority.urgent` becomes `PRIORITY_URGENT`), because protobuf scopes enum
  values to the package. Every enum also has the `unknown` value Ocean adds
  (see `dsl.datatype`), so it is numbered like any other value.

### 5.5 Errors

gRPC reports failures as status codes. The generated server maps an Ocean error
returned by an operation to a gRPC status. The error's `httpCode` is only the
input to that mapping; no HTTP response is produced:

| `httpCode` | gRPC status          |
| ---------- | -------------------- |
| 400        | `INVALID_ARGUMENT`   |
| 401        | `UNAUTHENTICATED`    |
| 403        | `PERMISSION_DENIED`  |
| 404        | `NOT_FOUND`          |
| 409        | `ALREADY_EXISTS`     |
| 429        | `RESOURCE_EXHAUSTED` |
| 501        | `UNIMPLEMENTED`      |
| 503        | `UNAVAILABLE`        |
| 504        | `DEADLINE_EXCEEDED`  |
| other      | `INTERNAL`           |

The error's code and message are carried in the status message. Unlike SOAP,
a gRPC operation does not declare typed faults.

### 5.6 Composition

gRPC APIs compose with `include` exactly as other styles do (see 2.2), and every
included API must also use `style:grpc`. The composing API owns its own
`protoPackage`, service, and port: the included operations become methods of the
composing API's protobuf service, and its `.proto` contains the complete
effective operation set. A `@service` gateway routes the included operations to
their providers as for any other style.

#### gRPC Rules

- Method names must be unique in the effective gRPC API.
- `protoPackage` must be a valid protobuf package name.
- Input and output types must resolve.
- `Map` key types are limited to `String`, `Int`, and `Boolean`.
- `engine` must be a supported gRPC engine; initially, `connect` is the only
  value.
- A streamed input must be the operation's only input, and a streamed output
  cannot be `_`.
- Streaming operations are rejected by validation until streaming is supported.
- gRPC API composition uses `include`; every included API must also use
  `style:grpc`.
- Metadata bindings such as `@from-context`, deadlines, explicit field numbers,
  and typed error details are future explicit extensions; they are not
  inferred from operation inputs.

---

## 6. GraphQL

An API may use the GraphQL style.

### 6.1 Syntax

```ocean-dsl
<ApiName> style:graphql
    query <name>(<params>) : <ReturnType>
    mutation <name>(<params>) : <ReturnType>
```

Example:

```ocean-dsl
ProductAPI style:graphql
    query getProduct(id: String) : Product
    query listProducts(category: String) : List<Product>
    mutation createProduct(input: ProductInput) : Product
    mutation updatePrice(id: String, price: Float) : Product
```

### 6.2 Operations

| Operation  | Description                                          |
| ---------- | ---------------------------------------------------- |
| `query`    | Defines a GraphQL read operation.                    |
| `mutation` | Defines a GraphQL write or state-changing operation. |

#### GraphQL Rules

- Every GraphQL query or mutation must define a return type.
- Parameters are named and typed.
- Operation and parameter names follow Ocean field naming conventions.
- Parameter and output type names follow Ocean datatype naming conventions.
- GraphQL operations cannot use `_` as the output type.
- `List<Type>` represents list results.
- Input validation and resolver contracts are derived from parameter and output datatypes.

---

## 7. WebSocket

An API may use the WebSocket style for streaming or bidirectional communication.

### 7.1 Syntax

```ocean-dsl
<ApiName> style:websocket
    ws <path> <MethodName>(<inputName>:<InputType>, ...) : <OutputType> [with <config>]
```

Example:

```ocean-dsl
ChatAPI style:websocket
    ws /chat/connect connectToChat(ChatInit) : ChatEvent
    ws /notifications listenForAlerts(_) : AlertEvent with mode:subscribe
    ws /orders/live getOrderUpdates(OrderFilter) : OrderUpdate
```

### 7.2 Elements

| Element      | Description                                                                                |
| ------------ | ------------------------------------------------------------------------------------------ |
| `ws`         | Declares a WebSocket or streaming endpoint.                                                |
| `path`       | Lowercase WebSocket endpoint path; supports placeholders such as `{id}` and `{id:String}`. |
| `MethodName` | Unique operation identifier.                                                               |
| `InputType`  | Datatype used when initiating or interacting with the connection. `_` indicates no input.  |
| `OutputType` | Datatype streamed from the server.                                                         |
| `config`     | Optional comma-separated `key:value` WebSocket configuration.                              |

#### Suggested Modes

| Mode        | Description                                                       |
| ----------- | ----------------------------------------------------------------- |
| `subscribe` | Server sends events after the initial connection or subscription. |
| `rpc`       | Request-response interaction over WebSocket.                      |
| `chat`      | Full-duplex communication.                                        |
| `log`       | Continuous stream of logs or updates.                             |

#### WebSocket Rules

- Input may be `_` when no initial message is required.
- Output type must be provided.
- Stream behavior may be configured using `with mode:<value>` where supported.

---

## 8. API Operation References

API operations are referenced using their fully qualified name:

```text
<ApiName>.<MethodName>
```

Examples:

```text
OrderAPI.createOrder
UserAPI.getUser
ChatAPI.connectToChat
```

Fully qualified operation references allow API methods to be referenced by other DSL constructs such as:

- `@fsm`;
- `@component`;
- `@integration`;
- `@service`.

---

## 9. Imports

Only datatypes may be imported into an `@api` file from the Ocean Repository.

Imported datatypes may be used as:

- request types;
- response types;
- operation parameters.

Example:

```ocean-dsl
@api

@import datatype O.domain.payment.CardInfo@1.2.0 as CardInfo

PaymentAPI
    post /cards addCard(cardInfo: CardInfo) : Response
```

The alias is required and is used in API method signatures. Other definition
kinds cannot be introduced with `@import` in an `@api` file.

Reusable definitions may also be made available through the supported Ocean include mechanism.

---

## 10. Rules and Constraints

The following general rules apply:

- An API file starts with `@api`.
- A file may define one or more APIs.
- REST is the default style.
- API operation syntax depends on the selected style.
- Operations may be written on one line or with input parameters on separate
  indented lines.
- `include` composes the effective operations of existing APIs. Included
  APIs may themselves include APIs; every included API must use the same style,
  and inclusion cycles are invalid.
- Operation names must be unique in an API's effective operation set. A local
  compatible operation may override an inherited operation with the same name.
- Input and output types use Ocean datatypes.
- `_` represents no input or no output where supported by the API style.
- API operations are referenced using `<ApiName>.<MethodName>`.
- Datatypes may be imported from the Ocean Repository.
- Style-specific rules apply in addition to these general rules.

---

## 11. Complete REST Example

The following is a complete `@api` file
(`ocean-examples/0008-invoice-approval-system/40-api.ocn`):

```ocean-dsl
@api

InvoiceApi style:rest
    engine = gin
    configType = ApiConfig
    version = 1.0.0
    generateSwagger = true



    # -------------------------
    # Invoice
    # -------------------------

    get  /invoice/{id}               getInvoice(id:String)              : *Invoice
    get  /invoices                   getAllInvoices()                   : List<*InvoiceSummary>
    get  /invoices/{offset}/{limit}  getInvoices(offset:Int, limit:Int) : List<*Invoice>
    #get  /invoices/pending/{userId}  getPendingInvoices(userId:String)  : List<*Invoice>

    post   /invoice/submit        submitInvoice(req:*SubmitInvoiceReq)  : *Invoice
    post   /invoice/update        updateInvoice(item: *Invoice)         : *Invoice
    post   /invoice/{id}/approve  approveInvoice(req:ApproveInvoiceReq) : *Invoice
    post   /invoice/{id}/reject   rejectInvoice(req:RejectInvoiceReq)   : Invoice
    delete /invoice/{id}/delete   deleteInvoice(id:String)              : _

    get  /invoice/find-by-status/{status}             findInvoiceByStatus(status: InvoiceStatus)    : List<*Invoice>
    get  /invoice/find-by-customer/{customerId}       findInvoiceByCustomer(customerId: String)     : List<*Invoice>
    get  /invoice/find-by-created-by/{createdById}    findInvoiceByCreatedBy(createdById: String)   : List<*Invoice>
    get  /invoice/find-by-approved-by/{approvedById}  findInvoiceByApprovedBy(approvedById: String) : List<*Invoice>
    get  /invoice/find-by-status-and-customer/{status}/{id}  findInvoiceByStatusAndCustomer(status: InvoiceStatus, id : String) : List<*Invoice>



    # -------------------------
    # User
    # -------------------------

    get /user/{id}                getUser(id:String)              : *User
    get /users                    getAllUsers()                   : List<User>
    get /find-user-by-name/{name} findUserByName(name:String)     : *User
    get /find-user-by-role/{role} findUserByRole(role:String)     : List<*User>
    get /users/{offset}/{limit}   getUsers(offset:Int, limit:Int) : List<*User>

    post   /user/create       createUser(user:User) : *User
    post   /user/update       updateUser(user:User) : *User
    delete /user/{id}         deleteUser(id:String) : _



    # -------------------------
    # Customer
    # -------------------------

    get /customer/{id}                getCustomer(id:String)              : *Customer
    get /customers                    getAllCustomers()                   : List<Customer>
    get /find-customer-by-name/{name} findCustomerByName(name:String)     : *Customer
    get /find-customer-by-department/{department} findCustomerByDepartment(department:String)     : List<*Customer>
    get /customers/{offset}/{limit}   getCustomers(offset:Int, limit:Int) : List<*Customer>

    post   /customer/create       createCustomer(customer:Customer) : *Customer
    post   /customer/update       updateCustomer(customer:Customer) : *Customer
    delete /customer/{id}         deleteCustomer(id:String)         : _
```

This example demonstrates:

- explicit REST style;
- API-level attributes (`engine`, `configType`, `version`, `generateSwagger`);
- several resources (Invoice, User, Customer) grouped in one API with comment banners;
- a commented-out endpoint;
- path parameters and typed parameters (`offset:Int`, `status:InvoiceStatus`);
- empty parameter lists (`getAllInvoices()`) for no input;
- pointer request and response types (`*Invoice`, `req:*SubmitInvoiceReq`, `List<*Invoice>`);
- list responses;
- `_` as an empty output type.

---

## 12. Related Knowledge

The `@api` DSL is related to:

- `dsl.datatype` — defines datatypes used by API inputs, outputs, and parameters.
- `dsl.context` — defines context data that may be supplied implicitly through `@from-context`.
- `dsl.import` — defines the mechanism for importing selected datatype definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
