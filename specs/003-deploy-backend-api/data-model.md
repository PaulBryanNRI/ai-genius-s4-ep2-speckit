# Data Model: Deploy Backend API Workflow

## Entity: ApiDeploymentWorkflow

- **Description**: Declarative GitHub Actions workflow configuration for backend API deployment.
- **Fields**:
  - `workflowFile` (string, required): `.github/workflows/003-deploy-api.yml`
  - `triggerBranches` (string[], required): must include only `main`
  - `environmentName` (string, required): resolved via `github.event.inputs.environment || 'dev'`
  - `concurrencyGroup` (string expression, required): `${{ github.workflow }}-${{ github.ref }}`
  - `cancelInProgress` (boolean, required): `true`
  - `runner` (string, required): `ubuntu-latest`
  - `dotnetVersion` (string, required): `10.0.x`
  - `projectPath` (string, required): `src/ai-genius-api`
  - `deployAction` (string, required): `azure/webapps-deploy@v3`
  - `appServiceNameSource` (string, required): `vars.APP_SERVICE_NAME`
- **Validation Rules**:
  - `triggerBranches` must not include non-main branches.
  - `deployAction` must equal `azure/webapps-deploy@v3`.
  - `dotnetVersion` must resolve to .NET 10.

## Entity: PublishArtifact

- **Description**: Packaged deployment artifact generated from backend API publish output.
- **Fields**:
  - `runtime` (string, required): `linux-x64`
  - `selfContained` (boolean, required): `true`
  - `publishDirectory` (string, required): workflow-defined output path
  - `zipPath` (string, required): zip package consumed by deploy step
- **Validation Rules**:
  - `runtime` must be `linux-x64`.
  - `zipPath` must point to an existing zip created after publish step.

## Entity: DeploymentRunOutcome

- **Description**: Observable result of a workflow run for maintainers.
- **Fields**:
  - `runId` (string, required)
  - `status` (enum, required): `queued | in_progress | success | failure | cancelled`
  - `failedStage` (enum, optional): `build | package | deploy`
  - `logsUrl` (string, required)
- **Validation Rules**:
  - `status=failure` requires `failedStage` to be identifiable from step outputs.
  - `status=success` requires successful completion of build, package, and deploy.

## State Transitions

- `queued` → `in_progress`
- `in_progress` → `success` (all required steps complete)
- `in_progress` → `failure` (any required step fails)
- `in_progress` → `cancelled` (concurrency cancellation on newer run)
