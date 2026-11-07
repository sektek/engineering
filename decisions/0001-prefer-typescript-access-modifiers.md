---
title: Prefer TypeScript Access Modifiers Over JavaScript Hard-Private Fields
status: accepted
date: 2026-11-07
tags:
  - typescript
  - encapsulation
  - coding-standards
---

# Prefer TypeScript Access Modifiers Over JavaScript Hard-Private Fields

## Context

JavaScript hard-private fields (`#member`) enforce encapsulation at runtime.
We adopted them when we wrote more plain JavaScript, where the only
alternative to hard privacy was underscore-prefixed names such as `_member`.

TypeScript provides the `private` and `protected` access modifiers. They
enforce visibility at compile time and don't change the runtime representation
of class members.

Hard-private fields have caused practical problems:

- **Inheritance:** Subclasses can't access hard-private fields, even when
  extending the parent's behavior legitimately requires it. Keeping the field
  hard-private means adding accessors or methods.
- **Mixins:** Accessing a hard-private field on an object that the declaring
  class didn't construct throws a `TypeError`. This limits some class
  composition patterns.
- **Serialization:** Hard-private state is excluded from property enumeration
  and ordinary serialization. State that must cross a serialization boundary,
  such as when requeueing events, needs an additional mechanism.
- **Performance:** Hard-private access can cost more than ordinary property
  access. The difference depends on the JavaScript engine and the workload.
- **Evolution:** Turning a hard-private field into an extension point for
  subclasses means changing its representation or adding an accessor.

Hard privacy also has benefits: runtime enforcement, no property-name
collisions, and no accidental exposure through reflection or serialization.

TypeScript's access modifiers already state and enforce the intended
visibility for TypeScript consumers. Code that bypasses them relies on
implementation details and takes responsibility for that reliance. Not every
internal member needs runtime enforcement.

## Decision

**Prefer TypeScript `private` and `protected` access modifiers over JavaScript
hard-private (`#`) fields.**

Follow these guidelines:

1. Use `private` for internal members that consumers and subclasses shouldn't
   access.
2. Use `protected` when subclass access is an intentional part of the class's
   extension contract.
3. Use `#private` only when runtime-enforced encapsulation has a specific,
   demonstrable benefit.
4. Don't add accessors or other abstractions only to preserve hard privacy
   when direct `protected` access is appropriate.
5. Define an explicit serialization contract for state that crosses a
   serialization boundary. Access modifiers don't determine what gets
   serialized.
6. Benchmark performance-sensitive code with representative workloads. Don't
   assume either mechanism is faster.

This decision applies to new code and to changes to existing classes. Existing
hard-private fields don't need migration unless migrating them provides a
practical benefit.

## Rationale

### Compile-time privacy is a sufficient default

Access modifiers give consumers and maintainers an explicit visibility
contract. JavaScript consumers and TypeScript escape hatches can bypass it,
but doing so goes outside the supported contract. Ordinary encapsulation
doesn't require preventing every such violation at runtime.

### Inheritance shouldn't require indirection

When a subclass needs inherited state, `protected` expresses that directly.
Wrapping a hard-private field in protected accessors only to keep it private at
runtime adds complexity without improving the design.

Accessors are still appropriate when they enforce invariants, control
mutation, or abstract over the underlying representation.

### Runtime enforcement should justify its cost

Hard privacy provides real guarantees, particularly against untyped JavaScript
consumers and property collisions. Choose it deliberately for the cases that
need those guarantees instead of applying it everywhere.

## Consequences

### Positive

- Fewer accessors and less indirection.
- Simpler subclass extension.
- Fewer runtime constraints on mixins and other composition patterns.
- Property enumeration and serialization can see class state. Serializable
  state still needs an explicit contract (guideline 5).
- Compile-time encapsulation is preserved.
- Changing a member from `private` to `protected` doesn't change its runtime
  representation.

### Negative

- JavaScript consumers can access `private` and `protected` members at
  runtime.
- Internal properties can appear in enumeration, object spreading, reflection,
  and serialization.
- A subclass can collide with or overwrite an internal property at runtime.
- Consumers can come to depend on implementation details, which we don't
  support.
- Runtime property access doesn't get the isolation that hard-private fields
  provide.

We accept these tradeoffs.

## Alternatives considered

### Use hard-private fields by default

This keeps the strongest runtime encapsulation but also keeps the inheritance,
composition, and serialization limitations that motivated this decision.

**Rejected:** The benefit of runtime enforcement doesn't justify the recurring
complexity.

### Use hard-private fields with protected accessors

This keeps runtime privacy and allows controlled subclass access.

**Rejected as a default:** It adds indirection when direct access is
intentional. It remains appropriate when the accessors enforce invariants or
add behavior.

### Drop privacy modifiers entirely

This relies on conventions and documentation to mark implementation details.

**Rejected:** TypeScript's access modifiers provide compile-time enforcement at
little cost.

## Implementation guidance

Use `private` and `protected` by default:

```typescript
class BaseComponent {
  private internalState = 0;
  protected sharedState = 0;
}
```

Use `#private` selectively, where runtime isolation matters:

```typescript
class IsolatedComponent {
  #sensitiveState = new Map<string, unknown>();
}
```

When you modify a class that already uses a hard-private field, you can keep
the field if hard privacy is still justified. An existing `#` field isn't a
reason to keep hard privacy on its own.
