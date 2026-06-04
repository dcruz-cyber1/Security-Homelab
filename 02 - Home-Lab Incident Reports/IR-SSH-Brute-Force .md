**INCIDENT REPORT**

SSH Brute Force Attack — Detection & Response

IR-2026-001 | April 2, 2026 | Homelab SOC Environment

| **Field** | **Detail** |
| --- | --- |
| Incident ID | IR-2026-001 |
| Date | April 2, 2026 |
| Time of Attack | 22:14 – 22:33 EDT |
| Detection Time | 22:30 EDT (within seconds of attack intensification) |
| Analyst | Daniel |
| Classification | Brute Force / Credential Attack |
| Severity | HIGH |
| Status | Detected — Active Response Partial |
| Attacker IP | 192.168.1.209 (test1 — Ubuntu Laptop) |
| Target IP | 192.168.1.200 (homelab server) |
| Target Service | SSH (port 22) |
| Attack Tool | Hydra v9.5 |
| Targeted User | daniel |

# **1. Executive Summary**

On April 2, 2026, a simulated SSH brute force attack was conducted against the homelab server (192.168.1.200) from an authorized machine (192.168.1.209). The attack was using Hydra v9.5 targeting the 'daniel' user account over SSH port 22. The attack was detected within seconds by the Graylog SIEM platform through auth log ingestion, and was captured by Suricata IDS. The Grafana dashboard displayed real-time spikes across multiple panels. Wazuh active response did not trigger an automated IP block during this test, which has been identified as a gap requiring investigation.

# **2. Environment**

| **Component** | **Detail** |
| --- | --- |
| Server | 192.168.1.200 — Ubuntu 24.04 — Ryzen 7 2700X — 16GB RAM |
| Attack Machine | 192.168.1.209 (test1) — Ubuntu 24.04 Laptop — Wazuh Agent installed |
| SIEM | Graylog (native) + OpenSearch backend (wazuh-cluster, port 9200) |
| IDS | Suricata v7.0.3 — monitoring enp5s0 — 49,314 rules active |
| EDR/IPS | Wazuh Manager (native) — Active Response enabled |
| Firewall | UFW — default deny — LAN + Tailscale allowed |
| Visualization | Grafana (Docker) — connected to OpenSearch |
| Log Pipeline | rsyslog → Graylog via UDP 5515 — Docker → Graylog via GELF 12201 |

# **3. Attack Timeline**

| **Time (EDT)** | **Event** | **Detection Source** |
| --- | --- | --- |
| 22:14:59 | Hydra initiated — 10 password attempts against SSH | auth.log |
| 22:15:06 | First Hydra run completed — no valid credentials found | Hydra output |
| 22:30:00 | Hydra re-launched with expanded wordlist (1,000+ passwords) in loop | auth.log |
| 22:30:35 | SSH Auth Events spike detected in Grafana dashboard | Graylog / Grafana |
| 22:31:00 | Suricata IDS spike — attack traffic pattern detected on enp5s0 | Suricata / Grafana |
| 22:32:30 | Multiple 'Failed password for daniel from 192.168.1.209' logged | auth.log / Graylog |
| 22:32:33 | 'PAM 5 more authentication failures' — max attempts exceeded | auth.log / Graylog |
| 22:32:33 | 'Too many authentication failures [preauth]' — SSH disconnecting attacker | auth.log / Graylog |
| 22:33:00 | Attack confirmed detected — Hydra stopped — evidence captured | Analyst action |
| 22:48:28 | Post-incident evidence collection — auth.log queried | Analyst action |

# **4. Attack Detail**

## **4.1 Attack Method**

The attack used Hydra v9.5, an open-source network login cracker, configured to perform a dictionary-based brute force attack against SSH. Hydra was run with 4 parallel threads (-t 4) and a custom wordlist containing common passwords and variations.

Attack command executed on test1:

hydra -l daniel -P ~/bigpasswords.txt 192.168.1.200 ssh -t 4

## **4.2 Auth Log Evidence**

The following entries from /var/log/auth.log confirm the attack:

| **Timestamp** | **Log Entry** |
| --- | --- |
| 22:32:30 | Failed password for daniel from 192.168.1.209 port 35674 ssh2 |
| 22:32:30 | Failed password for daniel from 192.168.1.209 port 35686 ssh2 |
| 22:32:30 | Failed password for daniel from 192.168.1.209 port 35688 ssh2 |
| 22:32:33 | error: maximum authentication attempts exceeded for daniel from 192.168.1.209 |
| 22:32:33 | Disconnecting authenticating user daniel 192.168.1.209: Too many authentication failures [preauth] |
| 22:32:33 | PAM 5 more authentication failures; logname= uid=0 euid=0 tty=ssh rhost=192.168.1.209 user=daniel |

