# Hardware Specifications

## Processor
- **Model:** Intel Pentium G3240 @ 3.10GHz
- **Architecture:** Haswell (2013)
- **Cores/Threads:** 2C/2T
- **TDP:** 53W
- **Cache:** 3MB L3

## Motherboard
- **Chipset:** Intel H81
- **Form Factor:** Micro-ATX
- **Manufacturer:** Intel
- **BIOS:** Standard UEFI/Legacy Support
- **Era:** 2013-2014

## Memory
- **Total Capacity:** 12GB
- **Type:** DDR2 (DDR2-800 or DDR2-1066)
- **Available to Host:** ~9.7GB (rest allocated to services)
- **Swap Allocated:** 8GB

## Storage

### Primary Drive
- **Model:** WD Purple WD10PURX
- **Capacity:** 931.5GB (1TB)
- **RPM:** 5400 (Surveillance-grade drive)
- **Interface:** SATA 3.0
- **Current Allocation:**
  - Proxmox Root FS: 96GB
  - LVM Data Pool: 794.3GB
  - MyCloud Container: 500GB
  - Swap: 8GB

### Secondary Drive (Non-Functional)
- **Model:** Hikvision HS-SSD-E100
- **Capacity:** 256GB
- **Status:** ❌ Dead (Hardware failure - NAND chip degradation)
- **Diagnosis Method:** Ubuntu Live USB + ddrescue revealed 2.3M read errors, 0 bytes recovered

## Network
- **Adapter:** Realtek RTL8110 Fast Ethernet Controller
- **Interface:** PCI Express
- **Speed:** 100 Mbps (Fast Ethernet)
- **MAC Address:** 00:f3:0c:81:24:f8
- **Connection Type:** Hardwired Ethernet to TP-Link TL-WR841N Router

## Power Consumption
- **Idle Draw:** 50-70W
- **Annual Cost:** ~₹3,000-4,200 @ ₹10/unit (24/7 operation)
- **Justification:** Cheaper than streaming subscriptions

---

# The Big Discovery: Hardware Bottleneck

## The Problem
Transferring files from laptop to server capped at **5 MB/s** despite:
- NVMe SSDs in laptop (capable of 3500+ MB/s)
- Modern Intel Wi-Fi 6 AX203 in laptop (capable of 600+ Mbps)
- 1TB surveillance drive ready for writes
- Gigabit fiber internet

**Something was clearly wrong.**

## The Investigation

### Step 1: Check Laptop Wi-Fi Speed
```bash
netsh wlan show interfaces
```
**Result:**
```
Network band (channel): 2.4 GHz (10)
Receive rate (Mbps): 130
Transmit rate (Mbps): 52
Radio type: 802.11n (Wi-Fi 4)
```
**Finding:** Laptop locked on 2.4GHz, theoretical max 130 Mbps

### Step 2: Scan Available Networks
```bash
netsh wlan show networks mode=bssid
```
**Result:**
```
SSID: Shanthi Ashok
BSSID 1: b0:19:21:1b:12:04
Radio type: 802.11n
Band: 2.4 GHz
```
**Finding:** Only ONE antenna broadcasting, only 2.4GHz band available

### Step 3: Check Router Specs
Logged into `192.168.0.1` router dashboard:
```
Model: TP-Link TL-WR841N v13
Wireless: 802.11bgn mixed (No 5GHz radio)
LAN Ports: Fast Ethernet (100 Mbps, NOT Gigabit)
```

## The Root Cause

**Physical Network Limitation:**
```
100 Mbps ÷ 8 bits/byte = 12.5 MB/s theoretical maximum
Real-world overhead (Samba, TCP/IP, router processing) = ~40% loss
Actual achievable speed = 5-8 MB/s
```

**This is NOT a software issue.** No amount of optimization can bypass the physical 100 Mbps ports.

## The Workaround

Instead of pushing files across slow local network:
- **Old approach:** Laptop → (slow Wi-Fi) → Server
- **New approach:** Server → (fast Ethernet) → Internet → Direct downloads

Server now pulls content directly from internet sources, bypassing laptop and local Wi-Fi entirely.
