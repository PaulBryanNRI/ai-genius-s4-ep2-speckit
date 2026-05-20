# Tasks: Deploy Backend API Workflow

**Feature**: `003-deploy-backend-api`  
**Input**: `specs/003-deploy-backend-api/plan.md`, `spec.md`, `research.md`, `data-model.md`, `contracts/deploy-api-workflow-contract.md`, `quickstart.md`

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Parallelizable task (different file, no blocking dependency)
- **[Story]**: User story label (`[US1]`, `[US2]`, `[US3]`)
- Every task includes an exact file path

---

## Phase 1: Setup

**Purpose**: Create the feature workflow file scaffold and shared constants.

- [ ] T001 Create workflow scaffold with `name`, `on`, `env`, and `jobs` root blocks in `.github/workflows/003-deploy-api.yml`
- [ ] T002 Define shared workflow env variables `API_PROJECT_PATH`, `PUBLISH_OUTPUT_DIR`, and `DEPLOY_ZIP_PATH` in `.github/workflows/003-deploy-api.yml`

---

## Phase 2: Foundational

**Purpose**: Add blocking workflow infrastructure required by all user stories.

**⚠️ CRITICAL**: Complete this phase before implementing US1/US2/US3.

- [ ] T003 Add `deploy-api` job baseline (`runs-on: ubuntu-latest`, `permissions.contents: read`, `permissions.id-token: write`) in `.github/workflows/003-deploy-api.yml`
- [ ] T004 Add Azure authentication step using `azure/login@v1` with `${{ secrets.AZURE_CREDENTIALS }}` in `.github/workflows/003-deploy-api.yml`
- [ ] T005 Add explicit stage step names (`Checkout`, `Setup .NET`, `Publish API`, `Create deployment zip`, `Deploy to App Service`) in `.github/workflows/003-deploy-api.yml` for failure-stage visibility

**Checkpoint**: Base deployment job exists and can be extended story-by-story.

---

## Phase 3: User Story 1 - Automatic API Deployment on Main (Priority: P1) 🎯 MVP

**Goal**: Ensure pushes to `main` automatically start backend deployment with clear run outcomes.

**Independent Test**: Push a commit to `main` and verify `.github/workflows/003-deploy-api.yml` runs automatically and ends in explicit success/failure status.

- [ ] T006 [US1] Configure trigger to `on.push.branches: [main]` only in `.github/workflows/003-deploy-api.yml`
- [ ] T007 [US1] Add first execution step `actions/checkout@v4` in `.github/workflows/003-deploy-api.yml`
- [ ] T008 [US1] Set workflow and job display names to clearly identify API deployment run outcomes in `.github/workflows/003-deploy-api.yml`
- [ ] T009 [US1] Update runtime validation instructions for auto main-branch trigger in `specs/003-deploy-backend-api/quickstart.md`

**Checkpoint**: US1 is independently testable through automatic trigger behavior on `main`.

---

## Phase 4: User Story 2 - Consistent Build Packaging for Deployment (Priority: P2)

**Goal**: Produce and deploy a self-contained `linux-x64` zip package for the backend API.

**Independent Test**: Run workflow from `.github/workflows/003-deploy-api.yml` and verify `dotnet publish` uses `linux-x64` + self-contained output, zip artifact is produced, and deploy step uses that zip.

- [ ] T010 [US2] Add `actions/setup-dotnet@v4` with `.NET 10` (`dotnet-version: '10.0.x'`) in `.github/workflows/003-deploy-api.yml`
- [ ] T011 [US2] Add `dotnet publish` step for `src/ai-genius-api` with `-c Release -r linux-x64 --self-contained true -o $PUBLISH_OUTPUT_DIR` in `.github/workflows/003-deploy-api.yml`
- [ ] T012 [US2] Add zip creation step that packages `$PUBLISH_OUTPUT_DIR` to `$DEPLOY_ZIP_PATH` in `.github/workflows/003-deploy-api.yml`
- [ ] T013 [US2] Add deployment step `azure/webapps-deploy@v3` using `app-name: ${{ vars.APP_SERVICE_NAME }}` and `package: ${{ env.DEPLOY_ZIP_PATH }}` in `.github/workflows/003-deploy-api.yml`
- [ ] T014 [US2] Enforce required step order (`checkout` → `setup-dotnet` → `dotnet publish` → zip → `azure/webapps-deploy@v3`) in `.github/workflows/003-deploy-api.yml`
- [ ] T015 [P] [US2] Update build/package/deploy validation checklist for zip deploy and `APP_SERVICE_NAME` in `specs/003-deploy-backend-api/quickstart.md`

