<!--
Sync Impact Report
- Version change: 1.1.0 -> 2.0.0
- Modified principles:
  - 1. No debt creation -> 1. Dependency Discipline
  - 2. Backend definition -> 2. Rust Backend
  - 3. HTTP Layer -> 3. Hyper HTTP Foundation
  - 4. Storage layer -> 4. Local-First Realm Storage
  - 5. Mobile development -> 5. Kotlin Multiplatform Mobile
  - 6. Android support -> 6. Android 10 and Later
  - 7. Maintainability -> 7. Domain-Driven Maintainability
  - 8. Testing -> 8. Test-First Development
  - 9. Security -> 9. Security and Synchronization Boundary
- Added sections: none
- Removed sections: none
- Follow-up TODOs: none
-->
# Rules

## Core Principles

### 1. Dependency Discipline

- Every new plan and implementation MUST begin with the latest stable, generally available version
  of each selected language, library, framework, build tool, and platform dependency.
- Selected versions MUST be recorded and pinned through build or lock files so builds are reproducible.
- Dependencies MUST NOT be upgraded incidentally during unrelated work. Each later upgrade MUST be
  planned, reviewed for compatibility and security, and tested as an explicit change.

These rules keep new work current while making upgrades controlled and auditable.

### 2. Rust Backend

- All production backend services and backend domain logic MUST be implemented in Rust.
- Rust code MUST compile without warnings under the repository's configured quality gates.
- Build scripts, test harnesses, and operational tooling MAY use Python only when they contain no
  production behavior. Python code MUST be formatted, follow PEP guidance, and use strict typing.

A consistent backend language gives the project one type system, runtime model, and maintenance
profile for production services.

### 3. Hyper HTTP Foundation

- Backend HTTP clients and servers MUST use Hyper as their HTTP protocol foundation.
- Higher-level Rust components MAY wrap Hyper only when the plan documents their value and confirms
  that Hyper remains the underlying HTTP implementation.
- Introducing a competing HTTP protocol stack requires a constitution amendment.

This standard concentrates transport behavior, security review, and operational knowledge.

### 4. Local-First Realm Storage

- All mobile task data MUST be stored locally in a Realm database and remain usable without an
  account, authentication, or network connectivity.
- Creating, reading, editing, completing, searching, reviewing, reminding, and deleting local tasks
  MUST NOT depend on the synchronization service.
- Remote storage MAY hold an account-scoped synchronization copy only when the user explicitly enables
  and invokes synchronization. Remote storage MUST NOT replace Realm as the mobile source of persisted
  task data.
- A synchronization failure, expired credential, disabled account, or deleted account MUST NOT delete,
  lock, or prevent access to local task data.
- Plans MUST define Realm schema versioning, migrations, backup implications, and protection of
  sensitive local data before implementation.

Local ownership keeps the task system useful offline and prevents account-service availability from
controlling access to a user's on-device work.

### 5. Kotlin Multiplatform Mobile

- Mobile application code MUST use Kotlin and Kotlin Multiplatform architecture.
- Android presentation and platform integrations MUST remain in Android source sets.
- Database access MUST remain in Kotlin. Realm-specific code MUST be isolated behind persistence
  contracts so domain behavior does not depend on database APIs.
- Shared domain or application rules MAY be implemented in Rust and exported through UniFFI when the
  plan documents why cross-language reuse outweighs the added boundary and build complexity.

These boundaries keep platform code explicit and preserve separation between domain and persistence.

### 6. Android 10 and Later

- Android is the only supported client platform for the current product scope.
- The minimum supported operating system MUST be Android 10, API level 29.
- Features MUST work on API level 29 and every supported later level. Automated verification MUST
  include API level 29.
- Plans and tasks MUST NOT add iOS, desktop, web-client, or pre-Android-10 compatibility work without
  a constitution amendment.

A narrow platform target keeps product and security support commitments explicit.

### 7. Domain-Driven Maintainability

- Development MUST follow Domain-Driven Design principles.
- Each feature MUST identify its bounded context, use a documented ubiquitous language, and keep
  domain rules independent of transport, persistence, and user-interface concerns.
- Aggregates MUST protect their invariants. Dependencies between bounded contexts MUST pass through
  explicit contracts or translation layers.
- Modules and types MUST have a single, cohesive responsibility. Architecture decisions that cross
  context boundaries MUST be documented.

These rules make ownership, change impact, and maintenance costs visible.

### 8. Test-First Development

- Production behavior MUST follow the red-green-refactor cycle: write an executable failing test,
  implement the minimum behavior needed to pass, then refactor while the suite remains green.
