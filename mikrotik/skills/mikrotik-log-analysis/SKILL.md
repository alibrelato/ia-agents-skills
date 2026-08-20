---
name: mikrotik-log-analysis
description: Use to analyze MikroTik RouterOS logs to diagnose problems and propose fixes: system log, topics, remote logging (syslog/RADIUS), /log print filtering, common failures (interfaces down, DHCP, VPN, firewall drops, cpu/memory, wireless, boot errors), correlation with configuration and step-by-step resolution. Keywords: logs, log, troubleshooting, diagnosis, problem, log analysis, errors, alerts, syslog, journal, remote logging, interface down, dhcp, wireguard, ipsec, firewall, cpu, memory.
---

# MikroTik Log Analysis (Diagnosis & Resolution)

Analyze RouterOS v6/v7 logs to identify problem causes, correlate with the configuration and present an actionable solution.

## Initial rules
- Get context: `/system resource print`, `/system routerboard print`, exact time (`/system clock print`).
- Collect the relevant older logs by topic, filtering by prefix/interface/IP.
- Correlate with the current config (`/export`) — a vpn/routing failure almost always requires looking at routes and rules too.
- Distinguish error vs warning vs "expected"; do not turn noise into an incident.

## 1. How to collect and filter logs
- View all: `/log print`.
- Filter by word: `/log print where message~"keyword"` (regex `~`, `filter`).
- By topic: `/log print where topics~"dhcp"`; common topics: `system,error,warning,info,debug,dhcp,ppp,vpn,rip,ospf,bgp,wifi,firewall`.
- Remote logging: config `/system logging action add name=remote target=remote remote=<IP:PORT>`, `/system logging add topics=all action=remote` (see topic `remote`).
- Log files: `/log` uses memory; consider `/system logging` for persistence or export via script (skill `mikrotik-scripts`).

## 2. Main symptoms and causes (quick troubleshooting)
- **Link/interface down**: `link-down`, `link-up`, `watchdog` → check cable/fiber, PoE, SFP, negotiation (`/interface ethernet print`), speed error.
- **DHCP not delivering**: `dhcp` logs without offer → check pool, server, interface, ranges, `arp`, subnet.
- **VPN (WireGuard/IPsec)**: errors `ikep`,`vpn`, `ipsec`, `wireguard` → check routes/peers/input firewall, mtu, timeouts.
- **CPU/memory drops or spikes**: `error,` resource, `out of memory` logs → correlate with processes (skill `mikrotik-improvements`), e.g.: much fasttrack off + flood.
- **Firewall blocking**: `drop`, `rate-limit`, `invalid` → review input/output/forward rules and dynamic address-lists.
- **Boot/upgrade**: messages `failed`, `no configuration`, `reboot` → version validation, storage space, hotfix order.
- **Wireless/Wireless-QoS**: interference, `noise`, `signal` → channel, tx-power, deauth.

## 3. Correlation with the configuration
- When finding `message~"eth3"` open `/interface ethernet print detail` and the interface config.
- Firewall drop messages → locate the corresponding rule (by `comment`) and evaluate the need for an exception.
- VPN errors → check `/routing`, `/interface wireguard peers`, `/ip ipsec` and routes.
- Routing errors → `/ip route print detail`, `check-gateway`, `routing-table`.

## 4. Resolution checklist (methodology)
1. Isolate the time range and topic.
2. Replicate/reproduce and capture logs.
3. Correlate with the config.
4. Propose a minimal fix (one change at a time) and reversible.
5. Validate on test/CHR when critical; monitor after the change.
6. Document the incident and root cause.

## Support features
- `/tool ping`, `/tool traceroute`, `/tool bandwidth` for measurement during resolution.
- `/torch`, `/tool sniffer quick interface=eth1` for traffic and diagnostics.
- `/system history` (recent commands) to see who changed what.

## Expected output
Structured diagnosis: symptom → relevant log excerpts (with date/time/topic) → probable cause with evidence → solution with commands/adjustments → validation and care. Include a persistent remote logging suggestion if the problem recurs.