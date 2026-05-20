# Contract: Backend API Deployment Workflow

## Scope

Defines required behavior for `.github/workflows/003-deploy-api.yml`.

## Trigger Contract

- Event: `push`
- Branch filter: `main` only
- Non-main pushes: MUST NOT trigger workflow

## Environment & Concurrency Contract

- Must align with `.github/workflows/001-deploy-infra.yml` pattern:
  - `ENVIRONMENT` resolved from `${{ github.event.inputs.environment || 'dev' }}`
  - `concurrency.group` = `${{ github.workflow }}-${{ github.ref }}`
  - `concurrency.cancel-in-progress` = `true`

## Build & Package Contract

- Required step order:
  1. `actions/checkout@v4`
  2. `actions/setup-dotnet@v4` with .NET 10
  3. `dotnet publish` for `src/ai-genius-api`
  4. zip artifact creation from publish output
  5. `azure/webapps-deploy@v3`
- `dotnet publish` requirements:
  - runtime: `linux-x64`
  - self-contained: `true`

## Deployment Contract

- Deployment action MUST be `azure/webapps-deploy@v3`
- `app-name` MUST be resolved from `${{ vars.APP_SERVICE_NAME }}`
- `package` MUST reference the generated zip artifact
- Authentication MUST use GitHub secret-backed Azure credentials (no inline secrets)

## Failure Signaling Contract

- Any publish, zip, or deploy failure MUST fail the workflow run
- Step naming MUST make build-vs-deploy failure stage obvious in run logs