- Every user-visible behavior and security-sensitive rule MUST have a natural-language acceptance
  scenario using Given/When/Then and MUST remain traceable to an automated test.
- The selected test toolchain MUST support mutation testing for the code it validates, or the plan
  MUST document a constitution exception for an unsupported boundary.
- A change is incomplete when its required tests were written only after production behavior or its
  acceptance scenarios are not executable.

Test-first development provides evidence that domain and security rules work as specified.

### 9. Security and Synchronization Boundary

- Security is the primary design and release criterion.
- Each feature MUST document assets, trust boundaries, threat cases, authentication and authorization
  rules, data classification, and failure behavior before implementation.
- Local task operations MUST NOT require account authentication. Authentication and current account
  authorization MUST be required for synchronization and synchronization-account management only.
- Synchronization MUST fail closed when credentials are missing, invalid, expired, or revoked, when the
  account is disabled or deleted, or when current account status cannot be verified. Such failure MUST
  preserve local tasks and pending local changes.
- Implementations MUST use least privilege, explicit input validation, protected secret storage,
  encryption for sensitive data in transit and at rest, and auditable security events.
- Internet-facing password authentication SHOULD use account-aware and source-aware throttling,
  progressive delays, temporary lockout, and auditable security events. A feature MAY defer these
  controls only when its specification records the threat, rationale, risk, and planned follow-up.
- Reviews MUST block release for unresolved critical or high-severity vulnerabilities, failed security
  tests, exposed secrets, or unmitigated threats.

The authentication boundary protects remote account data without making local task access depend on a
remote service.

## Technology Constraints

- Libraries MUST be pinned through a lockfile or an explicit compatible-version constraint.
- Mobile plans MUST name the selected stable Kotlin, Kotlin Multiplatform, Realm Kotlin, Android Gradle
  Plugin, and Android SDK versions and record where each is pinned.
- The Android minimum SDK MUST remain 29. Raising it is a planned compatibility change; lowering it or
  adding another client platform requires a constitution amendment.
- Libraries added beyond the mandated foundation MUST serve a documented domain, security, testing,
  persistence, or operational need. Their licenses, maintenance status, and known vulnerabilities MUST
  be reviewed.
- Cross-boundary data contracts and synchronization protocols MUST be versioned, validated, and covered
  by automated contract tests.
- Realm migrations MUST be deterministic, tested from every supported schema version, and preserve
  local data when authentication or synchronization is unavailable.

## Development Workflow and Quality Gates

- Specifications MUST express behavior as user scenarios and measurable acceptance criteria, including
  offline behavior, synchronization failures, security abuse, and recovery cases.
- Plans MUST declare bounded contexts, responsibilities, dependency direction, data ownership, trust
  boundaries, Realm schema ownership, synchronization boundaries, selected stable technology versions,
  and any justified exceptions.
- Tasks MUST preserve test-first order: acceptance scenarios and failing tests precede implementation,
  followed by refactoring, integration verification, migration testing, and security validation.
- Reviews MUST verify constitution compliance, architectural boundaries, single responsibility,
  Given/When/Then traceability, Realm migration safety, offline local access, sync-only authentication,
  Android API 29 support, dependency pinning, and security evidence.
- Continuous integration MUST run formatting, static analysis, unit tests, contract tests, integration
  tests, executable acceptance scenarios, Realm migration tests, dependency and secret scanning, and
  the Android API 29 checks relevant to the change.
- A temporary exception MUST identify its owner, rationale, risk, compensating control, and expiry date.
  An exception MUST NOT waive a security release gate or technology mandate.

## Governance

This constitution is the highest project-level engineering authority. Specifications, plans, tasks,
implementations, and reviews MUST comply with it. When another project document conflicts with this
constitution, that document MUST be corrected before implementation proceeds.

An amendment MUST state its rationale, affected principles, migration impact, and validation plan.
Approval requires project-maintainer review and recorded acceptance. An amendment takes effect only
after the constitution is versioned and dated.

Constitution versions follow semantic versioning:

- MAJOR for removing or incompatibly redefining a principle.
- MINOR for adding a principle or materially expanding mandatory guidance.
- PATCH for non-semantic clarifications, wording improvements, and typo fixes.

Every feature plan MUST include a constitution compliance check, and every release review MUST retain
evidence that the applicable quality and security gates passed. Compliance MUST be reviewed whenever
the specification, architecture, dependency set, platform target, storage model, synchronization model,
or threat model changes.

**Version**: 2.0.0 | **Ratified**: 2026-09-15 | **Last Amended**: 2026-09-16
