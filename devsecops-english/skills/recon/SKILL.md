---
name: recon
description: Use at the beginning of any web security test, when you need to map the target's attack surface before testing: subdomains, DNS, ports and services, technologies, WAF, security headers, endpoints, APIs, authentication and roles. RECONNAISSANCE phase of the DevSecOps agent methodology. Keywords: recon, reconnaissance, enumeration, footprinting, subdomain, attack surface, OSINT, httpx, nmap, ffuf.
---

# Reconnaissance (Recon)

Map the target's complete attack surface before any vulnerability testing.

## Rules before you start
- Confirm the target is within the authorized scope (contract, RoW/ALP, bug bounty, lab).
- Do not run aggressive scans (high speed, rate-limit bypass) unless authorized.
- Record the date/time of each activity to compose report evidence.

## 1. Scope and passive OSINT
- Subdomains: `subfinder -d target.com -all`, `amass enum -passive -d target.com`, query `crt.sh` (`https://crt.sh/?q=%25.target.com`).
- DNS resolution: `dnsx -l subdomains.txt -a -cname -resp`, `dig`, `nslookup`.
- Google dorks / Shodan / Censys for exposed assets (public read-only only).
- Always save the subdomain list to `recon/subdomains.txt`.

## 2. Active reconnaissance
- Probe live hosts: `httpx -l subdomains.txt -title -tech-detect -status-code -server -location -o recon/live.txt`.
- Ports/services on the main target: `nmap -sV -sC --top-ports 1000 -oA recon/target` (and a full scan on small scopes).
- Technology fingerprinting: `whatweb -a 3 URL`, Wappalyzer extension, `Server`, `X-Powered-By` headers, cookies.
- Identify deployment technologies: container/UUID, CDN platform, framework.

## 3. Content discovery
- Directories/files: `ffuf -w wordlist -u http://target.com/FUZZ -mc 200,204,301,302,307,401,403`.
- Useful wordlists: `raft-large-directories.txt`, `directory-list-2.3-medium.txt`, `common.txt`.
- Special files: `robots.txt`, `sitemap.xml`, `.well-known/*`, `favicon.ico` (hash), backup files (`.bak`, `.zip`, `.git/`), documentation files.
- Test for exposure of `.git/` (git-dumper), `.env`, `.svn`, admin panels.

## 4. API discovery
- Search for `/api`, `/api/v1`, `/swagger`, `/swagger-ui`, `/v3/api-docs`, `/openapi.json`, `/graphql`.
- If GraphQL: test whether `introspection` is enabled.
- Collect endpoints, parameters, authenticated/unauthenticated routes (HTTP method + roles).
- Record API contracts in `recon/api-endpoints.md`.

## 5. WAF and infrastructure
- Detect WAF: `wafw00f URL`, analyze headers/cookies (`cf-ray`, `__cfduid`, `x-sucuri-id`, `set-cookie: xd_visitor`).
- Identify CDN/cloud (Cloudflare, AWS CloudFront, Azure Front Door, Akamai, Incapsula).
- Map the real IP behind the CDN (DNS history, non-proxied subdomains) only if authorized.
- Use the `waf-analysis` skill for deep analysis and bypass.

## 6. Security headers (baseline)
Audit and document per host:
- `Strict-Transport-Security` (HSTS), `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, `X-XSS-Protection`.
- CORS: send a request with `Origin: https://evil.com` and check for echoing/credentialing in `Access-Control-Allow-Origin`.

## 7. Authentication and roles
- Locate: login pages, SSO/OAuth/OIDC flows, JWTs (audit algorithm, exp, payload), session cookies (flags `Secure`, `HttpOnly`, `SameSite`).
- Map roles/permissions (admin, user) and administrative endpoints.
- Record in `recon/auth-surfaces.md` (login, register, reset password, MFA, API keys).

## 8. Evidence organization
Mandatory output structure (to feed the next skills and the report):
```
recon/
├── subdomains.txt
├── live.txt
├── nmap/            (target.nmap, target.gnmap, target.xml)
├── endpoints.md     (URLs, methods, parameters, relevant headers)
├── api-endpoints.md (API contract, required authentication)
├── auth-surfaces.md (login/register/reset, tokens, roles)
└── notes.md         (technologies, WAF, CDN, CSP/CORS, preliminary findings)
```

## Expected output
A compact summary: list of live hosts, identified technologies, present WAF/CDN, discovered endpoints, authentication surfaces, and the populated `recon/` structure as the foundation for the `waf-analysis`, `owasp-web-testing`, `llm-agentic-testing` and `exploitation` skills.