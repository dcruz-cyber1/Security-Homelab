# 🔐 Security Stack

---

## Security Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│                   PERIMETER                         │
│  UFW Firewall — default deny, explicit allow only   │
│  Tailscale — zero-trust mesh, no open public ports  │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                 NETWORK LAYER                        │
│  Pi-hole — DNS-layer threat blocking                │
│  br0 bridge — Docker network segmentation           │
│  tailscale0 — VPN-only service exposure             │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│                   HOST LAYER                         │
│  Wazuh Agent — EDR, FIM, rootkit, vuln detection    │
│  Graylog — log ingestion, alerting, dashboards       │
│  Copilot — hardware telemetry                       │
└─────────────────────────────────────────────────────┘
```

---

## Firewall — UFW Rules

See [[05 - Firewall Rules]] for the full annotated breakdown.

**Summary of current policy:**
- Default: **DENY** all inbound
- LAN (`192.168.0.0/16`): **ALLOW** inbound
- Tailscale interface (`tailscale0`): **ALLOW** inbound
- SSH (`22/tcp`) on Tailscale: **RATE LIMITED** (brute-force protection)
- DNS (`53`) on Docker bridge (`br0`): **ALLOW** inbound (Pi-hole)

---

## Wazuh — SIEM / EDR / IPS

### What It Does
| Function | Description |
|----------|-------------|
| **Log Analysis** | Parses system, auth, application logs for threat indicators |
| **File Integrity Monitoring (FIM)** | Detects unauthorized changes to critical files |
| **Rootkit Detection** | Scans for hidden processes, files, and kernel modules |
| **Vulnerability Detection** | Maps installed packages to CVE database |
| **Active Response** | Can auto-block IPs (IPS behavior) on detected attacks |
| **IDS** | Detects intrusions based on log patterns and system events |

### Key Rules to Monitor
- `5710` — SSH brute-force attempts
- `5501` — Host-based anomaly detection
- `80790` — Shellshock attempt
- `31151` — Sudo privilege escalation
- Custom rules: `/var/ossec/etc/rules/local_rules.xml`

### Active Response Config
Wazuh can automatically block IPs via `firewall-drop` active response.  
Config: `/var/ossec/etc/ossec.conf`

---

## Graylog — Log Management

### Input Sources
| Source | Protocol | Port |
|--------|---------|------|
| System syslog | Syslog UDP/TCP | `5514` |
| Docker containers | GELF | `12201` |
| Wazuh alerts | Syslog / API | configurable |

### Key Dashboards to Build
- SSH login attempts & failures
- UFW BLOCK events by source IP
- DNS query volume by client
- Wazuh alert severity over time
- Docker container restart events

### Alert Examples
- `>10 failed SSH logins from same IP in 5 min` → alert + notify
- `UFW BLOCK from external IP` → alert
- `New sudo user added` → alert

---

## Threat Model — What This Lab Defends Against

| Threat | Mitigation |
|--------|-----------|
| SSH brute force | UFW rate limit + Wazuh active response |
| Malicious DNS resolution | Pi-hole blocklists |
| Unauthorized remote access | Tailscale zero-trust (device auth required) |
| Lateral movement | Network segmentation via br0/VLAN (in progress) |
| Malware / rootkits | Wazuh FIM + rootkit detection |
| Exposed services | All services behind Tailscale — no public ports |
| Log tampering | Centralized Graylog (logs shipped off-host) |

---

## What Real Companies Implement (That This Lab Mirrors)

| Enterprise Concept | Lab Implementation |
|--------------------|--------------------|
| SIEM | Wazuh + Graylog |
| EDR | Wazuh Agent |
| Zero Trust Network Access (ZTNA) | Tailscale |
| DNS Security / Filtering | Pi-hole |
| Network Segmentation | br0 bridge, Tailscale ACLs |
| Log Aggregation | Graylog centralized logging |
| IDS/IPS | Wazuh Active Response |
| Firewall policy | UFW with default-deny |
| Vulnerability Management | Wazuh vuln detection |

---

*← Back to [[00 - Home Lab Overview]]*
