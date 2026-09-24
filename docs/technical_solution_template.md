Below is a reusable **Technical Solution document outline** designed for software features involving frontend, backend, APIs, databases, cloud infrastructure, security, and release controls.

## 1. Technical Solution Outline

### 1. Document Overview

| Field          | Description                                    |
| -------------- | ---------------------------------------------- |
| Feature        | Feature name                                   |
| Jira/Epic      | Related Jira ticket or Epic                    |
| Author         | Technical owner                                |
| Reviewers      | Tech Lead, Security, QC, DevOps, Product Owner |
| Status         | Draft / In Review / Approved                   |
| Target Release | Planned release                                |
| Repositories   | Relevant repositories and services             |

### 2. Executive Summary

Briefly describe:

* The problem being solved.
* The proposed technical approach.
* The main systems affected.
* The expected outcome.
* The most important technical decision.

This section should be understandable without reading the entire document.

### 3. Problem Statement and Current Situation

Describe:

* Current system behaviour.
* Current architecture and workflow.
* Existing limitations or defects.
* Business and technical impact.
* Why the current implementation cannot fully support the requirement.
* Evidence from source code, logs, API responses, database records, or production incidents.

Codex should identify the relevant existing files, classes, functions, APIs, tables, and infrastructure resources.

### 4. Goals and Non-Goals

#### Goals

Define what the solution must achieve.

Examples:

* Support the new business workflow.
* Maintain backward compatibility.
* Enforce tenant or zone-level access.
* Provide auditable and deterministic results.
* Avoid disruption to existing production behaviour.

#### Non-Goals

Clearly identify what is intentionally excluded.

Examples:

* No redesign of unrelated modules.
* No migration of historical data.
* No replacement of the existing authentication framework.
* No new operational dashboard in this release.

### 5. Scope

#### In Scope

List affected areas such as:

* Frontend pages and components.
* Backend services and Lambda functions.
* API endpoints.
* Database tables or schemas.
* Configuration and feature settings.
* Infrastructure and permissions.
* Logging, audit, and monitoring.
* Automated tests.

#### Out of Scope

List related items that will not be delivered as part of this feature.

### 6. Requirements and Acceptance Criteria

Separate the requirements into:

#### Functional Requirements

Describe system behaviours, user actions, business rules, and expected outputs.

#### Non-Functional Requirements

Cover:

* Security.
* Performance.
* Availability.
* Reliability.
* Scalability.
* Auditability.
* Maintainability.
* Accessibility.
* Data retention and privacy.

#### Acceptance Criteria Mapping

Map each acceptance criterion to:

* Proposed component.
* Implementation location.
* Validation method.
* Test coverage.

### 7. Assumptions, Constraints, and Dependencies

#### Assumptions

Document assumptions that affect the solution.

#### Constraints

Examples:

* Existing API contracts must remain compatible.
* Production database schema changes must be backward-compatible.
* Existing authentication mechanisms must be reused.
* Infrastructure must remain within the current AWS or Azure architecture.

#### Dependencies

Identify dependencies on:

* Other teams.
* External APIs.
* Identity providers.
* Infrastructure changes.
* UX designs.
* Feature flags.
* Data migrations.
* Security or release approvals.

### 8. Current-State Technical Analysis

Codex should inspect the repository and document:

* Current request and data flow.
* Existing components and responsibilities.
* Existing API contracts.
* Current database schema.
* Current authentication and authorization.
* Current configuration and inheritance behaviour.
* Existing error handling.
* Existing tests.
* Known technical debt.

A Mermaid sequence or component diagram can be included.

### 9. Options Considered

For each option, document:

| Area               | Details                                      |
| ------------------ | -------------------------------------------- |
| Description        | How the option works                         |
| Advantages         | Technical and operational benefits           |
| Disadvantages      | Complexity, risks, limitations               |
| Security impact    | Authentication, authorization, data exposure |
| Performance impact | Latency, throughput, resource usage          |
| Delivery impact    | Development and testing effort               |
| Operational impact | Monitoring, support, rollback                |
| Decision           | Selected or rejected                         |
| Reason             | Why the option was selected or rejected      |

Include at least:

1. Minimal-change option.
2. Recommended architecture option.
3. Alternative or long-term option, when applicable.

