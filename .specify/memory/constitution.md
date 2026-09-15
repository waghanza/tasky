<!--
Sync Impact Report
- Version change: unratified template -> 1.0.0
- Modified principles: none; initial project-specific ratification
- Added principles:
  - I. Current Stable Versions
  - II. Rust Backend
  - III. Hyper HTTP Foundation
  - IV. Kotlin Multiplatform Mobile
  - V. Android 10 and Later
  - VI. Domain-Driven Design and Maintainability
  - VII. Behavior-Driven Test-First Development
  - VIII. Security First
- Added sections:
  - Technology Constraints
  - Development Workflow and Quality Gates
- Removed sections: none
- Follow-up TODOs: none
-->
# Tasky Constitution

## Core Principles

### 1. No debt creation

+ Every new plan and implementation MUST begin with the latest stable, generally available version of each selected language, library, framework, build tool, and platform dependency. 
+ The selected versions MUST be recorded and pinned through the appropriate build or lock files so builds remain reproducible. 
+ A dependency MUST NOT be upgraded incidentally during unrelated work; every later upgrade MUST be planned, reviewed for compatibility and security, and tested as an explicit change.
+ This keeps new work current while making subsequent upgrades controlled and auditable.

### 2. Backend definition

+ All production backend services and backend domain logic MUST be implemented in Rust. 
+ Rust code MUST compile without warnings under the repository's configured quality gates. This constraint gives the backend a consistent type system, runtime model, and maintenance profile.
+ Build scripts, test harnesses, and operational tooling MUST be implemented in python if, and only if, they do not contain production.
+ Pÿthon code MUST formatted, PEP aware and strictly typed.


### 3. HTTP Layer

+ Backend HTTP clients and servers MUST use Hyper as their HTTP protocol foundation. 
+ Higher-level Rust components MAY wrap Hyper only when the plan documents the added value and confirms that Hyper remains the underlying HTTP implementation. 
+ Introducing a competing HTTP protocol stack requires a constitution amendment. 
+ This standard keeps transport behavior, security review, and operational knowledge focused.

### 4. Mobile development

+ Mobile application code MUST use Kotlin and the Kotlin Multiplatform architecture.
+ Kotlin should be used for all part of code, except domain logic / application rules that should be written in Rust and exported with Uniffi
+ Database access should remain in Kotlin, since their is no shared logic with backend
+ Android-specific presentation and platform integrations MUST remain in Android source sets.

### 5. Android support

+ Android is the only supported mobile platform as for now. The minimum supported operating system MUST be Android 10, corresponding to API level 29. 
+ Features MUST work on API level 29 and every supported later level, and automated verification MUST include API level 29.
+ Plans and tasks MUST NOT add any iOS, desktop, weblient, or pre-Android-10 compatibility work unless this constitution is amended first. A narrow platform target keeps product and security support commitments explicit.

### 6. Maintainability

+ Development MUST follow Domain-Driven Design principles. 
+ Each feature MUST identify its bounded context, use a documented ubiquitous language, and keep domain rules independent of transport, persistence, and user interface concerns. 
+ Aggregates MUST protect their invariants, and dependencies between bounded contexts MUST pass through explicit contracts or translation layers. 
+ Modules and types MUST have a single, cohesive responsibility, and architecture decisions that cross context boundaries MUST be documented.
+ These rules make ownership, change impact, and maintenance costs visible.

### 7. Testing

+ Production behavior MUST be developed with the red-green-refactor TDD cycle: write an executable failing test, implement the minimum behavior needed to pass, then refactor while the suite remains green. 
+ Each user-visible behavior and security-sensitive rule MUST have acceptance scenarios written in natural language using Gherkin Given/When/Then syntax. 
+ The test runner MAY be Robot Framework or another tool, but the scenarios MUST remain traceable to automated tests. 
+ A change is incomplete if its required tests were written only after the production behavior or if its acceptance scenarios are not executable.
+ Any tool should have a mutation testing feature

### 8. Security

+ Security is the primary design and release criterion. 
+ Every feature MUST document its assets, trust boundaries, threat cases, authentication and authorization rules, data classification, and failure behavior before implementation. 
+ Implementations MUST use secure defaults, least privilege, explicit input validation, protected secret storage, encryption for sensitive data in transit and at rest, and auditable security events. 
+ Reviews MUST block release for unresolved critical or high-severity vulnerabilities, failed security tests, exposed secrets, or unmitigated threats. 
+ Enterprise readiness MUST be demonstrated through traceable controls, repeatable evidence, and documented operational ownership rather than asserted without verification.

### 9. Changes

+ Any change introduce MUST respect this constitution. Other it SHOULD be amended first

## Technology Constraints

+ Any library should be pinned throught a lockfile or a semver constraint.
+ Mobile plans MUST name the selected stable Kotlin, Kotlin Multiplatform, Android Gradle Plugin, and Android SDK versions and record where each is pinned.
+ The Android minimum SDK MUST be 29. Raising it is a planned compatibility change; lowering it or adding another client platform requires a constitution amendment.
+ Libraries added above the required foundation MUST serve a documented domain, security, testing, or operational need. 
+ Their licenses, maintenance status, and known vulnerabilities MUST be reviewed.
+ Cross-boundary data contracts MUST be versioned, validated, and covered by automated contract tests.

## Development Workflow and Quality Gates

+ Specifications MUST express behavior as user scenarios and measurable acceptance criteria, including security abuse and failure cases.
+ Plans MUST declare bounded contexts, their responsibilities, dependency direction, data ownership, trust boundaries, selected stable technology versions, and any justified exceptions.
+ Tasks MUST preserve the test-first order: acceptance scenarios and failing tests precede implementation, followed by refactoring, integration verification, and security validation.
+ Reviews MUST verify constitution compliance, architectural boundaries, single responsibility, Gherkin traceability, Android API 29 support, dependency pinning, and security evidence.
+ Continuous integration MUST run formatting, static analysis, unit tests, contract tests, integration tests, executable acceptance scenarios, dependency and secret scanning, and the Android API 29 checks relevant to the change.
+ Any temporary exception MUST identify its owner, rationale, risk, compensating control, and expiry date. An exception MUST NOT waive the security release gates or a technology mandate in this constitution.

## Governance

This constitution is the highest project-level engineering authority. 
Specifications, plans, tasks, implementation, and reviews MUST comply with it. When another project document conflicts with this constitution, that document MUST be corrected before implementation proceeds.

Amendments MUST be proposed as an explicit constitution change with the rationale, affected principles, migration impact, and validation plan. 
Approval requires project maintainer review and recorded acceptance. Changes take effect only after the amended constitution is versioned and dated.

Constitution versions follow semantic versioning: 
+ MAJOR for removing or incompatibly redefining a principle; 
+ MINOR for adding a principle or materially expanding mandatory guidance; 
+ PATCH for non-semantic clarifications. 
Every feature plan MUST include a constitution compliance check, and every release review MUST retain evidence that the applicable quality and security gates passed. 
Compliance MUST be reviewed whenever the specification, architecture, dependency set, platform target, or threat model changes.

**Version**: 1.0.0 | **Ratified**: 2026-09-15 | **Last Amended**: 2026-09-15
