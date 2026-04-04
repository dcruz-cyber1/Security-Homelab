# 🏠 Home Lab Overview

> **Purpose:** A hands on security and networking lab that replicates real world enterprise infrastructure — covering security hardening, network segmentation, log analysis, DNS, IDS/IPS, SIEM, containerization, and scripting.

---

## 🎯 Learning Goals

- Security hardening (OS, network, application layer)
- Log analysis, alerting & dashboards
- DNS security and ad-blocking
- Intrusion Detection & Prevention (IDS/IPS)
- SIEM deployment and tuning
- Endpoint Detection & Response (EDR)
- Docker containerization
- Remote access & Zero Trust networking
- Media server stack management

---

## 🖥️ Lab Machines

| Role | Hostname | IP | Description |
|------|----------|----|-------------|
| Server | `homelab-server` | `192.168.1.200` | Main services host — see [[01 - Hardware]] |
| Security Analyst | `laptop` | DHCP | Monitoring, log review, tooling |
| Attacker / Red Team | `desktop-attacker` | DHCP | Penetration testing, attack simulation |

---

## 🔗 Quick Links

- [[01 - Hardware]] — Server specs & components
- [[02 - Network Map]] — IP layout, devices, Tailscale mesh
- [[03 - Services]] — All running services & stacks
- [[04 - Security Stack]] — Firewall rules, Wazuh, IDS/IPS
- [[05 - Firewall Rules]] — UFW rules breakdown
- [[06 - Runbooks]] — Setup guides & procedures

---

## 📅 Lab Status

| Area | Status |
|------|--------|
| Media Stack (Arr + Jellyfin) | ✅ Running |
| Pi-hole DNS | ✅ Running |
| Tailscale VPN mesh | ✅ Running |
| Wazuh SIEM/EDR | ✅ Running |
| Graylog | ✅ Running |
| Copilot System Monitor | ✅ Running |
| IDS/IPS Tuning | ✅ Running |
| Network Segmentation | 🔧 In Progress |
