# Staging Release Runbook

## Objective
Provide a repeatable promotion and rollback process for the staging environment with explicit ownership.

## Ownership
- Primary owner: CTO
- Backup owner: CEO

## Pipeline entry points
- Automatic staging deploy: push to `main` triggers `.github/workflows/staging-release.yml` (`deploy-staging` job).
- Manual deploy: `workflow_dispatch` with `action=deploy`.
- Rollback rehearsal: `workflow_dispatch` with `action=rollback_rehearsal` and `rehearsal_ref`.

## Promotion flow (main -> staging)
1. PR is merged into `main` through required PR gates.
2. `Staging Release` workflow starts automatically on the `push` event.
3. `deploy-staging` generates and stores deployment evidence artifact:
   - `staging-deploy-<run_id>`
   - payload: `release/staging-deploy.json`
4. Owner checks workflow summary and confirms release SHA in staging.

## Rollback flow (rehearsal + real rollback procedure)
### Rehearsal procedure
1. Open `Staging Release` workflow and choose `action=rollback_rehearsal`.
2. Provide `rehearsal_ref` (commit SHA/tag to test).
3. Confirm successful run and artifact:
   - `rollback-rehearsal-<run_id>`
   - payload: `release/rollback-rehearsal.json`

### Real rollback procedure
1. Determine target good SHA from latest successful staging deploy artifact.
2. Create revert PR or hotfix PR to restore that SHA-equivalent state on `main`.
3. Merge PR through required gates.
4. Automatic `deploy-staging` run re-promotes the recovered state.
5. Post incident note with:
   - triggering issue
   - target SHA
   - deploy run URL
   - confirmation timestamp

## Operational checks
- SLA: staging pipeline failures acknowledged within 30 minutes (working hours).
- Triage owner posts first diagnosis in issue thread.
- Any rollback action must capture run URL and resulting SHA.

## Evidence capture for issue completion
- At least one successful `deploy-staging` run URL.
- At least one successful `rollback-rehearsal` run URL.
