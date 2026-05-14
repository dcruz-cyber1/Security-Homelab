# IR-2026-001 — SSH Brute Force Attack

**Date:** April 2, 2026 | **Severity:** HIGH | **Status:** Detected — Active Response Partial

| Field | Detail |
|-------|--------|
| Incident ID | IR-2026-001 |
| Date | April 2, 2026 |
| Time | 22:14 – 22:33 EDT |
| Attacker IP | 192.168.1.209 (test1 — Ubuntu Laptop) |
| Target IP | 192.168.1.200 (homelab server) |
| Target Service | SSH port 22 |
| Attack Tool | Hydra v9.5 |
| Targeted User | daniel |

## Tools Used

### Attack (Red Team)
| Tool | Purpose |
|------|---------|
| Hydra v9.5 | SSH dictionary brute force |
| Custom wordlist | 1,000+ common password variations |

### Defense (Blue Team)
| Tool | Purpose |
|------|---------|
| Graylog | Auth log ingestion and alerting |
| Suricata v7.0.3 | Network IDS — attack traffic detection |
| Grafana | Real-time SOC dashboard visualization |
| Wazuh v4.14.5 | EDR — host-based detection |

## Attack Command
```bash
hydra -l daniel -P ~/bigpasswords.txt 192.168.1.200 ssh -t 4
```

## Timeline

| Time (EDT) | Event | Source |
|------------|-------|--------|
| 22:14:59 | Hydra initiated — 10 attempts | auth.log |
| 22:30:00 | Hydra re-launched with expanded wordlist | auth.log |
| 22:30:35 | SSH Auth Events spike in Grafana | Graylog / Grafana |
| 22:31:00 | Suricata IDS spike detected | Suricata / Grafana |
| 22:32:30 | Multiple Failed password entries logged | auth.log / Graylog |
| 22:32:33 | PAM 5 more authentication failures | auth.log / Graylog |
| 22:32:33 | Too many authentication failures [preauth] | auth.log / Graylog |
| 22:33:00 | Attack stopped — evidence captured | Analyst |

## Detection

- Graylog captured auth logs via rsyslog (facility: security/authorization, UDP 5515)
- SSH Auth Events panel spiked to 20 events per 30s interval in Grafana
- Suricata spiked to 800+ events per interval concurrently
- Source IP 192.168.1.209 identified across all log entries

## Auth Log Evidence

| Timestamp | Log Entry |
|-----------|-----------|
| 22:32:30 | Failed password for daniel from 192.168.1.209 port 35674 ssh2 |
| 22:32:33 | error: maximum authentication attempts exceeded for daniel from 192.168.1.209 |
| 22:32:33 | Disconnecting authenticating user daniel 192.168.1.209: Too many authentication failures [preauth] |
| 22:32:33 | PAM 5 more authentication failures; rhost=192.168.1.209 user=daniel |

## Gaps Identified

| Gap | Priority | Status |
|-----|----------|--------|
| Wazuh active response did not trigger | HIGH | Pending fix |
| Wazuh Grafana panel showed no data | HIGH | Pending fix |
| Attack ran on flat LAN (no VLAN isolation) | MEDIUM | ✅ Resolved — VLAN deployed May 10 |
| rockyou.txt not installed on test machine | LOW | ✅ Resolved |

## Lessons Learned

- Graylog + Grafana pipeline detected the attack in real time — core detection chain works
- Auth log rsyslog forwarding required facility filter fix (auth.* and authpriv.*) — resolved during setup
- Suricata provided independent network-layer corroboration of host-based findings
- Wazuh active response needs end-to-end pipeline validation

---

# IR-2026-002 — Nmap Network Reconnaissance Scan

**Date:** May 10, 2026 | **Severity:** MEDIUM | **Status:** Detected

| Field | Detail |
|-------|--------|
| Incident ID | IR-2026-002 |
| Date | May 10, 2026 |
| Time | 22:13 – 22:20 EDT |
| Attacker IP | 192.168.1.50 (Kali — VLAN 3) |
| Target IP | 192.168.1.209 (victim laptop — VLAN 2) |
| Attack Type | Network Reconnaissance / Port Scanning |
| Attack Tool | Nmap v7.98 |

## Tools Used

### Attack (Red Team)
| Tool | Purpose |
|------|---------|
| Nmap v7.98 | Port scanning, service detection, OS fingerprinting |

### Defense (Blue Team)
| Tool | Purpose |
|------|---------|
| Suricata v7.0.3 | Network IDS — scan pattern detection |
| Graylog | Log ingestion and search |
| Grafana | Real-time dashboard visualization |

