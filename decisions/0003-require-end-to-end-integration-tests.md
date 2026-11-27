---
title: Require End-to-End Integration Tests for Application Flows
status: accepted
date: 2026-11-27
tags:
  - testing
  - integration-testing
  - coding-standards
---

# Require End-to-End Integration Tests for Application Flows

## Context

Unit tests establish confidence in the behavior of individual components. They
do not establish that the composition of those components implements the
intended application behavior.

In component-oriented architectures, composition often defines a significant
portion of the application's logic. Routing, filtering, transformation, and
coordination can emerge from the relationships between components rather than
their individual implementations.

Mocked dependencies can also conceal discrepancies between component contracts
and infrastructure behavior. For example, a provider's unit tests may pass while
the results of an actual SQL query fail to satisfy the provider's expected
contract. Experience with database integrations has demonstrated the importance
of exercising these interactions without mocks.

End-to-end integration tests, also referred to as flow tests, address these
limitations by exercising complete application flows under conditions that
approximate production.

## Decision

**Require end-to-end integration tests for application flows. Exercise actual
application logic and infrastructure dependencies while isolating external
system boundaries where appropriate.**

### Require flow tests

A pull request introducing a flow's composition is incomplete without
corresponding flow tests.

CI must successfully execute the tests before the pull request advances to the
next stage of the development lifecycle. When code review is the next stage,
resolve test failures before requesting review by a developer or an agent.

Flow tests should:

- Verify the application's happy path.
- Verify known error cases.
- Exercise conditions that may expose edge cases and unexpected interactions.
- Verify observable application outcomes, including emitted events where
  applicable.

Comprehensive unit test coverage does not replace flow tests.

### Exercise actual application logic

Use actual component implementations and their production composition. Do not
mock internal application dependencies when testing a complete flow.

The test should exercise the same application logic used in production,
including interactions between components and infrastructure dependencies.

### Isolate transport and external system boundaries

Transport mechanisms should be sufficiently abstracted that substituting them
does not change application logic.

For event-driven flows, inject events directly into the flow rather than through
the production gateway. Outbound transport components can be replaced with
implementations that make delivered events accessible for verification.

For flows exposed through HTTP endpoints, exercise the flow through its HTTP
interface.

External HTTP services can be simulated at the HTTP boundary. This allows the
application to exercise its actual request construction, response handling, and
error handling without depending on the external service.

These substitutions isolate system boundaries rather than replace application
logic.

### Preserve infrastructure fidelity

Use production-equivalent infrastructure where infrastructure behavior
contributes to application correctness.

When a flow interacts with a database, exercise the actual persistence
implementation against the corresponding database engine and schema. Do not
replace persistence components with mocks in complete flow tests.

Provision infrastructure dependencies within a self-contained test environment.
Database schemas and other infrastructure definitions should support testing
dependent applications without requiring access to shared systems.

### Complement unit tests

Unit tests remain responsible for verifying the behavioral contracts of
individual components, including:

- Happy paths.
- Error cases.
- Edge cases.
- Emitted events and other observable behavior.

Treat subflows as independent units when appropriate. This allows developers to
implement and test a subflow before assembling the complete application flow.
Mocks are permitted when testing a subflow as a unit.

Independent subflow tests do not replace integration tests for the complete
application flow.

### Protect against regressions

When addressing a defect, add a flow test that covers the affected application
behavior.

If the defect originates in the flow's composition, the flow test is sufficient.

If the defect originates within a component or an independently unit-tested
subflow, also add a unit test that covers the defect at that level.

This decision requires regression coverage without prescribing the sequence in
which tests and implementation changes are developed.

## Rationale

### Verify composition as application logic

Correct components do not guarantee correct application behavior.

The selection, configuration, and composition of components determine how an
application behaves. Flow tests verify the resulting behavior rather than
relying on the correctness of each component in isolation.

This complements
[EDR 0002: Favor Small, Composable Components with Focused Responsibilities](./0002-favor-small-composable-components.md).

### Expose infrastructure contract discrepancies

Mocked infrastructure dependencies reproduce the behavior expected by the
developer. They cannot independently establish that the infrastructure behaves
as expected.

Exercising actual database engines, schemas, and queries exposes discrepancies
that isolated component tests may conceal. It also verifies that persistence
implementations satisfy the contracts expected by their consumers.

### Establish confidence in changes

Flow tests provide regression protection at the application level.

Developers can modify individual components, replace implementations, or
reorganize composition while verifying that established application behavior
remains intact.

