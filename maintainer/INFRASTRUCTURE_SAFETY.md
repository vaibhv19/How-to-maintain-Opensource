# Infrastructure Safety

This document covers what can break **beyond your codebase** when a PR is merged, and how to review it safely.

Code correctness and infrastructure safety are separate concerns. A PR can be perfectly correct code and still break your deployment, expose your secrets, corrupt your database, or compromise your security.

Related: [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md) for the full PR review checklist.

---

## The three categories of change

Before reviewing any PR, classify it:

### Category A — Normal code change
Logic, algorithms, UI components, utility functions, documentation.
- Standard review per [CONTRIBUTION_REVIEW.md](./CONTRIBUTION_REVIEW.md).
- No special infrastructure concerns.

### Category B — Change requiring additional maintainer review
Touches areas that can break running systems without being obviously dangerous.
- New environment variables or configuration keys
- Dependency additions or major upgrades
- New external API integrations
- Changes to background jobs, cron schedules, or queue workers
- Changes to file handling or storage access
- New feature flags or runtime configuration
- Changes to logging or observability (can accidentally log secrets)

**Action**: Apply the specific checklist for that area from this document before merging.

### Category C — Maintainer-only. External contributions must not make these changes.
Changes that can break or compromise the project for all users at the infrastructure level.
- CI/CD workflow files
- Deployment scripts and configuration
- Production environment variable definitions
- Secret management configuration
- Database migration tooling or migration file ordering
- Container/Docker configuration
- Infrastructure-as-code (Terraform, Pulumi, CDK, etc.)
- DNS configuration
- Cloud resource permissions and IAM policies
- Authentication provider configuration
- SSL/TLS certificate management

**Action**: If an external contributor's PR touches any of these, close or request the changes be removed from the PR. These areas need to be replicated by a maintainer if the underlying logic is valid.

---

## Environment variables and secrets

### What to check

- [ ] Does the PR add new environment variable references in code?
- [ ] Was `.env.example` updated with the new variable (name only, safe placeholder value)?
- [ ] Is the variable genuinely needed, or could the value be a constant?
- [ ] Is the variable ever logged, printed, or returned in an API response?
- [ ] Does the PR **ever** hardcode a real value where an env var should be used?
- [ ] Does the PR introduce a new secrets pattern inconsistent with how existing secrets are managed?

### What must never happen

- Real credentials, tokens, or keys in any committed file
- Secrets in test fixtures, seed data, or mock files
- Logging of secret values (even at debug level)
- Passing secrets as command-line arguments (visible in process lists)
- Baking secrets into Docker images at build time

### If a secret was accidentally committed

1. Assume it is compromised immediately. Rotate it now, before anything else.
2. Do not just remove it in a follow-up commit — the git history still contains it.
3. Remove the secret from git history using `git filter-repo` or the equivalent.
4. Force-push the cleaned history.
5. Contact the relevant service provider and rotate credentials.
6. Review access logs for that secret to assess any unauthorized use.

See [INCIDENT_RESPONSE.md](./INCIDENT_RESPONSE.md) for the full exposed secret procedure.

---

## Authentication and authorisation

### What to check

- [ ] Does the change modify who can access what?
- [ ] Does it introduce a new auth flow, token type, or session mechanism?
- [ ] Does it change how permissions are enforced?
- [ ] Could any endpoint or resource become accessible without authentication after this change?
- [ ] Is the change backward compatible with existing sessions/tokens?
- [ ] Does the change disable or weaken any existing auth check?

### High-risk patterns

- Changing middleware order (auth middleware being moved after route handlers)
- Bypass conditions added for "development" that could affect production
- New public routes that should be protected
- Permission checks moved from server-side to client-side
- Caching of auth responses without understanding the cache key

**One auth mistake can expose all user data. Do not rush this review.**

---

## Databases and migrations

### What to check

- [ ] Does the PR add new migration files?
- [ ] Are the migrations in the correct sequence (no gaps, no reordering)?
- [ ] Does the migration add or rename columns safely (not dropping required fields)?
- [ ] Could the migration cause table locks on a live production database?
- [ ] Is the migration reversible (does a `down` migration exist and work)?
- [ ] If the migration changes existing data: is there a tested backfill plan?
- [ ] Does the application code handle **both old and new schema** during the migration window? (Critical for rolling deploys)
- [ ] Is there data that would be permanently deleted? Is that intentional?

### Table-locking risk (high traffic systems)

Certain operations lock tables and cause downtime:
- Adding a column with a non-null constraint and no default
- Creating an index without `CONCURRENTLY` (PostgreSQL)
- Altering column types on large tables
- Adding a foreign key constraint without `NOT VALID` first

If the project runs on significant traffic, these operations need a migration strategy, not just a migration file.

### Never allow an external contributor to

- Reorder existing migration files
- Modify existing migration files that have already been run in production
- Delete migration files
- Change the migration runner configuration