## Attack Commands
```bash
nmap -sS -sV -O 192.168.1.209
nmap -A 192.168.1.209
nmap --script vuln 192.168.1.209
```

## Timeline

| Time (EDT) | Event | Source |
|------------|-------|--------|
| 22:13:00 | Nmap SYN scan initiated from 192.168.1.50 | Suricata |
| 22:14:00 | Suricata IDS spike — scan signatures triggered | Suricata / Grafana |
| 22:15:00 | Service version detection (-sV) scan started | Suricata |
| 22:18:00 | Suricata spike to 1,250+ events — peak scan activity | Grafana |
| 22:20:00 | Scan completed | Analyst |

## Detection

- Suricata spiked to 1,250+ events at 22:18 — highest spike of the session
- Multiple Suricata signatures triggered including port scan detection rules
- UFW Blocks showed background activity during scan window
- Grafana dashboard showed clear spike distinguishable from baseline

## Gaps Identified

| Gap | Priority | Status |
|-----|----------|--------|
| No automated alert configured for port scan detection in Graylog | MEDIUM | Pending |
| Wazuh did not generate agent alert for inbound scan on laptop | LOW | Pending |

## Lessons Learned

- Suricata is highly effective at detecting nmap scans — signatures fired immediately
- SYN scans (-sS) are detectable even without completing the TCP handshake
- VLAN isolation confirmed working — Kali (192.168.1.50) reached laptop but not server

---

# IR-2026-003 — Metasploit SSH Login Scanner

**Date:** May 10, 2026 | **Severity:** HIGH | **Status:** Detected

| Field | Detail |
|-------|--------|
| Incident ID | IR-2026-003 |
| Date | May 10, 2026 |
| Time | 22:28 – 22:38 EDT |
| Attacker IP | 192.168.1.50 (Kali — VLAN 3) |
| Target IP | 192.168.1.209 (victim laptop — VLAN 2) |
| Attack Type | Credential Brute Force / Exploitation Framework |
| Attack Tool | Metasploit v6.4 |

## Tools Used

### Attack (Red Team)
| Tool | Purpose |
|------|---------|
| Metasploit v6.4 | Exploitation framework — SSH login scanner module |
| rockyou.txt | Password wordlist — 14 million passwords |

### Defense (Blue Team)
| Tool | Purpose |
|------|---------|
| Graylog | Auth log ingestion |
| Suricata v7.0.3 | Network IDS |
| Grafana | Real-time dashboard |
| Wazuh v4.14.5 | EDR host monitoring |

## Attack Commands
```bash
msfconsole
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 192.168.1.209
set USERNAME daniel1
set PASS_FILE /usr/share/wordlists/rockyou.txt
set THREADS 4
run
```

## Timeline

| Time (EDT) | Event | Source |
|------------|-------|--------|
| 22:28:00 | Metasploit SSH login scanner initiated | auth.log |
| 22:28:00 | SSH Auth Events spike to 40 events per interval | Grafana |
| 22:30:00 | Suricata concurrent spike — attack traffic detected | Suricata / Grafana |
| 22:33:00 | UFW Blocks spike to 18 at 22:28 window | Grafana |
| 22:38:00 | Scan stopped — evidence captured | Analyst |

## Detection

- SSH Auth Events panel spiked to 40 per 30s interval — highest SSH spike of session
- Suricata simultaneously spiked indicating network-level detection
- Auth log on laptop showed repeated Failed password entries from 192.168.1.50
- Multiple PAM authentication failure messages logged

## Gaps Identified

| Gap | Priority | Status |
|-----|----------|--------|
| Wazuh active response did not block 192.168.1.50 | HIGH | Pending — indexer pipeline issue |
| No Graylog alert configured for Metasploit scanner signatures | MEDIUM | Pending |

## Lessons Learned

- Metasploit's ssh_login module generates the same auth.log patterns as Hydra — both are detectable with the same rules
- Multi-tool attack correlation confirmed — Grafana showed SSH and Suricata spikes simultaneously

---

# IR-2026-004 — FTP Brute Force Attack

**Date:** May 10, 2026 | **Severity:** MEDIUM | **Status:** Detected

| Field | Detail |
|-------|--------|
| Incident ID | IR-2026-004 |
| Date | May 10, 2026 |
| Time | 22:35 – 22:42 EDT |
| Attacker IP | 192.168.1.50 (Kali — VLAN 3) |
| Target IP | 192.168.1.209 (victim laptop — VLAN 2) |
| Target Service | FTP port 21 |
| Attack Type | Credential Brute Force |
| Attack Tool | Hydra v9.6 |

