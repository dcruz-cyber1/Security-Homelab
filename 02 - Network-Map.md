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

## Network Diagram (Text)

```
Internet
    │
    ▼
 [Router/Modem]
    │  192.168.1.0/24
    ├──── [Homelab Server] 192.168.1.200
    │         ├── br0 (Docker bridge)
    │         └── tailscale0 (VPN mesh)
    │
    ├──── [Pi-hole]  ← DNS for all LAN + Tailscale devices
    │
    ├──── [Security Laptop] 192.168.1.209 (analyst)
    │
    └──── [Attacker Desktop]  (red team)

Tailscale Mesh (10.x.x.x / 100.x.x.x overlay)
    ├── Server
    ├── Phone
    ├── TVs
    └── Any approved remote device
```

---

## Tailscale VPN Mesh

- **Purpose:** Zero-trust remote access — only approved/authenticated devices join the tailnet
- **What's connected:** Server, phone, TVs, any approved remote device
- **Benefit:** No exposed ports to public internet; SSH and services accessible only through Tailscale
- **Interface:** `tailscale0` on server


## Pi-hole DNS

- **Scope:** Handles DNS for all LAN devices + Tailscale-connected devices
- **Function:** Blocks ad domains, tracking domains, malicious domains at DNS layer
- **Upstream DNS:** (e.g., `1.1.1.1`, `8.8.8.8` — update with your config)
- **DNS-over-HTTPS:** (enabled/disabled — update)

Port `53` is allowed inbound only on `br0` (Docker bridge) — see [[05 - Firewall Rules]].

---

## Subnets in Use

| Subnet | Use |
|--------|-----|
| `192.168.0.0/16` | Full LAN range — allowed inbound by UFW |
| `100.x.x.x` | Tailscale overlay network |
| Docker bridge | Internal container networking |

---

*← Back to [[00 - Home Lab Overview]]*