# **5. Detection**

## **5.1 Graylog SIEM**

* Auth logs forwarded to Graylog via rsyslog (facility: security/authorization) on UDP port 5515
* Graylog search query 'failures' returned 48 results within the attack window
* Source IP 192.168.1.209 clearly identified across all log entries
* Message Count graph showed three distinct spikes at 22:31, 22:32, and 22:33

## **5.2 Grafana SOC Dashboard**

* SSH Auth Events panel spiked to 20 events per 30-second interval during peak attack
* Suricata IDS Alerts panel showed concurrent spike to 800+ events per interval
* UFW Blocks panel showed background noise — no direct block correlation to this attack
* Dashboard set to Last 15 minutes with 10-second auto-refresh — real-time visibility confirmed

## **5.3 Suricata IDS**

* Suricata rule engine (49,314 rules active on enp5s0) detected attack traffic patterns
* Wazuh rule 100005 (Suricata alert detected) fired 21,180+ times on the day of the attack
* Network-level detection corroborated host-level auth log findings

# **6. Response**

## **6.1 Automated Response**

Wazuh active response (firewall-drop + host-deny) is configured to trigger after 5+ SSH authentication failures within 60 seconds (rule 100001). During this test, the active response did NOT trigger. No entries were found in /var/ossec/logs/active-responses.log.

Investigation revealed that Wazuh alerts are currently being written to the local alerts.json file but are NOT being indexed into the OpenSearch wazuh-alerts-4.x index in real time. This is because the wazuh-indexer connection is configured to use 0.0.0.0:9200 but the indexer only listens on 127.0.0.1:9200. This is a known gap identified during this exercise.

## **6.2 Manual Response**

* Attack confirmed and stopped by analyst after evidence capture
* Auth logs reviewed and preserved
* Grafana and Graylog screenshots captured as evidence
* Incident documented in this report

# **7. Detection Gaps Identified**

| **Gap** | **Description** | **Priority** | **Next Step** |
| --- | --- | --- | --- |
| Wazuh Active Response | Rule 100001 did not fire and block 192.168.1.209 during brute force | HIGH | Fix wazuh-indexer connection — verify alerts flow to OpenSearch |
| Wazuh Grafana Panel | Wazuh alerts panel showed no data during attack due to indexing gap | HIGH | Resolve wazuh-indexer indexing pipeline |
| Wordlist Size | Only custom wordlist used — rockyou.txt not installed on test machine | LOW | Install wordlists package: sudo apt install wordlists |
| VLAN Isolation | Attack ran on same flat LAN as production server | MEDIUM | Complete VLAN segmentation with TP-Link TL-SG108E switch |

# **8. Lessons Learned**

* The Graylog + Grafana pipeline successfully detected the SSH brute force attack in real time — the core detection chain works
* Auth log forwarding via rsyslog required fixing the facility filter (auth.\* and authpriv.\* were not being forwarded) — this was resolved during setup
* OpenSearch password management across multiple instances (Graylog OpenSearch vs Wazuh indexer) adds complexity — both now use NewPassword123!
* Suricata provided a second independent detection layer at the network level, corroborating the host-based findings
* Active response configuration needs end-to-end validation — rule firing alone is insufficient if the indexer pipeline is broken
* The Grafana dashboard proved its value as a single pane of glass — SSH spike, Suricata spike, and UFW activity all visible simultaneously

# **9. Next Steps**

| **Priority** | **Task** | **Status** |
| --- | --- | --- |
| 1 | Fix Wazuh indexer pipeline — ensure alerts reach OpenSearch in real time | Pending |
| 2 | Re-run brute force test — confirm rule 100001 fires and active response blocks IP | Pending |
| 3 | Install rockyou wordlist on test machine for more realistic attack simulation | Pending |
| 4 | Complete VLAN segmentation — isolate attack machine to VLAN 3 | Pending |
| 5 | Add Prometheus + Node Exporter for system metrics in Grafana | Pending |
| 6 | Write full incident response runbook based on this exercise | Pending |
| 7 | Apply to NJCCIC internship with this incident report as portfolio evidence | Pending |

# **10. Conclusion**

This demonstrated a functioning homelab capable of detecting a real SSH brute force attack in real time. The detection chain from attack to log to SIEM to dashboard visualization performed as designed. Gaps in the automated response pipeline have been identified and documented for remediation. The environment mirrors enterprise security architecture including SIEM (Graylog + Wazuh), IDS (Suricata), firewall (UFW), DNS security (Pi-hole), and zero-trust remote access (Tailscale).

This incident report serves as evidence of hands-on security operations experience including attack simulation, log analysis, SIEM configuration, dashboard creation, and incident documentation.
