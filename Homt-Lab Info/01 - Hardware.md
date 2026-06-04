# 🖥️ Hardware — Server Specs

**Role:** Primary homelab server — hosts all services via Docker  
**IP:** `192.168.1.200`

---

## CPU & Platform

| Component | Details |
|-----------|---------|
| **CPU** | AMD Ryzen 7 2700X (8-core / 16-thread, 3.7GHz base / 4.3GHz boost) |
| **RAM** | 16 GB DDR4 @ 3200MHz |

## GPU

| Component | Details |
|-----------|---------|
| **GPU** | NVIDIA GTX 1660 |
| **Use Cases** | Hardware transcoding (Jellyfin) |

## Networking

| Component | Details |
|-----------|---------|
| **Bridge Interface** | `br0` — used for Docker + firewall rules |
| **Tailscale Interface** | `tailscale0` — VPN mesh interface |

## ⚔️ Attack Machine — Lenovo IdeaCentre K330B

| Component | Details |
|-----------|---------|
| **CPU** | Intel Core i3-2120 |
| **GPU** | NVIDIA GTX 1660 Super |
| **OS** | Kali Linux |
| **Role** | Dedicated red team / attack machine |
| **Network** | VLAN 3 — isolated, port 4 on switch |
| **IP** | 192.168.1.50 |

---

## OS & Base Config

| Item | Details |
|------|---------|
| **OS** | Ubuntu Server (LTS) |
| **Container Runtime** | Docker + Docker Compose |
| **Firewall** | UFW — see [[05 - Firewall Rules]] |
| **Remote Access** | Tailscale (SSH over VPN) |

---

## Resource Monitoring

Copilot system overview tool is running and provides:
- CPU / GPU usage graphs
- RAM and storage utilization
- Network throughput

See [[03 - Services#Copilot System Monitor]] for details.

---

*← Back to [[00 - Home Lab Overview]]*
