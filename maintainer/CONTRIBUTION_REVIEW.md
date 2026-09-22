# Contribution Review

**A contribution working on the contributor's machine does not mean it is safe to merge.**

This document answers: *"A contributor submitted a PR. What do I actually do before merging it?"*

Use this alongside the [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md) checklist for PRs touching infrastructure.

---

## Before you read a single line of code

Ask yourself these questions first:

- [ ] Does the PR description explain what changed and why?
- [ ] Is there a linked issue or context for the change?
- [ ] Is the scope reasonable? A PR that changes 40 files across unrelated areas is a red flag.
- [ ] Did the contributor follow the guidelines in [CONTRIBUTING.md](../CONTRIBUTING.md)?

If the description is missing or inadequate, **request changes before reviewing code**. You cannot review what you do not understand.

---

## Step 1 — Understand what is actually changing

Read the diff with intention, not just for correctness.

- What does this change do in the **running system**, not just in isolation?
- What is the **blast radius** if this is wrong? (Small isolated utility vs. core auth path vs. database schema)
- Is the PR doing what the description says? Sometimes PRs drift from their stated intent.
- Are there changes the contributor did **not** mention in the description?

**Pay particular attention to files the contributor may not have intended to change.** Accidentally modified files, reformatted files, or changes pulled in from a stale branch are all common.

---

## Step 2 — Verify the scope of the change

- Does this change only what it needs to change?
- Are there unrelated changes bundled in (reformatting, dependency updates, refactors)?
- If yes: request that they be split into separate PRs. Mixed-scope PRs make attribution, rollback, and debugging harder.

---

## Step 3 — Architecture and code impact

- Does the change fit the existing architecture, or is it working around it?
- Does it introduce a new pattern that will now need to be maintained?
- Does it duplicate logic that already exists elsewhere?
- Does it change public interfaces, function signatures, or API contracts? If so, is it backward compatible?
- Does it introduce global state, side effects, or implicit dependencies?
- If this is a performance change: does it come with benchmarks?

---

## Step 4 — Tests

- [ ] Are tests present for the new behaviour?
- [ ] Do the tests actually test the behaviour, or do they just execute code paths?
- [ ] Are existing tests still passing?
- [ ] Have any tests been deleted or weakened? If so, why?
- [ ] Is the test coverage adequate for the risk level of the change?

A PR with no tests for new behaviour is not ready to merge. Request them.

If CI is passing but tests were removed or bypassed: investigate before merging.

---

## Step 5 — Dependencies

- [ ] Were any new dependencies added?
  - What does the dependency do?
  - Is it actively maintained?
  - Does it have a history of security issues?
  - Is it really necessary, or could this be done without adding a dependency?
  - What license does it carry? Is it compatible with this project?
- [ ] Were existing dependencies upgraded?
  - Is the upgrade intentional, or a side effect?
  - Does the new version have breaking changes?
  - Is there a migration guide you should follow?
- [ ] Were any dependencies removed?
  - Is anything else depending on them that could break?

---

## Step 6 — API and interface compatibility

- [ ] Does this change any public API, CLI interface, or external-facing endpoint?
- [ ] Is it backward compatible? If not, is a major version bump warranted?
- [ ] Are there clients, integrations, or users who will break on this change?
- [ ] Has the relevant documentation been updated?

---

## Step 7 — Configuration changes

- [ ] Were any configuration files modified (app config, build config, etc.)?
- [ ] Are new configuration keys required? Are they documented and do they have defaults?
- [ ] Were any configuration keys removed or renamed? Will existing deployments break on upgrade?
- [ ] Was `.env.example` updated if new environment variables were added?

For deeper environment variable and secret review: see [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md).

---

## Step 8 — Security implications

- [ ] Does the change touch authentication, authorisation, session handling, or token validation?
- [ ] Does it change data access patterns? Could a user access data they should not?
- [ ] Does it introduce new input handling? Is that input validated and sanitised?
- [ ] Does it add external HTTP calls? Are those calls secure (HTTPS, certificate validation)?
- [ ] Does it change CORS, CSP, or security header configuration?
- [ ] Could this introduce a timing attack, race condition, or IDOR?
- [ ] Does the PR include any credentials, tokens, or secrets (even in comments or test fixtures)?