---

## CI/CD configuration

CI/CD files are **Category C** — maintainer-only.

### Why

A malicious or careless change to a CI/CD workflow can:
- Exfiltrate secrets (secrets are available to workflow steps)
- Replace production deployments with attacker-controlled code
- Modify release artifacts before they are published
- Disable security checks that protect the pipeline

### What to check even for maintainer-authored changes

- [ ] Are any `secrets.*` references being added or changed?
- [ ] Are any steps running untrusted external scripts or actions at an unpinned version?
- [ ] Are `pull_request_target` triggers being used? (These run with full repo permissions on PRs from forks — extremely dangerous)
- [ ] Is a new deployment environment being added? Are its secrets scoped correctly?
- [ ] Are any manual approval gates being removed?

### Safe practice for CI changes

- Pin third-party actions to a specific commit SHA, not a tag. Tags are mutable.
- Review every `run:` step for potential secret exposure.
- Do not give CI workflows more permissions than they need (use `permissions:` scoping in GitHub Actions).

---

## Docker and container configuration

### What to check

- [ ] Does the change modify base images? Is the new image from a trusted source?
- [ ] Are any `ENV` instructions in the Dockerfile setting secret values?
- [ ] Are build arguments (`ARG`) being used to pass secrets into the image? (These end up in image layers.)
- [ ] Does the container run as root when it does not need to?
- [ ] Are new ports being exposed?
- [ ] Does a `docker-compose.yml` change affect how services connect or what credentials they use?

### What never to do

- Bake credentials into Docker images at build time
- Pull images without pinning them to a specific digest
- Use `latest` tags in production-facing configurations

---

## Cloud resources and infrastructure-as-code

This is **Category C** territory. External contributors should not be modifying infrastructure definitions.

### For maintainer-authored changes

- [ ] Does this change create, modify, or destroy cloud resources?
- [ ] Are any IAM permissions being broadened? Can they be more restrictive instead?
- [ ] Are public-facing resources being created (public S3 buckets, open security groups)?
- [ ] Does the change affect production or staging specifically?
- [ ] Has the change been previewed with a dry-run (`terraform plan`, `pulumi preview`, etc.) before applying?
- [ ] Is state stored and locked properly to prevent concurrent modifications?

**Destroy operations in infrastructure-as-code are permanent. Verify twice.**

---

## External APIs and services

### What to check

- [ ] Does the PR introduce calls to a new external service?
- [ ] Is that service authenticated? How are the credentials managed?
- [ ] Can the service calls fail and cause cascading failures? Is there a timeout and retry policy?
- [ ] Could this change trigger unexpected charges on a metered service?
- [ ] If webhooks are involved: are the incoming webhooks validated (signature verification)?
- [ ] Is any user data being sent to the external service? Is that permitted by your privacy policy?

---

## Production configuration

Changes to production-specific configuration files (not env vars, but config YAML/JSON/TOML files deployed alongside the app):

- [ ] Is the change intentionally for production, or did a dev config accidentally get modified?
- [ ] Does the change affect feature flags, rate limits, or caching behaviour that production depends on?
- [ ] Is there a staged rollout, or does this affect all users immediately?

---

## DNS and domains

**Maintainer-only. Do not accept PRs that modify DNS or domain configuration.**

DNS changes propagate globally and can take minutes to 48 hours to take effect. A mistake can make the entire project unreachable. DNS changes should be made directly by the maintainer in the DNS provider's interface, not through code PRs.

---

## Storage

- [ ] Does the change create, modify, or delete files or objects in persistent storage?
- [ ] Could it delete data that cannot be recovered?
- [ ] Does it change file permissions or access controls on stored objects?
- [ ] Is it writing to a path it should not be writing to?
- [ ] For user uploads: is there validation of file type, size, and content?

---

## Permissions and access control

- [ ] Does the change alter user roles, permissions, or group memberships?
- [ ] Could any role gain access to something it should not?
- [ ] Is the least-privilege principle maintained?
- [ ] Are admin/superuser functions properly protected?

---

## Quick classification guide

Use this table when a PR arrives to decide how much scrutiny is needed:

| PR touches | Category | Action |
|---|---|---|
| Business logic, UI components | A | Standard review |
| New npm/pip/go dependency | B | Check security and license |
| New env var or config key | B | Check docs, no real values committed |
| External API integration | B | Check auth, timeouts, data handling |
| Background job / cron | B | Check timing, idempotency, failure handling |
| Database migration | B | Full migration checklist above |
| Auth / permissions | B+ | Slow, careful review |
| `.github/workflows/` | C | Maintainer only — reject or rewrite yourself |
| `Dockerfile`, `docker-compose.yml` | C | Maintainer only |
| Deployment scripts | C | Maintainer only |
| Terraform / infrastructure files | C | Maintainer only |
| DNS, domain config | C | Maintainer only |
| Secret manager configuration | C | Maintainer only |
