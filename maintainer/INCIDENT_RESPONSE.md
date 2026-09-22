# Incident Response

What to do when something breaks.

The goal is always the same: **stop further damage, restore service, understand what happened, and prevent recurrence**.

---

## Core sequence (apply to every incident)

```
1. Stop further damage
2. Determine what changed
3. Restore service or safe state
4. Preserve evidence
5. Fix the root cause
6. Verify recovery
7. Document what happened
8. Improve the process
```

Do not skip to step 5 without completing steps 1-4. Fixing a symptom without understanding the cause leads to repeat incidents.

---

## Production deployment failure

**Symptoms**: Deployment fails mid-way. Application is in an unknown state. Users may be affected.

**Immediate actions**:

1. Do not re-run the deployment blindly — the second run may compound the damage.
2. Check if the previous version is still running (rolling deploy) or if the application is down.
3. If down: roll back to the previous stable version now. Speed matters.
4. Check deployment logs for the specific failure point.
5. If a migration ran before the code deployed, check whether the migration completed successfully.

**Rollback procedure**:
- Redeploy the last known good version (use the git tag from the previous release).
- If a database migration already ran: the new schema is now live. Do not redeploy old code that does not understand the new schema — check compatibility first.

**Once service is restored**:
- Identify the deployment failure cause from logs.
- Fix the issue in a branch.
- Test the fix in staging.
- Re-release per [RELEASE_PROCESS.md](./RELEASE_PROCESS.md).

---

## Regression introduced by a contribution

**Symptoms**: A feature that was working stopped working after a PR was merged.

**Immediate actions**:

1. Identify which PR introduced the regression. Check the git log since the last known-good state.
2. If identified: revert the PR commit and deploy the revert immediately.

```bash
git revert <commit-sha>
git push origin main
```

3. Deploy the revert as an emergency release.
4. Inform the contributor that the PR was reverted and why.

**Do not**:
- Attempt to hotfix the regression without reverting first, unless the revert itself would cause a migration conflict.
- Leave a broken `main` branch without taking action.

**Once service is restored**:
- Write a test that would have caught the regression.
- Have the contributor re-submit the PR with the fix included.
- Review the PR review process — what should have caught this?

---

## Database migration failure

**Symptoms**: Migration fails partway through. Database may be in a partially migrated state. Application may be failing because schema is inconsistent.

**Immediate actions**:

1. Stop any application instances that are writing to the database — they may corrupt data trying to operate on partial schema.
2. Do not re-run the migration without understanding why it failed.
3. Check the migration tool's transaction state. Most tools wrap migrations in a transaction — if it failed and rolled back, the database is in its pre-migration state. If there was no transaction, assess the damage.
4. Connect to the database directly and verify the actual schema state.

**Recovery paths**:

| State | Action |
|---|---|
| Migration rolled back (transaction failed cleanly) | Fix the migration, test on staging, re-run |
| Migration partially applied (no transaction) | Requires manual database surgery — proceed carefully |
| Migration succeeded but app is failing | Code–schema compatibility issue — redeploy correct code version |

**Never**:
- Manually edit a production database without a backup verified as restorable.
- Delete or alter migration history files to paper over a failed migration.

**After recovery**:
- Ensure all migrations use transactions where the database engine supports them.
- Add a rollback migration for any migration that doesn't have one.
- Update the release process to test migrations on staging first.

---

## Exposed secret or credential

**Symptoms**: A secret (API key, database password, token, etc.) has been committed to the repository, posted in an issue, or otherwise exposed publicly.

**Immediate actions — treat this as a breach, not a mistake**:

1. **Rotate the secret now.** Do not wait. Revoke the exposed credential and issue a new one.
2. Assume the secret has already been extracted. Check access logs for the relevant service if available.
3. Remove the secret from git history:
   ```bash
   # Using git filter-repo (recommended over filter-branch)
   git filter-repo --path <file-containing-secret> --invert-paths
   # or for a specific string:
   git filter-repo --replace-text <(echo 'EXPOSED_SECRET==>REMOVED')
   ```
4. Force-push the cleaned history: `git push --force-with-lease origin main`
5. Ask GitHub support to purge cached views if the secret appeared in a public PR or commit.
6. Notify any services whose credentials were exposed that a rotation occurred.
7. Update the project's secret management documentation if the exposure revealed a process gap.

**Do not**:
- Just remove the secret in a new commit. The old commit still contains it and is publicly accessible.
- Assume it was not seen because the exposure was brief.

---

## Dependency vulnerability

**Symptoms**: A CVE is published for a dependency you use. A security scanner flags a dependency.

**Assess first**:

1. Is the vulnerable code path actually reachable in your application?
2. What is the CVSS score and the nature of the vulnerability?
3. Is a patched version available?

**Actions by severity**:

