# Quickstart: Validate Backend API Deployment Workflow

## Prerequisites

- Branch: `003-deploy-backend-api`
- Required repository secret:
  - `AZURE_CREDENTIALS`
- Required repository variable:
  - `APP_SERVICE_NAME` (Linux B1 App Service name)

## Planned Workflow Location

- `.github/workflows/003-deploy-api.yml`

## Verification Checklist (Static)

1. Confirm trigger configuration:
   - `on.push.branches` contains only `main`.
   - `on.workflow_dispatch.inputs.environment` exists with `dev|qa|prod`.
2. Confirm workflow + job identity:
   - Workflow and job names clearly indicate backend API deployment.
3. Confirm environment/concurrency alignment with `001-deploy-infra.yml`:
   - `env.ENVIRONMENT: ${{ github.event.inputs.environment || 'dev' }}`
   - `concurrency.group: ${{ github.workflow }}-${{ github.ref }}`
   - `concurrency.cancel-in-progress: true`
   - `jobs.deploy-api.environment: ${{ github.event.inputs.environment || 'dev' }}`
4. Confirm build/package configuration:
   - Includes `actions/setup-dotnet@v4` with `.NET 10` (`10.0.x`).
   - Runs `dotnet publish` on `src/ai-genius-api` with:
     - `-c Release`
     - `-r linux-x64`
     - `--self-contained true`
     - output to `$PUBLISH_OUTPUT_DIR`
   - Creates zip package from `$PUBLISH_OUTPUT_DIR` to `$DEPLOY_ZIP_PATH`.
5. Confirm deployment configuration:
   - Uses Azure authentication via `azure/login@v1` and `${{ secrets.AZURE_CREDENTIALS }}`.
   - Uses `azure/webapps-deploy@v3`.
   - Reads app service name from `${{ vars.APP_SERVICE_NAME }}`.
   - Uses `${{ env.DEPLOY_ZIP_PATH }}` as package input.
6. Confirm required visible stage names exist:
   - `Checkout`
   - `Setup .NET`
   - `Publish API`
   - `Create deployment zip`
   - `Deploy to App Service`
7. Confirm failure semantics:
   - Build/package/deploy errors fail run.
   - Logs clearly identify failed stage from step name.

## Runtime Validation (After Implementation)

1. **Automatic trigger on `main`**
   - Push a commit to `main`.
   - Verify `003 Deploy Backend API to Azure` starts automatically.
   - Verify no run is triggered for non-`main` branch pushes.

2. **Build/package/deploy flow**
   - Open the run and verify stage order:
     `Checkout` → `Setup .NET` → `Publish API` → `Create deployment zip` → `Deploy to App Service`
   - Confirm `Publish API` uses `linux-x64` and `--self-contained true`.
   - Confirm the zip deployment package is created and consumed by deploy.
   - Confirm deployment targets `${{ vars.APP_SERVICE_NAME }}`.

3. **Concurrency and environment behavior**
   - Trigger two near-consecutive pushes to `main`.
   - Verify in-progress older run is cancelled (when superseded).
   - Trigger manual dispatch for each environment (`dev`, `qa`, `prod`) and verify selected environment is used.
   - Verify fallback environment resolves to `dev` when manual input is absent.

4. **Failure-stage visibility**
   - Intentionally break publish or deploy input in a test run.
   - Verify the workflow fails fast.
   - Verify the failed stage is obvious from step name and logs.

## Final Consistency Check

- Confirm this quickstart matches `.github/workflows/003-deploy-api.yml` exactly for trigger, step order, concurrency, environment resolution, auth, and deploy action inputs.
