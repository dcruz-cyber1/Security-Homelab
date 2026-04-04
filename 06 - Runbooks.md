# 📘 Runbooks & Procedures

---

## Starting / Stopping All Services

```bash
# Start all Docker services
cd ~/docker && docker compose up -d

# Stop all services
docker compose down

# Restart a single service
docker compose restart jellyfin

# View running containers
docker ps

# View logs for a container
docker logs -f wazuh-manager
```

---

## SSH into Server (via Tailscale)

```bash
# From any Tailscale-connected device
ssh user@<tailscale-ip-of-server>

# Or use the machine name if MagicDNS is enabled
ssh user@homelab-server
```

---

## Checking Wazuh Alerts

1. Open Wazuh dashboard: `https://192.168.1.200:5601` (or via Tailscale IP)
2. Navigate to **Security Events** → filter by `rule.level >= 7` for high-severity
3. Review **FIM** tab for file integrity changes
4. Review **Vulnerability Detector** for CVEs on installed packages

```bash
# View Wazuh agent status on server
sudo /var/ossec/bin/agent_control -l

# Check Wazuh manager logs
sudo tail -f /var/ossec/logs/ossec.log

# Restart Wazuh manager
sudo systemctl restart wazuh-manager
```

---

## Checking Graylog

1. Open: `http://192.168.1.200:9000`
2. Use **Search** to query logs: e.g., `source:server AND message:FAILED`
3. Check **Streams** for pre-filtered alert streams
4. Review **Dashboards** for visual summaries

---

## Pi-hole Maintenance

```bash
# Update blocklists
pihole -g

# Check Pi-hole status
pihole status

# Temporarily disable for 5 minutes
pihole disable 5m

# View query log
pihole -t
```

---

## Adding a New Tailscale Device

1. Install Tailscale on the device
2. Run: `tailscale up`
3. Authenticate via browser
4. Approve device in Tailscale admin console: `https://login.tailscale.com/admin/machines`
5. Update [[02 - Network Map]] with new device entry

---

## Updating All Containers

```bash
# Pull new images
docker compose pull

# Restart with new images
docker compose up -d

# Remove old images
docker image prune -f
```

---

## Incident Response Checklist

If you detect suspicious activity:

- [ ] Check Wazuh alerts dashboard for triggered rules
- [ ] Search Graylog for related log events (auth, UFW blocks, process events)
- [ ] Run `who` / `last` to check active and recent sessions
- [ ] Check `ss -tulnp` for unexpected listening ports
- [ ] Check `ps aux` for unexpected processes
- [ ] Review `/var/log/auth.log` for auth events
- [ ] If compromised: isolate via `ufw default deny incoming` then investigate
- [ ] Document findings in [[06 - Incident Log]] (create as needed)

---

*← Back to [[00 - Home Lab Overview]]*
