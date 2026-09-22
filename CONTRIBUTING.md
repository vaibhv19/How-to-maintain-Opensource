# Contributing

Thanks for wanting to contribute. This document covers how to set up your environment, what to work on, and how to submit changes.

---

## Before you start

- Check [open issues](../../issues) and [existing PRs](../../pulls) before starting work to avoid duplication.
- For significant changes, open an issue first to discuss the approach. Do not spend days on a PR that may be declined.
- Read [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) before participating.

---

## What contributors can work on

Generally welcome:
- Bug fixes with a clear reproduction case
- Documentation improvements
- Performance improvements with benchmarks
- New features that have been discussed and agreed in an issue

Out of scope for external contributors (maintainer-only):
- Changes to CI/CD configuration
- Changes to deployment scripts or workflows
- Changes to environment variable definitions or secret handling
- Changes to production configuration files
- Changes to database migration tooling or migration execution order
- Changes to DNS, cloud resource definitions, or infrastructure-as-code
- Changes to authentication or authorization logic without explicit prior discussion

When in doubt, open an issue and ask.

---

## Development setup

1. Fork the repository and clone your fork.
2. Create a branch from `main` (see Branching below).
3. Install dependencies as described in the project README.
4. Copy any required environment variable templates (e.g. `.env.example` → `.env.local`) and populate with safe local values only — never real credentials.
5. Run the test suite before making changes to confirm a clean baseline.

---

## Branching

| Branch | Purpose |
|---|---|
| `main` | Stable, releasable state. Direct pushes not accepted. |
| `feature/<name>` | New features |
| `fix/<name>` | Bug fixes |
| `docs/<name>` | Documentation-only changes |
| `chore/<name>` | Dependency updates, tooling, housekeeping |

Always branch from `main`. Do not stack PRs on top of unmerged branches unless explicitly coordinated with the maintainer.

---

## Commit expectations

- Write clear, one-line commit messages: `fix: prevent token refresh loop on 401 response`
- Use a recognised convention if the project has one (e.g. [Conventional Commits](https://www.conventionalcommits.org/)).
- One logical change per commit. Squash cleanup commits before submitting.
- Do not mix unrelated changes in a single commit or PR.

---

## Pull request expectations

Before opening a PR:

- [ ] Run the full test suite locally. Do not submit a PR with failing tests.
- [ ] Add or update tests for the change you are making.
- [ ] Update any relevant documentation.
- [ ] Confirm you have not accidentally committed secrets, credentials, or environment-specific configuration.
- [ ] Keep the PR focused. A PR that does five things at once is harder to review and more likely to introduce regressions.

When opening the PR:

- Write a clear description: what changed, why, and how to verify it.
- Reference the issue it addresses (e.g. `Closes #42`).
- If there is any infrastructure or deployment implication, call it out explicitly in the PR description.

---

## Testing requirements

- All new behaviour must be covered by tests.
- Existing tests must continue to pass.
- If you are fixing a bug, add a test that would have caught the bug.
- Do not delete or weaken existing tests to make your change pass.

---

## What contributors should not modify

The following areas are maintained exclusively by the project maintainer. Do not modify these in a PR without explicit prior agreement:

- `.github/workflows/` — CI/CD pipeline definitions
- `Dockerfile`, `docker-compose.yml` — container configuration
- `**/migrations/` — database migration files
- `.env.*` — environment configuration templates
- `infra/`, `terraform/`, `pulumi/`, `cdk/` — infrastructure definitions
- `scripts/deploy*`, `scripts/release*` — deployment scripts
- `SECURITY.md` — security policy
- `LICENSE` — license terms

Touching these files without maintainer sign-off will result in the PR being closed regardless of the code quality of other changes.

---

## What happens after you submit

1. A maintainer will review your PR. Response time varies — do not ping daily.
2. You may be asked to make changes. Please do so on the same branch and push; the PR will update automatically.
3. When the PR is approved and all checks pass, the maintainer will merge it.
4. External contributors do not merge their own PRs.

For the maintainer review process, see [docs/maintainer/CONTRIBUTION_REVIEW.md](./maintainer/CONTRIBUTION_REVIEW.md).