### 10. Recommended Solution

State the selected option and explain:

* Why it is preferred.
* How it satisfies the requirements.
* Why the other options were rejected.
* Expected implementation complexity.
* Key trade-offs.
* Any accepted limitations.

### 11. Target Architecture

Describe the proposed architecture at the appropriate levels.

#### System Context

Show the affected systems, actors, and external dependencies.

#### Component or Container Design

Show:

* Frontend.
* API Gateway or controllers.
* Backend services.
* Databases.
* Queues or event systems.
* External APIs.
* Authentication services.
* Monitoring systems.

#### Deployment Architecture

Describe:

* Cloud services.
* Runtime environments.
* Network boundaries.
* IAM roles and permissions.
* Secrets and encryption.
* Environment-specific configuration.

Use Mermaid diagrams where useful.

### 12. Detailed Processing Flow

Document the end-to-end workflow.

For example:

```text
User action
→ Frontend validation
→ API request
→ Authentication
→ Authorization and tenant validation
→ Business-rule evaluation
→ Database or external-service operation
→ Audit and logging
→ API response
→ UI update
```

Include:

* Main success flow.
* Validation failure flow.
* Authorization failure flow.
* Dependency failure flow.
* Retry flow.
* Timeout flow.
* Partial-success behaviour.
* Rollback or compensation flow.

### 13. Component-Level Changes

For every affected component, document:

| Component      | Current Behaviour      | Proposed Change         | Key Files                | Impact            |
| -------------- | ---------------------- | ----------------------- | ------------------------ | ----------------- |
| Frontend       | Current implementation | UI or state changes     | File paths               | User impact       |
| API            | Existing contract      | Endpoint changes        | File paths               | Compatibility     |
| Service        | Existing logic         | New business logic      | File paths               | Processing impact |
| Database       | Current schema         | Schema or query changes | Models/migrations        | Data impact       |
| Infrastructure | Current resources      | IAM/config changes      | Terraform/CloudFormation | Deployment impact |

Codex should reference actual repository file paths and symbols.

### 14. Frontend Design

Cover:

* Pages and components affected.
* User workflow.
* Form and client-side validation.
* State management.
* API integration.
* Loading, empty, error, and permission states.
* Read-only versus editable states.
* Accessibility.
* Feature visibility and role-based controls.
* Backward compatibility.

### 15. Backend Design

Cover:

* Controllers, handlers, and Lambda functions.
* Service-layer responsibilities.
* Domain or business logic.
* Validation.
* Authorization.
* Tenant, company, or zone scoping.
* Transaction handling.
* Idempotency.
* Concurrency control.
* Retry and timeout handling.
* Dependency isolation.
* Error mapping.

### 16. API Contract

For each endpoint, specify:

* HTTP method and path.
* Authentication requirements.
* Required roles and permissions.
* Request headers.
* Path and query parameters.
* Request body.
* Response body.
* Error responses.
* Idempotency behaviour.
* Pagination, filtering, and sorting.
* Backward compatibility.

Example:

```text
POST /api/example

Authentication:
- Valid JWT required

Authorization:
- Global Admin or Zone Admin
- Zone access must be validated

Request:
{
  "zoneId": 123,
  "enabled": true
}

Responses:
- 200: Updated successfully
- 400: Invalid request
- 401: Invalid authentication
- 403: Insufficient permission
- 404: Resource not found
- 409: Stale or conflicting update
- 500: Unexpected internal error
```

### 17. Data Model and Persistence

Document:

* Existing tables and models.
* New or modified attributes.
* Relationships.
* Indexes.
* Validation constraints.
* Default values.
* Data ownership and tenant scoping.
* Audit fields.
* Migration requirements.
* Rollback compatibility.
* Data-retention requirements.

Include example schemas where useful.

### 18. Business Rules and Decision Logic

Define rules precisely and deterministically.

For each rule, specify:

* Inputs.
* Preconditions.
* Evaluation order.
* Output.
* Default behaviour.
* Tie-breaking logic.
* Null or missing-data behaviour.
* Failure behaviour.
* Examples.
* Edge cases.

Avoid leaving business logic only in diagrams or prose. Include pseudocode when needed.

