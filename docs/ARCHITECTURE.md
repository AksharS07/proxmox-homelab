# System Architecture

## The Big Picture

```
┌─────────────────────────────────────────────┐
│  Old Pentium Desktop (Headless Server)      │
│  Plugged in 24/7 in corner, no monitor      │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│     Proxmox VE 9.2 (Hypervisor)             │
│     Running on 1TB WD Purple HDD            │
│     Role: Infrastructure Landlord           │
└─────────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────────┐
│  Ubuntu 22.04 LXC Container (MyCloud)       │
│  - 500GB allocated storage                  │
│  - 2 CPU cores, 2GB RAM                     │
│  - Role: Application Server                 │
│                                              │
│  ├─ CasaOS (Web Dashboard)                  │
│  ├─ Samba/SMB (File Sharing Protocol)       │
│  ├─ Tailscale (Secure VPN)                  │
│  ├─ [Ready for] Jellyfin (Media Server)     │
│  └─ [Ready for] Radarr/Sonarr (Automation)  │
└─────────────────────────────────────────────┘
              ↓
┌──────────────────────────┬──────────────────┐
│  Secure Tailscale Mesh   │                  │
│  (Private VPN Network)   │                  │
│                          │                  │
│  ✓ Phone (Android)       │ All encrypted,   │
│  ✓ Laptop (Windows/Linux)│ all secure       │
│  ✓ Server (Linux)        │                  │
└──────────────────────────┴──────────────────┘
```

## Software Stack

```text
Layer 4: Applications
├── CasaOS (Web UI Dashboard)
├── Samba/SMB (Network File Sharing)
├── Tailscale (VPN Mesh Network)
└── [Planned] Jellyfin + Radarr + Sonarr

Layer 3: Container OS
├── Ubuntu 22.04 LTS
├── 500GB LVM Storage
└── 2GB Memory Allocation

Layer 2: Hypervisor
├── Proxmox VE 9.2
├── LVM Partitioning
├── Virtual Bridge Networking
└── Container Management

Layer 1: Hardware
├── Pentium G3240 CPU
├── 12GB DDR2 RAM
├── 1TB WD Purple HDD
└── Realtek RTL8110 NIC (100 Mbps)
```

## Network Topology

```text
                    ┌─────────────────────┐
                    │  Tailscale Account  │
                    │   (Google OAuth)    │
                    └──────────┬──────────┘
                               │
                ┌──────────────┼──────────────┐
                │              │              │
         ┌──────▼─────┐  ┌────▼──────┐  ┌───▼──────┐
         │   Phone    │  │  Laptop   │  │  Server  │
         │(Tailscale) │  │(Tailscale)│  │(Tailscale│
         └──────┬─────┘  └────┬──────┘  └───┬──────┘
                │              │              │
                └──────────────┼──────────────┘
                               │
                    (Encrypted Mesh VPN)
                               │
                ┌──────────────┴──────────────┐
                │                            │
        On Home Wi-Fi          Outside Home
    Access via 192.168.0.x    Access via 100.x.x.x
     (Tailscale IP)            (Tailscale IP)
```
