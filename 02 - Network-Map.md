# 🌐 Network Map

---

## IP Address Layout

| Device | IP | Interface | Notes |
|--------|----|-----------|-------|
| **Homelab Server** | `192.168.1.200` | `br0` / `eth0` | Static — main services host |
| **Pi-hole** | `192.168.1.x` (update) | LAN | DNS resolver for all LAN devices |
| **Security Laptop** | 192.168.1.209 | LAN |Victim machine — Wazuh agent — VLAN 2 |
| **Attacker Desktop** | 192.168.1.50 | LAN | Kali Linux — dedicated attack machine — VLAN 3 isolated |
| **Phone** | Tailscale IP | `tailscale0` | Remote access via VPN |
| **Smart TVs** | Tailscale or LAN | LAN / `tailscale0` | Routed through Pi-hole DNS |

---
```
Internet
    │
    ▼
 [Router/Modem] 192.168.1.1
    │
    ▼
 [TP-Link TL-SG108E] 192.168.1.254
    │  192.168.1.0/24
    ├── Port 1 ── Router
    ├── Port 2 ── [Victim Laptop] 192.168.1.209 (VLAN 1+2)
    ├── Port 3 ── [Homelab Server] 192.168.1.200 (VLAN 1)
    │                 ├── br0 (Docker bridge)
    │                 └── tailscale0 (VPN mesh)
    └── Port 4 ── [Kali Attack Machine] 192.168.1.50 (VLAN 3 — isolated)



```
## 🔀 Switch & VLAN Configuration
---

**Switch:** TP-Link TL-SG108E — Management IP: 192.168.1.254

| Port | Device | VLAN | Notes |
|------|--------|------|-------|
| Port 1 | Router | 1 | Uplink |
| Port 2 | Laptop (victim) | 1+2 | Has internet + reachable by Kali |
| Port 3 | Server | 1 | Core services |
| Port 4 | Kali (attacker) | 3 | Isolated — no internet, laptop only |
| Ports 5-8 | Spare | 1 | General use |

---

**Isolation rules:**
- Kali can reach laptop ✅
- Kali cannot reach server ✅
- Kali has no internet ✅
- Server and laptop can communicate ✅


---
```

## Tailscale VPN Mesh

- **Purpose:** Zero-trust remote access — only approved/authenticated devices join the tailnet
- **What's connected:** Server, phone, TVs, any approved remote device
- **Benefit:** No exposed ports to public internet; SSH and services accessible only through Tailscale
- **Interface:** `tailscale0` on server

```

## Pi-hole DNS

- **Scope:** Handles DNS for all LAN devices + Tailscale-connected devices
- **Function:** Blocks ad domains, tracking domains, malicious domains at DNS layer
- **Upstream DNS:** (e.g., `1.1.1.1`, `8.8.8.8` — update with your config)
- **DNS-over-HTTPS:** (enabled/disabled — update)

Port `53` is allowed inbound only on `br0` (Docker bridge) — see [[05 - Firewall Rules]].

```
```
## Subnets in Use
---



| Subnet | Use |
|--------|-----|
| `192.168.0.0/16` | Full LAN range — allowed inbound by UFW |
| `100.x.x.x` | Tailscale overlay network |
| Docker bridge | Internal container networking |

---
```