### 19. Configuration and Inheritance

Where the feature supports configuration, document:

* Global defaults.
* Customer, company, tenant, or zone overrides.
* Inheritance behaviour.
* Enable and disable behaviour.
* Existing versus newly created customer behaviour.
* Configuration precedence.
* Cache behaviour.
* Configuration invalidation.
* Permission to view and modify configuration.

### 20. Security Design

Cover:

#### Authentication

* Token type and issuer.
* Token validation.
* Expiration and signature validation.
* Service-to-service authentication.

#### Authorization

* Required user roles.
* Resource-level access.
* Tenant and zone scoping.
* Prevention of horizontal privilege escalation.

#### Data Protection

* Encryption in transit.
* Encryption at rest.
* KMS or key requirements.
* Secret storage.
* PII handling.
* Sensitive-data masking.

#### Security Controls

* Input validation.
* Output encoding.
* Injection prevention.
* SSRF protection where applicable.
* Rate limiting.
* Audit logging.
* Least-privilege IAM permissions.

#### Threat Scenarios

Include misuse and abuse cases relevant to the feature.

### 21. Performance and Scalability

Document:

* Expected request volume.
* Maximum dataset size.
* Latency target.
* Throughput target.
* Database-query impact.
* External API limits.
* Lambda concurrency or service scaling.
* Connection-pool impact.
* Cache strategy.
* Batch-size limits.
* Pagination requirements.

Include load assumptions and worst-case calculations where possible.

### 22. Reliability and Failure Handling

Define behaviour for:

* Database unavailable.
* External service unavailable.
* Timeout.
* Invalid or incomplete data.
* Duplicate request.
* Partial operation.
* Retry exhaustion.
* Audit persistence failure.
* Notification failure.

Specify whether the feature should:

* Fail open.
* Fail closed.
* Preserve the current result.
* Retry.
* Queue for later processing.
* Return a partial result.
* Require manual intervention.

### 23. Logging, Monitoring, and Audit

Define:

* Structured log fields.
* Correlation or trace IDs.
* Security audit events.
* Business audit events.
* Metrics.
* Dashboards.
* Alarms.
* Error thresholds.
* Sensitive-data redaction.
* Support and troubleshooting information.

### 24. Backward Compatibility and Migration

Cover:

* Existing customer behaviour.
* New customer behaviour.
* API compatibility.
* Database compatibility.
* Configuration migration.
* Historical-data treatment.
* Deployment ordering.
* Mixed-version operation.
* Rollback behaviour.

### 25. Testing Strategy

#### Unit Testing

* Business rules.
* Validation.
* Error handling.
* Permission checks.

#### Integration Testing

* API and database integration.
* External-service integration.
* Authentication and authorization.
* Configuration inheritance.

#### End-to-End Testing

* Main user workflows.
* Role and tenant scenarios.
* Failure scenarios.

#### Non-Functional Testing

* Performance.
* Load.
* Stress.
* Security.
* Concurrency.
* Recovery.
* Regression.

#### Environment Validation

Define validation required in:

* Local or development.
* Staging.
* UAT.
* Production smoke testing.

### 26. Deployment and Release Plan

Document:

* Deployment sequence.
* Infrastructure prerequisites.
* Database migration ordering.
* Feature flag or enablement strategy.
* Environment configuration.
* Release approvals.
* Smoke tests.
* Monitoring period.
* Rollback conditions.
* Rollback steps.
* Support communication.

### 27. Risks and Mitigations

Use a table such as:

| Risk                            | Likelihood |   Impact | Mitigation                   | Contingency            | Owner         |
| ------------------------------- | ---------: | -------: | ---------------------------- | ---------------------- | ------------- |
| Existing behaviour regression   |     Medium |     High | Regression tests             | Disable or rollback    | Tech Lead     |
| External dependency unavailable |     Medium |     High | Timeout and retry            | Preserve existing flow | Backend Lead  |
| Incorrect tenant access         |        Low | Critical | Server-side scope validation | Disable feature        | Security Lead |

### 28. Trade-Offs and Technical Debt

Describe:

* Trade-offs accepted by the recommended solution.
* Temporary implementation decisions.
* Deferred improvements.
* Long-term architectural direction.
* Follow-up Jira tickets.

