# Deployment Requirements Checklist: Deploy Backend API Workflow

**Purpose**: Validate deployment workflow requirements for completeness, clarity, consistency, measurability, and scenario coverage.  
**Created**: 2026-05-20  
**Feature**: [Deploy Backend API Workflow Specification](../spec.md)

**Note**: This checklist evaluates requirement quality only (not implementation behavior).

## Requirement Completeness

- [ ] CHK001 Are trigger requirements fully specified for both included and excluded branches? [Completeness, Spec §FR-002, Spec §Edge Cases]
- [ ] CHK002 Are all required deployment stages explicitly required from source checkout through App Service deployment? [Completeness, Spec §FR-013]
- [ ] CHK003 Are required workflow inputs/secrets/variables fully enumerated with ownership and source of truth? [Completeness, Spec §FR-014, Spec §Dependencies]
- [ ] CHK004 Are required failure-reporting outputs specified for each major stage (publish, zip, deploy)? [Completeness, Spec §FR-009, Spec §FR-010]

## Requirement Clarity

- [ ] CHK005 Is “aligned with the pattern used in 001-deploy-infra.yml” defined with explicit fields that must match? [Clarity, Ambiguity, Spec §FR-003, Spec §FR-004]
- [ ] CHK006 Is “self-contained build targeted to linux-x64” defined with precise requirement wording that avoids interpretation drift? [Clarity, Spec §FR-006]
- [ ] CHK007 Is “clear reason” for configuration failure defined with minimum diagnostic detail expectations? [Clarity, Ambiguity, Spec §Edge Cases]
- [ ] CHK008 Is “enough run output” quantified so reviewers can objectively determine stage-level failure visibility? [Clarity, Ambiguity, Spec §FR-010]

## Requirement Consistency

- [ ] CHK009 Do branch-trigger requirements remain consistent between Functional Requirements and User Story acceptance scenarios? [Consistency, Spec §FR-002, Spec §User Story 1]
- [ ] CHK010 Do concurrency requirements align between FRs, edge cases, and success criteria without conflicting outcomes? [Consistency, Spec §FR-004, Spec §Edge Cases, Spec §SC-005]
- [ ] CHK011 Do deployment target requirements consistently reference Linux B1 across clarifications, FRs, and assumptions? [Consistency, Spec §Clarifications, Spec §FR-015, Spec §Assumptions]

## Acceptance Criteria Quality

- [ ] CHK012 Are all success criteria directly traceable to one or more functional requirements? [Traceability, Spec §SC-001..SC-007, Spec §FR-001..FR-015]
- [ ] CHK013 Are success metrics written so pass/fail can be judged objectively without implicit assumptions? [Measurability, Spec §SC-001..SC-007]
- [ ] CHK014 Is the acceptance criterion for “no manual intervention” bounded with explicit exceptions (if any)? [Clarity, Ambiguity, Spec §SC-002]

## Scenario Coverage

- [ ] CHK015 Are requirements complete for primary flow, alternate flow, and failure flow across trigger, package, and deploy stages? [Coverage, Spec §User Stories, Spec §Edge Cases]
- [ ] CHK016 Are recovery expectations defined when deployment fails after a successful publish/package stage? [Gap, Recovery, Spec §Edge Cases]
- [ ] CHK017 Are requirements explicit for back-to-back pushes and canceled-run observability under concurrency controls? [Coverage, Spec §User Story 3, Spec §SC-005]

## Edge Case Coverage

- [ ] CHK018 Are missing-variable and missing-secret behaviors both specified, including expected failure classification? [Coverage, Gap, Spec §Edge Cases, Spec §Dependencies]
- [ ] CHK019 Are non-main push outcomes defined beyond “must not trigger” (e.g., visibility/audit expectations)? [Coverage, Gap, Spec §Edge Cases]

## Non-Functional Requirements

- [ ] CHK020 Are reliability and timing expectations formally specified as enforceable NFRs rather than plan-only guidance? [Gap, Non-Functional, Plan §Performance Goals]
- [ ] CHK021 Are security requirements for credential handling, permission scope, and secret exposure prevention explicitly stated in the spec? [Gap, Security, Plan §Constitution Check]

## Dependencies & Assumptions

- [ ] CHK022 Are external dependency assumptions testable and assigned validation responsibility (repo config, Azure readiness, permissions)? [Dependencies, Assumption, Spec §Dependencies, Spec §Assumptions]
- [ ] CHK023 Is dependency ownership clear for `APP_SERVICE_NAME` lifecycle and environment-specific value management? [Dependency, Clarity, Spec §FR-014, Spec §Dependencies]

## Ambiguities & Conflicts

- [ ] CHK024 Is there any conflict between “push to main only” and environment-input patterns inherited from infra workflow conventions? [Conflict, Spec §FR-002, Spec §FR-003, Plan §Constraints]
- [ ] CHK025 Is terminology normalized for “artifact/package/zip deploy” so each term maps to one unambiguous requirement object? [Ambiguity, Spec §FR-007, Spec §FR-012, Spec §Key Entities]
