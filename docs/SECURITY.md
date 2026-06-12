# Security Architecture

## Security Decisions & Trade-offs

| Decision | Implementation | Why | Risk Mitigated |
|----------|-----------------|-----|-----------------|
| **VPN over Port Forwarding** | Tailscale mesh network | Encrypted tunnel, not public-facing | Bots scanning for open ports |
| **No Public Internet Access** | Server invisible on WAN | Complete invisibility | Network reconnaissance attacks |
| **Firewall Active** | Standard Linux iptables | Defense in depth | Assume-breach mentality |
| **SMB Credentials** | Root account + strong password | Centralized auth | Brute force attacks on file shares |
| **No UPnP/UPNP** | Disabled in router | Prevents auto port mapping | Malware opening router ports |
| **Tailscale 2FA** | Enabled on Google account | Multi-factor authentication | Account takeover |

## Attack Surface Analysis

### Threat: External Network Scanner
- **Attack Method:** Nmap scanning public IP addresses
- **Your Defense:** Server has NO public IP address (private Tailscale only)
- **Result:** ✅ Server completely invisible

### Threat: Brute Force SMB
- **Attack Method:** Attacker tries common passwords on SMB port 445
- **Your Defense:** SMB only accessible via Tailscale (encrypted tunnel)
- **Result:** ✅ Attacker can't even reach SMB port

### Threat: Compromised Router
- **Attack Method:** Attacker gains access to TP-Link router
- **Your Defense:** Server not accessible via router's LAN isolation
- **Result:** ⚠️ Partial risk (attacker could see Tailscale traffic but can't decrypt)

### Threat: Google Account Hijacking
- **Attack Method:** Attacker steals your Gmail credentials
- **Your Defense:** Tailscale 2FA + password manager
- **Result:** ✅ Mitigated via 2FA enforcement
