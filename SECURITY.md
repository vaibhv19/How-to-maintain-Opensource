# Security Policy

## Reporting a vulnerability

Do not report security vulnerabilities in public GitHub issues, pull requests, or discussions.

Use one of the following:

- **GitHub private vulnerability reporting**: Use the "Report a vulnerability" button on the Security tab of this repository (if enabled).
- **Email**: Contact the maintainer directly at the email address listed in the repository's GitHub profile or pinned contact issue.

Include:
- A clear description of the vulnerability
- Steps to reproduce it
- The potential impact
- Any suggested fix (optional)

You will receive a response within 7 days. Critical issues will be prioritised.

---

## What must never appear in issues or PRs

**Do not post the following in any public location:**

- API keys, tokens, or credentials of any kind
- Private environment variable values
- Database connection strings or credentials
- SSH keys or certificates
- Auth secrets (JWT secrets, OAuth client secrets, webhook signing keys)
- Internal hostnames, IP addresses, or service endpoints not intended to be public
- Personal data belonging to real users

If you find that any of the above has been accidentally committed to the repository, report it privately immediately. Do not document it in a public issue. Assume that any secret committed to a public repository is compromised and must be rotated.

---

## Security expectations for contributors

- Do not add dependencies without checking their security history.
- Do not disable security headers, CORS policies, rate limiting, or authentication checks in a PR without explicit prior discussion.
- Do not store sensitive values in code, config files committed to the repository, or test fixtures.
- Use the project's existing secrets management approach (environment variables, secret manager references) — do not introduce a new pattern.
- If your change touches authentication, authorisation, input validation, or data access, call this out explicitly in the PR description. These areas receive extra scrutiny.

For maintainer-side security review procedures, see [docs/maintainer/INFRASTRUCTURE_SAFETY.md](./maintainer/INFRASTRUCTURE_SAFETY.md).

---

## Supported versions

Security fixes are applied to the current `main` branch only. Older versions or releases are not backported unless explicitly noted.

---

## Disclosure policy

Once a vulnerability is fixed and a release is published, the maintainer may publish a summary of the issue. Reporters who wish to be credited will be acknowledged (with their permission).