| Severity | Timeframe | Action |
|---|---|---|
| Critical (CVSS 9+) | Immediately | Patch and deploy, or disable the affected functionality |
| High (CVSS 7-9) | Within days | Plan and execute patch |
| Medium/Low | Next scheduled release | Include in normal release cycle |

**If no patch is available**:
- Can the vulnerable feature be disabled or removed?
- Can you vendor or fork the dependency with a local fix?
- Document the risk and monitor for a patch.

**After patching**:
- Verify the upgrade did not introduce breaking changes.
- Run the full test suite.
- Deploy per [RELEASE_PROCESS.md](./RELEASE_PROCESS.md).

---

## Broken CI/CD

**Symptoms**: CI is failing on `main`. Builds are not completing. Deployments are not triggering.

**Immediate actions**:

1. Check whether `main` itself was broken by a recent merge, or if the CI infrastructure failed.
2. If CI infrastructure failed (flaky runner, expired token, third-party service outage): wait or manually re-run.
3. If a recent merge broke CI: revert the merge or fix the CI failure before anything else.

**Do not**:
- Merge PRs with failing CI "just to unblock" — this normalises broken CI and makes the next real failure invisible.
- Disable CI checks to force a merge through.
- Leave CI broken without filing a tracking issue and beginning resolution.

**Restoring CI**:
- Fix the failing configuration or code.
- Verify CI passes on a branch before merging.
- Update [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md) or the CI configuration review checklist if the failure revealed a gap.

---

## External service failure

**Symptoms**: An external API, database service, CDN, or third-party integration is down or returning errors.

**Immediate actions**:

1. Confirm the service is actually down (check the provider's status page before assuming it is your code).
2. If the service is down: is there a graceful degradation path in your application? Enable it.
3. If there is no graceful degradation and users are affected: consider whether a maintenance page is better than a broken UI.

**While the service is down**:
- Monitor the provider's status page.
- Communicate with users if the outage is visible.
- Do not attempt to fix something that is not yours to fix.

**After service restoration**:
- Check whether any data was lost or jobs failed during the outage.
- Re-run any failed background tasks if safe to do so.
- Consider adding circuit breakers, retries, or fallback behaviour to prevent user-facing impact in future outages.

---

## Production outage (application is down)

**Symptoms**: The application is returning errors or is unreachable for all or most users.

**Immediate sequence**:

1. **Stop making changes.** A panicked series of deploys during an outage often makes things worse.
2. **Determine what changed.** Check the deployment log: what was last deployed and when?
3. **Rollback to last known good** if the outage started after a deployment.
4. **Check infrastructure** if no code changed: server health, disk space, memory, database connections, external dependencies.
5. Communicate a status update (even "investigating") if users are affected.

**Checklist while investigating**:

- [ ] Application logs — what errors are appearing?
- [ ] Infrastructure metrics — CPU, memory, disk, connections
- [ ] Database — is it accepting connections? Are query times normal?
- [ ] Load balancer / reverse proxy — is it healthy?
- [ ] DNS — is the domain resolving correctly?
- [ ] SSL — is the certificate valid?
- [ ] External dependencies — any affecting the startup path?

**After service is restored**:
- Write a post-incident summary (see Post-incident documentation below).
- Fix the root cause before re-enabling any functionality that was involved.

---

## Post-incident documentation

For every significant incident, write a brief summary. This does not need to be a formal report — it needs to be useful.

Template:

```
## Incident: [Short description]
Date: YYYY-MM-DD
Duration: [How long was service affected]

### What happened
[Factual description of the sequence of events]

### Root cause
[What actually caused the failure]

### Impact
[Who was affected, what was unavailable]

### Timeline
[Key timestamps: when it started, when detected, when resolved]

### Resolution
[What was done to restore service]

### Follow-up actions
- [ ] [Specific action to prevent recurrence]
- [ ] [Specific action to improve detection]
```

Store post-incident summaries in `docs/incidents/` or an equivalent location. They accumulate into a pattern history that is valuable for identifying systemic issues.

---

## Evidence preservation

Before reverting, fixing, or cleaning up, capture the evidence you need to understand what happened:

- Copy relevant log output before it rotates.
- Screenshot error pages, monitoring dashboards, or alert states.
- Record the exact git commit that was deployed when the incident occurred.
- Save the database migration state (which migrations have run) if applicable.

You cannot diagnose a past incident with logs you deleted.

---

## Preventing recurrence — process improvements

After each incident:

- Did the existing review process fail to catch the issue? Update [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md).
- Was an infrastructure area involved that needs better documentation? Update [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md).
- Did the deployment process allow something unsafe? Update [RELEASE_PROCESS.md](./RELEASE_PROCESS.md).
- Was there a missing test? Write it.
- Was there missing monitoring or alerting? Add it.

An incident that does not produce at least one process improvement is a missed learning opportunity.
