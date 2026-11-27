---
title: Favor Minimal, Responsibility-Focused Component Interfaces
status: accepted
date: 2026-11-27
tags:
  - component-design
  - interfaces
  - coding-standards
---

# Favor Minimal, Responsibility-Focused Component Interfaces

## Context

Component interfaces define the responsibilities exposed to consumers and
establish how components participate in composition.

Interfaces that expose unnecessary operations or introduce distinctions based
solely on implementation details increase the number of contracts developers
must understand. Conversely, consolidating structurally equivalent interfaces
without considering their semantic differences can obscure the responsibilities
they represent.

This decision complements
[EDR 0002: Favor Small, Composable Components with Focused Responsibilities](./0002-favor-small-composable-components.md)
by applying its principles to component contracts.

## Decision

**Define minimal component contracts that express distinct responsibilities and
roles in composition.**

### Minimize individual contracts

Favor component interfaces that expose a single operation representing their
responsibility.

Introducing additional operations requires explicit justification. Grouping
operations within a cohesive composite interface is acceptable when doing so
provides meaningful value to consumers.

### Reuse semantically equivalent contracts

Reuse existing interfaces and component types when they adequately express the
responsibility and role of a component.

Introducing a new component contract requires explicit justification.
Differences in implementation strategy or terminology alone are insufficient
justification.

Evaluate equivalence based on:

- The responsibility represented by the contract.
- The role of the component within composition.
- The ability of the existing contract to express the required behavior.

### Preserve semantic distinctions

Structural equivalence does not imply semantic equivalence.

Maintain distinct contracts when components represent different responsibilities
or roles in composition, even when their type definitions are structurally
equivalent.

Do not introduce dependencies between semantically distinct contracts solely to
eliminate duplication in their type definitions.

### Distinguish implementation names from contracts

Name component implementations according to their specific purpose.
Implementation names do not need to match the terminology of the interfaces
they implement.

A transformer, converter, or enricher can implement a `Processor` interface
without requiring a corresponding specialized interface.

### Justify contract decisions

Establish explicit justification when introducing a new component contract or
expanding an existing contract beyond a single operation.

Establishing this justification during design or code review is sufficient.
A separate decision record is not required.

## Rationale

### Preserve focused responsibilities

Minimal interfaces reinforce focused component responsibilities by limiting the
behavior exposed to consumers.

A single-operation interface communicates the component's responsibility
without requiring consumers to understand unrelated capabilities.

### Promote interoperability

Reusing semantically equivalent contracts allows components with different
implementations to participate in the same composition.

Consumers can depend on the responsibility represented by a contract rather
than the strategy used to fulfill it.

### Preserve semantic clarity

Component contracts communicate intent as well as structural requirements.

Two components may accept equivalent arguments and return equivalent values
while serving different purposes. Maintaining distinct contracts preserves
these differences and allows each contract to evolve independently.

### Reduce unnecessary abstractions

Every additional contract introduces terminology and relationships that
developers must understand and maintain.

Reusing existing contracts where appropriate establishes a smaller, more
consistent vocabulary for component composition without restricting the
variety of available implementations.

## Consequences

### Positive

- More focused and understandable component contracts.
- Greater interoperability between component implementations.
- Reduced proliferation of unnecessary interfaces and types.
- Clearer semantic distinctions between component responsibilities.
- Greater freedom to name implementations according to their purpose.

### Negative

- Additional design consideration when introducing or expanding contracts.
- Some structurally equivalent type definitions may be duplicated.
- Consumers may occasionally need to adapt operations exposed by composite
  components.

We accept these tradeoffs in favor of minimal, semantically meaningful
component contracts.

## Alternatives considered

### Require single-operation interfaces without exception

Require every component interface to expose exactly one operation.

**Rejected:** Cohesive composite components can provide meaningful value by
grouping related operations. An absolute restriction would prevent legitimate
composite contracts.

### Define specialized contracts for each implementation category

Introduce distinct interfaces for builders, transformers, converters,
enrichers, and other implementation categories.

**Rejected:** Differences in implementation purpose or strategy do not
necessarily represent distinct responsibilities within composition.
Specialized contracts can introduce unnecessary abstractions and reduce
interoperability.

### Consolidate structurally equivalent contracts

Reuse contracts whenever their type definitions are structurally compatible.

**Rejected:** Structural compatibility alone does not establish semantic
equivalence. Consolidating contracts on this basis can obscure meaningful
differences in responsibility and introduce artificial dependencies.

## Implementation guidance

### Define focused component interfaces

Expose the operation that represents the component's responsibility.

```typescript
interface Provider<T> {
  get(): T | PromiseLike<T>;
}

interface Channel<T> {
  send(event: T): void | PromiseLike<void>;
}

interface Command {
  execute(): void | PromiseLike<void>;
}
```

These interfaces illustrate focused contracts rather than prescribe canonical
type definitions.

### Reuse contracts across implementation strategies

A builder constructs an object to provide it to a consumer. Its construction
strategy does not require a distinct component contract.

A builder can implement the `Provider` interface through `get` rather than
introduce a separate `Builder` interface with a `create` operation.

The implementation can retain a name that identifies it as a builder.

### Preserve semantically distinct types

Consider the following functional component types:

```typescript
type CommandFn<T = void, R = void> =
  (arg: T) => R | PromiseLike<R>;

type PredicateFn<T = void> =
  (arg: T) => boolean | PromiseLike<boolean>;
```

Although `PredicateFn` could be expressed in terms of `CommandFn`, doing so
would establish a relationship between semantically distinct contracts.

A command performs an operation. A predicate evaluates a condition. Their
structural compatibility does not make them conceptually interchangeable
within composition.

Maintain separate definitions to preserve these distinctions.

### Name implementations according to purpose

A processor accepts a value, performs an operation, and returns a value.

Implementations such as `OrderTransformer`, `CurrencyConverter`, and
`CustomerEnricher` can all implement the `Processor` interface.

Their names communicate the specific processing they perform, while the shared
interface establishes their role within composition.

Introduce specialized contracts only when they represent a justified semantic
distinction.

### Accommodate composite components

A `Store` may expose multiple providers, mutators, and commands through a
cohesive composite interface.

When a composite component structurally satisfies a narrower component
contract, consumers can use it directly. For example, a store exposing `get`
can participate in composition as a `ProviderComponent`, while a store
exposing `values` can participate as an `IterableProvider`.

When the required operation does not directly satisfy the component contract,
adapt it as a functional component. For example, `store.keys.bind(store)` can
provide an iterable over the store's keys.

A composite interface is a justified exception to the single-operation
preference, not a default approach to component design.
