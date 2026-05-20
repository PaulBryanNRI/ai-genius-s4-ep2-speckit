# Feature Specification: Deploy Backend API Workflow

**Feature Branch**: `003-deploy-backend-api`  
**Created**: 2026-05-20  
**Status**: Draft  
**Input**: User description: "Deploy the AI Genius backend API via GitHub Actions with automatic main-branch deployment and Azure App Service release flow."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Automatic API Deployment on Main (Priority: P1)

As a maintainer, I want every push to main to automatically run the API deployment workflow so that backend releases are consistently deployed without manual execution.

**Why this priority**: This is the core value: reliable deployment of backend changes as part of the mainline delivery flow.

**Independent Test**: Push a commit to main and verify a deployment workflow run starts automatically and reaches a clear success or failure status.

**Acceptance Scenarios**:

1. **Given** a commit is pushed to main, **When** workflows are evaluated, **Then** the API deployment workflow starts automatically.
2. **Given** the workflow starts, **When** it completes, **Then** maintainers can confirm whether deployment succeeded or failed from workflow results.

---

### User Story 2 - Consistent Build Packaging for Deployment (Priority: P2)

As a maintainer, I want the backend to be packaged in a deployment-ready format for Linux runtime so that the deployed API artifact is compatible with the target app service.

**Why this priority**: Even if triggering works, deployment is not useful unless the artifact format matches the target runtime expectations.

**Independent Test**: Run the workflow and verify the produced build package is self-contained and targeted for linux-x64 before deployment is attempted.

**Acceptance Scenarios**:

1. **Given** the workflow is running, **When** the build stage executes, **Then** the backend project at `src/ai-genius-api` is built as linux-x64 and self-contained.
2. **Given** the build stage succeeds, **When** deployment begins, **Then** the generated package is used as the deployment input.

---

### User Story 3 - Safe, Non-Overlapping Deployments (Priority: P3)

As a maintainer, I want the workflow to follow the established environment and concurrency behavior so that deployments remain consistent with existing repository deployment controls.

**Why this priority**: Consistency with current deployment controls reduces operational risk and prevents conflicting runs.

**Independent Test**: Compare workflow behavior to the established deployment pattern and run two close-together pushes to verify concurrency handling is applied.

**Acceptance Scenarios**:

1. **Given** multiple pushes happen in close succession, **When** deployment workflows are queued, **Then** concurrency behavior prevents conflicting parallel deployments for the same reference.
2. **Given** no manual environment input is provided, **When** the workflow evaluates deployment context, **Then** it resolves environment behavior using the same pattern as the existing infrastructure deployment workflow.

---

### Edge Cases

- A push to a non-main branch must not trigger this deployment workflow.
- If required deployment configuration values are missing, the workflow must fail with a clear reason rather than appearing successful.
- If a new push to main occurs while a deployment is running, concurrency rules must determine which run continues and which run is canceled.
- If build succeeds but deployment fails, the workflow result must still be failed and clearly identify the deployment failure.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The system MUST define a GitHub Actions workflow file at `.github/workflows/003-deploy-api.yml` dedicated to backend API deployment.
- **FR-002**: The workflow MUST trigger on every push to the `main` branch.
- **FR-003**: The workflow MUST include environment context behavior aligned with the pattern used in `.github/workflows/001-deploy-infra.yml`.
- **FR-004**: The workflow MUST include concurrency controls aligned with the pattern used in `.github/workflows/001-deploy-infra.yml`.
- **FR-005**: The workflow MUST build the backend project located at `src/ai-genius-api`.
- **FR-006**: The workflow MUST produce a self-contained build targeted to `linux-x64`.
- **FR-007**: The workflow MUST deploy the built API artifact to Azure App Service.
- **FR-008**: The workflow MUST use `azure/webapps-deploy@v3` for the deployment step.
- **FR-009**: The workflow MUST mark the run as failed when build or deployment steps fail.
- **FR-010**: The workflow MUST expose enough run output for maintainers to identify whether failure occurred during build preparation or deployment.

### Key Entities

- **API Deployment Workflow**: The automation definition responsible for triggering, building, and deploying the backend API.
- **Backend API Project**: The source project at `src/ai-genius-api` that is packaged and deployed.
- **Deployment Artifact**: The self-contained linux-x64 package generated from the backend project for deployment.
- **Deployment Run Outcome**: The success/failure result and logs that communicate deployment status to maintainers.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: 100% of pushes to `main` trigger a run of the API deployment workflow.
- **SC-002**: 100% of successful workflow runs deploy the backend API to the target app service without manual intervention.
- **SC-003**: 100% of workflow runs use the backend source located at `src/ai-genius-api` and package it as a self-contained linux-x64 artifact.
- **SC-004**: 100% of failed runs are visibly marked failed and indicate whether failure occurred in build or deployment stages.
- **SC-005**: During a controlled test of two consecutive pushes to `main`, concurrency behavior prevents ambiguous overlapping deployment outcomes for the same ref.

## Dependencies

- GitHub repository environments, variables, and secrets needed for deployment are configured and accessible to workflow runs.
- Azure App Service target is provisioned and reachable for deployment operations.
- Workflow permissions allow authentication and deployment actions to execute successfully.

## Assumptions

- The repository already has required deployment credentials and app service identifiers configured in GitHub repository settings.
- The existing infrastructure deployment workflow (`001-deploy-infra.yml`) is the source of truth for environment and concurrency behavior patterns.
- The backend API remains a single deployable unit rooted at `src/ai-genius-api`.
