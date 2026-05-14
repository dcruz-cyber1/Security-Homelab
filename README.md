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
<img width="1919" height="991" alt="Full Attack Session Overview" src="https://github.com/user-attachments/assets/8383abcb-1c78-4412-b4ee-0fe08e23f5a9" /><img width="1919" height="989" alt="SSH Brute Force Attack Peak (Hydra with rockyou txt) — May 10, 2026" src="https://github.com/user-attachments/assets/a7fad408-ae7c-420a-93e5-f9863ef89f57" />
<img width="1919" height="991" alt="Full Attack Session Overview" src="https://github.com/user-attachments/assets/dd8943b5-c6a3-49b7-a231-7c56d8d1d210" />
<img width="1918" height="989" alt="DoSFlood Attack FTP Brute Force" src="https://github.com/user-attachments/assets/505e9174-d0e5-47a6-9551-8f43785ed14f" />
<img width="1919" height="958" alt="Nmap Port Scan + Beginning of SSH Brute Force — May 10, 2026" src="https://github.com/user-attachments/assets/4063b72d-b5a1-4dde-96fb-bb38d9dce2f2" />
<img width="1919" height="991" alt="Metasploit SSH Login Scanner + Continued Brute Force" src="https://github.com/user-attachments/assets/8e14a672-c11d-4490-910f-f770dea45025" />