### 29. Implementation Plan and Jira Breakdown

Suggested ticket groups:

1. Requirement and repository investigation.
2. Technical solution and approval.
3. Security review and approval.
4. Backend implementation.
5. Frontend implementation.
6. Database migration.
7. Infrastructure and IAM changes.
8. Unit and integration testing.
9. QC test preparation and execution.
10. Performance and security validation.
11. Release preparation and approval.
12. Production deployment and post-release validation.

For each ticket, include:

* Scope.
* Dependencies.
* Acceptance criteria.
* Technical notes.
* Estimate.
* Owner role.

### 30. Open Questions and Decisions Required

Track unresolved items:

| Question                                            | Impact                        | Owner         | Required By        | Status |
| --------------------------------------------------- | ----------------------------- | ------------- | ------------------ | ------ |
| Should existing customers be enabled automatically? | Migration and customer impact | Product Owner | Before development | Open   |
| Is a new IAM permission required?                   | Deployment dependency         | DevOps        | Before staging     | Open   |

### 31. Approval Checklist

* Product requirements approved.
* UX design approved.
* Technical solution approved.
* Security review approved.
* Test strategy approved.
* Performance evidence accepted.
* Infrastructure changes approved.
* Release plan approved.
* Production release authorized.

---

## Codex Prompt Template

The following prompt instructs Codex to inspect the repository and produce the technical solution using the outline.

Act as a Senior Technical Architect and Senior Full-Stack Engineer.

Your task is to investigate the following feature and produce a repository-grounded technical solution document.

FEATURE

Feature name:
[FEATURE NAME]

Business requirement:
[PASTE THE BUSINESS REQUIREMENT]

Acceptance criteria:
[PASTE ACCEPTANCE CRITERIA]

Known constraints:
[LIST KNOWN CONSTRAINTS]

Relevant repositories or folders:
[LIST REPOSITORIES OR FOLDERS, IF KNOWN]

DELIVERABLE

Create a detailed technical solution using the structure below:

1. Document Overview
2. Executive Summary
3. Problem Statement and Current Situation
4. Goals and Non-Goals
5. Scope
6. Requirements and Acceptance Criteria
7. Assumptions, Constraints, and Dependencies
8. Current-State Technical Analysis
9. Options Considered
10. Recommended Solution
11. Target Architecture
12. Detailed Processing Flow
13. Component-Level Changes
14. Frontend Design
15. Backend Design
16. API Contract
17. Data Model and Persistence
18. Business Rules and Decision Logic
19. Configuration and Inheritance
20. Security Design
21. Performance and Scalability
22. Reliability and Failure Handling
23. Logging, Monitoring, and Audit
24. Backward Compatibility and Migration
25. Testing Strategy
26. Deployment and Release Plan
27. Risks and Mitigations
28. Trade-Offs and Technical Debt
29. Implementation Plan and Jira Breakdown
30. Open Questions and Decisions Required
31. Approval Checklist

REPOSITORY INVESTIGATION RULES

Before proposing the solution:

1. Search the repository for all code related to the feature.
2. Identify the complete current request and data flow.
3. Inspect frontend, backend, APIs, database models, infrastructure, configuration, authentication, authorization, logging, and tests.
4. Identify reusable components and existing implementation patterns.
5. Find all callers and downstream consumers of any code or contract proposed for modification.
6. Check whether similar functionality already exists elsewhere in the repository.
7. Identify existing feature flags, tenant or zone settings, inheritance mechanisms, and permission checks.
8. Inspect Terraform, CloudFormation, deployment files, environment variables, IAM policies, and secrets where relevant.
9. Inspect existing tests to determine current coverage and expected behaviour.
10. Do not design the target solution until the current implementation has been investigated.

EVIDENCE REQUIREMENTS

For every important finding, provide:

* Repository path.
* Class, function, component, endpoint, table, or resource name.
* Relevant line number or approximate line range when available.
* Explanation of how the code currently behaves.
* Explanation of why the file is relevant.

Clearly label information as one of:

* Confirmed from repository.
* Inferred from repository.
* Proposed change.
* Assumption requiring confirmation.
* Open question.

Do not present assumptions as confirmed facts.

Do not invent files, APIs, database fields, services, permissions, or system behaviour.