**Checkpoint**: US2 is independently testable by artifact/package correctness regardless of concurrency behavior.

---

## Phase 5: User Story 3 - Safe, Non-Overlapping Deployments (Priority: P3)

**Goal**: Align environment resolution and concurrency behavior with existing infra workflow controls.

**Independent Test**: Trigger two close-together `main` pushes and verify `.github/workflows/003-deploy-api.yml` applies matching concurrency behavior and deterministic environment resolution pattern.

- [ ] T016 [US3] Add workflow-level `concurrency.group: ${{ github.workflow }}-${{ github.ref }}` and `concurrency.cancel-in-progress: true` to `.github/workflows/003-deploy-api.yml`
- [ ] T017 [US3] Add `workflow_dispatch` `environment` input and set `env.ENVIRONMENT: ${{ github.event.inputs.environment || 'dev' }}` in `.github/workflows/003-deploy-api.yml`
- [ ] T018 [US3] Set `jobs.deploy-api.environment: ${{ github.event.inputs.environment || 'dev' }}` in `.github/workflows/003-deploy-api.yml` to match `.github/workflows/001-deploy-infra.yml` pattern
- [ ] T019 [P] [US3] Update concurrency and environment runtime validation steps in `specs/003-deploy-backend-api/quickstart.md`

**Checkpoint**: US3 is independently testable with back-to-back push behavior and environment pattern verification.

---

## Phase 6: Polish

**Purpose**: Final compliance and cross-cutting documentation alignment.

- [ ] T020 [P] Validate `.github/workflows/003-deploy-api.yml` against `specs/003-deploy-backend-api/contracts/deploy-api-workflow-contract.md` and resolve any contract mismatches directly in `.github/workflows/003-deploy-api.yml`
- [ ] T021 [P] Add deployment workflow usage notes (required secret/variable and trigger behavior) to `docs/guide.md`
- [ ] T022 Validate final execution checklist consistency between `specs/003-deploy-backend-api/quickstart.md` and `.github/workflows/003-deploy-api.yml`

---

## Dependencies & Execution Order

### Phase Dependencies

1. **Setup (Phase 1)** → no dependencies  
2. **Foundational (Phase 2)** → depends on Phase 1; blocks all user stories  
3. **US1 (Phase 3)** → depends on Phase 2  
4. **US2 (Phase 4)** → depends on Phase 2 and US1 workflow trigger baseline  
5. **US3 (Phase 5)** → depends on Phase 2 and can proceed after US1 baseline exists  
6. **Polish (Phase 6)** → depends on completion of selected user stories

### User Story Dependencies

- **US1 (P1)**: first deliverable (MVP), no dependency on other stories
- **US2 (P2)**: builds on US1 workflow baseline to implement packaging/deploy path
- **US3 (P3)**: builds on shared workflow baseline; should remain independently testable from US2 packaging logic

---

## Parallel Execution Examples

### US1 Parallel Example

```bash
# After T008:
Task: "T009 Update runtime validation instructions in specs/003-deploy-backend-api/quickstart.md"
```

### US2 Parallel Example

```bash
# After T014:
Task: "T015 Update quickstart checklist in specs/003-deploy-backend-api/quickstart.md"
```

### US3 Parallel Example

```bash
# After T018:
Task: "T019 Update concurrency/environment validation steps in specs/003-deploy-backend-api/quickstart.md"
```

---

## Implementation Strategy

### MVP First (US1 only)

1. Complete Phase 1 (T001-T002)
2. Complete Phase 2 (T003-T005)
3. Complete Phase 3 (T006-T009)
4. Validate independent US1 trigger behavior on `main`

### Incremental Delivery

1. Add US2 packaging/deployment path (T010-T015), validate artifact/deploy behavior
2. Add US3 concurrency/environment behavior (T016-T019), validate non-overlapping runs
3. Finish Polish (T020-T022) for contract and docs alignment

### Parallel Team Strategy

- Engineer A: workflow implementation tasks in `.github/workflows/003-deploy-api.yml`
- Engineer B: quickstart/documentation updates in `specs/003-deploy-backend-api/quickstart.md` and `docs/guide.md`
- Merge after each story checkpoint to preserve independent testability
