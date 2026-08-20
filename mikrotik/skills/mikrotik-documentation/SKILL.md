---
name: mikrotik-documentation
description: Use to find, read and apply the official MikroTik documentation for the specific RouterBOARD model and RouterOS firmware version in question: help.mikrotik.com manuals (per RouterOS version), routerboard.com hardware specs and FAQ, changelogs, and how to fetch the right docs before configuring, comparing or troubleshooting. Keywords: documentation, official docs, help.mikrotik.com, routerboard, manual, changelog, version, firmware, model, support, specs, hardware, requirements, faq.
---

# Official MikroTik Documentation by Model and Version

Locate, navigate and apply the official MikroTik documentation regarding the exact RouterBOARD model and installed RouterOS version — avoiding generic/outdated versions.

## Official sources
- **RouterOS Manuals (per version)**: `https://help.mikrotik.com/docs/` — "RouterOS" section with content specific per version (v6 and v7). English documentation is the canonical reference.
- **Hardware per model**: `https://routerboard.com/` and product pages at `https://mikrotik.com/` (data sheet, photos, ports, LEDs, specs).
- **Changelog / Versions**: `https://mikrotik.com/download/` and changelogs at `help.mikrotik.com` (check improvements, deprecations, fixes).
- **FAQ / Official forum**: `https://help.mikrotik.com/docs/` FAQ, and the community at `https://forum.mikrotik.com/`.
- **Command reference manual**: embedded Console `help` (`?` and `/?` in each path) which demands the installed version — always confirm before proposing commands.

## Mandatory steps
1. **Identify the device** (do not assume): run/require
   - `/system routerboard print` → model, firmware, serial.
   - `/system resource print` → version, build-time, CPU, memory.
   - `/system package print` → packages/channel (stable/beta).
2. **Map to the correct official document**:
   - Model → product page at routerboard.com (ports, switch chip, PoE, SFP specs).
   - Version → specific RouterOS manual (e.g.: "RouterOS v7" vs "RouterOS 6.x").
   - Rule of thumb: if the manual item cites a feature missing/different from the installed version, prioritize the real version's manual.
3. **Extract the relevant sections** for the task: interfaces, bridge/VLAN, firewall/NAT, routing, services, security, upgrade.
4. **Verify package/feature compatibility** with the model (e.g.: switch chip in CRS, hardware offload in hEX/CCR).
5. **Check version milestones**: native `wireguard` (v7+), `/routing` structure (v7), `fasttrack` (several versions), end-of-life of v6.x — recommend an upgrade when the version is out of support.

## Efficient web lookup
- Search the manual with official terms: `site:help.mikrotik.com <feature> RouterOS v7`.
- Use webfetch to open the manual page, extract official command examples and translate syntax differences.
- For hardware specs use `site:routerboard.com <model> specifications`.
- Always cross-check the changelog for security fixes of the installed version (skill `mikrotik-security`).

## Handling ambiguities
- If the user does not know the model/version, ask for the diagnostic command outputs (do not guess).
- If there is a manual page for multiple branches, prioritize the used branch (v7) and note the v6 divergence when relevant.
- Do not confuse "Wireless" generic docs with model-specific "Wireless/AC" docs; validate the SKU.

## Expected output
Documentation summary applied to that device: confirmed model/version, cited official links, documentation excerpts relevant to the task, commands with correct syntax for the version, and compatibility/upgrade warnings when applicable.