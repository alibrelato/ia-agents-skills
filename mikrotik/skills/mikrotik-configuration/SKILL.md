---
name: mikrotik-configuration
description: Use when you need to create, configure or administer MikroTik RouterOS settings: interfaces, bridges, VLANs, IP addressing, routing, DHCP, NAT, firewall, services and the RouterOS v6/v7 CLI. Covers the full configuration workflow from base setup to production. Keywords: mikrotik, routeros, configuration, configure, administer, routerboard, bridge, vlan, dhcp, nat, routing, ip, interface, license, packages.
---

# MikroTik RouterOS Configuration & Administration

Create, configure and administer MikroTik routers (RouterOS v6/v7) in a complete, organized and production-ready way.

## Rules before starting
- Identify the model (`/system routerboard print`) and version (`/system resource print`).
- Confirm all installed packages (`/system package print`).
- Take a `.backup` backup and a `.rsc` export (skill `mikrotik-backup-analysis`) before any relevant change.
- Work in functional blocks, in the order below, validating each step.

## 1. System basics
- Set identity: `/system identity set name=<name>` (e.g.: `RB-BR-DATACENTER-01`).
- Set time zone and NTP: `/system clock set time-zone-name=America/Sao_Paulo`, `/system ntp client set enabled=yes server=<srv>`.
- Set DNS: `/ip dns set servers=1.1.1.1,8.8.8.8 allow-remote-requests=yes`.
- Addressing grid: document LAN, WAN, VLANs, DHCP Pool, reservations, default routes.
- Note the license (`/system license print`) and validate support for the features used (some features require higher levels).

## 2. Interfaces, bridges and VLANs
- Rename interfaces: `/interface ethernet set ether2 name=WAN1`.
- Create a bridge: `/interface bridge add name=bridge-LAN`; configure `protocol-mode`, `vlan-filtering`.
- VLANs: `/interface vlan add name=LAN10 vlan-id=10 interface=bridge-LAN`.
- Add ports: `/interface bridge port add bridge=bridge-LAN interface=ether3`.
- Switch chip (CRS): use the `switch` ifaces when applicable to the model; check the official hardware doc.
- Checks: `/interface bridge port print`, `/interface vlan print`, `ping` per VLAN.

## 3. IP addressing and network services
- IPs: `/ip address add address=192.168.10.1/24 interface=bridge-LAN network=192.168.10.0`.
- Default route: `/ip route add dst-address=0.0.0.0/0 gateway=<WAN-GW>` (skill `mikrotik-improvements` for failover).
- DHCP server: `/ip pool add name=pool-LAN ranges=192.168.10.100-192.168.10.200`, `/ip dhcp-server add address-pool=pool-LAN interface=bridge-LAN`.
- DHCP client on WAN: `/ip dhcp-client add interface=WAN1 disabled=no`.
- DHCP reservations: `/ip dhcp-server lease add address=192.168.10.50 mac-address=AA:BB:.. server=bridge-LAN`.

## 4. Routing
- Static: `/ip route add dst-address=10.0.0.0/8 gateway=10.20.0.1`.
- OSPF: `/routing ospf instance add name=inst0`, `/routing ospf area add name=backbone instance=inst0` (v7); `/routing ospf network add network=...` (v6).
- BGP: `/routing bgp connection add ...` (v7) or `/routing bgp instance/template set ...` (v6).
- PBR: address-lists + route tables (`/routing table`).

## 5. NAT and firewall (basics for operation)
- NAT out: `/ip firewall nat add chain=srcnat out-interface=WAN1 action=masquerade`.
- NAT in (port): `/ip firewall nat add chain=dstnat in-interface=WAN1 protocol=tcp dst-port=80 action=dst-nat to-addresses=192.168.10.80 to-ports=80`.
- Drop invalid `connection-track`: `/ip firewall filter add chain=input protocol=tcp connection-state=invalid action=drop`.
- Outbound rules (responses): depends on output/established chains (skill `mikrotik-security`).
- Test connectivity after each rule (do not leave the operator "locked out").

## 6. Access services
- SSH: `/ip service set ssh disabled=no port=2222`; use keys (`/user ssh-keys import`).
- WinBox: `/ip service set winbox address=192.168.10.0/24 disabled=no`.
- API: enable only if required; restrict by IP.
- WebFig: disable in production (skill `mikrotik-security`).
- Always check `/ip service print`.

## 7. Final validation
- `.backup` backup + `.rsc` export saved in `/files`.
- Full test: WAN ping, DNS resolves, DHCP delivers IPs, internal services access, NAT to internet.
- Scheduled reboot to validate persistence (`/system reboot` in a maintenance window) or `/system scheduler`.
- Review `/log print` for post-configuration errors (skill `mikrotik-log-analysis`).

## v6 vs v7 differences to always remember
- `/routing` (v7) consolidates OSPF/BGP/OSPFv3; v6 separates `/routing ospf`, `/routing bgp`.
- WireGuard is native in v7; `fasttrack` available in most v6 upgrades and v7+.
- v7 config syntax may change parameters (e.g.: `protocol-mode`, `vlan-filtering`). Always confirm with the CLI `help` (`?`) of the installed version.
- v7 exports use `/interface bridge port`/`ethernet` like v6, but BGP routes differ.

## Expected output
Ready-to-copy command blocks, organized by function, with a safe execution order, a comment on impact (reversible or not) and the validation checklist after applying.