## Tools Used

### Attack (Red Team)
| Tool | Purpose |
|------|---------|
| Hydra v9.6 | FTP dictionary brute force |
| rockyou.txt | Password wordlist |
| vsftpd | FTP server installed on victim for testing |

### Defense (Blue Team)
| Tool | Purpose |
|------|---------|
| Suricata v7.0.3 | Network IDS — FTP brute force detection |
| Graylog | Log ingestion |
| Grafana | Real-time visualization |

## Attack Commands
```bash
# On victim laptop
sudo apt install vsftpd -y
sudo systemctl start vsftpd

# On Kali
hydra -l daniel1 -P /usr/share/wordlists/rockyou.txt ftp://192.168.1.209 -t 4 -V
```

## Timeline

| Time (EDT) | Event | Source |
|------------|-------|--------|
| 22:35:00 | FTP brute force initiated from 192.168.1.50 | Suricata |
| 22:35:00 | Suricata FTP anomaly signatures triggered | Suricata / Grafana |
| 22:38:00 | SSH Auth Events showing residual activity | Grafana |
| 22:40:00 | UFW Blocks spike to 25+ | Grafana |
| 22:42:00 | Attack stopped | Analyst |

## Detection

- Suricata detected FTP brute force pattern signatures
- UFW Blocks spiked during attack window
- Grafana dashboard showed concurrent multi-panel activity

## Gaps Identified

| Gap | Priority | Status |
|-----|----------|--------|
| No dedicated FTP stream in Graylog | LOW | Pending |
| vsftpd logs not forwarded to Graylog | LOW | Pending |

## Lessons Learned

- FTP is an insecure protocol — credentials sent in plaintext, easily brute forced
- Suricata detects FTP brute force without any custom rules due to protocol anomaly signatures
- vsftpd should be disabled after testing — not needed in production lab

---

# IR-2026-005 — Denial of Service (DoS) Attack

**Date:** May 10, 2026 | **Severity:** HIGH | **Status:** Detected

| Field | Detail |
|-------|--------|
| Incident ID | IR-2026-005 |
| Date | May 10, 2026 |
| Time | 22:42 – 22:50 EDT |
| Attacker IP | 192.168.1.50 (Kali — VLAN 3) |
| Target IP | 192.168.1.209 (victim laptop — VLAN 2) |
| Attack Type | Denial of Service — SYN Flood |
| Attack Tool | hping3 |

## Tools Used

### Attack (Red Team)
| Tool | Purpose |
|------|---------|
| hping3 | SYN flood — high volume TCP SYN packets to port 22 |
| nmap --flood | Packet flood scan |

### Defense (Blue Team)
| Tool | Purpose |
|------|---------|
| Suricata v7.0.3 | Network IDS — DoS and flood detection |
| UFW | Firewall — blocked flood traffic |
| Grafana | Real-time visualization |
| Graylog | UFW block log ingestion |

## Attack Commands
```bash
hping3 -S --flood 192.168.1.209 -p 22
nmap --flood 192.168.1.209
```

## Timeline

| Time (EDT) | Event | Source |
|------------|-------|--------|
| 22:42:00 | SYN flood initiated from 192.168.1.50 | Suricata / UFW |
| 22:42:00 | UFW Blocks spike to 30+ — flood packets blocked | Grafana |
| 22:43:00 | Suricata DoS signatures triggered | Suricata / Grafana |
| 22:45:00 | UFW Blocks sustained at 20-30 per interval | Grafana |
| 22:50:00 | Attack stopped — evidence captured | Analyst |

## Detection

- UFW Blocks panel spiked to 30+ events per interval — highest UFW activity of session
- Suricata detected SYN flood pattern — DoS signatures triggered
- Both panels showed sustained elevated activity for duration of flood
- Grafana full 1-hour overview clearly shows DoS window as distinct spike

## Gaps Identified

| Gap | Priority | Status |
|-----|----------|--------|
| No automated Graylog alert for DoS pattern (sustained UFW spike) | HIGH | Pending |
| Wazuh did not alert on network flood condition | MEDIUM | Pending |
| No rate limiting configured on victim machine | LOW | Pending |

## Lessons Learned

- UFW successfully blocked SYN flood packets — firewall held under flood conditions
- Suricata DoS signatures fired without any custom rule tuning
- The combination of UFW Blocks + Suricata spikes together in Grafana is a reliable DoS indicator
- VLAN isolation critical — flood traffic contained to VLAN 2/3, server (VLAN 1) unaffected
- DoS attack is the most visually obvious attack type on the Grafana dashboard
