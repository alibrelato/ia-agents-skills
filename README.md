# ia-agents-skills

A collection of specialized AI agents and skills for opencode, organized by domain. Each domain ships a ready-to-use agent and its supporting skills.

## What's inside

- **Agents** — full agent definitions (system prompts, methodology, expected output) that turn the model into a domain specialist.
- **Skills** — specialized, on-demand instructions that the agent loads when the task matches the skill's trigger keywords.

## Repository structure

```
.
├── devsecops-waf/          # Offensive web security (DevSecOps) agent
│   ├── agent/agent.md      # Agent definition
│   └── skills/             # Supporting skills
│       ├── recon/          # Attack surface mapping
│       ├── waf-analysis/   # WAF/ModSecurity rules and bypass
│       ├── owasp-web-testing/  # Systematic OWASP Top 10 tests
│       ├── llm-agentic-testing/ # LLM/Agentic App Top 10 tests
│       ├── exploitation/   # Controlled PoC exploitation
│       └── reporting/      # Standardized technical report
├── mikrotik/               # MikroTik RouterBOARD expert agent
│   ├── agent/agent.md      # Agent definition
│   └── skills/             # Supporting skills
│       ├── mikrotik-configuration/     # Create/configure/administer RouterOS
│       ├── mikrotik-improvements/      # Config audit and optimization insights
│       ├── mikrotik-security/          # RouterOS hardening and access control
│       ├── mikrotik-backup-analysis/   # Read .backup / .rsc files
│       ├── mikrotik-scripts/           # RouterOS scripting and automation
│       ├── mikrotik-documentation/     # Official docs per model/firmware
│       └── mikrotik-log-analysis/      # Log analysis and troubleshooting
├── opencode.json           # Registers skill paths for opencode
└── LICENSE
```

## Domains

### devsecops-waf — Offensive Web Security
An ethical DevSecOps agent that performs reconnaissance, WAF analysis, systematic OWASP Top 10 (2021) and LLM/Agentic App Top 10 testing, controlled exploitation and standardized reporting (severity + CVSS, PoC reproduction, remediation). Operates only in authorized environments and follows PTES, OWASP Testing Guide, NIST SP 800-115 and OSSTMM.

### mikrotik — MikroTik RouterBOARD Expert
A specialist that creates, configures, administers and audits MikroTik routers (RouterOS v6/v7). Reads and restores `.backup`/`.rsc` files, writes configuration scripts and scheduler automation, finds and applies the official documentation for the exact RouterBOARD model and firmware, hardens the device (firewall, access control, brute-force/DDoS protection) and analyzes logs to diagnose and resolve problems.

## Using with opencode

Skill paths are registered in `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "skills": {
    "paths": ["./devsecops-waf/skills", "./mikrotik/skills"]
  }
}
```

After adding or editing agents/skills, restart opencode to reload the configuration.

## Ethical use

The DevSecOps agent skills are intended exclusively for authorized environments (labs, CTFs, bug bounty programs, contracted penetration tests). Always obtain explicit permission and respect local laws and security policies.

## License

GPL-3.0 — see [LICENSE](LICENSE).