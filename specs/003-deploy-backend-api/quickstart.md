# Quickstart: Validate Backend API Deployment Workflow Plan

## Prerequisites

- Branch: `003-deploy-backend-api`
- Required repository secret:
  - `AZURE_CREDENTIALS`
- Required repository variable:
  - `APP_SERVICE_NAME` (Linux B1 App Service name)

## Planned Workflow Location

- `.github/workflows/003-deploy-api.yml`

## Verification Checklist

1. Confirm workflow trigger:
   - `on.push.branches` contains only `main`.
2. Confirm environment/concurrency alignment with `001-deploy-infra.yml`:
   - `ENVIRONMENT` fallback pattern matches.
   - Concurrency group and cancel behavior match.
3. Confirm build/package configuration:
   - Includes `actions/setup-dotnet@v4` for .NET 10.
   - Runs `dotnet publish` on `src/ai-genius-api` with `linux-x64` and `--self-contained true`.
   - Creates a zip package from publish output.
4. Confirm deployment configuration:
   - Uses `azure/webapps-deploy@v3`.
   - Reads app service name from `${{ vars.APP_SERVICE_NAME }}`.
   - Uses zip package as deploy input.
5. Confirm failure semantics:
   - Build/package/deploy errors fail run.
   - Step names make failed stage obvious.

## Runtime Validation (after implementation)

1. Push a commit to `main` and verify workflow starts automatically.
2. Confirm successful run deploys API to configured App Service.
3. Trigger two near-consecutive pushes to `main` and verify concurrency behavior avoids conflicting overlapping runs.
4. Intentionally break publish/deploy input in a test run and confirm workflow fails with stage-identifiable logs.
