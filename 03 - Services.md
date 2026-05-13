# ⚙️ Services Running

All services run as Docker containers on the homelab server (`192.168.1.200`).

---

## 🎬 Media Stack (Arr + Jellyfin)

| Service | Purpose | Port |
|---------|---------|------|
| **Jellyfin** | Media server — streams movies, shows, music locally and remotely | `8096` |
| **Sonarr** | TV show automation — monitors, downloads, organizes shows | `8989` |
| **Radarr** | Movie automation — monitors, downloads, organizes movies | `7878` |
| **Lidarr** | Music automation | `8686` |
| **Prowlarr** | Indexer manager for all Arr apps | `9696` |
| **qBittorrent** | Torrent client — handles downloads from Arr stack | `8080` |

**Flow:**  
`Prowlarr (indexers) → Sonarr/Radarr/Lidarr (automation) → qBittorrent (download) → Jellyfin (stream)`

**Storage:** Media stored on server local disk, mounted into containers.

---

## 🛡️ Pi-hole

| Item | Detail |
|------|--------|
| **Purpose** | Network wide ad blocking + DNS hardening |
| **Scope** | All LAN devices + Tailscale mesh devices |
| **Interface** | Web dashboard on port `80`/`443` |
| **DNS blocking** | Ad networks, tracking pixels, known malicious domains |

**Security value:** Reduces attack surface by blocking malicious domains before any connection. Provides visibility into DNS traffic across the entire network.

---

## 🌐 Tailscale

| Item | Detail |
|------|--------|
| **Purpose** | Zero-trust VPN mesh for secure remote access |
| **Devices** | Server, phone, TVs, approved remote machines |
| **SSH** | Rate-limited on `tailscale0` — only reachable via VPN |
| **Security model** | MFA-backed device auth; no open inbound ports needed |

---

## 📊 Copilot System Monitor

| Item | Detail |
|------|--------|
| **Purpose** | Real-time hardware telemetry dashboard |
| **Data** | CPU, GPU, RAM, disk, network utilization |
| **Output** | Hardware graphs for performance tracking |

Useful for correlating system load with security events (e.g., CPU spike during suspicious process).

---

## 🔐 Wazuh (SIEM / EDR / IDS / IPS)

| Item | Detail |
|------|--------|
| **Purpose** | All-in-one security monitoring framework |
| **Components** | Wazuh Manager, Wazuh Agent, OpenSearch/Kibana dashboard |
| **Functions** | EDR, IDS, IPS, log collection, file integrity monitoring (FIM), vulnerability detection |
| **Agents** | Deployed on server, and laptop as victim |
| **Dashboard port** | `443` / `5601` |

**What it monitors:**
- Authentication events (SSH logins, sudo, failed logins)
- File integrity (detects unauthorized file changes)
- Rootkit detection
- CVE / vulnerability scanning
- Network anomalies
- Custom rules and active response

See [[04 - Security Stack]] for Wazuh rule configuration.

---

## 📋 Graylog (Log Management & SIEM)

| Item | Detail |
|------|--------|
| **Purpose** | Centralized log ingestion, parsing, alerting, dashboards |
| **Input sources** | Syslog, Docker logs, Wazuh, system logs |
| **Port** | `9000` (web UI), `514` (syslog), `12201` (GELF) |

**Use cases:**
- Create dashboards for SSH login attempts, UFW drops, DNS queries
- Build alerts for brute-force patterns
- Correlate events across services
- Long-term log storage and search

---

## 📊 Grafana (SOC Dashboard)

| Item | Detail |
|------|--------|
| **Purpose** | Real-time SOC visualization dashboard |
| **Runtime** | Docker — host network mode |
| **Port** | `3000` |
| **Datasources** | OpenSearch (Graylog data + Wazuh alerts) |

**Panels:**
- SSH Auth Events — detects brute force spikes
- Suricata IDS Alerts — network attack detection
- UFW Blocks — firewall block events
- Wazuh Alerts — EDR rule triggers

---


