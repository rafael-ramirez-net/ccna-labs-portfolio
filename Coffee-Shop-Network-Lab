# Network Engineering Portfolio | CCNA 200-301

## Coffee Shop Network Simulation: VLAN Segmentation, Secure Guest Wireless, Router DHCP, and ISP WAN Gateway

**Author:** Rafael Andrés Ramírez Benavides
**Role:** Junior Network Engineer
**Certifications in preparation:** Cisco Certified Network Associate (CCNA 200-301)
**Tools:** Cisco Packet Tracer

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Network Topology and IP Addressing Scheme](#2-network-topology-and-ip-addressing-scheme)
3. [Key Technical Implementations](#3-key-technical-implementations)
4. [Configuration Highlights (Cisco IOS)](#4-configuration-highlights-cisco-ios)
5. [Verification and Testing Results](#5-verification-and-testing-results)
6. [Demonstrated Engineering Competencies](#6-demonstrated-engineering-competencies)

---

## 1. Executive Summary

This project documents the end-to-end design, implementation, and security hardening of a commercial Coffee Shop network infrastructure. The goal of this architecture is to deliver reliable network operations for business services while isolating public guest Wi-Fi traffic from internal administrative and Point-of-Sale (POS) systems.

The solution deploys:

- IEEE 802.1Q VLAN segmentation
- Router-on-a-Stick (ROAS) inter-VLAN routing
- Cisco IOS DHCP Server pools per VLAN
- A Point-to-Point (PtP) WAN connection to an Internet Service Provider (ISP)
- An Autonomous Access Point providing WPA2-secured Wi-Fi
- Access Control Lists (ACLs) to strictly isolate guest users from internal corporate assets

---

## 2. Network Topology and IP Addressing Scheme

```text
                                  [ ISP Router ]
                                        | 10.1.1.2/30 (Gi0/0)
                                  ( WAN /30 PtP )
                                        | 10.1.1.1/30 (Gi0/0)
                                  [ R_Cafeteria ]
                                        | (Gi0/1 Trunk 802.1Q)
                                    [ Switch ]
          +-----------------------------+-----------------------------+
          | (Fa0/23 Access V10)         | (Fa0/22 & Fa0/24 Access V20)| (Gi0/2 Trunk V30)
  [ VLAN 10: Management ]     [ VLAN 20: POS / Cash ]     [ AccessPoint0 (Autonomous) ]
  - PCmanager                 - PCCaja                    - LaptopCliente
  - Printer0                  - Printer1                  - Tablet Clientes
                                                          - SmartphoneCliente 1,2,3
```

| Device | Interface / Subinterface | IP Address / Mask | VLAN / Purpose | DHCP Pool / Range |
| :--- | :--- | :--- | :--- | :--- |
| **ISP** | Gi0/0 | `10.1.1.2 /30` | WAN Gateway (Simulated ISP) | N/A (Static) |
| **R_Cafeteria** | Gi0/0 | `10.1.1.1 /30` | WAN PtP Interface | N/A (Static) |
| **R_Cafeteria** | Gi0/1.10 | `192.168.10.1 /24` | Gateway - Management (VLAN 10) | `VLAN10_MGMT` (`.1`–`.10` excluded) |
| **R_Cafeteria** | Gi0/1.20 | `192.168.20.1 /24` | Gateway - POS / Cash (VLAN 20) | `VLAN20_POS` (`.1`–`.10` excluded) |
| **R_Cafeteria** | Gi0/1.30 | `192.168.30.1 /24` | Gateway - Guest Wi-Fi (VLAN 30) | `VLAN30_WIFI` (`.1`–`.10` excluded) |
| **Switch** | Gi0/1 | Unassigned | 802.1Q Trunk to R_Cafeteria | N/A |
| **Switch** | Fa0/23 | Unassigned | Access Port (VLAN 10) | N/A |
| **Switch** | Fa0/22, Fa0/24 | Unassigned | Access Ports (VLAN 20) | N/A |
| **Switch** | Gi0/2 | Unassigned | Link to AccessPoint0 (VLAN 30) | N/A |
| **PCmanager** | NIC (DHCP) | `192.168.10.11 /24` | Management Workstation | Assigned via `VLAN10_MGMT` |
| **PCCaja** | NIC (DHCP) | `192.168.20.11 /24` | Cash Register POS System | Assigned via `VLAN20_POS` |
| **Wireless Clients** | Wireless NIC (DHCP) | `192.168.30.11+ /24` | Guest Laptops / Tablets / Phones | Assigned via `VLAN30_WIFI` |

---

## 3. Key Technical Implementations

### A. Layer 2 Segmentation & Trunking

Three dedicated VLANs were created to separate network traffic based on function and security level:

- **VLAN 10 (Management):** Internal management workstations (`PCmanager`) and administrative printers (`Printer0`).
- **VLAN 20 (POS):** Point-of-Sale cash registers (`PCCaja`) and receipt printers (`Printer1`).
- **VLAN 30 (WIFI_Clientes):** Public wireless access for customers.

Access ports on the main switch were mapped strictly to their respective VLANs, and the uplink interface to `R_Cafeteria` (`Gi0/1`) was configured as an 802.1Q trunk.

### B. Inter-VLAN Routing (ROAS) & Security ACL Isolation

- **Subinterface encapsulation:** Subinterfaces `Gi0/1.10`, `Gi0/1.20`, and `Gi0/1.30` were created on `R_Cafeteria` with `encapsulation dot1Q` to enable Layer 3 routing between segments.
- **Inter-VLAN security restriction:** Full inter-routing is enabled between administrative VLAN 10 and operational VLAN 20, but **VLAN 30 (Guest Wi-Fi) is blocked from accessing internal subnets** (`192.168.10.0/24` and `192.168.20.0/24`). An extended ACL applied inbound on `Gi0/1.30` enforces this while still allowing outbound traffic to the Internet.

### C. Cisco IOS Integrated DHCP Server

Centralized DHCP services were configured on `R_Cafeteria` for all three VLANs:

- **Address exclusions:** Reserved the first ten addresses (`.1` through `.10`) of each subnet for gateways and infrastructure, preventing address conflicts (`%DHCPD-4-PING_CONFLICT`).
- **Pool definitions:** Configured network ranges, default gateways, and public DNS servers (`8.8.8.8`) for each pool.

### D. ISP WAN Gateway & Default Static Route

- Established a `/30` point-to-point network (`10.1.1.0/30`) between `R_Cafeteria` (`10.1.1.1`) and `ISP` (`10.1.1.2`).
- Configured a default static route (`ip route 0.0.0.0 0.0.0.0 10.1.1.2`) on `R_Cafeteria` to direct all external, Internet-bound traffic through the ISP gateway.

### E. Autonomous Wireless Access Point Integration

- Deployed an **Autonomous Access Point (AP-PT)** connected to the switch port serving VLAN 30.
- Configured an SSID (`Cafeteria_Guest_WiFi`) with **WPA2-PSK** security to encrypt customer wireless sessions.
- Wireless clients (laptops, tablets, smartphones) associate automatically, request IP parameters dynamically from the `VLAN30_WIFI` pool, and obtain Internet access without exposure to internal devices.

---

## 4. Configuration Highlights (Cisco IOS)

### Router Configuration (`R_Cafeteria`)

```text
! 1. Parent Interface & Subinterfaces (ROAS)
interface GigabitEthernet0/1
 no ip address
 no shutdown
!
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
!
interface GigabitEthernet0/1.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
 ip access-group BLOCK_GUEST_TO_INTERNAL in
!
! 2. DHCP Pools & Exclusions
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp excluded-address 192.168.30.1 192.168.30.10
!
ip dhcp pool VLAN10_MGMT
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
!
ip dhcp pool VLAN20_POS
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
!
ip dhcp pool VLAN30_WIFI
 network 192.168.30.0 255.255.255.0
 default-router 192.168.30.1
 dns-server 8.8.8.8
!
! 3. Extended ACL for Guest Wi-Fi Isolation
ip access-list extended BLOCK_GUEST_TO_INTERNAL
 deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
 deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
 permit ip 192.168.30.0 0.0.0.255 any
!
! 4. WAN Interface & Default Static Route to ISP
interface GigabitEthernet0/0
 ip address 10.1.1.1 255.255.255.252
 no shutdown
!
ip route 0.0.0.0 0.0.0.0 10.1.1.2
```

---

## 5. Verification and Testing Results

| Test Scenario | Source Device | Target IP | Expected Outcome | Actual Result |
| :--- | :--- | :--- | :--- | :---: |
| **Management to POS routing** | `PCmanager` (VLAN 10) | `192.168.20.11` (`PCCaja`) | Successful ICMP ping | ✅ PASSED |
| **POS to Management routing** | `PCCaja` (VLAN 20) | `192.168.10.11` (`PCmanager`) | Successful ICMP ping | ✅ PASSED |
| **Guest Wi-Fi to ISP** | `LaptopCliente` (VLAN 30) | `10.1.1.2` (`ISP`) | Successful ICMP ping | ✅ PASSED |
| **Guest Wi-Fi to Management (ACL)** | `SmartphoneCliente1` (VLAN 30) | `192.168.10.11` (`PCmanager`) | Destination host unreachable / blocked | ✅ PASSED |
| **Guest Wi-Fi to POS (ACL)** | `Tablet Clientes` (VLAN 30) | `192.168.20.11` (`PCCaja`) | Destination host unreachable / blocked | ✅ PASSED |
| **DHCP address allocation** | Wireless Clients | VLAN 30 range | Dynamic IP assigned in `192.168.30.0/24` | ✅ PASSED |

---

## 6. Demonstrated Engineering Competencies

- Designing multi-VLAN segmentation models tailored for business operations and public access.
- Implementing Router-on-a-Stick (ROAS) and securing inter-VLAN communications using extended Access Control Lists (ACLs).
- Configuring Cisco IOS DHCP Server services with IP exclusions to mitigate address conflicts.
- Integrating Autonomous Wireless Access Points with WPA2 security into segmented LAN environments.
- Configuring Point-to-Point WAN interfaces and default static routing toward an ISP edge.
