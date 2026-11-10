# Expression DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content 8776e975). Canonical source: https://atlas.ocean-lab.ai/dsl/expression/

**Contents**

- [1. Overview](#1-overview)
- [2. Explicit Expressions](#2-explicit-expressions)
- [3. Expression Elements](#3-expression-elements)
- [4. Inputs](#4-inputs)
- [5. Outputs](#5-outputs)
- [6. Logic Expressions](#6-logic-expressions)
- [7. External Expressions](#7-external-expressions)
- [8. Built-in Functions](#8-built-in-functions)
- [9. Boolean Expressions](#9-boolean-expressions)
- [10. Mapping and Projection](#10-mapping-and-projection)
- [11. Computation](#11-computation)
- [12. Expression Language](#12-expression-language)
- [13. Literals](#13-literals)
- [14. Variables and Field Access](#14-variables-and-field-access)
- [15. Boolean Operations](#15-boolean-operations)
- [16. Comparisons](#16-comparisons)
- [17. Arithmetic](#17-arithmetic)
- [18. Conditional Expressions](#18-conditional-expressions)
- [19. Enum Values](#19-enum-values)
- [20. Expression Composition](#20-expression-composition)
- [21. Implicit Expressions](#21-implicit-expressions)
- [22. Explicit vs Implicit Expressions](#22-explicit-vs-implicit-expressions)
- [23. Use in FSMs](#23-use-in-fsms)
- [24. Use in Components](#24-use-in-components)
- [25. Use in Services](#25-use-in-services)
- [26. Imports](#26-imports)
- [27. Complete Example](#27-complete-example)
- [28. Rules and Constraints](#28-rules-and-constraints)
- [29. Related Knowledge](#29-related-knowledge)

Defines the Ocean expression model for stateless, side-effect-free reusable logic, including explicit named expressions, implicit inline expressions, typed inputs and outputs, built-in functions, logic blocks, and external implementations.

Source: https://atlas.ocean-lab.ai/dsl/expression/

Status: Accepted · Authority: Normative

## 1. Overview

Expressions define stateless logic that operates on well-typed inputs and produces well-typed outputs.

Expressions may represent:

- conditions;
- validations;
- mappings and transformations;
- calculations;
- derived values;
- routing decisions;
- reusable business logic.

Expressions are designed to be:

- stateless;
- side-effect-free;
- deterministic where possible;
- reusable;
- composable;
- independent from persistence and transport technologies.

Expressions do not directly:

- mutate persistent state;
- access databases;
- perform external I/O;
- own runtime state.

Ocean supports two forms of expressions:

1. **Explicit Expressions** — named reusable expressions defined under `@expression`.
2. **Implicit Expressions** — inline expression logic embedded directly where it is used.

---

## 2. Explicit Expressions

Explicit expressions are named reusable logic units.

They are defined under:

```ocean-dsl
@expression
```

A file may contain multiple explicit expression definitions.

General syntax:

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

Alternatively, an expression may delegate its implementation to external code:

```ocean-dsl
<ExpressionName>
    context : <name1:DataType1> & ...
    input  : <name1:DataType1> & ...
    output : <name1:DataType1> & ...

    external
        source: "<file-name>"
        method: "<method-name>"
```

Each explicit expression uses either:

- `logic`; or
- `external`.

---

## 3. Expression Elements

| Element          | Description                                                   |
| ---------------- | ------------------------------------------------------------- |
| `@expression`    | Starts an expression definition file.                         |
| `ExpressionName` | Unique name of the reusable expression.                       |
| `authorize`      | Optional declarative authorization precondition.              |
| `context`        | Optional typed values supplied from service context.          |
| `input`          | Typed input values available to the expression.               |
| `output`         | Typed values produced by the expression.                      |
| `include`        | Makes built-in functions available to the expression.         |
| `logic`          | Defines expression logic using the Ocean expression language. |
| `external`       | Delegates implementation to external source code.             |

Expression names follow the same naming convention as Ocean datatype names.

Examples:

```text
IsHighPriority
MapToSummary
CalculateDiscount
ComputeTax
```

Input and output names follow Ocean field naming conventions.

---

### 3.1 Context Bindings

An explicit expression may declare optional named and typed service-context
bindings.

Syntax:

```ocean-dsl
context : <name>:<Datatype>
```

Multiple context bindings are separated using `&`:

```ocean-dsl
context : requestId:String & actor:UserIdentity
```

Context bindings follow the same name-and-datatype rules as inputs and outputs.
They expose only the context values explicitly connected by the surrounding
service. Expressions treat those values as read-only and must remain stateless
and side-effect-free.

Some expressions instead receive context as an ordinary typed input — for
example `input: ctx:Context & req:SignUpInput` — rather than through a separate
`context :` line.

### 3.2 Authorization Preconditions

The policy-rule grammar and evaluation semantics are canonical in
[`dsl.authorization`](authorization.md). This section defines only how an
expression attaches that shared policy to its trusted context actor.

An explicit expression may declare an authorization precondition. It defines
the policy that must pass before the expression's business logic or external
implementation can run.

Syntax:

```ocean-dsl
<ExpressionName>
    authorize <actor-context-name>
        <policy-rule>
        <policy-rule>

    input  : ctx:Context & ...
    output : ...
    logic
        ...
```

`<actor-context-name>` identifies a trusted actor supplied by the surrounding
runtime. The initial standard actor is `actor`: an `OceanActor` established by
a gateway after external authentication and propagated in the signed Ocean
request context. It is not a caller-supplied `User` value.

An authorization declaration requires an input named `ctx` of type `Context`.
It does not make an `actor` variable available in the `logic` block. If the
business logic also needs a local domain projection of the actor, declare and
connect that value explicitly through the normal service-context binding
mechanism.

At an HTTP boundary, an endpoint that reaches an authorized expression must
also declare the actor as required request context:

```ocean-dsl
delete /users/{id} deleteUser(id:String) : _ @from-context:actor
```

The transport declaration requires an authenticated actor; `authorize`
evaluates the business authorization policy. The two concerns are deliberately
separate so the same protected expression behaves consistently when it is
called through another supported transport.

#### 3.2.1 Policy Structure

The direct rules in an `authorize` block are combined with **AND**. `anyOf`
and `allOf` create nested policy groups. A group must contain at least one
rule.

```ocean-dsl
UpdateUser
    authorize actor
        authenticated
        anyOf
            subject equals user.id
            role "platform-admin", "admin"

    input  : ctx:Context & user:User
    output : result:User
    logic
        result = User.updateUser(user)
```

The example permits an authenticated actor to update their own user record, or
an actor with either listed role. The values on a single leaf rule are combined
with **OR**; separate rules remain **AND**.

The supported initial leaf rules are:

| Rule | Meaning |
| --- | --- |
| `authenticated` | A verified actor must be present. |
| `scope "a", "b"` | The actor must hold at least one listed scope. |
| `role "a", "b"` | The actor must hold at least one listed role. |
| `subject equals ref1, ref2` | The actor subject ID must equal at least one String expression-input reference. |
| `tenant equals ref1, ref2` | The actor tenant ID must equal at least one String expression-input reference. |
| `organization contains ref1, ref2` | The actor organization IDs must contain at least one String expression-input reference. |
| `anyOf` | At least one nested rule or group must pass. |
| `allOf` | Every nested rule or group must pass. |

`ref` is an expression input reference, optionally with field access, such as
`id`, `user.id`, or `account.organizationId`. It is evaluated before the
expression logic runs and must resolve to `String`.

Examples:

```ocean-dsl
# Any authenticated actor.
GetCurrentSession
    authorize actor
        authenticated
    input  : ctx:Context
    output : result:SessionInfo
    logic
        ...

# One of the listed administration roles is required.
CreateUser
    authorize actor
        authenticated
        role "platform-admin", "admin"
    input  : ctx:Context & user:User
    output : result:User
    logic
        ...

# The caller must belong to the target organization and have one of the
# permitted capabilities.
UpdateOrganizationBankAccount
    authorize actor
        authenticated
        organization contains account.organizationId
        scope "bank-accounts:write", "bank-accounts:manage"
    input  : ctx:Context & account:OrganizationBankAccount
    output : result:OrganizationBankAccount
    logic
        ...
```

Authorization is deny-by-default. A missing or invalid actor produces an
`Unauthorized` outcome (HTTP `401` at an HTTP boundary); a verified actor that
does not satisfy the policy produces a `Forbidden` outcome (HTTP `403`). The
policy is evaluated before all expression logic and before delegating to an
`external` implementation.

---

## 4. Inputs

Expression inputs are named and typed.

Syntax:

```ocean-dsl
input : <name>:<Datatype>
```

Example:

```ocean-dsl
input : order:Order
```

Multiple inputs are separated using `&`.

Example:

```ocean-dsl
input : price:Float & quantity:Int
```

An expression that takes no input declares `_`:

```ocean-dsl
input : _
```

Each input datatype must resolve to a valid Ocean datatype.

Inputs are read-only from the perspective of the expression.

### 4.1 Function-Typed Inputs

An input may be a function type, written `Func<InputType, OutputType>`. It is
supplied by the caller and invoked from `logic` like a call:

```ocean-dsl
RequestHandler
    input : a:Int & b:Int & requester:Func<MathRequest, MathResponse>
    output : result:Int
    logic
        var req MathRequest
        req.num1 = a
        req.num2 = b
        var res MathResponse
        res = requester(req)
        result = res.resValue
```

This is used, for example, to hand an expression a broker request-response
call without coupling it to the transport.

---

## 5. Outputs

Expression outputs are named and typed.

Syntax:

```ocean-dsl
output : <name>:<Datatype>
```

Example:

```ocean-dsl
output : result:Boolean
```

Multiple outputs are separated using `&`.

Example:

```ocean-dsl
output : amount:Float & eligible:Boolean
```

Expressions produce values through their declared outputs.

---

## 6. Logic Expressions

A `logic` block defines an expression using the Ocean expression language.

Syntax:

```ocean-dsl
logic
    <expression logic>
```

Example:

```ocean-dsl
IsHighPriority
    input  : order:Order
    output : result:Boolean

    logic
        result = order.priority == "HIGH"
```

Logic may:

- read input values;
- access nested fields;
- declare intermediate values;
- perform calculations;
- evaluate conditions;
- invoke included built-in functions;
- invoke other available expressions;
- assign declared outputs.

Logic must remain stateless and side-effect-free.

### 6.1 Statement Forms

A `logic` block is a sequence of statements, one per line.

**Assignment** — to an output, an output field, or a local variable:

```ocean-dsl
r = a + b
r.result = a + b
r.isValid = true
result = res
```

A whole value may be assigned when the types match (`result = res`), or fields
may be assigned individually (`r.result = ...`).

**Local variable declaration** — `var <name> <Type>` introduces a mutable local:

```ocean-dsl
var res MathResponse
res.resValue = req.num1 + req.num2
result = res
```

**Block conditional** — `if <condition> then` … optional `else` … `end`. Branch
bodies contain statements, not a single value:

```ocean-dsl
if b == 0 then
    r.result = 0
    r.isValid = false
else
    r.result = a / b
    r.isValid = true
end
```

```ocean-dsl
if req.reqType == RequestType.time then
    res.resContent = GetTimeString()
end
```

The inline `if <cond> then <value> else <value>` form in Section 18 is an
expression that produces a value; the block form above is control flow around
statements.

**Error** — `error "<message>"` (or `error <ErrorName>` for a declared error)
stops evaluation and raises the error to the caller:

```ocean-dsl
if movementItem.quantity <= 0 then
    error "quantity must be a positive number"
end
```

---

## 7. External Expressions

An expression may delegate its implementation to external code.

Syntax:

```ocean-dsl
external
    source: "<file-name>"
    method: "<method-name>"
```

Example:

```ocean-dsl
@expression

ComputeTax
    input  : invoice:Invoice
    output : taxAmount:Float

    external
        source: "tax-rules.go"
        method: "ComputeTax"
```

External expressions are useful when:

- logic is too complex for inline expression syntax;
- existing implementation code should be reused;
- specialized algorithms are required;
- generated code needs to delegate to user-provided implementation.

The external implementation must respect the same expression contract:

- declared inputs;
- declared outputs;
- stateless behavior;
- no unintended side effects.

---

## 8. Built-in Functions

Expressions may include built-in Ocean functions.

Syntax:

```ocean-dsl
include: <name>@<version>
```

Examples:

```ocean-dsl
include: normalizeString@1.0.0
```

```ocean-dsl
include: now
```

```ocean-dsl
include: uuid@latest
```

Multiple `include:` lines are allowed.

Each `include:` declaration is defined on one line.

Example:

```ocean-dsl
include: normalizeString@1.0.0
include: now
include: uuid@latest
```

A built-in reference follows the conceptual format:

```text
<name>@<version>
```

If no version is specified, the latest applicable version is used.

Built-in functions are distinct from `@import` and the Ocean Repository include mechanism.

`include:` inside an expression makes expression-language built-ins available to the expression logic.

---

## 9. Boolean Expressions

A Boolean Expression evaluates a condition and produces a Boolean result.

Example:

```ocean-dsl
@expression

IsHighPriority
    input  : order:Order
    output : result:Boolean

    include: normalizeString@1.0.0
    include: now
    include: uuid@latest

    logic
        result = order.priority == "HIGH"
```

Typical uses include:

- FSM guards;
- conditional routing;
- validation;
- filtering;
- runtime decisions.

---

## 10. Mapping and Projection

Expressions may transform one datatype or structure into another.

Example:

```ocean-dsl
@expression

MapToSummary
    input  : order:Order
    output : summary:OrderSummary

    logic
        summary.id = order.id
        summary.total = order.amount * order.quantity
        summary.status = order.status
```

Mappings may:

- copy fields;
- rename values;
- calculate derived fields;
- combine multiple inputs;
- construct output structures.

This makes expressions suitable for explicit transformation logic between typed DSL elements.

---

## 11. Computation

Expressions may perform calculations.

Example:

```ocean-dsl
@expression

CalculateDiscount
    input  : item:Item
    output : discount:Float

    logic
        discount = if item.quantity > 10 then 0.2 else 0.0
```

Computations may produce:

- numeric values;
- strings;
- Boolean results;
- compound datatypes;
- multiple typed outputs.

---

## 12. Expression Language

The Ocean expression language supports common constructs for declarative logic.

| Feature        | Syntax / Example                              | Description                             |
| -------------- | --------------------------------------------- | --------------------------------------- |
| Literals       | `"Paid"`, `true`, `42`                        | Constant values.                        |
| Variables      | `status`, `amount`, `input.name`              | Access variables and fields.            |
| Element access | `items[index]`, `values[key]`                 | Access a `List` element or `Map` value. |
| Conditionals   | `if x > 0 then "OK" else "Fail"`              | Conditional evaluation.                 |
| Ternary        | `x > 0 ? "pos" : "neg"`                       | Short conditional form.                 |
| Booleans       | `x and y`, `not x`, `x or y`                  | Logical operations.                     |
| Comparisons    | `==`, `!=`, `>`, `<`, `>=`, `<=`              | Value comparison.                       |
| Math           | `a + b`, `price * quantity`, `total % 10`     | Arithmetic operations.                  |
| Function calls | `map(id, status)`, `concat(a, b)`             | Invoke functions or expressions.        |
| Let bindings   | `let total = a + b in total * 2`              | Intermediate values.                    |
| Match / Case   | `match status { "A" -> x, "B" -> y, _ -> z }` | Exact-match branching where supported.  |
| Chaining       | `validate(input).transform("mode")`           | Compose operations.                     |

Some expression-language constructs may evolve as the parser and generators are extended.

The canonical behavior of each construct should be updated here together with its implementation.

---

## 13. Literals

Expressions may use literal values.

Examples:

```ocean-dsl
"Paid"
```

```ocean-dsl
true
```

```ocean-dsl
42
```

```ocean-dsl
0.2
```

Literal values must be compatible with the datatype expected by the surrounding expression.

---

## 14. Variables and Field Access

Inputs and intermediate values may be referenced by name.

Example:

```ocean-dsl
order
```

Fields are accessed using dot notation:

```ocean-dsl
order.priority
```

Nested access may follow the same model:

```ocean-dsl
order.customer.loyaltyLevel
```

Expressions may also access locally defined intermediate values where supported by the expression language.

### 14.1 List and Map Element Access

Square brackets read an element from a `List` or a value from a `Map`:

```ocean-dsl
result = items[index]
result = values[key]
```

The bracket expression is evaluated before access. A list index must evaluate
to `Int`; a map key must match the key type declared by the map.

Validation and evaluation rules:

- A negative or out-of-range list index produces an evaluation error.
- A missing map key produces an evaluation error.
- Access on a value that is not a `List` or `Map` is rejected during expression
  validation.
- A non-`Int` list index is rejected during expression validation.
- A map key whose type does not match the declared key type is rejected during
  expression validation.

The current expression grammar supports one level of element access when it is
the complete right-hand side of an assignment.

Supported:

```ocean-dsl
result = items[index]
result = values[key]
```

Not yet supported:

```ocean-dsl
result = matrix[row][column]
result = items[index] + 1
result = items[indexes[position]]
```

Chained access, access combined with another expression, and nested access
inside the key expression require future grammar support.

---

## 15. Boolean Operations

Supported Boolean operations include:

```ocean-dsl
a and b
```

```ocean-dsl
a or b
```

```ocean-dsl
not a
```

Example:

```ocean-dsl
order.total > 500 and customer.active == true
```

---

## 16. Comparisons

Expressions may compare compatible values using:

```text
==
!=
>
<
>=
<=
```

Example:

```ocean-dsl
order.priority == "HIGH"
```

Example:

```ocean-dsl
item.quantity > 10
```

Comparison operands must be type-compatible.

---

## 17. Arithmetic

Expressions may perform arithmetic using operators such as:

```text
+
-
*
/
%
```

Example:

```ocean-dsl
summary.total = order.amount * order.quantity
```

Arithmetic operations require compatible numeric datatypes.

---

## 18. Conditional Expressions

Conditional logic may be expressed inline.

Example:

```ocean-dsl
discount = if item.quantity > 10 then 0.2 else 0.0
```

A short ternary form may also be supported:

```ocean-dsl
result = x > 0 ? "pos" : "neg"
```

Conditional branches must produce compatible result types.

---

## 19. Enum Values

Enum values are accessed through their enum type.

Syntax:

```ocean-dsl
<EnumType>.<value>
```

Example:

```ocean-dsl
TaskLifecycle.inProgress
```

This makes enum references explicit and avoids ambiguity between enum values and ordinary variables.

---

## 20. Expression Composition

Explicit expressions may invoke other expressions.

Example:

```ocean-dsl
@expression

IsPreferredCustomer
    input  : customer:Customer
    output : result:Boolean

    logic
        result = customer.loyaltyLevel == "GOLD"

IsDiscountEligible
    input  : order:Order
    output : result:Boolean

    logic
        result = order.total > 500 and IsPreferredCustomer(order.customer).result
```

This allows complex logic to be composed from smaller reusable expressions.

Expression composition must remain stateless and side-effect-free.

---

## 21. Implicit Expressions

Implicit Expressions are lightweight inline expressions defined directly where they are used.

Unlike explicit expressions, implicit expressions:

- have no reusable name;
- are local to their point of use;
- cannot be referenced elsewhere;
- use the same underlying expression-language concepts.

Typical uses include:

- guards;
- conditions;
- routing decisions;
- one-off transformations.

Conceptual examples include:

```ocean-dsl
if input.amount > 100
```

```ocean-dsl
connect input -> output if input.status == "Ready"
```

```ocean-dsl
value = input.amount * 2
```

For complex or reusable logic, an explicit `@expression` should be preferred.

The exact embedding syntax depends on the DSL construct in which the implicit expression is used.

---

## 22. Explicit vs Implicit Expressions

| Characteristic              | Explicit                | Implicit                         |
| --------------------------- | ----------------------- | -------------------------------- |
| Named                       | Yes                     | No                               |
| Reusable                    | Yes                     | No                               |
| Defined under `@expression` | Yes                     | No                               |
| Used inline                 | Referenced by name      | Yes                              |
| Typed contract              | Explicit input/output   | Derived from surrounding context |
| Best for                    | Shared or complex logic | Small local logic                |

Both forms use the same conceptual expression language.

---

## 23. Use in FSMs

FSMs may use explicit expressions as reusable logic.

Example:

```ocean-dsl
@expression

IsHighValueOrder
    input  : order:Order
    output : result:Boolean

    logic
        result = order.total > 1000
```

An FSM may then use the expression according to FSM expression invocation syntax.

Expressions are useful in FSMs for:

- guards;
- calculations;
- validation;
- decision logic;
- transformations.

Because expressions are stateless, state transitions remain the responsibility of the FSM.

---

## 24. Use in Components

Components may use expressions for:

- transformation between connected ports;
- mapping;
- conditional routing;
- reusable calculations.

Conceptually:

```ocean-dsl
use expression MyMapper
```

A mapper expression may transform a source datatype into the datatype required by a target connection.

The exact component mapper syntax should be finalized together with `@component` implementation.

---

## 25. Use in Services

Services may use expressions as stateless logic units within service orchestration.

Expressions may provide:

- validation;
- transformation;
- calculations;
- decision logic;
- reusable business rules.

The service controls orchestration and side effects, while expressions remain pure logic units.

The exact service invocation and connection semantics are defined by `@service`.

---

## 26. Imports

An `@expression` file may import reusable definitions from the Ocean Repository.

Supported imported item types include:

- `datatype`;
- `expression`.

Example:

```ocean-dsl
@expression

@import datatype   O.common.math.Vector@2.0.0     as Vector
@import expression O.common.logic.DotProduct@2.1  as Dot

ComputeNorm
    input  : v:Vector
    output : norm:Float

    logic
        # use Dot(...) as part of logic
```

Imported datatypes may be used in:

```ocean-dsl
input:
```

and:

```ocean-dsl
output:
```

Imported expressions may be invoked from a `logic` block using their alias.

Every imported datatype or expression requires an `as <alias>` clause and must
be referenced through that alias.

Reusable definitions may also be made available through the supported Ocean include mechanism.

---

## 27. Complete Example

A complete `@expression` file
(`ocean-examples/0004-answer-sync/40-ans-expression.ocn`):

```ocean-dsl
@expression

HandleDateTimeRequest
    input:  req:TimeDateRequest
    output: result:TimeDateResponse
    logic
        var res TimeDateResponse
        res.resType = req.reqType
        if req.reqType == RequestType.time then
            res.resContent = GetTimeString()
        end
        if req.reqType == RequestType.date then
            res.resContent = GetDateString()
        end
        result = res

GetTimeString
    input: _
    output: result:String
    logic
        nowTime = nowTimeString()
        result = nowTime

GetDateString
    input: _
    output: result:String
    logic
        nowDate = nowDateString()
        result = nowDate

GetDayString
    input: _
    output: result:String
    logic
        today = today()
        result = today

GetDayTip
    input: _
    output: result:String
    logic
        dayTip = tipOfTheDay()
        result = dayTip

PrettyTime
    input: i:String
    output: o:String
    logic
        o = StringConcat2("⏰ ", i)

PrettyDate
    input: i:String
    output: o:String
    logic
        o = StringConcat2("📅 ", i)

PrettyTip
    input: i:String
    output: o:String
    logic
        o = StringConcat2("💡 ", i)

TimeAnswerToString
    input: i:TimeAnswer
    output: o:String
    logic
        o = "Today is "
        o = StringConcat2(o, i.day)
        o = StringConcat2(o, "! ")
        o = StringConcat2(o, i.date)
        o = StringConcat2(o, " ")
        o = StringConcat2(o, i.time)
        o = StringConcat2(o, " ")
        o = StringConcat2(o, i.tip)
```

This example demonstrates:

- multiple expressions in one file;
- `input: _` for an expression that takes no input;
- `var` local declarations and whole-value output assignment (`result = res`);
- block `if <cond> then … end` control flow;
- enum comparison (`req.reqType == RequestType.time`);
- calls to other expressions (`GetTimeString()`) and to built-in helpers
  (`nowTimeString()`, `today()`, `tipOfTheDay()`, `StringConcat2(...)`);
- typed inputs and outputs, including user-defined datatypes.

The example does not use `external`, `include:`, or expression imports; see
Sections 7, 8, and 26 for those. Note that helper functions such as
`nowTimeString()` and `StringConcat2(...)` are called directly here without an
`include:` line.

---

## 28. Rules and Constraints

The following rules apply:

- An explicit expression file starts with `@expression`.
- A file may define multiple expressions.
- Expression names follow Ocean datatype naming conventions.
- Optional context bindings are named and typed, separated using `&`, and
  exposed read-only to expression logic.
- `authorize <actor-context-name>` is optional and declares a deny-by-default
  authorization precondition for the expression.
- An authorized expression requires an input named `ctx` of type `Context`.
- Direct authorization rules are combined with AND; comma-separated values on
  one `role`, `scope`, `subject`, `tenant`, or `organization` rule are combined
  with OR.
- `anyOf` and `allOf` may nest policy rules; each policy group must contain at
  least one child rule.
- `subject`, `tenant`, and `organization` references must resolve to `String`
  expression inputs or fields and are evaluated before expression logic.
- At HTTP boundaries, the endpoint leading to an authorized expression must
  declare the same actor with `@from-context:<actor-context-name>`.
- Inputs and outputs are named and typed.
- Multiple inputs and outputs are separated using `&`.
- An expression that takes no input declares `input : _`.
- An input may be a function type `Func<InputType, OutputType>`, invoked from `logic` like a call.
- `logic` is a sequence of statements: assignments, `var <name> <Type>` declarations, block `if <cond> then … [else …] end` control flow, and `error "<message>"` / `error <ErrorName>`.
- Expressions are stateless.
- Expressions are side-effect-free.
- Expressions do not directly access databases or external I/O.
- An explicit expression uses either a `logic` block or an `external` block.
- `logic` contains Ocean expression-language logic.
- `external` delegates implementation to external code.
- External expressions require `source:` and `method:`.
- Multiple `include:` lines are allowed.
- Each `include:` declaration is defined on one line.
- Explicit expressions may compose other expressions.
- Implicit expressions are local and non-reusable.
- Enum values are referenced through their enum type.
- `List` and `Map` element access follows the validation and current grammar
  limitations defined in Section 14.1.
- Imported datatypes and expressions require explicit aliases.
- Imported datatypes and expressions may be used through their aliases.

---

## 29. Related Knowledge

The `@expression` DSL is related to:

- `dsl.datatype` — defines the type system used by expression inputs, outputs, variables, and values.
- `dsl.fsm` — uses expressions for reusable conditions, calculations, validation, and transition logic.
- `dsl.component` — may use expressions for mapping, transformation, and routing between component elements.
- `dsl.service` — uses expressions as stateless logic within service orchestration.
- `dsl.import` — defines the mechanism for importing selected datatype and expression definitions from the Ocean Repository.
- `dsl.include` — defines the mechanism for including reusable definitions from the Ocean Repository.

These semantic relationships are declared in the document metadata.
