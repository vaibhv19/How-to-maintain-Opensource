# Maintainer Guide

This is your operating manual. It covers everything from daily triage to release cadence to keeping old projects healthy over time.

Related documents:
- [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md) — the detailed PR review checklist
- [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md) — what can break beyond the code
- [RELEASE_PROCESS.md](./RELEASE_PROCESS.md) — how changes reach production
- [INCIDENT_RESPONSE.md](./INCIDENT_RESPONSE.md) — what to do when something breaks

---

## Maintainer responsibilities

You are responsible for:

- Deciding what gets merged and what does not
- Keeping the codebase in a releasable state
- Protecting the project's infrastructure, users, and data
- Triaging issues and PRs in a reasonable timeframe
- Communicating clearly with contributors
- Managing releases
- Monitoring dependencies for security issues
- Keeping documentation accurate

You are **not** responsible for:
- Implementing every requested feature
- Reviewing PRs that do not meet contribution standards
- Explaining every decision in detail
- Accepting every contribution that technically works

---

## Daily/weekly operations

### Issue triage

When a new issue arrives:

1. **Read it fully** before doing anything.
2. Label it: `bug`, `enhancement`, `question`, `documentation`, `needs-reproduction`, `wontfix`, etc.
3. If it is a bug: can you reproduce it? If not, ask for reproduction steps. Label `needs-reproduction`.
4. If it is a feature request: does it fit the project scope? If not, close it politely with an explanation.
5. If it is a question: answer it or point to existing documentation.
6. Assign a priority if you have a milestone system.

**A triaged issue is not a commitment to fix it.** It is an acknowledgement that it was seen.

### PR triage

When a new PR arrives:

1. Check it against [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md).
2. If the PR does not meet basic standards (no description, no tests, touches restricted files), request changes immediately rather than letting it sit.
3. If the PR is high risk (infrastructure changes, security-adjacent), do not rush.
4. Assign a `needs-review` or `needs-changes` label so state is visible.

---

## PR lifecycle

```
Opened → Triage → Review → Changes Requested → Re-review → Approved → Merged
                     ↓
                  Closed (scope/quality/policy)
```

**Triage**: Is this worth reviewing at all? Does it meet minimum standards?

**Review**: Full review per [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md).

**Changes Requested**: Be specific. List exactly what needs to change. Do not make contributors guess.

**Re-review**: Once changes are pushed, review again. Check that the changes addressed your feedback and did not introduce new issues.

**Approved → Merged**: You merge it. External contributors do not merge their own PRs.

---

## When to request changes

- Tests are missing or inadequate
- The change breaks something you know about
- The code is correct but the approach does not fit the project's architecture
- The PR is too large to review safely
- The description is insufficient to understand what changed
- The change touches infrastructure or security areas and lacks the required sign-off
- The PR mixes multiple unrelated changes

---

## When to close a contribution

Close (not just request changes) when:

- The feature is not in scope for this project
- The PR has been abandoned (no response to requested changes after a reasonable period — typically 2-4 weeks)
- The contributor is unwilling to make necessary changes
- The approach is fundamentally incompatible with the project architecture
- The PR would introduce unacceptable risk and cannot be reworked to remove it
- The PR violates the security policy or contribution restrictions

Close politely. Explain the reason. Do not feel obligated to apologise for protecting the project.

---

## When to merge

Merge when:

- [ ] The change does what it claims to do
- [ ] Tests pass (CI and local if necessary)
- [ ] Your review per [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md) is complete
- [ ] No open review comments remain unresolved
- [ ] The changelog has been updated (or you will update it)
- [ ] You are confident the change can be rolled back if needed
- [ ] For infrastructure-touching changes: [INFRASTRUCTURE_SAFETY.md](./INFRASTRUCTURE_SAFETY.md) checklist is satisfied

Do not merge on a Friday afternoon before going offline if the change has any deployment risk.

---

## Documentation maintenance

- Keep [CONTRIBUTING.md](../CONTRIBUTING.md) up to date with actual project conventions.
- Update [CHANGELOG.md](../CHANGELOG.md) when merging or releasing.
- Update [SECURITY.md](../SECURITY.md) if the vulnerability reporting process changes.
- Update the maintainer docs whenever your infrastructure or release process evolves.
- When you add a new environment variable, deployment step, or external service dependency, document it immediately — not later.

**Outdated documentation is worse than no documentation.** If you cannot keep a section current, remove it.

---

## Dependency maintenance

- Review dependency update PRs (from bots like Dependabot or Renovate) carefully — they can break things just as easily as manual changes.
- Prioritise security patches. Do not defer them indefinitely.
- Test dependency updates with the same thoroughness as code changes.
- Before updating a major version of a core dependency, read its migration guide and check your integration points.
- Periodically audit for dependencies that are no longer maintained.

---

## Release cadence

There is no universal right answer. Pick one and stick to it:

| Style | When it works |
|---|---|
| On-demand (release when ready) | Solo projects, low-traffic tools |
| Time-boxed (e.g. monthly) | Active projects with regular contributions |
| Semantic (release on breaking/feature/patch events) | Libraries, APIs |
| Continuous (every merge to main ships) | Projects with solid CI/CD and quick rollback |

Regardless of cadence:
- Every release goes through [RELEASE_PROCESS.md](./RELEASE_PROCESS.md).
- Every release has a changelog entry.
- Every release has a git tag.

---

## Keeping old projects healthy without rewriting them

If you maintain a project that is working but not actively developed:

- **Do not rewrite it because it feels old.** If it is working and serving users, rewrites introduce risk for no immediate gain.
- Apply security patches promptly. Everything else can wait.
- Keep CI green. If CI is broken, fix it before anything else.
- Update the README if it no longer reflects reality.
- Set contributor expectations appropriately — a low-maintenance project should say so.
- Archive the repository on GitHub if the project is genuinely abandoned. Do not leave it in a limbo state.

Signs a project needs attention, not a rewrite:
- CI is failing
- Dependencies have critical security vulnerabilities
- Documentation is actively misleading
- The project's external dependencies (APIs, services) have changed

Signs that a rewrite might be warranted (rare):
- The architecture actively prevents fixing bugs or adding essential features
- The underlying technology platform is end-of-life and has no migration path
- The security model is fundamentally broken

Even then: migrate, do not rewrite unless you have to.

---

## Contributor communication

- Respond to PRs and issues within a reasonable window. What is reasonable depends on your project's activity level — be explicit about it in your README.
- If you are going offline for an extended period, say so.
- If a PR will not be merged soon because of competing priorities, say so. Silence is demoralising for contributors.
- Thank contributors for genuine effort even when you do not merge their work.
- Be firm on standards. Being kind and being strict are not mutually exclusive.

---

## Adapting this guide to a specific project

This guide is a template. For a real project:

- Replace generic references (CI system, deployment platform, etc.) with the actual tools you use.
- Add project-specific triage labels and workflows.
- Remove sections that do not apply (e.g. the database migration guidance if you have no database).
- If you have multiple maintainers, define responsibilities and decision-making authority clearly.
