# Cybersecurity Testing Lab Setup (Week 1 - Task WK1-PM1)

## Overview
This repository documents the deployment and configuration of an isolated virtual cybersecurity testing lab built using Oracle VirtualBox and Kali Linux as part of the NetworkWalks Academy Cybersecurity Internship program.

The objective is to establish an isolated virtual network environment capable of hosting attacking platforms and target machines safely without exposing local network devices.

---

## Lab Architecture & Specifications

| Component | Configuration |
| :--- | :--- |
| **Hypervisor** | Oracle VirtualBox 7.x |
| **Attacking Machine** | Kali Linux (x86_64) |
| **Virtual Network Mode** | Custom NAT Network |
| **Subnet CIDR** | `10.0.0.0/24` |
| **Gateway Address** | `10.0.0.1` |
| **Kali Assigned IP** | `10.0.0.10/24` |
| **DNS Resolvers** | `1.1.1.1`, `8.8.8.8` |

---

## Implementation Steps

### 1. Hypervisor & NAT Network Setup
- Configured a dedicated **NAT Network** named `NatNetwork` with IPv4 prefix `10.0.0.0/24`.
- Enabled internal DHCP service for scalable lab target integration.

![NAT Network Configuration](screenshots/01-nat-network.png)

### 2. Attacking Machine Deployment (Kali Linux)
- Imported the pre-built Kali Linux 64-bit appliance into VirtualBox.
- Attached Adapter 1 to `NAT Network` (`NatNetwork`).
- Configured bidirectional clipboard sharing and drag-and-drop.
- Configured host-guest shared folders for secure file transfer.

![Kali Linux Desktop](screenshots/02-kali-desktop.png)

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

### 4. Baseline Recovery Snapshot
- Captured an initial, clean-state snapshot (`Snapshot 1`) before installing tools or running exploits.
- Enables instant rollback and deterministic lab testing.

![Baseline Snapshot](screenshots/04-snapshot.png)
