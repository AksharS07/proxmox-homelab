# Phase-by-Phase Setup Journey

## Phase 1: Hardware Rescue (June 10, 2026)
**Goal:** Determine what's salvageable from dead rig

**Actions:**
- Booted with Ubuntu Live USB
- Discovered Hikvision SSD completely dead (NAND failure)
- Confirmed WD Purple HDD still functional
- Identified 12GB DDR2 RAM intact
- Motherboard responding normally

**Decision:** Don't throw away—repurpose as headless server
**Output:** Confirmed viable hardware path forward

---

## Phase 2: Proxmox Installation (June 11, 2026)
**Goal:** Install hypervisor directly onto 1TB drive

**Actions:**
- Downloaded Proxmox VE 9.2 ISO
- Flashed to Lexar USB via `dd` command on laptop
- Booted old rig from USB
- Selected 1TB WD Purple as installation target
- Configured DHCP networking
- Set root password
- Set hostname: proxmox.local
- Recorded IP: 192.168.0.100:8006

**Key Decision:** Hypervisor directly on hardware = maximum efficiency
**Output:** Proxmox running, accessible from laptop browser

---

## Phase 3: Container Creation (June 11, 2026)
**Goal:** Build lightweight Ubuntu environment for applications

**Actions:**
- Downloaded Ubuntu 22.04 Standard template
- Created LXC container (MyCloud)
- Allocated 500GB storage from LVM pool
- Allocated 2 CPU cores (out of 2)
- Allocated 2GB RAM (out of 12GB)
- Set DHCP networking
- Started container

**Why LXC not VM?**
- Containers are lightweight (2 seconds boot time vs 30+ for full VM)
- Shared kernel = less resource overhead
- Perfect for file server + lightweight services

**Output:** Ubuntu 22.04 container running at 192.168.0.X

---

## Phase 4: CasaOS Installation (June 11, 2026)
**Goal:** Beautiful web dashboard for file management

**Actions:**
- Logged into container via Proxmox console
- Ran: `wget -qO- https://get.casaos.io | sudo bash`
- Container installed Docker + CasaOS automatically
- Accessed web interface at container's IP
- Created admin account: akshar
- Configured file sharing

**Why CasaOS?**
- Zero command-line required for daily use
- Mobile-friendly interface
- Built-in app store (Jellyfin, AdGuard, etc.)
- Professional-grade but user-friendly

**Output:** Web dashboard live and accessible

---

## Phase 5: Samba Configuration (June 11, 2026)
**Goal:** Make server visible in Windows File Explorer

**Actions:**
- Created SMB password for root account: `smbpasswd -a root`
- Shared Cloud_Drive folder via CasaOS
- Attempted Windows mapping: `\\192.168.0.102`
- Hit "Network Error 1208" (credential rejection)
- Flushed Windows ghost connections: `net use * /delete`
- Successfully mapped drive using root + SMB password

**Learning:** CasaOS web login ≠ Samba network login (different subsystems)
**Output:** Cloud_Drive now appears in Windows File Explorer

---

## Phase 6: Tailscale VPN (June 11, 2026)
**Goal:** Secure remote access from anywhere

**Actions:**
- Installed Tailscale in container: `curl -fsSL https://tailscale.com/install.sh | sh`
- Hit TUN device permission issue (Proxmox security)
- Added LXC device permissions in `/etc/pve/lxc/100.conf`
- Restarted container
- Ran: `tailscale up`
- Clicked authentication link
- Logged in with Google account
- Container joined mesh network

**Why Tailscale?**
- No port forwarding (security-first)
- Encrypted WireGuard tunnels
- Works on home Wi-Fi AND cellular data
- Automatic subnet routing

**Key Lesson:** Hypervisors have security boundaries—had to grant explicit permissions
**Output:** Server accessible via 100.x.x.x IP from anywhere

---

## Phase 7: Mobile Integration (June 12, 2026)
**Goal:** Native file access on Android phone

**Actions:**
- Installed Tailscale app on Realme phone
- Installed CX File Explorer app
- Added SMB network drive: 100.x.x.x (Tailscale IP)
- Entered credentials: root + SMB password
- Successfully accessed Cloud_Drive from phone

**Why not CasaOS web interface?**
- Native file manager is faster
- Works exactly like internal storage
- No browser tabs cluttering screen
- Offline access possible

**Output:** Phone now has native file explorer access

---

## Phase 8: Bottleneck Diagnosis (June 12, 2026)
**Goal:** Understand why 5 MB/s transfer speed

**Actions:**
- Ran `netsh wlan show interfaces` on laptop
- Discovered 2.4GHz lock + 130 Mbps cap
- Ran `netsh wlan show networks mode=bssid`
- Found router broadcasting ONLY 2.4GHz
- Accessed router settings: TP-Link TL-WR841N v13
- Confirmed 100 Mbps Fast Ethernet ports (not Gigabit)
- Calculated: 100 Mbps ÷ 8 = 12.5 MB/s max

**Realization:** This is a hardware problem, not software
**Impact:** Shifted from "optimize software" to "work around hardware limitation"
**Output:** Strategy change—server pulls from internet, not relying on local transfers
