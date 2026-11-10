# Authorization Policy DSL Reference

> Generated from Ocean-Atlas v0.1.0 (content baec097c). Canonical source: https://atlas.ocean-lab.ai/dsl/authorization/

**Contents**

- [1. Purpose](#1-purpose)
- [2. Policy Rules](#2-policy-rules)
- [3. Expression Authorization](#3-expression-authorization)
- [4. Connection Authorization](#4-connection-authorization)
- [5. Trusted Context](#5-trusted-context)
- [6. Related Knowledge](#6-related-knowledge)

Defines declarative authorization policies (authenticated, role, scope, subject, tenant, organization, anyOf, allOf) and how expressions and connections attach them.

Source: https://atlas.ocean-lab.ai/dsl/authorization/

Status: Approved · Authority: Normative

## 1. Purpose

Ocean authorization policies are declarative preconditions evaluated before a
protected operation runs. They are distinct from authentication: authentication
establishes a trusted actor; authorization decides whether that actor may
perform a particular operation.

Policies are deny-by-default. A missing or invalid trusted actor produces an
unauthorized outcome (`401` at an HTTP boundary). A verified actor that does
not satisfy a policy produces a forbidden outcome (`403`).

## 2. Policy Rules

The direct rules in a policy block are combined with **AND**. `anyOf` and
`allOf` create nested groups. A group must contain at least one rule.

| Rule | Meaning |
| --- | --- |
| `authenticated` | A verified actor must be present. |
| `role "a", "b"` | The actor must hold at least one listed role. |
| `scope "a", "b"` | The actor must hold at least one listed scope. |
| `subject equals ref1, ref2` | The actor subject ID must equal one listed String reference. |
| `tenant equals ref1, ref2` | The actor tenant ID must equal one listed String reference. |
| `organization contains ref1, ref2` | The actor organization IDs must contain one listed String reference. |
| `anyOf` | At least one nested rule or group must pass. |
| `allOf` | Every nested rule or group must pass. |

Values on `role` and `scope` are quoted strings. References must resolve to a
`String` before the protected operation executes.

## 3. Expression Authorization

Expressions authorize against an actor made available by their runtime context:

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

The expression input must include `Context`, named `ctx`. `actor` identifies the
trusted actor context name; it is not a caller-supplied domain `User` value.

## 4. Connection Authorization

Services, integrations, and UIs may attach the same policy grammar directly to
a `connect` declaration. A connection policy uses an explicit trusted context
reference:

```ocean-dsl
connect api.findUserById -> User.findUserById
    authorize context.actor
        authenticated
        anyOf
            subject equals id
            role "platform-admin"
```

For service API connections, the source endpoint must declare that context item
with `@from-context:actor`. The generated guard runs before the connection
target. Connection authorization can coexist with context parameter bindings:

```ocean-dsl
connect api.softDeleteUser -> User.softDeleteUser
    bind deletedBy <- context.actor
    authorize context.actor
        authenticated
        role "platform-admin"
```

Connection policy references resolve against the source operation's inputs.
They must not rely on client-submitted ownership fields for mutable existing
resources. Policies requiring a persisted resource lookup require an explicit
resource-authorization capability rather than trusting the request payload.

## 5. Trusted Context

Only trusted runtime context may be used as a policy actor. At an HTTP boundary
the standard `actor` is established by gateway authentication and propagated in
signed Ocean request context. A DSL declaration such as `@from-context:actor`
requires that context at the endpoint; it does not itself grant access.

## 6. Related Knowledge

- [Expression DSL Reference](expression.md) — the `authorize` precondition on expressions.
- [Service DSL Reference](service.md) — `authorize` on `connect` declarations.
- [Integration DSL Reference](integration.md) — authorization on integration connections.
- [UI DSL Reference](ui.md) — authorization on UI connections.
- [API DSL Reference](api.md) — trusted actor context at the HTTP boundary.
