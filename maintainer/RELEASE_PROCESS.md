# Release Process

A repeatable process for getting changes from a merged PR to production safely.

Adapt this to your deployment platform and CI/CD system. The steps are universal; the tooling is not.

Related: [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md) — what happens before a PR is merged.

---

## Overview

```
PR merged to main
  → Version decision
  → Changelog updated
  → Build verified
  → Tests pass
  → Preview/staging validation (if available)
  → Production deployment
  → Smoke tests
  → Production verification
  → Tag and release
  → Rollback readiness confirmed
```

---

## Step 1 — Confirm main is in a releasable state

Before starting a release:

- [ ] All CI checks on `main` are green.
- [ ] No partially-merged feature is in a broken intermediate state.
- [ ] The changelog reflects everything that has merged since the last release.
- [ ] Any open security issues have been addressed.

If any of these are not true: fix them before continuing, or wait for the next release window.

---

## Step 2 — Decide the version

Use Semantic Versioning (`MAJOR.MINOR.PATCH`):

| Change type | Version bump |
|---|---|
| Bug fix, documentation, internal change | PATCH (1.0.0 → 1.0.1) |
| New feature, backward-compatible change | MINOR (1.0.0 → 1.1.0) |
| Breaking change to any public API/interface | MAJOR (1.0.0 → 2.0.0) |

If you are not publishing a versioned library or package, a simpler date-based scheme (`2026.09`) or just git tags (`release-2026-09-23`) is fine. The important thing is that every production deployment is tagged and traceable.

---

## Step 3 — Update the changelog

Update [CHANGELOG.md](../CHANGELOG.md):

- Move items from `[Unreleased]` into a new versioned section.
- Review the entries — are they accurate? Do they describe the change from a user's perspective?
- Add the release date.

```markdown
## [1.2.0] - 2026-09-23

### Added
- ...

### Fixed
- ...
```

Commit the changelog update to `main`. This commit is part of the release, not a pre-release step.

---

## Step 4 — Build

Run your build process:

- [ ] Build completes without errors or warnings that were not there before.
- [ ] Build artifacts are produced in the expected locations.
- [ ] Build is reproducible (running it twice produces the same output).

If the build fails: stop. Do not attempt to release a broken build. Investigate before continuing.

---

## Step 5 — Run the full test suite

- [ ] Unit tests pass.
- [ ] Integration tests pass.
- [ ] Any end-to-end tests pass.

If tests that were passing before the last merge are now failing: identify which merge caused it before releasing. Do not release over a regression.

---

## Step 6 — Preview or staging validation

If the project has a staging environment:

- [ ] Deploy to staging first.
- [ ] Run smoke tests against staging (see Step 8 for what smoke tests to run).
- [ ] Verify that any new features or fixes work as expected in the staging environment.
- [ ] If there are database migrations: run them on staging first and verify the application works correctly after migration.

If there is no staging environment: accept the additional risk and be especially thorough with Step 8 after production deployment.

---

## Step 7 — Production deployment

### Before deploying

- [ ] You are deploying during a low-traffic window if possible.
- [ ] You are not deploying on a Friday afternoon unless you have solid rollback confidence.
- [ ] You know what the rollback procedure is for this deployment (see Step 10).
- [ ] If there are database migrations: understand whether they run before or after the code deployment and plan accordingly.
- [ ] Anyone else who needs to know about the deployment is aware.

### Deployment order (when migrations are involved)

**Deploy the code, then run migrations** (preferred for additive migrations — new columns, new tables):
- The new code should be written to handle old schema gracefully until migration runs.

**Run migrations, then deploy the code** (required for destructive migrations — removing columns, renaming):
- Only acceptable if old code can run against new schema without errors.

Do not run migrations and deploy simultaneously without understanding the interaction.

### During deployment

- Monitor logs during and immediately after deployment.
- Watch for restart failures, crashes on startup, or unexpected error spikes.

---

## Step 8 — Smoke tests

Immediately after production deployment, manually verify the critical paths:

- [ ] The application starts and serves requests (not a 500 or blank page).
- [ ] Core user-facing functionality works (log in, primary workflow, etc.).
- [ ] Any functionality that was specifically changed in this release works as expected.
- [ ] No unusual error rate spike in logs or monitoring.

Smoke tests should take no more than 5-10 minutes. If it takes longer than that, the smoke test list is too long — trim it to the true critical path.

---

## Step 9 — Production verification

After smoke tests pass, do a broader check over the next 15-30 minutes:

- [ ] Background jobs and cron tasks are running normally.
- [ ] External integrations are responding as expected.
- [ ] Database connections and query performance look normal.
- [ ] Memory/CPU/disk usage is not spiking unexpectedly.
- [ ] No alerts firing in monitoring or error tracking.

If anything looks off: investigate immediately. Do not wait to see if it resolves.

---

## Step 10 — Rollback readiness

Before the release is considered complete, confirm you know how to roll back:

| Scenario | Rollback approach |
|---|---|
| Code change with no migration | Revert the git commit and redeploy the previous version |
| Code change with additive migration | Redeploy previous code — old code should tolerate new schema |
| Code change with destructive migration | **Requires a pre-planned rollback migration** — do not deploy without one |
| Infrastructure change | Depends on the infrastructure — verify before deploying |

If you cannot roll back quickly: your monitoring and alerting needs to be especially good so you catch problems fast.

---

## Step 11 — Tag and release

Once production is verified:

```bash
git tag -a v1.2.0 -m "Release v1.2.0"
git push origin v1.2.0
```

If the project publishes to a package registry (npm, PyPI, etc.), publish the package now.

Create a GitHub release from the tag. Paste the changelog entry into the release notes.

---

## After the release

- Update the `[Unreleased]` section of the changelog (should now be empty or nearly empty).
- Close any issues marked for this release milestone.
- Communicate the release to users if the project has a communication channel.

---

## Emergency releases (hotfixes)

When a critical bug or security issue requires an immediate release:

1. Branch from the affected production tag or `main` (whichever is deployed).
2. Make the minimal fix. Do not bundle anything else.
3. Apply accelerated review — still check the relevant sections of [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md), but focus on the specific fix.
4. If infrastructure is affected, apply [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md).
5. Deploy with the same steps as above, but compress the timeline.
6. Tag with a PATCH version bump.
7. Backport or merge the fix to `main` immediately after.

An emergency release still needs a changelog entry, a tag, and smoke tests. Shortcuts in an emergency create the next incident.

---

## Adapting this process

| Project type | What to adjust |
|---|---|
| Static site / documentation | Steps 6 and 9 are simpler; no migrations |
| Library / package | Step 11 (package registry publish) is the main deployment step |
| Backend service | All steps apply |
| AI/ML model | Add model validation and evaluation steps between Steps 5 and 6 |
| Multi-service / microservices | Define deployment order for dependent services |