When repository evidence is unavailable, explicitly state:

“Not confirmed from the current repository evidence.”

CURRENT-STATE ANALYSIS

Document:

* Entry points.
* Frontend workflow.
* API request flow.
* Authentication and authorization.
* Tenant, company, or zone scoping.
* Business logic.
* Database access.
* External integrations.
* Configuration lookup.
* Error handling.
* Logging and audit.
* Infrastructure dependencies.
* Existing automated tests.

Include a Mermaid sequence diagram for the current flow.

OPTIONS ANALYSIS

Evaluate at least three options when technically relevant:

1. Minimal-change option.
2. Recommended maintainable solution.
3. Alternative or longer-term architecture.

For each option, compare:

* Architecture.
* Implementation effort.
* Security.
* Performance.
* Reliability.
* Backward compatibility.
* Operational complexity.
* Testing effort.
* Release risk.
* Advantages.
* Disadvantages.

Select one recommended option and explain the decision.

TARGET SOLUTION

The recommended solution must specify:

* Components to add, modify, or remove.
* Exact responsibilities of each component.
* Frontend state and user flow.
* Backend processing flow.
* API contracts.
* Data-model changes.
* Business rules.
* Validation rules.
* Permission and tenant-scoping rules.
* Configuration and inheritance behaviour.
* Error handling.
* Retry and timeout behaviour.
* Idempotency and concurrency handling.
* Logging, monitoring, metrics, and audit.
* Backward compatibility.
* Migration and rollback.

Include Mermaid diagrams for:

1. Target component architecture.
2. Main success sequence.
3. Important failure flow, when applicable.

SECURITY REVIEW

Analyze:

* Authentication.
* Role-based authorization.
* Resource-level authorization.
* Tenant or zone isolation.
* Horizontal privilege-escalation risks.
* Input validation.
* Sensitive-data handling.
* PII exposure.
* Encryption.
* Secrets.
* IAM permissions.
* Audit events.
* Abuse and misuse scenarios.

List required security controls and security test cases.

PERFORMANCE AND RELIABILITY

Estimate or identify:

* Expected request volume.
* Dataset size.
* Database-query impact.
* Connection impact.
* External API limits.
* Concurrency.
* Timeout requirements.
* Retry behaviour.
* Cache requirements.
* Batch or pagination limits.
* Failure behaviour.

Do not invent numeric performance targets. Where targets are unavailable, identify them as open questions and recommend what should be measured.

TESTING STRATEGY

Include:

* Unit tests.
* Component tests.
* API integration tests.
* Database integration tests.
* Frontend tests.
* End-to-end tests.
* Permission and tenant-isolation tests.
* Negative tests.
* Regression tests.
* Performance tests.
* Security tests.
* Deployment smoke tests.
* Production validation.

Map the important acceptance criteria to test scenarios.

IMPLEMENTATION PLAN

Produce a Jira-ready ticket breakdown.

For every ticket provide:

* Title.
* Objective.
* Scope.
* Main implementation tasks.
* Acceptance criteria.
* Dependencies.
* Risks.
* Suggested owner role.
* Estimated complexity or story points.
* Files or components likely to change.

Separate tickets for:

* Investigation.
* Technical solution approval.
* Security review.
* Backend implementation.
* Frontend implementation.
* Database changes.
* Infrastructure changes.
* Automated tests.
* QC execution.
* Performance validation.
* Release preparation.
* Production validation.

OUTPUT QUALITY RULES

* Use clear technical English.
* Prefer tables for comparisons and structured information.
* Use pseudocode for complicated decision logic.
* Include example API payloads where relevant.
* Separate current behaviour from proposed behaviour.
* Highlight breaking changes.
* Highlight security-critical changes.
* Highlight production-release blockers.
* Keep the design consistent with existing repository patterns unless there is a strong reason not to.
* Avoid unnecessary architecture changes.
* Avoid unrelated refactoring.
* State all unresolved decisions in the Open Questions section.
* End with a concise recommendation and approval status.

Start by presenting the repository investigation findings. Then produce the proposed technical solution.

A lighter Codex prompt can be created from this by keeping sections 3, 8, 10–18, 20, 25–27, and 29 when the feature is small.