This is particularly valuable when reviewing agent-generated changes. Passing
flow tests provide evidence that the application continues to satisfy the
behaviors covered by those tests, regardless of who or what produced the
implementation.

### Verify behavior before requesting review

Automated test verification should precede code review, whether performed by a
developer or an agent. A failing test establishes that further investigation or
modification is required.

Requiring passing tests before requesting review reduces unnecessary review
cycles, conserves engineering time and computational resources, and allows
reviewers to focus on correctness, design, maintainability, and risk.

Resolve known verification failures before incurring the cost of subsequent
verification activities.

### Prioritize engineering time over execution time

Higher-fidelity tests generally require more execution time and infrastructure
setup than isolated unit tests.

Accept this additional cost when it provides meaningful regression protection
and reduces the engineering effort required to verify changes and investigate
defects.

Engineering time is generally more valuable than test execution time. Optimize
the overall cost of developing and maintaining the application rather than test
execution time in isolation.

Accepting longer execution times does not justify inefficient tests. Avoid
unnecessary work and optimize test execution where doing so preserves testing
fidelity, coverage, and maintainability. Evaluate the engineering effort
required for these optimizations against their expected benefit.

### Avoid dependencies on shared environments

Self-contained test environments reduce reliance on the availability and state
of shared infrastructure.

Provisioning infrastructure alongside the tests allows application behavior to
be verified without coordinating access to externally managed test environments.

## Consequences

### Positive

- Verification of application behavior beyond individual component contracts.
- Increased confidence that implementation and composition changes preserve
  established behavior.
- Detection of discrepancies between application contracts and infrastructure
  behavior.
- Regression coverage at the application boundary.
- Greater confidence when reviewing agent-generated implementations.
- Fewer unnecessary human and agent review cycles.
- Reduced reliance on shared testing infrastructure.

### Negative

- Longer test execution times.
- Additional CI resource consumption.
- Additional effort to provision and maintain test infrastructure.
- More test setup and infrastructure configuration.
- Failures may require investigation across multiple components.
- Substituted transport boundaries leave production transport behavior outside
  the scope of flow tests.
- Some infrastructure dependencies may be difficult or expensive to reproduce
  with production fidelity.

We accept these tradeoffs in favor of greater confidence in application behavior
and reduced long-term engineering effort.

## Alternatives considered

### Rely primarily on unit tests

Unit tests execute quickly, isolate failures, and provide focused verification
of component behavior.

**Rejected:** Component correctness does not establish application correctness.
This approach leaves composition and infrastructure interactions insufficiently
verified.

### Mock infrastructure dependencies in flow tests

Mocking infrastructure reduces setup requirements and test execution time while
allowing application composition to be exercised.

**Rejected:** Mocking infrastructure can conceal discrepancies between actual
infrastructure behavior and application contracts.

Mocks remain appropriate for isolated unit tests and independently tested
subflows.

### Require the complete production transport path

Exercise flows through their production gateways and outbound transport
implementations.

This provides broader coverage of the deployed system, including transport
configuration and behavior.

**Rejected as a requirement for flow tests:** The purpose of flow tests is to
verify application logic. Transport-independent application logic can be
verified without requiring the complete production transport path.

Additional tests can verify transport behavior when necessary.

### Use shared integration environments

Run integration tests against externally provisioned infrastructure shared
between applications or developers.

**Rejected as a default:** Shared environments introduce external dependencies
and require coordination around availability, configuration, and state.

## Implementation guidance

### Event-driven flows

Inject an event into the application's composed flow. Exercise the actual
components and infrastructure dependencies. Replace outbound transport
components where necessary to capture and verify delivered events.

### HTTP flows

Exercise HTTP endpoints through their HTTP interface. When the application calls
an external HTTP service, simulate that service at the HTTP boundary rather than
mocking the application's HTTP client or application components.

### Database-backed flows

Provision the corresponding database engine and initialize its schema. Exercise
the application's actual queries and persistence components. Verify application
outcomes against the expected behavior.

### Defect resolution

Add regression coverage to the affected flow. When the defect originates within
an independently unit-tested component or subflow, extend its unit tests to
cover the defect as well.

### Scope

This decision does not prescribe testing frameworks, infrastructure provisioning
tools, test isolation strategies, state management, parallel execution, or local
test execution practices.

These implementation decisions can vary between projects according to their
requirements and constraints.

Beyond requiring successful CI test verification before advancement, this
decision does not prescribe the stages of the development lifecycle.
