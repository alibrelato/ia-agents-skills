---
name: mikrotik-scripts
description: Use to create RouterOS configuration scripts and .rsc files: RouterOS scripting language (variables, conditionals, loops, functions, arrays), /system scheduler automation, auto-provisioning, exports, CLI batch scripts for RouterOS v6/v7, and safe script execution. Keywords: script, scripts, rsc, config file, routeros script, scripting, scheduler, scheduling, variables, loops, functions, automation, provisioning, export, import.
---

# MikroTik Configuration Scripts (RouterOS)

Create safe, reusable and auditable configuration scripts for RouterOS v6/v7, from isolated commands to scheduler automation.

## Script safety rules
- Never paste clear-text passwords in the script; use script environment variables or parameterize.
- Test on a VM/CHR; start with `:log` and precondition validations before changing anything.
- Add `comment` and a header with version/purpose.
- Use no void commands; set up rollback where possible.
- Keep a backup before execution (`/system backup save`).

## 1. RouterOS language fundamentals
- Comments: `#` (in scripts) and `:global`/`:local` variables.
- Variables: `:local var 5`, `:global total + 10`.
- Conditionals: `:if (cond) do={...} else={...}`.
- Loops: `:for i from=1 to=10 do={...}`, `:while (cond) do={...}`, `:foreach i in=[/ip address find] do={...}`.
- Functions: `:global myFunc do={ ... }` and calling `$myFunc arg`.
- Arrays/hashes and properties: `$arr->property`.
- Outputs/text: `:put`, `:log info "msg"`, string formatting with `:set`.
- Capturing results: `:local x [/ip address get [/ip address find where interface=ether1] address]`.

## 2. Structure of a configuration script (`.rsc`)
Example of a parameterized block:
```
# ===== Basic Provisioning Script =====
# Purpose: configure IP, DHCP and NAT
:local lanIP "192.168.50.1"
:local pool "192.168.50.100-192.168.50.200"
:comment set identity "MY-ROUTER"
/interface bridge add name=bridge-lan
/ip address add address=$lanIP/24 interface=bridge-lan
/ip pool add name=pool-lan ranges=$pool
/ip dhcp-server add name=dhcp-lan interface=bridge-lan address-pool=pool-lan
/ip firewall nat add chain=srcnat out-interface=WAN1 action=masquerade
:log info "Provisioning completed"
```
- Multi-line commands require `\` as continuation: in RouterOS, line breaks between properties work when putting `\` at the end of the line or inside parentheses.
- Always run `/import file-name=<file>.rsc` with `check=yes` when available (syntax validation).

## 3. Automation with Scheduler
- Create job: `/system scheduler add name=backup-daily start-time=03:00 interval=1d on-event=/"system backup save name=auto"`.
- Also: smaller `interval` for loop tasks; `run-count` limit.
- Use `/system scheduler print` and test with `run`: `/system scheduler run <name>`.
- Scripts run asynchronously; use `:delay` to wait for resources.

## 4. Failover / routine automation (example)
- Gateway check and route switch:
```
:global wan1 "10.0.0.1"
:global wan2 "10.0.1.1"
:if ([:ping $wan1 count=1] = 0) do={
  /ip route set [find where dst-address=0.0.0.0/0] gateway=$wan2 distance=1
  :log warning "WAN1 down; route migrated to WAN2"
} else={
  /ip route set [find where dst-address=0.0.0.0/0] gateway=$wan1 distance=1
}
```
- Visual feedback: `/tool fetch`, e-mail via `/tool e-mail send`.

## 5. Mass provisioning / auto-config
- Use model data collection (`/system routerboard print`) and conditionals to apply different config per model.
- Templates per role (network, gateway-focused, core-focused) in `.rsc` files.
- Redeploy: server-side bash script generates `.rsc` by parameterization and uploads via WinBox/SSH.
- Do not apply to the whole network at once; keep idempotency (existence checks before `add`).

## 6. Import, versioning and best practices
- Version scripts in a repo (git-like) together with exports.
- Users: script with group rights (e.g.: `script,policy` or `full` if necessary) — principle of least privilege.
- Error handling: `:if ([/ip route get [find] ... ]=...)` and `:log error`.
- Document each script with a header: purpose, version, author, model/version dependency.

## Expected output
Ready-to-download/paste scripts, commented, with header, preconditions, logs and rollback; plus the scheduler indication if applicable and suggested tests (VM/CHR before production).