Act as a Senior Technical Architect and Senior Full-Stack Engineer.

Your task is to investigate the following feature and produce a repository-grounded technical solution document.

FEATURE

Feature name:
[FEATURE NAME]

Business requirement:
[PASTE THE BUSINESS REQUIREMENT]

Acceptance criteria:
[PASTE ACCEPTANCE CRITERIA]

Known constraints:
[LIST KNOWN CONSTRAINTS]

Relevant repositories or folders:
[LIST REPOSITORIES OR FOLDERS, IF KNOWN]

DELIVERABLE

Create a detailed technical solution using the structure below:

1. Document Overview
2. Executive Summary
4. Goals and Non-Goals
5. Scope
6. Requirements and Acceptance Criteria
7. Assumptions, Constraints, and Dependencies
8. Current-State Technical Analysis
9. Options Considered
10. Recommended Solution
11. Target Architecture
12. Detailed Processing Flow
13. Component-Level Changes
14. Frontend Design
15. Backend Design
16. API Contract
17. Data Model and Persistence
18. Business Rules and Decision Logic
19. Configuration and Inheritance
20. Security Design
21. Performance and Scalability
22. Reliability and Failure Handling
23. Logging, Monitoring, and Audit
24. Backward Compatibility and Migration
25. Testing Strategy
27. Risks and Mitigations
28. Trade-Offs and Technical Debt
31. Approval Checklist

REPOSITORY INVESTIGATION RULES

Before proposing the solution:

1. Search the repository for all code related to the feature.
2. Identify the complete current request and data flow.
3. Inspect frontend, backend, APIs, database models, infrastructure, configuration, authentication, authorization, logging, and tests.
4. Identify reusable components and existing implementation patterns.
5. Find all callers and downstream consumers of any code or contract proposed for modification.
6. Check whether similar functionality already exists elsewhere in the repository.
7. Identify existing feature flags, tenant or zone settings, inheritance mechanisms, and permission checks.
8. Inspect Terraform, CloudFormation, deployment files, environment variables, IAM policies, and secrets where relevant.
9. Inspect existing tests to determine current coverage and expected behaviour.
10. Do not design the target solution until the current implementation has been investigated.

EVIDENCE REQUIREMENTS

For every important finding, provide:

* Repository path.
* Class, function, component, endpoint, table, or resource name.
* Relevant line number or approximate line range when available.
* Explanation of how the code currently behaves.
* Explanation of why the file is relevant.

Clearly label information as one of:

* Confirmed from repository.
* Inferred from repository.
* Proposed change.
* Assumption requiring confirmation.
* Open question.

Do not present assumptions as confirmed facts.

Do not invent files, APIs, database fields, services, permissions, or system behaviour.

When repository evidence is unavailable, explicitly state:

“Not confirmed from the current repository evidence.”

CURRENT-STATE ANALYSIS

Document:

* Entry points.
* Frontend workflow.
* API request flow.
* Authentication and authorization.
* Tenant, company, or zone scoping.
* Business logic.
* Database access.
* External integrations.
* Configuration lookup.
* Error handling.
* Logging and audit.
* Infrastructure dependencies.
* Existing automated tests.

Include a Mermaid sequence diagram for the current flow.

OPTIONS ANALYSIS

Evaluate at least three options when technically relevant:

1. Minimal-change option.
2. Recommended maintainable solution.
3. Alternative or longer-term architecture.

For each option, compare:

* Architecture.
* Implementation effort.
* Security.
* Performance.
* Reliability.
* Backward compatibility.
* Operational complexity.
* Testing effort.
* Release risk.
* Advantages.
* Disadvantages.

Select one recommended option and explain the decision.

TARGET SOLUTION

The recommended solution must specify:

* Components to add, modify, or remove.
* Exact responsibilities of each component.
* Frontend state and user flow.
* Backend processing flow.
* API contracts.
* Data-model changes.
* Business rules.
* Validation rules.
* Permission and tenant-scoping rules.
* Configuration and inheritance behaviour.
* Error handling.
* Retry and timeout behaviour.
* Idempotency and concurrency handling.
* Logging, monitoring, metrics, and audit.
* Backward compatibility.
* Migration and rollback.

Include Mermaid diagrams for:

1. Target component architecture.
2. Main success sequence.
3. Important failure flow, when applicable.

SECURITY REVIEW

Analyze:

* Authentication.
* Role-based authorization.
* Resource-level authorization.
* Tenant or zone isolation.
* Horizontal privilege-escalation risks.
* Input validation.
* Sensitive-data handling.
* PII exposure.
* Encryption.
* Secrets.
* IAM permissions.
* Audit events.
* Abuse and misuse scenarios.

List required security controls and security test cases.

PERFORMANCE AND RELIABILITY

Estimate or identify:

* Expected request volume.
* Dataset size.
* Database-query impact.
* Connection impact.
* External API limits.
* Concurrency.
* Timeout requirements.
* Retry behaviour.
* Cache requirements.
* Batch or pagination limits.
* Failure behaviour.

Do not invent numeric performance targets. Where targets are unavailable, identify them as open questions and recommend what should be measured.

TESTING STRATEGY

Include:

* Unit tests.
* Component tests.
* API integration tests.
* Database integration tests.
* Frontend tests.
* End-to-end tests.
* Permission and tenant-isolation tests.
* Negative tests.
* Regression tests.
* Performance tests.
* Security tests.
* Deployment smoke tests.
* Production validation.

Map the important acceptance criteria to test scenarios.

IMPLEMENTATION PLAN

Produce a Jira-ready ticket breakdown.

For every ticket provide:

* Title.
* Objective.
* Scope.
* Main implementation tasks.
* Acceptance criteria.
* Dependencies.
* Risks.
* Suggested owner role.
* Estimated complexity or story points.
* Files or components likely to change.

Separate tickets for:

* Investigation.
* Technical solution approval.
* Security review.
* Backend implementation.
* Frontend implementation.
* Database changes.
* Infrastructure changes.
* Automated tests.
* QC execution.
* Performance validation.
* Release preparation.
* Production validation.

OUTPUT QUALITY RULES

* Use clear technical English.
* Prefer tables for comparisons and structured information.
* Use pseudocode for complicated decision logic.
* Include example API payloads where relevant.
* Separate current behaviour from proposed behaviour.
* Highlight breaking changes.
* Highlight security-critical changes.
* Highlight production-release blockers.
* Keep the design consistent with existing repository patterns unless there is a strong reason not to.
* Avoid unnecessary architecture changes.
* Avoid unrelated refactoring.
* State all unresolved decisions in the Open Questions section.
* End with a concise recommendation and approval status.

Start by presenting the repository investigation findings. Then produce the proposed technical solution.

------------------------------------------------------------------------------------------------------------------SMALL FEATURE-----------------
Yes. For a **small feature**, the current outline is too detailed and contains several overlapping sections. It can be reduced from 27 sections to **10 practical sections** without losing important technical coverage.

DELIVERABLE

Create a detailed technical solution using the structure below:

## Recommended Reduced Outline

|  # | Section                                        | Includes                                                                                          |
| -: | ---------------------------------------------- | ------------------------------------------------------------------------------------------------- |
|  1 | **Overview and Summary**                       | Document overview, executive summary, feature background                                          |
|  2 | **Goals, Scope, and Requirements**             | Goals, non-goals, in-scope, out-of-scope, requirements, acceptance criteria                       |
|  3 | **Assumptions, Constraints, and Dependencies** | Technical assumptions, limitations, external dependencies, pending decisions                      |
|  4 | **Current-State Analysis**                     | Existing architecture, current processing flow, relevant code, limitations, technical gap         |
|  5 | **Options and Recommendation**                 | Options considered, pros and cons, recommended solution, trade-offs                               |
|  6 | **Target Technical Design**                    | Target architecture, detailed flow, frontend changes, backend changes, component responsibilities |
|  7 | **Contracts, Data, and Business Logic**        | API contract, data model, persistence, decision rules, configuration, inheritance                 |
|  8 | **Security, Reliability, and Operations**      | Security, performance, scalability, failure handling, logging, monitoring, audit                  |
|  9 | **Compatibility, Testing, and Risks**          | Backward compatibility, migration, testing strategy, risks, mitigations, technical debt           |
| 10 | **Approval Checklist**                         | Technical, security, QC, DevOps, Product Owner, and release approvals                             |