If any of these are yes: slow down. Do not merge until you have fully understood the implications.

See [SECURITY.md](../SECURITY.md) for the project's security policy.

---

## Step 9 — Database and data implications

- [ ] Does this change add, modify, or remove database tables or columns?
- [ ] Are there migration files? Are they correct and reversible where possible?
- [ ] Could this migration fail partway through and leave the database in an inconsistent state?
- [ ] Could this migration lock tables and cause downtime on a live system?
- [ ] Does this change the shape of data that is already in production? Is there a backfill plan?
- [ ] Does it delete or transform data that cannot be recovered?

**Database migrations deserve the same level of review as production deployments.** A bad migration is one of the hardest things to recover from.

See [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md) for migration-specific guidance.

---

## Step 10 — Deployment implications

- [ ] Does this change affect how the application starts up?
- [ ] Does it require new infrastructure resources (new queue, new bucket, new service)?
- [ ] Does it require zero-downtime consideration (e.g. old and new code must run simultaneously during rolling deploys)?
- [ ] Does it change startup/shutdown behaviour in a way that could cause crashes during deployment?
- [ ] Does the deployment order matter (migrate before deploy, deploy before migrate)?

---

## Step 11 — CI/CD implications

- [ ] Were any CI configuration files modified (`.github/workflows/`, `.circleci/`, etc.)?
- [ ] If yes: treat this as an infrastructure change and apply the [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md) review.
- [ ] Do the CI checks that are passing actually cover the changed code paths?
- [ ] Is CI being gamed (skipped tests, flaky tests being suppressed)?

---

## Step 12 — External services

- [ ] Does this change interact with any external API, webhook, or service?
- [ ] Could it cause unexpected charges, quota consumption, or rate limiting?
- [ ] If it changes webhook endpoints or API keys: has the service side been updated or is it part of the deployment plan?
- [ ] If it removes an integration: is anything else depending on it?

---

## Step 13 — Integration testing

After review, before merging on anything with deployment risk:

- Pull the branch locally.
- Run the full test suite.
- Test the specific functionality that changed, not just CI green.
- If the change has a UI component: test it in a browser.
- If the change involves infrastructure: verify in a staging or non-production environment if one exists.

You are not obligated to do this for every small documentation fix. You are obligated to do it for any change touching data, auth, deployment, or external services.

---

## Step 14 — Rollback and recovery

Before merging, ask: **if this breaks in production, what do I do?**

- [ ] Can this be reverted with a standard `git revert`?
- [ ] If there is a database migration, is it reversible? Is there a rollback plan?
- [ ] If this changes external service configuration, can that change be undone quickly?
- [ ] Do I have a way to detect breakage quickly after deploying (monitoring, alerts, smoke tests)?

If rollback is difficult: plan it before merging, not after.

---

## Decision: merge / request changes / close

**Merge when:**
- All of the above checks are satisfactory
- CI passes
- No open review comments
- You are confident the change is correct, safe, and reversible if needed
- You would be comfortable deploying this today

**Request changes when:**
- Specific, fixable issues were found
- Tests are missing but the core change is sound
- The approach needs minor adjustment
- Infrastructure or security areas need extra verification

**Close when:**
- The change is out of scope
- The approach is fundamentally wrong and cannot be salvaged
- The contributor has abandoned the PR after requested changes
- The change introduces unacceptable infrastructure or security risk
- The PR violates contribution restrictions from [CONTRIBUTING.md](../CONTRIBUTING.md)

---

## High-risk change categories (escalate your review)

| Category | Why it is risky |
|---|---|
| Auth / session changes | Can lock out all users or expose data |
| Database migrations | Can corrupt or destroy data |
| CI/CD changes | Can break your entire deployment pipeline |
| Secret / env var changes | Can expose credentials |
| Dependency major upgrades | Can introduce breaking changes at scale |
| New external service integrations | Can cause unexpected charges or data leaks |
| File storage / deletion logic | Data loss is often unrecoverable |
| Cron / background job changes | Can run at wrong times, cause duplication, or fail silently |

For all of these: see [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md).
