---
title: Favor Small, Focused Pull Requests
status: accepted
date: 2026-11-27
tags:
  - development-workflow
  - code-review
  - verification
---

# Favor Small, Focused Pull Requests

## Context

Large pull requests increase the cognitive burden of review. Combining
component implementation with component composition requires reviewers to
evaluate both individual component behavior and interactions between components.

Small, focused pull requests allow developers and agents to evaluate changes
against a narrower set of requirements.

## Decision

**Favor small pull requests with a single, focused purpose.**

Prefer pull requests that implement or modify an individual component or
compose existing components. Avoid combining component implementation and
composition when they can reasonably be delivered separately.

Larger pull requests require justification for why further decomposition is
undesirable or impractical.

Every pull request must independently satisfy all applicable verification
requirements, including when it is part of a stack of dependent pull requests.

Do not artificially fragment cohesive changes merely to reduce pull request
size.

## Rationale

Separating component implementation from composition allows reviewers to
evaluate distinct concerns independently:

- Whether a component fulfills its contract.
- Whether components are composed correctly to fulfill application requirements.

Reducing the scope of individual pull requests reduces review complexity, makes
defects easier to isolate, and limits the risk associated with individual
changes.

These benefits apply to both human and agent reviewers.

[Small, composable components with focused responsibilities](0002-favor-small-composable-components.md)
provide natural boundaries for decomposing changes.

## Consequences

### Positive

- Reduced cognitive burden during review.
- More focused verification and feedback.
- Smaller, more isolated changes.
- Greater opportunities for parallel development.

### Negative

- Features may require multiple pull requests.
- Dependent pull requests may require additional coordination.
- Decomposing work requires additional planning.

## Alternatives considered

### Establish fixed pull request size limits

**Rejected:** Line counts vary across languages, implementation patterns, and
testing requirements. They do not reliably measure review complexity or change
cohesion.

### Organize pull requests around complete features

**Rejected:** Feature boundaries may encompass multiple independently verifiable
changes, unnecessarily increasing review complexity.

## Implementation guidance

Identify component implementation and composition boundaries when decomposing
work.

For example, implementing a new order eligibility predicate and incorporating
it into an order processing flow can be delivered as separate pull requests:

1. Implement the predicate and its unit tests.
2. Incorporate the predicate into the flow and add the relevant
   [end-to-end integration tests](0003-require-end-to-end-integration-tests.md).

Each pull request must satisfy its applicable verification requirements before
proceeding to the next stage of development.

Evaluate pull request size based on the scope, complexity, and cohesion of the
changes rather than the number of lines changed.
