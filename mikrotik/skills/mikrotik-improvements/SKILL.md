---
name: mikrotik-improvements
description: Use to audit an existing MikroTik/RouterOS configuration and propose improvement insights: performance tuning, fasttrack, WAN redundancy/failover, aggregation/bonding, MTU/queue optimization, feature utilization, cleanup and best practices per RouterOS version. Judge the current config and deliver a prioritized action plan. Keywords: improvements, insights, audit, tuning, performance, fasttrack, failover, redundancy, bond, mtu, queue, optimization, best practices.
---

# MikroTik Configuration Improvements & Insights

Audit the existing configuration and propose prioritized improvements by impact, risk and effort. Use this skill ANYTIME there is a `/export` or a real configuration to analyze.

## Rules before starting
- Gather the real state: `/export compact`, `/system resource print`, `/system routerboard print`, `/system package print`, `/ip firewall print`, `/interface print`, `/routing` (v7).
- Do not propose changes without confirming version (v6 vs v7) and model (switch chip/CPU resources).
- Deliver a prioritized plan: Impact (High/Medium/Low) × Risk × Effort.

## 1. Basic performance analysis
- `fasttrack`: enable for established connections where applicable (careful with heavy NAT/connection tracking):
  ```
  /ip firewall raw add action=notrack chain=prerouting connection-state=established,related
  /ip firewall filter add chain=forward connection-state=established,related action=fasttrack-connection
  ```
  (configure before the established accept rule).
- Remove obsolete, duplicated and non-optimized rules (unnecessary mangle).
- Mangle: avoid rules on the hot path when there is an alternative (static address-lists, swap chains).
- Check CPU saturation (`/system resource cpu print`, `/tool profile`) and identify the responsible process (e.g.: missing `fasttrack` on a high-throughput link).

## 2. WAN redundancy (dual link)
- Failover with check-gateway:
  ```
  /ip route add dst-address=0.0.0.0/0 gateway=WAN-GW1 distance=1 check-gateway=ping
  /ip route add dst-address=0.0.0.0/0 gateway=WAN-GW2 distance=2 check-gateway=ping
  ```
- Policy routing (PBR): route critical traffic (VoIP, backup) through the right WAN via address-lists + route tables.
- NAT per WAN: keep two masquerade rules with `out-interface` and correct order.
- Link monitoring with `/tool netwatch` + failover script (skill `mikrotik-scripts`) and dynamic update of VPN IP/endpoints.

## 3. Aggregation and VLANs
- Bond: `mode=802.3ad` to aggregate throughput when the switch supports LACP; confirm model support.
- VLAN trunking: use `vlan-filtering` on the bridge, not n VLANs per port.
- MTU: configure consistent MTU/MRU across the whole path (WAN, PPPoE, Wireless, L2TP) to avoid fragmentation.
- Enable `arp=proxy-arp` only when necessary; avoid duplicated routes.

## 4. Routing improvements
- OSPF/BGP: summarization, area structure, filters (prefix-lists), `bfd` for fast convergence, `ecmp` when allowed.
- v7: use `routing/rule` and tables for PBR instead of multipath mangle.
- Remove dead blackhole/static routes; check `pref-src` and `scope/target-scope`.

## 5. Organization and operational best practices
- Standard naming: interfaces, bridges, VLANs, address-lists (e.g.: `VLAN10_ADM`, `WAN1_GW`).
- Comments: use `comment=` on firewall/NAT/route rules for easy maintenance.
- Scheduled backup: `/system scheduler` + automatic export script to a server (skill `mikrotik-scripts`).
- Firmware/package updates (skill `mikrotik-documentation`) with a maintenance window.
- Monitor resources: `/system resource print` and CPU/memory alerts (script + e-mail).

## 6. Platform feature utilization
- Use the switch chip (CRS) for ports, not CPU crossing.
- Hardware/native routing and `hw-offload` when the model supports it (`/interface bridge port set ... hw=yes`).
- Avoid switch filters (use chip ACL) when supported.
- Check power supply and redundant source (PoE) on critical equipment.

## 7. Prioritization matrix
Present findings as a table:
| Adjustment | Impact | Risk | Effort | Compatible Version | Notes |
|------------|--------|------|--------|--------------------|-------|
| Enable fasttrack | High | Medium | Low | v7+ | watch rule order |
| Failover route | High | Low | Low | v6/v7 | test in lab |

## Expected output
Audit report: numbered findings with evidence (current config), recommendation with concrete commands, impact/risk/effort, and a suggested implementation order. Include a "before/after" checklist and post-change validation (skill `mikrotik-log-analysis`).