## Consolidation Mapping

### 1. Overview and Summary

Merge:

* Document Overview
* Executive Summary

These sections are usually short and do not need to be separate for a small feature.

### 2. Goals, Scope, and Requirements

Merge:

* Goals and Non-Goals
* Scope
* Requirements and Acceptance Criteria

These describe what the feature must achieve and its boundaries.

### 3. Assumptions, Constraints, and Dependencies

Keep as one section because it provides important context before the design.

### 4. Current-State Analysis

Keep:

* Current-State Technical Analysis

The current architecture and processing flow can be included in the same section rather than creating a separate current-flow section.

### 5. Options and Recommendation

Merge:

* Options Considered
* Recommended Solution
* Trade-Offs and Technical Debt

The trade-offs should be explained as part of the option decision. Any deferred technical debt can also be recorded here or in the final risk section.

### 6. Target Technical Design

Merge:

* Target Architecture
* Detailed Processing Flow
* Component-Level Changes
* Frontend Design
* Backend Design

For a small feature, separate frontend and backend design chapters create unnecessary repetition. They can be subsections under one target-design section.

Suggested subsections:

```text
6.1 Target Architecture
6.2 End-to-End Processing Flow
6.3 Frontend Changes
6.4 Backend Changes
6.5 Component Change Summary
```

### 7. Contracts, Data, and Business Logic

Merge:

* API Contract
* Data Model and Persistence
* Business Rules and Decision Logic
* Configuration and Inheritance

These areas define how information enters, is processed, is stored, and is configured.

Suggested subsections:

```text
7.1 API Changes
7.2 Data Changes
7.3 Business Rules
7.4 Configuration and Inheritance
```

Sections that are not affected should contain a simple statement such as:

> No API contract change is required.

### 8. Security, Reliability, and Operations

Merge:

* Security Design
* Performance and Scalability
* Reliability and Failure Handling
* Logging, Monitoring, and Audit

These are the main non-functional and operational concerns.

Suggested subsections:

```text
8.1 Security
8.2 Performance and Scalability
8.3 Failure Handling
8.4 Logging, Monitoring, and Audit
```

For small changes, only include relevant findings rather than creating generic content for every subsection.

### 9. Compatibility, Testing, and Risks

Merge:

* Backward Compatibility and Migration
* Testing Strategy
* Risks and Mitigations
* Remaining Technical Debt

These topics are closely connected to implementation and release safety.

Suggested subsections:

```text
9.1 Backward Compatibility and Migration
9.2 Testing Strategy
9.3 Risks and Mitigations
9.4 Deferred Work
```

### 10. Approval Checklist

Keep:

* Approval Checklist

This can be very short and should only include approvals relevant to the feature.

## Final Small-Feature Outline

```text
1. Overview and Summary

2. Goals, Scope, and Requirements
   2.1 Goals and Non-Goals
   2.2 In Scope and Out of Scope
   2.3 Requirements and Acceptance Criteria

3. Assumptions, Constraints, and Dependencies

4. Current-State Analysis
   4.1 Current Architecture and Flow
   4.2 Relevant Repository Components
   4.3 Existing Limitations or Technical Gap

5. Options and Recommendation
   5.1 Options Considered
   5.2 Recommended Solution
   5.3 Trade-Offs

6. Target Technical Design
   6.1 Target Architecture
   6.2 End-to-End Processing Flow
   6.3 Frontend Changes
   6.4 Backend Changes
   6.5 Component Change Summary

7. Contracts, Data, and Business Logic
   7.1 API Changes
   7.2 Data and Persistence Changes
   7.3 Business Rules
   7.4 Configuration and Inheritance

8. Security, Reliability, and Operations
   8.1 Security
   8.2 Performance and Scalability
   8.3 Error and Failure Handling
   8.4 Logging, Monitoring, and Audit

9. Compatibility, Testing, and Risks
   9.1 Backward Compatibility and Migration
   9.2 Testing Strategy
   9.3 Risks and Mitigations
   9.4 Deferred Work or Technical Debt

10. Approval Checklist
```

This structure remains detailed enough for technical review while preventing Codex from producing an unnecessarily large enterprise-level document for a small feature.

