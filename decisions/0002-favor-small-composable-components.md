---
title: Favor Small, Composable Components with Focused Responsibilities
status: accepted
date: 2026-11-27
tags:
  - architecture
  - composition
  - coding-standards
---

# Favor Small, Composable Components with Focused Responsibilities

## Context

Software systems grow in complexity as requirements evolve. The organization
of their implementations influences the effort required to understand behavior,
introduce features, resolve defects, and verify changes.

Components that combine multiple responsibilities require developers to
understand a broader implementation when making changes. This can increase
the scope of changes, complicate testing, and make it more difficult to assess
the risk associated with a modification.

Agent-assisted development amplifies these concerns. Experience with
agent-generated implementations has shown that insufficient component
boundaries can lead to disproportionately large change sets, increasing the
effort required to review and verify otherwise focused changes.

The objective is to reduce the amount of complexity a developer needs to
understand at any given time and the scope of implementation changes required
as a system evolves.

## Decision

Favor small, composable components with focused responsibilities. Construct
complex behavior through the composition of these components.

Evaluate component boundaries according to responsibility, testability,
coupling, and scope of change rather than arbitrary measures of size.

### Separate responsibilities

A component should have a single, well-defined responsibility.

When a component requires additional behavior to fulfill its responsibility,
consider whether that behavior represents a separate responsibility. For
example, when a component requires information to perform an operation,
retrieving that information is the responsibility of a provider.

The existence of multiple consumers is not a prerequisite for separating
responsibilities. Independent testability and reduced coupling provide
sufficient justification.

### Inject dependencies

Supply supporting components through dependency injection. Provide default
implementations where a meaningful default exists.

Dependency injection allows consumers to substitute implementations without
modifying the component itself. It also allows unit tests to replace
dependencies with controlled implementations, isolating the behavior under
test from the behavior of its dependencies.

### Enable and encourage reuse

Reusability is a natural consequence of focused responsibilities and loosely
coupled dependencies.

When a component has identifiable uses beyond its initial purpose, proactively
place it within an appropriate shared scope. Reuse does not require waiting
until a second consumer exists.

Avoid introducing responsibilities or configuration solely to accommodate
speculative use cases. A reusable component should remain focused on the
responsibility it was designed to fulfill.

### Organize composition around intent

Composition introduces complexity of its own. As the number of components
grows, the code responsible for instantiating and composing them can become
difficult to navigate.

Apply the same principles of decomposition to composition code that apply to
functional implementations:

- Group related composition into focused, well-named functions.
- Name components and composition functions according to their responsibilities.
- Organize composition so developers can understand high-level behavior before
  examining implementation details.
- Maintain consistent naming and composition conventions throughout the codebase.

A developer should be able to navigate progressively deeper levels of detail
without needing to understand the entire composition at once.

### Support navigation through documentation

Provide conceptual documentation and guides that help developers understand
the organization of a system, its component relationships, and its execution
flows.

Documentation complements meaningful naming and code organization. It should
help developers establish a mental model of the system and identify where to
investigate or introduce changes.

### Scope

This decision concerns the logical organization of software. It does not
prescribe deployment boundaries, package boundaries, or a distributed system
architecture.

Apply these principles within libraries, modular monoliths, microservices,
and other architectural models.

Component size is a consequence of focused responsibility, not an independent
objective. The decision establishes no fixed limits on lines of code, methods,
interfaces, or source files.

## Rationale

### Reduce the scope of change

Focused responsibilities and well-defined dependency boundaries allow many
defects and feature enhancements to be addressed within individual components.

New behavior can often be introduced by adding or composing components rather
than modifying existing implementations. This reduces the need to change
established behavior and limits the potential for unintended regressions.

Changes to shared contracts and cross-cutting behavior can still affect
multiple components. Componentization reduces the typical scope of change but
does not eliminate broader changes.

### Improve reviewability and risk assessment

Component boundaries provide natural opportunities to decompose deliverables
into smaller, independently verifiable changes.

Smaller pull requests generally require reviewers to understand less
implementation context. When a change is confined to a component with a
well-defined responsibility, reviewers can more readily identify the affected
behavior, evaluate correctness, and assess risk.

This approach supports incremental delivery without requiring an entire
feature to be implemented and reviewed as a single change.

### Improve testability

Focused components allow unit tests to verify specific behavior without
exercising unrelated responsibilities.

Injectable dependencies give tests control over the conditions under which a
component operates. This makes it easier to reproduce failure scenarios,
verify edge cases, and identify the source of unexpected behavior.

Integration tests remain necessary to verify interactions between components
and the behavior of their composition.

### Reduce duplication and improve consistency

Reusable components allow established behavior to be applied across multiple
implementations.

Centralizing common behavior reduces duplication and provides a consistent
mechanism for applying engineering and operational practices. Improvements to
shared implementations can benefit their consumers without requiring
equivalent changes in each consumer.

### Support agent-assisted development

Focused component boundaries provide agents with narrower implementation
scopes and clearer behavioral contracts.

This supports both implementation and verification:

- Agents can implement and modify behavior without requiring extensive context
  about unrelated functionality.
- Developers can review generated changes against a more clearly defined
  responsibility.
- Deliverables can be decomposed into smaller tasks with independently
  verifiable outcomes.
- The potential impact of agent-generated changes is easier to assess.

The benefits apply regardless of whether a developer or an agent produces the
implementation.

## Consequences

### Benefits

- More focused implementations and behavioral contracts.
- Smaller, more localized changes when resolving defects and introducing
  features.
- Opportunities for smaller pull requests, faster reviews, and more focused
  risk assessments.
- Improved isolation in unit tests.
- Increased opportunities for reuse and consistent behavior.
- Reduced context requirements for developers and agents working on individual
  components.

### Costs and limitations

- Additional indirection between components can make execution paths harder
  to follow.
- Developers may need to navigate multiple source files to understand behavior.
- Composition code can grow complex and require its own decomposition and
  maintenance.
- Inconsistent application of component conventions can increase the
  difficulty of navigating a codebase.
- Changes to widely used components or shared contracts can have broad
  consequences.
- Maintaining conceptual documentation and navigation guides requires
  additional effort.

Evaluate these costs against the complexity of developing, understanding,
modifying, and verifying the system as a whole rather than the complexity of
an individual implementation.

## Alternatives considered

### Consolidate related behavior into larger components

Larger components reduce the number of implementation boundaries and can make
an execution path easier to follow within a single source file.

However, combining responsibilities can increase coupling, complicate isolated
testing, and require developers to understand unrelated behavior when making
changes.

### Extract reusable components only when multiple consumers exist

Deferring extraction until reuse is demonstrated reduces the risk of
introducing unnecessary abstractions.

However, reuse is only one justification for component separation. Waiting
for multiple consumers overlooks the immediate benefits of focused
responsibilities, independent testability, and localized changes.

## Implementation guidance

When a component needs to retrieve information to perform its operation,
separate retrieval into a provider and inject that provider into the
component. Unit tests can supply a controlled provider to verify the
operation independently of retrieval.

When a composition grows difficult to navigate, extract related portions
into named composition functions. Use names that describe the behavior
assembled by each function so readers can follow the high-level flow before
examining its details.

When planning a feature, use component boundaries to identify independently
verifiable changes that can be implemented and reviewed in separate pull
requests.
