# Phase 0 Research: Deploy Backend API Workflow

## Decision 1: Reuse environment and concurrency pattern from `001-deploy-infra.yml`

- **Decision**: Use `ENVIRONMENT: ${{ github.event.inputs.environment || 'dev' }}` style resolution and `concurrency.group: ${{ github.workflow }}-${{ github.ref }}` with `cancel-in-progress: true`.
- **Rationale**: The spec requires alignment with the established deployment pattern; reusing this structure ensures consistent behavior and low operational surprise.
- **Alternatives considered**:
  - Hard-code environment to `prod` only (rejected: diverges from established pattern).
  - Use no concurrency controls (rejected: violates FR-004 and increases deployment overlap risk).

## Decision 2: Publish API as self-contained linux-x64 with .NET 10

- **Decision**: Use `dotnet publish src/ai-genius-api -c Release -r linux-x64 --self-contained true -o <publish-dir>` after `actions/setup-dotnet@v4` with `dotnet-version: '10.0.x'`.
- **Rationale**: Meets FR-006/FR-011 and matches the backend project target framework (`net10.0`), producing a deployable Linux artifact.
- **Alternatives considered**:
  - Framework-dependent publish (rejected: does not satisfy self-contained requirement).
  - Non-Linux runtime publish (rejected: target App Service is Linux B1).

## Decision 3: Zip deploy package to `azure/webapps-deploy@v3`

- **Decision**: Zip the publish directory and pass the zip path to `azure/webapps-deploy@v3` via the `package` input.
- **Rationale**: Explicitly required by FR-012 and ensures deterministic deployment artifact handling.
- **Alternatives considered**:
  - Deploy unzipped directory (rejected: violates zip deploy requirement).
  - Use a different deployment action (rejected: violates FR-008).

## Decision 4: Resolve target app service name from GitHub variable

- **Decision**: Set `app-name: ${{ vars.APP_SERVICE_NAME }}` in deploy step and treat missing value as a hard failure.
- **Rationale**: Satisfies FR-014 and keeps environment-specific naming out of workflow code.
- **Alternatives considered**:
  - Hard-code app name in workflow (rejected: not environment-portable and violates spec intent).
  - Read from secret (rejected: not required; variable is correct classification for non-sensitive identifier).

## Decision 5: Preserve explicit step ordering and failure visibility

- **Decision**: Use one linear job with clearly named steps in mandated order and default fail-on-error behavior.
- **Rationale**: Satisfies FR-009/FR-010/FR-013 and makes failed stage immediately identifiable in workflow UI.
- **Alternatives considered**:
  - Combine build/package into a single script step (rejected: reduces failure-stage clarity).
  - Continue-on-error behavior (rejected: conflicts with required failed-run semantics).
