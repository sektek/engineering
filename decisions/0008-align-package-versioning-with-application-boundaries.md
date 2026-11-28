---
title: Align Package Versioning with Application Boundaries
status: accepted
date: 2026-11-27
tags:
  - package-versioning
  - application-architecture
  - dependency-management
---

# Align Package Versioning with Application Boundaries

## Context

Applications often contain multiple deployable services and shared libraries.
As applications evolve, components may become independently deployed without
becoming independent applications.

Independently versioning shared libraries introduces release management,
dependency management, and compatibility considerations. These costs are not
always justified by reuse within an application.

## Decision

**Align package versioning boundaries with application boundaries rather than
deployment boundaries.**

Prefer colocating application-specific libraries and deployable services within
an application monorepo. Shared libraries within an application do not require
independent versioning solely because they have multiple consumers.

Favor independently versioned packages for general-purpose libraries intended
for use across applications.

Consider extracting application-specific libraries when their use outside the
application becomes comparable to their use within it.

Shared domain contracts may warrant independent versioning earlier when they
establish compatibility boundaries between applications.

## Rationale

Colocating application-specific libraries with their consumers allows
coordinated changes without introducing unnecessary package release and
dependency management overhead.

Independent versioning provides consumers outside the application with control
over dependency adoption. Producers can evolve their libraries without
requiring consumers to adopt changes simultaneously.

Deployment, repository, and package versioning boundaries serve different
purposes and do not need to align.

## Consequences

### Positive

- Reduced release and dependency management overhead within applications.
- Atomic changes to internal libraries and their consumers.
- Independent release lifecycles for cross-application libraries.
- Consumer-controlled adoption of breaking changes.

### Negative

- Extracting an internal library later requires additional work.
- Independently versioned libraries require compatibility and release
  management.
- Application boundaries must be evaluated as systems evolve.

## Alternatives considered

### Independently version all shared libraries

**Rejected:** Reuse within an application does not inherently justify
independent release lifecycles or dependency management overhead.

### Colocate all libraries with their consumers

**Rejected:** Libraries consumed across application boundaries benefit from
independent release lifecycles and explicit compatibility contracts.

## Implementation guidance

Evaluate shared libraries according to their intended consumers and
responsibilities.

- General-purpose event processing infrastructure should be independently
  versioned.
- Shared event contracts consumed across applications may warrant independent
  versioning.
- Application-specific processing logic shared between services should remain
  within the application repository.

Extracting a component into an independently deployed service does not, by
itself, justify independently versioning its supporting libraries.

Reevaluate library boundaries as consumption patterns evolve.
