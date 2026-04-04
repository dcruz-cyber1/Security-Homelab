# Security-Homelab

A full security operations center built on Ubuntu 24.04 
for hands-on attack simulation and detection.

## Stack
- Wazuh — SIEM/EDR/IPS
- Graylog — log management and alerting
- Suricata — network IDS (49,314 rules)
- Grafana — real-time SOC dashboard
- Pi-hole — DNS security
- UFW — firewall
- Tailscale — zero trust remote access

## What I've Done
- Configured end-to-end log pipelines 
  (Docker, auth, UFW, Suricata → Graylog)
- Simulated SSH brute force attack using Hydra
- Detected attack in real time across 3 tools simultaneously
- Produced formal incident report IR-2026-001

## Incidents
- [IR-2026-001 — SSH Brute Force Attack](incidents/IR-2026-001-SSH-Brute-Force.md)

## Screenshots
![Grafana Dashboard](screenshots/grafana-dashboard-attack.png)
![Graylog SSH Failures](screenshots/graylog-ssh-failures.png)
