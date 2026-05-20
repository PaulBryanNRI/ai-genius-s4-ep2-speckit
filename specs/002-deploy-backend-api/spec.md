# Feature Specification: Deploy Backend API Workflow

**Feature Branch**: `002-deploy-backend-api`  
**Created**: 2026-05-20  
**Status**: Draft  
**Input**: User description: "Deploy the AI Genius backend API via GitHub Actions. The backend is a .NET API in `src/ai-genius-api`. New GitHub Actions workflow `.github/workflows/003-deploy-api.yml`"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Automatic Backend Deployment on Main Changes (Priority: P1)

As a maintainer, I want backend deployments to run automatically when changes are merged so that the API is consistently deployed without manual steps.

**Why this priority**: This is the core business value of the feature: dependable, repeatable API deployments.

**Independent Test**: Merge a backend change to the main branch and confirm a deployment run starts and completes with the backend published successfully.

**Acceptance Scenarios**:

1. **Given** a change is pushed to the main branch, **When** the deployment workflow is triggered, **Then** the backend deployment process runs automatically.
2. **Given** the workflow run finishes successfully, **When** maintainers review the run summary, **Then** it clearly indicates the backend deployment completed.

---

### User Story 2 - Manual Deployment When Needed (Priority: P2)

As a maintainer, I want to trigger deployment manually so that I can redeploy or validate deployment behavior without requiring a code push.

**Why this priority**: Manual execution is important for recovery, validation, and controlled operational workflows.

**Independent Test**: Start the workflow manually from the Actions interface and verify the backend deploys successfully.

**Acceptance Scenarios**:

1. **Given** a maintainer starts the workflow manually, **When** the run begins, **Then** the same deployment flow is executed as the automatic trigger.
2. **Given** no new code changes are present, **When** the manual run completes, **Then** the deployment still succeeds and reports status clearly.

---

### User Story 3 - Failure Visibility and Safe Halt (Priority: P3)

As a maintainer, I want deployment failures to stop the process and surface clear error information so that I can quickly diagnose and fix issues.

**Why this priority**: Reliable failure handling prevents unnoticed broken releases and reduces mean time to recovery.

**Independent Test**: Introduce a controlled deployment failure and verify the workflow exits as failed with clear diagnostic output.

**Acceptance Scenarios**:

1. **Given** a deployment step fails, **When** the workflow executes, **Then** the run is marked failed and no success state is reported.
2. **Given** a failed run, **When** maintainers review workflow output, **Then** they can identify which step failed and why.

---

### Edge Cases

- What happens when backend files are unchanged but workflow is triggered manually? The deployment should still execute and return a valid success or failure result.
- What happens when two deployment runs overlap? The workflow should avoid ambiguous state and clearly indicate each run outcome.
- What happens when required deployment settings are missing? The workflow should fail early with a clear configuration error.
- What happens when the repository path for the backend is incorrect? The workflow should fail before deployment and report path resolution failure.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST define a dedicated backend deployment workflow at `.github/workflows/003-deploy-api.yml`.
- **FR-002**: The workflow MUST support automatic triggering on updates to the primary integration branch.
- **FR-003**: The workflow MUST support manual triggering by authorized repository maintainers.
- **FR-004**: The workflow MUST deploy the backend API from `src/ai-genius-api`.
- **FR-005**: The workflow MUST clearly report deployment success status in run results.
- **FR-006**: The workflow MUST fail the run when any deployment-critical step fails.
- **FR-007**: The workflow MUST provide failure details sufficient for maintainers to identify the failed step.
- **FR-008**: The workflow MUST use a consistent deployment sequence so repeated runs behave predictably.
- **FR-009**: The workflow MUST complete without requiring direct interactive input during execution.
- **FR-010**: The workflow MUST operate using repository-configured deployment credentials and settings, without embedding secrets in workflow source.

### Key Entities

- **Deployment Workflow**: The automation definition that governs backend deployment triggers, steps, and outcomes.
- **Backend API Artifact**: The deployable output generated from the backend project at `src/ai-genius-api`.
- **Deployment Run Record**: The execution result containing status, logs, and failure context for each workflow run.
- **Deployment Configuration**: Repository-level deployment settings and credentials used during workflow execution.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of qualifying pushes to the primary integration branch automatically start a backend deployment run.
- **SC-002**: Maintainers can start a manual deployment run in under 1 minute from opening the workflow page.
- **SC-003**: At least 95% of successful runs complete end-to-end deployment in under 10 minutes.
- **SC-004**: 100% of failed runs are marked failed and include a clear failed-step indicator in run output.
- **SC-005**: Within the first month of adoption, manual ad-hoc deployment steps outside the workflow are reduced to zero for routine releases.

## Dependencies

- Access to a deployment target environment that accepts backend API releases.
- Repository deployment credentials and required environment settings configured prior to workflow execution.
- GitHub Actions enabled for the repository with permissions to execute deployment runs.

## Assumptions

- The backend service currently deploys as a single API unit from `src/ai-genius-api`.
- The repository already has an established primary branch used for production-facing integration.
- Authorized maintainers have permission to run workflows manually.
- Existing deployment credentials are valid and have sufficient permissions for API deployment.
