---
name: mikrotik-security
description: Use to secure MikroTik RouterOS administration and the device itself: hardening of users and access (SSH/WinBox/API/WebFig), firewall input/output/forward policies, DDoS and brute-force protection, connection tracking, disabling insecure services, update policy and secure tunneling (WireGuard/IPsec). Follows MikroTik security guidelines per RouterOS version. Keywords: security, hardening, firewall, mikrotik security, brute force, ddos, winbox, ssh, api, wireguard, ipsec, connections, input chain, protection.
---

# MikroTik Administration Security

Harden the MikroTik router (RouterOS v6/v7): administrative access control, firewall protection of the device itself and of the networks served, and mitigation of common attacks.

## Rules before starting
- Connect via local/secure access before applying restrictive rules so you do not lose access.
- Keep an escape rule (e.g.: allow WinBox/SSH only from a known range + management path).
- Order firewall rules efficiently (most used first); keep `comment`.
- Account for the installed version (some parameters change v6/v7).

## 1. Administrative access control
- Disable unnecessary services: `/ip service disable telnet,ftp,webfig` (keep only ssh and/or winbox as needed).
- Restrict by network: `/ip service set ssh address=192.168.10.0/24`; same rule for `winbox`, `api`.
- Change default ports if exposed to the internet (`/ip service set ssh port=2222`), without conflicts.
- Users: `/user add name=admin2 group=full password=<strong>`; disable orphan users, use groups with least privilege (`/user group`).
- SSH key authentication instead of passwords (`/user ssh-keys import user=admin2 public-key-file=chave.pub`).
- `/ip service print` to confirm the final state.

## 2. Firewall to protect the router itself (input chain)
- Drop invalid: `/ip firewall filter add chain=input connection-state=invalid action=drop comment="drop invalid"`.
- Allow only essentials in input with explicit exception of the management range:
  - `add chain=input connection-state=established,related action=accept`
  - Move access rules (ssh/winbox) up.
- Final defensive rule: `/ip firewall filter add chain=input action=drop comment="drop all input"` (default drop-all).

## 3. Forward firewall (client traffic)
- Drop invalid (`connection-state=invalid`).
- Accept established/related: `add chain=forward connection-state=established,related action=accept`.
- Final `drop` rule so not everything is allowed.
- Filter by network interface (e.g.: block internal access to management network, DMZ).
- Consider `connection-state=new` + `dst-port` to restrict external services.

## 4. Brute-force protection (SSH/WinBox/API)
- Rate-limit new: `add chain=input protocol=tcp dst-port=22,8291 connection-state=new src-address-list=bruteforce action=drop`
- Populate the list dynamically:
  ```
  add chain=input protocol=tcp dst-port=22,8291 connection-state=new action=add-src-to-address-list address-list=bruteforce address-list-timeout=1h
  ```
- In v6, combine with connection rate rules (`/ip firewall filter add chain=input ... limit`) or a sniper script; for simplicity, keep the address-list with timeout.

## 5. DDoS and flood mitigation (external connections)
- Notrack for external hosts when necessary.
- Limit new TCP connections per IP (`/ip firewall filter add ... protocol=tcp connection-state=new src-address-list=... action=add-dst-to-address-list`).
- Limit ICMP (ping) and UDP floods by policy (`limit=...`).
- Connection tracking tuning: `/ip firewall connection tracking set tcp-established-timeout=...`.
- Be careful not to drop legitimate traffic; test with `/tool bandwidth` and logs.

## 6. Secure VPN and controlled external access
- WireGuard (v7): generate keys, peer policy, avoid exposing the whole setup.
- IPsec/IKEv2 when required by compliance.
- Keep forwarding ports minimized (only really public services).
- Do not leave `winbox`/`api` exposed directly to the internet; use a VPN.

## 7. Update policy and vulnerability posture
- Keep the RouterOS version current/stable; check the changelog (skill `mikrotik-documentation`).
- Removable packages (new v7 architecture: `installed-packages` via `/system packages`).
- Do not use default passwords; change the admin password on new equipment.
- Document password and permission policies (users/groups).

## 8. Final security checklist
- [ ] Services: only ssh/winbox (and api if needed) active, restricted by IP.
- [ ] Input chain with drop-default and minimal exceptions.
- [ ] Forward chain with established/related accept + final drop.
- [ ] Brute-force protection on ports 22/8291.
- [ ] Notrack/fasttrack evaluated for performance without opening holes.
- [ ] Strong, non-default passwords, users by role.
- [ ] Backup before/after every sensitive change.
- [ ] Access test after each rule block (do not leave the operator out).

## Expected output
Numbered copyable hardening command blocks, with application order, a security checklist and reversibility care (maintenance window, emergency access via local console).