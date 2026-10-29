# Cybersecurity Testing Lab Setup (Week 1 - LinuxVM, AndroidVM, WindowsVM)

## Overview
This repository documents the deployment, configuration, and verification of an isolated virtual cybersecurity testing lab built with Oracle VirtualBox as part of the NetworkWalks Academy Cybersecurity Internship program.

The objective is to establish an isolated virtual subnet capable of hosting both attacking platforms and target machines safely without exposing local network devices, validating end-to-end routing, DNS resolution, and cross-machine reachability.

---

## Lab Architecture & Specifications

| Component | Role | OS / Platform | IP Address | Subnet CIDR |
| :--- | :--- | :--- | :--- | :--- |
| **Hypervisor** | Virtualization Host | Oracle VirtualBox 7.x | — | — |
| **Attacking Machine (WK1-PM1)** | Security Testing Node | Kali Linux (x86_64) | `10.0.0.10` | `10.0.0.0/24` |
| **Target Machine (WK1-PMx)** | Target / Mobile Node | Android-x86 9.0 | `10.0.0.9` | `10.0.0.0/24` |
| **Default Gateway** | Virtual NAT Router | VirtualBox Internal | `10.0.0.1` | `10.0.0.0/24` |

---

## Part 1: Attacking Platform Deployment (Kali Linux)

### 1. Hypervisor & NAT Network Setup
- Configured a dedicated **NAT Network** named `NatNetwork` with IPv4 prefix `10.0.0.0/24`.
- Enabled internal DHCP service for scalable lab target integration.

![NAT Network Configuration](<Screenshots for KaliVM/nat-network.png>)

### 2. Attacking Machine Deployment
- Imported the pre-built Kali Linux 64-bit appliance into VirtualBox.
- Attached Adapter 1 to `NAT Network` (`NatNetwork`).
- Configured bidirectional clipboard sharing and drag-and-drop.
- Configured host-guest shared folders for secure file transfer.

![Kali Linux Desktop](<Screenshots for KaliVM/kali-desktop.png>)

### 3. Static IP Configuration & Routing
- Assigned Kali Linux a static configuration (`10.0.0.10/24`) to avoid reserved address conflicts with the VirtualBox gateway and internal services.
- Verified gateway reachability and external DNS resolution.

```bash
# Verify assigned IP address
ip a

# Test default gateway connectivity
ping -c 3 10.0.0.1

# Test internet routing & DNS resolution
ping -c 3 1.1.1.1
ping -c 3 google.com
```

![Verifying IP address](<Screenshots for KaliVM/network-test-1.png>)
![Testing internet routing](<Screenshots for KaliVM/network-test-2.png>)

### 4. Baseline Recovery Snapshot
- Captured an initial, clean-state snapshot (`Snapshot 1`) before running exploits or installing tools.
- Enables instant rollback and deterministic lab testing.

![Baseline Snapshot](<Screenshots for KaliVM/snapshot.png>)

---

## Part 2: Target Integration 1 - Android

### 1. Target Deployment & Network Configuration
- Deployed an **Android-x86 9.0 (r2)** virtual appliance attached to the shared `NatNetwork` (`10.0.0.0/24`).

![Android Installation](<Screenshots for AndroidVM/installation.png>)
![Android Homepage](<Screenshots for AndroidVM/android-homepage.png>)

- Partitioned a dedicated 10 GB virtual drive formatted with `ext4` and installed legacy GRUB for full state persistence.
- Assigned a static IP configuration to interface `wlan0`:
  * **IP Address:** `10.0.0.9`
  * **Subnet Mask:** `255.255.255.0` (`/24`)
  * **Gateway:** `10.0.0.1`
  * **Primary DNS:** `8.8.8.8`
- Added explicit default gateway route entries (`ip route add default via 10.0.0.1 dev wlan0`) to ensure routing table persistence.

### 2. Dual-Node Verification & Connectivity Test
Confirmed cross-node connectivity across the isolated subnet as well as external internet access:

