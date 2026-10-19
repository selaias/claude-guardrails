---
description: Full security audit — code, infrastructure, keys/credentials. Report only, no fixes.
---

# /security-scan — Full security audit

Audit the code, the servers and the credentials, then report. This is a report only: don't fix anything until the user has read it and decided what matters.

Start by working out the stack from the package manager files, frameworks, templating engine and database, then adapt each check below to it. If a check doesn't apply, skip it and say so in the report rather than leaving it out silently.

## Scope

### Code Audit
- **Secrets in source:** grep all tracked files for API keys, tokens, passwords, connection strings with credentials, private keys. Also check git history for previously committed secrets (`git log -p -S <pattern>`).
- **XSS vectors:** unescaped user input rendered in templates (e.g. React `dangerouslySetInnerHTML`, Vue `v-html`, Handlebars/Blaze `{{{triple}}}`, Jinja `|safe`, raw `innerHTML`).
- **Injection risks:** string-concatenated SQL, unsanitized NoSQL query objects, shell commands built from user input.
- **Auth coverage:** API routes, server methods, and data endpoints without input validation or auth/role guards.
- **Data exposure:** endpoints or queries returning full records where a field projection/DTO should be used.
- **CSP & security headers:** review the CSP/helmet/equivalent config — flag unnecessary wildcards and missing headers.
- **PII in logs:** logger calls that write emails, tokens, passwords, or payment data in plain text.
- **Exposed routes:** routes reachable without authentication that should be protected.
- **Rate limiting:** login, signup, password reset, OTP, form submissions, and public APIs without rate limits.
- **Dependencies:** run the ecosystem's audit tool (`npm audit`, `pip-audit`, etc.) and report high/critical items.

### Infrastructure Audit
Only for servers the user has given access to. Read-only commands only.
- **Firewall:** status and rules (e.g. `ufw status verbose`).
- **SSH:** password auth disabled? Root login disabled? Key-only?
- **Brute-force protection:** fail2ban or equivalent running?
- **Open ports:** `ss -tlnp` — flag anything listening publicly that shouldn't be.
- **TLS certificates:** expiry dates (`certbot certificates` or `openssl`).
- **Database:** auth enabled? Bound to localhost/private network? IP allow-list current?
- **Reverse proxy (nginx/caddy/etc.):** security headers, proxy settings, directory listing off.

### Keys & Credentials Audit
- **Rotation age:** when each third-party key (payments, email, OAuth, cloud) was last rotated — ask the user if it can't be determined.
- **CI secrets:** all required secrets set, no stale or unused ones.
- **Environment files on servers:** not world-readable (`ls -l`), owned by the app user.
- **Config files:** secret-bearing config is gitignored; no copies committed anywhere in the repo.
- **Backups:** check whether backup files or dumps contain raw tokens or passwords.

**Never print a secret value in the report.** Show only the file, line, and key name — values entering the transcript count as leaked.

## Output

Report as a table per category:

| Finding | Severity | File/Location | Details |
|---------|----------|---------------|---------|
| ... | P0/P1/P2 | path:line | description |

Summary:
- P0 (critical): X findings — must fix before any deploy
- P1 (high): X findings — fix soon
- P2 (medium): X findings — fix when convenient

**Do NOT fix anything automatically. Report and wait for the user to prioritize.**

## Guardrails this enforces
- #1 Security comes first
- #3 Nothing irreversible without approval (this audit is read-only)
- #4 Check before you claim (every finding cites a location you actually opened)
