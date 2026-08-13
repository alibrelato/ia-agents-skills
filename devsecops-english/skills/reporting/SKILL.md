---
name: reporting
description: Use at the end of the engagement, when all tests and exploitations are complete, to generate the technical report in the DevSecOps agent's standardized format: title, severity with CVSS, affected asset, technical description, evidence, step-by-step PoC, impact, remediation recommendations and references. Also generates the executive summary and payload/log attachments. Keywords: report, documentation, markdown, CVSS, executive summary, PoC, severity, critical severity.
---

# Technical Report

Consolidate all findings into the agent's standardized report, reproducible and actionable. This skill consumes artifacts from the `recon`, `waf-analysis`, `owasp-web-testing`, `llm-agentic-testing` and `exploitation` skills.

## Mandatory report structure

For EACH vulnerability found, generate the following sections:

### 1. Title
- Format: `<Vulnerability class> <context> in <target>` (e.g., "Authenticated SQL Injection in /api/v1/users (POST)").

### 2. Severity
- Level: Critical / High / Medium / Low / Informational.
- CVSS 3.1/4.0 when applicable: full vector + score (e.g., `AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` → 7.5).
  - Calculate with the FIRST CVSS Calculator or official software; justify each metric.

### 3. Affected asset
- URL/IP/hostname, application, component, environment (prod/staging/lab) and version when identified.

### 4. Technical description
- What it is, why it occurs (root cause), context (authentication, role, parameters, HTTP method, relevant headers).

### 5. Evidence
- Full HTTP request (headers + body), relevant response excerpt, screenshots (if any), SHA256 hashes of critical payloads, timestamps.

### 6. Proof of concept (PoC) — Reproduction
- Numbered, copy-pasteable steps (including test credentials and an example token), re-executable in an identical environment:
  ```bash
  curl -X POST 'https://target.com/api/v1/users' \
    -H 'X-Auth-Token: <token>' \
    -H 'Content-Type: application/json' \
    -d '{"id": "1 OR 1=1--"}'
  ```
- Indicate the expected result that proves the vulnerability.

### 7. Impact
- What an attacker achieves (e.g., full DB access, RCE, session theft, exposed PII).
- Business impact: financial, reputational, compliance (LGPD/GDPR, PCI-DSS, etc.).

### 8. Remediation recommendations
- Specific fix and code examples (prepared statements, input validation, CSP, CORS fix, etc.).
- Prioritization by severity (fix critical first) and references (OWASP, CWE, vendor advisories).

### 9. References
- OWASP Top 10 (item and link), CWE ID, related CVEs, vendor documentation.

## Executive summary (1 page)
For non-technical stakeholders:
- Test context (scope, authorization, period).
- Number of findings by severity (table).
- Top 3 risks in plain language.
- Priority recommendations.
- Overall risk status (e.g., High).

## Mandatory attachments
1. **List of tested payloads** (`payloads.csv` or `.json`): payload field, technique, status (blocked/passed/validated), SHA256 hash, reference to the finding.
2. **Tool logs**: outputs of nuclei, sqlmap, nmap, ffuf, wafw00f etc. (summarized, without sensitive data).
3. **Glossary** of acronyms (if there is a mixed audience).

## Redaction and integrity
- Mask any real sensitive data (passwords, tokens, PII) with `***`.
- Do not include instructions that allow exploiting systems without authorization.
- Order findings by severity (Critical → Informational).
- Standard file naming: `report/<target>-pentest-report.md` (or `.pdf` via conversion), alongside `executive-summary.md` and the attachments.

## Final verification (checklist)
- [ ] Every vulnerability has all 9 sections.
- [ ] Every PoC was tested/validated (not theoretical).
- [ ] Severity + CVSS calculated.
- [ ] Evidence without real sensitive data.
- [ ] Executive summary and attachments generated.
- [ ] Methodology/tools/versions documented (PTES, OWASP Testing Guide, NIST SP 800-115, OSSTMM).