```bash
# From Android Terminal:
ping -c 3 10.0.0.10     # Reachability to Kali Attacker
ping -c 3 8.8.8.8       # Outbound Internet Reachability

# From Kali Terminal:
ping -c 3 10.0.0.9      # Reachability to Android Target
```

![Bilateral Network Verification - Android to Kali](<Screenshots for AndroidVM/android to kali.png>)
![Bilateral Network Verification - Kali to Android](<Screenshots for AndroidVM/kali to android.png>)

### 3. Baseline Recovery Snapshot
- Captured an initial, clean-state snapshot (`Snapshot 1 - clean lab setup`) before running exploits or installing tools.
- Enables instant rollback and deterministic lab testing.

![Baseline Snapshot](<Screenshots for AndroidVM/android snapshot.png>)

---

## Part 3: Target Integration 2 — Windows 10

### 1. Target Deployment & Network Configuration
- Deployed a **Windows 10 (64-bit)** virtual machine with 4 GB RAM and a 40 GB dynamically allocated VDI drive attached to `NatNetwork` (`10.0.0.0/24`).

![Desktop](<Screenshots for WindowsVM/windows desktop.png>)
 
- Configured a static IPv4 address via Network Adapter settings (`ncpa.cpl`):
  * **IP Address:** `10.0.0.11`
  * **Subnet Mask:** `255.255.255.0` (`/24`)
  * **Default Gateway:** `10.0.0.1`
  * **Preferred DNS:** `8.8.8.8`
- Added an inbound ICMPv4 Echo Request rule in Windows Defender Firewall (`netsh advfirewall firewall add rule name="Allow ICMPv4-In" protocol=icmpv4:8,any dir=in action=allow`) to permit ping verification from Kali Linux.
- Captured a clean baseline snapshot (`Snapshot 1 - Clean Lab Setup`) for rapid rollback.

### 2. Dual-Node Verification & Connectivity Test
Confirmed bidirectional reachability between Kali Linux, Windows 10, and external DNS:

```cmd
:: From Windows Command Prompt:
ping 10.0.0.10          :: Reachability to Kali Attacker
ping 8.8.8.8            :: Outbound Internet Reachability
```

```bash
# From Kali Terminal:
ping -c 3 10.0.0.11     # Reachability to Windows 10 Target
```

![Bilateral Network Verification - Windows to Kali](<Screenshots for WindowsVM/windows to kali.png>)
![Bilateral Network Verification - Kali to Windows](<Screenshots for WindowsVM/kali to windows.png>)

### 3. Baseline Recovery Snapshot
- Captured an initial, clean-state snapshot (`Snapshot 1 - clean lab setup`) before running exploits or installing tools.
- Enables instant rollback and deterministic lab testing.

![Baseline Snapshot](<Screenshots for WindowsVM/windows snapshot.png>)

---

## Key Troubleshooting & Engineering Notes
- **VirtualBox NAT Network IP Reservations:** In a VirtualBox `10.0.0.0/24` NAT Network, addresses `10.0.0.1` through `10.0.0.3` are internally reserved by the hypervisor architecture for default routing, DHCP dispatch, and DNS proxying. Setting client VMs to addresses within this reserved pool causes routing failure upon reboot; shifting hosts to the usable pool starting at `.4` (`10.0.0.9`, `10.0.0.10`, `10.0.0.11`) eliminates IP collisions and ensures persistent connectivity.
- **Android-x86 Virtual Interface Routing:** Android-x86 exposes the virtualized adapter as `wlan0`. If the interface fails to propagate the default gateway upon static assignment, explicitly defining the gateway using `ip route add default via 10.0.0.1 dev wlan0` resolves the missing route without requiring hypervisor bridge rebuilds.
- **Windows 10 ICMP Filtering:** By default, Windows Defender Firewall drops inbound ICMP Echo requests on newly joined networks, causing pings from the attacking node to time out even when routing is functional. Explicitly allowing `icmpv4:8` inbound enables accurate reachability verification without disabling the entire firewall service.

---

## Author
- **Irfan K**
- Cybersecurity Intern @ NetworkWalks Academy
