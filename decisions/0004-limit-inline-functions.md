---
title: Limit Inline Functions to Single Expressions
status: accepted
date: 2026-11-27
tags:
  - coding-standards
  - component-design
  - testing
---

# Limit Inline Functions to Single Expressions

## Context

Inline functions provide a convenient way to supply behavior without defining a
separate function or component. They are particularly useful when composing
components, writing tests, and prototyping.

However, inline functions in production application code can conceal
responsibilities that would otherwise warrant independent components and unit
tests. This can undermine established principles for component design and
testing.

## Decision

**Limit inline functions in production application code to a single
expression.**

This constraint applies to the function's implementation, not the number of
physical lines it occupies.

A single expression does not automatically justify inlining behavior. Extract
complex expressions or independently meaningful responsibilities into named
functions or components as appropriate.

### Exceptions

- **Test code:** Inline functions may contain multiple statements, including
  when composing components within tests.
- **Prototyping:** Inline functions may contain multiple statements during
  exploratory development. Refactor them before incorporating the implementation
  into production application code.

## Rationale

### Preserve responsibility boundaries

Inline functions can introduce additional responsibilities without making those
responsibilities apparent in the component structure.

Limiting inline implementations encourages developers to identify and extract
distinct responsibilities, reinforcing
[EDR 0002: Favor Small, Composable Components with Focused Responsibilities](./0002-favor-small-composable-components.md).

### Encourage independent testing

Embedding behavior within component composition can encourage reliance on flow
tests to verify behavior that should also be independently unit tested.

Extracting distinct responsibilities into independently testable components
reinforces the complementary roles of unit tests and flow tests established in
[EDR 0003: Require End-to-End Integration Tests for Application Flows](./0003-require-end-to-end-integration-tests.md).

### Preserve composition readability

Composition should communicate how components interact. Embedding implementation
details within composition makes it more difficult to distinguish component
relationships from application behavior.

### Recognize the benefits of inlining

Inlining can improve test readability by keeping test-specific behavior close
to the setup and expectations that depend on it.

Inlining also reduces the effort required to explore an implementation during
prototyping. However, agent-assisted development reduces the cost of extracting
components and writing corresponding unit tests, diminishing this advantage.

## Consequences

### Positive

- More visible responsibility boundaries.
- Greater opportunities for independent unit testing.
- Improved readability of component composition.
- More consistent application of component design principles.

### Negative

- Additional named functions and components.
- Potentially greater navigation between composition and implementation.
- Additional abstraction for behavior that might otherwise be expressed concisely.

We accept these tradeoffs while preserving the benefits of inlining in tests
and prototypes.

## Alternatives considered

### Allow inline functions at developer discretion

Allow developers to determine when inline implementations warrant extraction.

**Rejected:** This provides insufficient guidance and allows inline
implementations to undermine established component design and testing
principles.

### Prohibit inline functions entirely

Require all functions to be defined independently before use.

**Rejected:** Simple inline expressions can communicate intent clearly without
introducing unnecessary indirection.

## Implementation guidance

### Inline simple behavior

Use inline functions for simple expressions that do not warrant independent
components.

```typescript
const channel = new FilterChannel({
  predicate: event => event.type === 'order.created',
});
```

### Extract multistatement implementations

Avoid embedding multistatement implementations within component composition.

```typescript
const channel = new FilterChannel({
  predicate: event => {
    const { order, customer } = event.payload;

    if (!customer.active || customer.suspended) {
      return false;
    }

    if (order.status !== 'pending') {
      return false;
    }

    return order.items.every(item => item.inStock);
  },
});
```

Extract the implementation into a named function or component.

```typescript
const channel = new FilterChannel({
  predicate: isOrderEligible,
});
```

When extracting an implementation, evaluate whether it represents an independent
responsibility that warrants its own component and unit tests.

Do not restructure multistatement implementations into complex single
expressions solely to satisfy this decision.
