# How to Maintain Open Source

A practical maintainer playbook for running open-source projects safely — covering contribution workflows, infrastructure protection, release processes, and incident response.

This repository is a reusable reference system. Use it as a template when starting a new project or as an audit checklist for an existing one.

---

## What this repository contains

| Document | Purpose |
|---|---|
| [CONTRIBUTING.md](./CONTRIBUTING.md) | How contributors participate |
| [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) | Expected contributor behaviour |
| [SECURITY.md](./SECURITY.md) | How to report vulnerabilities |
| [CHANGELOG.md](./CHANGELOG.md) | Notable changes over time |
| [maintainer/MAINTAINER_GUIDE.md](./maintainer/MAINTAINER_GUIDE.md) | How to operate the project day-to-day |
| [maintainer/CONTRIBUTION_REVIEW.md](./maintainer/CONTRIBUTION_REVIEW.md) | What to actually check before merging a PR |
| [maintainer/INFRASTRUCTURE_SAFETY.md](./maintainer/INFRASTRUCTURE_SAFETY.md) | What can break beyond the codebase |
| [maintainer/RELEASE_PROCESS.md](./maintainer/RELEASE_PROCESS.md) | How changes safely reach production |
| [maintainer/INCIDENT_RESPONSE.md](./maintainer/INCIDENT_RESPONSE.md) | What to do when something goes wrong |

---

## Who this is for

**You, the maintainer.** This playbook assumes you are managing one or more open-source projects, potentially across different technology stacks — web applications, backend services, AI/ML systems, deployed infrastructure.

It is intentionally technology-agnostic at its core. Framework-specific notes are secondary.

---

## Where to start

**If you are setting up a new project:**
1. Copy the project-facing documents (README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, CHANGELOG, LICENSE) into your new repo.
2. Adapt them to your project's specifics.
3. Keep the maintainer docs in `maintainer/` as your private operating reference.

**If you are reviewing a contributor's PR:**
→ Start at [CONTRIBUTION_REVIEW.md](./maintainer/CONTRIBUTION_REVIEW.md)

**If something just broke in production:**
→ Start at [INCIDENT_RESPONSE.md](./maintainer/INCIDENT_RESPONSE.md)

**If you want to understand the full maintainer workflow:**
→ Start at [MAINTAINER_GUIDE.md](./maintainer/MAINTAINER_GUIDE.md)

**If a PR touches environment variables, secrets, databases, or deployment config:**
→ Read [INFRASTRUCTURE_SAFETY.md](./maintainer/INFRASTRUCTURE_SAFETY.md) before doing anything else

---

## Document dependency map

```
CONTRIBUTING.md
  └─ explains how contributors participate

MAINTAINER_GUIDE.md
  └─ explains how you operate the project

CONTRIBUTION_REVIEW.md
  └─ explains how you evaluate incoming changes

INFRASTRUCTURE_SAFETY.md
  └─ explains what can break beyond the codebase

RELEASE_PROCESS.md
  └─ explains how changes safely reach users

INCIDENT_RESPONSE.md
  └─ explains what to do when something goes wrong
```

Each document links to the others rather than duplicating content.

---

## How to adapt this to a specific project

This repo is a template, not a finished product. When applying it to a real project:

- Replace generic references with project-specific details (repo name, tech stack, deployment platform, CI system).
- Remove sections that do not apply (e.g. database migration guidance for a static site).
- Add sections for things specific to your project that are not covered here.
- Keep the maintainer docs updated as the project's infrastructure evolves.

Which documents are **essential** for any project:
- README, CONTRIBUTING, LICENSE, SECURITY, CODE_OF_CONDUCT

Which documents are **recommended** once you have contributors:
- CHANGELOG, MAINTAINER_GUIDE, CONTRIBUTION_REVIEW

Which documents matter **as the project grows in complexity**:
- INFRASTRUCTURE_SAFETY, RELEASE_PROCESS, INCIDENT_RESPONSE

---

## License

See [LICENSE](./LICENSE).
