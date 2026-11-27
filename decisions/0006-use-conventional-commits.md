---
title: Use Conventional Commits
status: accepted
date: 2026-11-27
tags:
  - version-control
  - commit-conventions
  - release-management
---

# Use Conventional Commits

## Context

Commit messages communicate the intent of changes to developers, agents, and
automated tooling. Inconsistent commit messages make change history harder to
interpret and automate.

## Decision

**All commits and pull request titles must follow the
[Conventional Commits specification](https://www.conventionalcommits.org/en/v1.0.0/).**

This requirement applies to all commits, including intermediate development
commits.

Individual repositories determine their enforcement mechanisms, versioning
policies, and release automation.

## Rationale

Conventional Commits establishes a consistent format for communicating the
intent of changes.

Adopting an established specification provides a shared convention for
developers and agents while enabling automated changelog generation, semantic
versioning, and release management.

Applying the convention to pull request titles preserves these benefits when
multiple commits are consolidated into a single commit through squash merging.

## Consequences

### Positive

- Consistent, understandable commit history.
- Compatibility with established release automation tooling.
- Predictable commit message and pull request title conventions across
  repositories.

### Negative

- Developers and agents must classify changes when creating commits and pull
  requests.
- Enforcement may require additional tooling.

## Alternatives considered

### Use unstructured commit messages

**Rejected:** Unstructured messages lack the consistency needed to reliably
interpret and automate change history.

### Define a custom commit convention

**Rejected:** An established specification satisfies our requirements without
introducing a proprietary convention or requiring custom tooling.

## Implementation guidance

Use types that accurately communicate the intent of each change.

```text
feat: add order eligibility processor
fix: handle expired subscriptions
refactor: extract customer predicate
docs: document component interfaces
test: cover order eligibility edge cases
```

A commit may include the affected module or component as a scope in parentheses.

```text
feat(event-channel): support asynchronous dispatch
```

Indicate breaking changes with `!` before the colon. Include a
`BREAKING CHANGE:` footer describing what breaks. Both indicators are required:
the first highlights breaking changes in commit history, while the second
provides details for changelog entries.

```text
refactor(provider)!: replace create with get

BREAKING CHANGE: Providers now expose get instead of create.
Consumers must update their calls accordingly.
```

When creating pull requests, use Conventional Commits for the title so that
squash merges using the pull request title produce compliant commit messages.
