# 🛡️ Firewall Rules — UFW

**Server:** `192.168.1.200`  
**Tool:** UFW (Uncomplicated Firewall)  
**Default policy:** DENY inbound, ALLOW outbound

---

## Current Rules

```
To                          Action      From
──────────────────────────────────────────────────────
Anywhere                    ALLOW IN    192.168.0.0/16
Anywhere on tailscale0      ALLOW IN    Anywhere
22/tcp on tailscale0        LIMIT IN    Anywhere
53 on br0                   ALLOW IN    Anywhere
Anywhere (v6) on tailscale0 ALLOW IN    Anywhere (v6)
22/tcp (v6) on tailscale0   LIMIT IN    Anywhere (v6)
53 (v6) on br0              ALLOW IN    Anywhere (v6)
```

---

## Rule Breakdown & Rationale

### Rule 1 — Allow LAN Inbound
```bash
ufw allow in from 192.168.0.0/16
```
| Field | Value |
|-------|-------|
| **Source** | `192.168.0.0/16` (entire LAN range) |
| **Destination** | All ports / protocols |
| **Action** | ALLOW |
| **Rationale** | Trust devices on your local network to reach server services |
| **Risk** | Any compromised LAN device has full access — consider restricting by port as you mature |

---

### Rule 2 — Allow All Inbound on Tailscale Interface
```bash
ufw allow in on tailscale0
```
| Field | Value |
|-------|-------|
| **Interface** | `tailscale0` |
| **Source** | Any (but Tailscale enforces device auth upstream) |
| **Action** | ALLOW |
| **Rationale** | Only authenticated Tailscale devices can reach this interface — safe to allow broadly |
| **Benefit** | All Tailscale-approved devices can reach services without opening public ports |

---

### Rule 3 — Rate Limit SSH on Tailscale (IPv4 + IPv6)
```bash
ufw limit in on tailscale0 to any port 22 proto tcp
```
| Field | Value |
|-------|-------|
| **Interface** | `tailscale0` |
| **Port** | `22/tcp` (SSH) |
| **Action** | LIMIT (rate limit) |
| **Rationale** | Prevents brute-force attacks against SSH even from within the Tailscale mesh |
| **Behavior** | UFW blocks IPs with >6 connection attempts in 30 seconds |
| **Note** | SSH is only reachable via Tailscale — not exposed on public internet |

---

### Rule 4 — Allow DNS on Docker Bridge (IPv4 + IPv6)
```bash
ufw allow in on br0 to any port 53
```
| Field | Value |
|-------|-------|
| **Interface** | `br0` (Docker bridge) |
| **Port** | `53` (DNS — UDP + TCP) |
| **Action** | ALLOW |
| **Rationale** | Docker containers need to resolve DNS via Pi-hole |
| **Scope** | Scoped to `br0` only — DNS not exposed on other interfaces |

---

## Hardening Recommendations (Next Steps)

| Improvement | Command | Rationale |
|------------|---------|-----------|
| Restrict LAN by port | `ufw allow in from 192.168.0.0/16 to any port 80,443,8096` | Reduce exposure per service instead of all ports |
| Deny forward by default | `ufw default deny forward` | Prevent routing between Docker networks unintentionally |
| Log UFW drops | `ufw logging on` | Feed into Graylog for alerting on blocked traffic |
| Enable fail2ban | install `fail2ban` | Complements UFW rate-limiting with persistent bans |

---

## Useful Commands

```bash
# View rules with numbers
sudo ufw status numbered

# Add a rule
sudo ufw allow in on tailscale0 to any port 9000  # example: Graylog UI

# Delete a rule by number
sudo ufw delete 3

# Reload UFW
sudo ufw reload

# View UFW logs
sudo tail -f /var/log/ufw.log
```

---

*← Back to [[04 - Security Stack]] | [[00 - Home Lab Overview]]*
