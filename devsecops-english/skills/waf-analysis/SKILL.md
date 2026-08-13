---
name: waf-analysis
description: Use when the target has a WAF or ModSecurity and you need to identify the active rules, false positives, bypass techniques and detection logs. Covers Cloudflare WAF, AWS WAF, Azure WAF, Nginx + ModSecurity (OWASP CRS). Keywords: WAF, ModSecurity, CRS, bypass, evasion, Cloudflare, AWS WAF, Azure WAF, wafw00f, detection rules, paranoia level.
---

# WAF / ModSecurity Analysis

Identify the WAF, map the active rules and evaluate evasion techniques in a controlled manner.

## 1. WAF identification
- `wafw00f URL` for fingerprinting.
- Indicators in the response:
  - Headers: `cf-ray`, `__cfduid` (Cloudflare); `x-sucuri-id`, `x-cdn` (Sucuri); `x-azure-ref` (Azure Front Door); `x-aws-*` (CloudFront/AWS); `server: cloudflare`.
  - Block response: page with incident ID/`cf-error`, 403/406/418/429 status, JS challenge/captcha.
  - Cookies: `__cf_bm`, `AWSALB`, `AzureAdSessionCookie`, `_cflb`.
- Record vendor/version and whether it is in detect or block mode.

## 2. Enumerate active rules
- For ModSecurity/CRS: identify the active paranoia levels (PL1–PL4) by testing basic payloads that trigger rules at each level.
- Test rule categories: SQLi (930xxx), XSS (941xxx), LFI/Path Traversal (930/931), Command Injection (932xxx), Java/deserialization (934xxx), generic RCE (932xxx), protocol (920xxx).
- Differentiate blocking by detection rule vs. rate limit vs. challenge (response behavior).

## 3. Bypass techniques (test each with a valid and an invalid request)
All techniques below must be tested with real payloads and documented (blocked = WAF response, passed = application response):

- **Encoding**: double URL encoding (`%2527`), full-width/Unicode (`％２７`), HTML encoding (`&#x27;`), base64/hex in interpreted parameters.
- **Case variation**: `SeLeCt`, `UNION sElEcT` when the backend is case-insensitive.
- **Comment injection**: `UN/**/ION SE/**/LECT`, `a/**/b`, `%0a`/`%0d` (newline/tab `%09`) inside payloads.
- **Null byte/truncation**: `%00` in arguments (rare in HTTP, common in filesystems).
- **HTTP Parameter Pollution (HPP)**: repeat parameters (`?id=1&id=1 OR 1=1`), argument in JSON+form, `id[]`, duplicate parameter names (WAF inspects only the first, backend uses the last).
- **HTTP method override**: `X-HTTP-Method-Override`, `X-Method-Override`, `_method`, POST method for payloads the WAF only inspects on GET (and vice versa).
- **Chunked Transfer-Encoding**: send the body in chunks (`Transfer-Encoding: chunked`), a classic technique against ModSecurity on older servers.
- **Content-Type confusion**: send the payload with `Content-Type: multipart/form-data`, `application/json`, `text/plain`, `application/x-www-form-urlencoded` and observe which one the WAF inspects.
- **Protocol/edge**: HTTP/2 → HTTP/1.1 downgrade, connection reuse, `Host`/`User-Agent` variations, paths with `/./` and `//`, trailer headers.
- **Oversized/limits**: send payloads above `SecRequestBodyLimit` to trigger a "truncation" bypass of the WAF (only if the backend accepts a larger body).
- **WebSocket / internal API**: endpoints the WAF does not inspect (WebSocket, internal addresses, `/admin`, direct IP).

## 4. Bypass validation
- Confirm the payload **actually** reached the application: response with SQL/syntax error, behavior change, or operation success.
- A generic 200 status is NOT proof of bypass — validate the response.
- Cross-check each bypass against the detection logs (if accessible) to confirm the rule did not fire.

## 5. False positives
- Differentiate: WAF blocking vs. application error (generic 404, 500 without detection body) vs. rate limit (429/Retry-After) vs. challenge (JS/captcha).
- Record which rules produce false positives on legitimate traffic.

## 6. Detection logs
- Where to look: ModSecurity (`SecAuditLog`), Nginx access logs, WAF dashboards/analytics (Cloudflare analytics, AWS WAF sampled requests), SIEM alerts.
- Log patterns to record: rule ID, `SecRule ID:932230`, URI, source, timestamp, detected offensive payload.

## 7. Documentation
Mandatory output:
```
waf/
├── fingerprint.md      (vendor, version, mode, real IP if mapped)
├── rules-detected.md   (rule IDs/paranoia levels identified)
├── bypass-validated.md (each technique: payload, full request, blocked/passed, evidence)
└── fp-analysis.md      (false positives and blocking behavior)
```
Each validated bypass must include the full HTTP request (headers + body) and a response excerpt, for reuse in the `owasp-web-testing` and `exploitation` skills.