[Lab-EtherChannel-LACP-Port-Security.md]

# Network Engineering Portfolio | CCNA 200-301

## Lab: EtherChannel LACP and Layer 2 Security (Port Security)

**Author:** Rafael Andrés Ramírez Benavides
**Role:** Junior Network Engineer
**Certifications in preparation:** Cisco Certified Network Associate (CCNA 200-301)
**Tools:** Cisco Packet Tracer

---

## Table of Contents

1. [Lab Overview](#1-lab-overview)
2. [Topology and Interface Table](#2-topology-and-interface-table)
3. [Step-by-Step Configuration](#3-step-by-step-configuration)
4. [Operational Verification](#4-operational-verification)
5. [Troubleshooting Case Studies](#5-troubleshooting-case-studies)
6. [Key Takeaways](#6-key-takeaways)

---

## 1. Lab Overview

This lab documents the implementation and verification of **Layer 2 link aggregation using LACP (802.3ad)** over trunk links, together with **Port Security** policies on user access ports of Cisco Catalyst switches.

It includes a **Layer 2 troubleshooting** section covering common configuration errors, the requirement for a static access mode, and the transition of the port security status from `Disabled / Secure-down` to `Secure-up`.

---

## 2. Topology and Interface Table

### Network Diagram

```text
   [ PC0 ]                                                        [ PC1 ]
 192.168.10.2/24                                              192.168.10.3/24
      | Fa0                                                          | Fa0
      | Fa0/24 (Access VLAN 10)                  Fa0/24 (Access VLAN 10) |
 [ Switch1 ] ==== Gig0/1 ==== Port-Channel 1 (LACP Trunk) ==== Gig0/1 ==== [ Switch2 ]
             ==== Gig0/2 ====                               ==== Gig0/2 ====
```

### Interface Table

| Device | Interface | Port Mode | Assigned VLAN | Function / Description |
| :--- | :--- | :--- | :--- | :--- |
| **Switch1** | `Gig0/1 - Gig0/2` | EtherChannel (LACP) | Trunk (10, 20, 99) | Aggregated link Po1 toward SW2 |
| **Switch1** | `Po1` | Trunk | Allowed: 10,20,99 (Native 99) | Logical interface Port-Channel 1 |
| **Switch1** | `Fa0/24` | Static Access | VLAN 10 (`ADMIN`) | PC0 user port (Port Security enabled) |
| **Switch2** | `Gig0/1 - Gig0/2` | EtherChannel (LACP) | Trunk (10, 20, 99) | Aggregated link Po1 toward SW1 |
| **Switch2** | `Po1` | Trunk | Allowed: 10,20,99 (Native 99) | Logical interface Port-Channel 1 |
| **Switch2** | `Fa0/24` | Static Access | VLAN 10 (`ADMIN`) | PC1 user port |
| **PC0** | `Fa0` | Static / DHCP | VLAN 10 (`192.168.10.2/24`) | Test host |
| **PC1** | `Fa0` | Static / DHCP | VLAN 10 (`192.168.10.3/24`) | Test host |

---

## 3. Step-by-Step Configuration

### 3.1 VLAN Configuration on Both Switches (`SW1` and `SW2`)

```text
Switch1> enable
Switch1# configure terminal
Switch1(config)# vlan 10
Switch1(config-vlan)# name ADMIN
Switch1(config-vlan)# vlan 20
Switch1(config-vlan)# name Production
Switch1(config-vlan)# vlan 99
Switch1(config-vlan)# name NATIVE_VLAN
Switch1(config-vlan)# exit
```

### 3.2 EtherChannel Configuration with LACP (802.3ad)

The physical interfaces `Gig0/1` and `Gig0/2` are aggregated using the standard **LACP protocol in active mode**, and the logical `Port-Channel 1` interface is then converted into an 802.1Q trunk.

**On Switch1 and Switch2:**

```text
! 1. Bind the physical ports to the channel group
Switch(config)# interface range gigabitEthernet 0/1 - 2
Switch(config-if-range)# channel-group 1 mode active
Switch(config-if-range)# exit

! 2. Configure the trunking parameters on the logical Port-Channel interface
Switch(config)# interface port-channel 1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# switchport trunk allowed vlan 10,20,99
Switch(config-if)# exit
```

### 3.3 Access Port and Port Security Configuration on `SW1` (`Fa0/24`)

Static assignment of the user port to VLAN 10 and application of Layer 2 security policies:

```text
Switch1(config)# interface fastEthernet 0/24
Switch1(config-if)# switchport mode access
Switch1(config-if)# switchport access vlan 10

! Enable the Port Security master switch
Switch1(config-if)# switchport port-security

! Define the security policies
Switch1(config-if)# switchport port-security maximum 2
Switch1(config-if)# switchport port-security mac-address sticky
Switch1(config-if)# switchport port-security violation restrict
Switch1(config-if)# spanning-tree portfast
Switch1(config-if)# exit
```

---

## 4. Operational Verification

### 4.1 EtherChannel Status (`show etherchannel summary`)

Confirms that the port channel operates at Layer 2 (`S`), is in use (`U`), and bundles both physical interfaces (`P`) using the LACP protocol:

```text
Switch1# show etherchannel summary
Flags:  D - down        P - in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      f - failed to allocate aggregator

Group  Port-channel  Protocol    Ports
------+-------------+-----------+----------------------------------------------
1      Po1(SU)           LACP   Gig0/1(P) Gig0/2(P)
```

### 4.2 Trunk Status (`show interfaces trunk`)

Confirms that the logical `Po1` interface actively carries the VLANs in the *Forwarding* state:

```text
Switch1# show interfaces trunk
Port        Mode         Encapsulation  Status        Native vlan
Po1         on           802.1q         trunking      99

Port        Vlans allowed on trunk
Po1         10,20,99

Port        Vlans in spanning tree forwarding state and not pruned
Po1         10
```

### 4.3 Port Security Status (`show port-security interface fa0/24`)

Validates that port security is enabled and actively protecting the port (`Secure-up`):

```text
Switch1# show port-security interface fastEthernet 0/24
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Restrict
Aging Time                 : 0 mins
Aging Type                 : Absolute
SecureStatic Address Aging : Disabled
Maximum MAC Addresses      : 2
Total MAC Addresses        : 1
Configured MAC Addresses   : 0
Sticky MAC Addresses       : 1
Last Source Address:Vlan   : 0001.C7D2.A1B2:10
Security Violation Count   : 0
```

---

## 5. Troubleshooting Case Studies

### Problem 1: Port Security Command Rejected (Dynamic Port Error)

**Symptom:**
When trying to run the security command on the access interface, Cisco IOS rejects it with the following error:

```text
Switch(config-if)# switchport port-security
Command rejected: FastEthernet0/24 is a dynamic port.
```

**Root cause:**
The port is negotiating its operating mode through DTP (*Dynamic Trunking Protocol*) in `dynamic auto` or `dynamic desirable` state. Cisco IOS does not allow *Port Security* on ports that do not have a fixed, static role declared.

**Solution:**
Explicitly force the port role to static access before enabling security:

```text
Switch(config)# interface fastEthernet 0/24
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
```

---

### Problem 2: Security Status Shows `Disabled` or `Secure-down`

**Symptom:**
When checking the interface with `show port-security interface fa0/24`, security appears inactive even though the maximum limits and violation rules were configured:

```text
Port Security              : Disabled
Port Status                : Secure-down
```

**Root cause:**
Two possible causes were identified:

1. **Missing master command:** subcommands such as `switchport port-security maximum` or `violation` were entered, but the base command `switchport port-security`, which activates the security engine on the interface, was omitted.
2. **No active host on the port:** `Secure-down` indicates that the secured port has no active link. Once the host is connected and sends its first frame, the switch learns the MAC address and the status changes to `Secure-up`.

**Solution:**

1. Explicitly enable the master command on the interface:

   ```text
   Switch(config-if)# switchport port-security
   ```

2. Generate traffic from the host (for example, a `ping` to another IP in the VLAN) so the switch learns the MAC address and the operational status changes to **`Secure-up`**.

---

### Problem 3: Connectivity Failure (Ping) Between Hosts in the Same VLAN

**Symptom:**
`ping` between `PC0` and `PC1` results in total packet loss (`Request timed out`), even though the EtherChannel shows `Po1(SU)`.

**Root cause:**
The VLANs were not explicitly created in the VLAN database (`vlan.dat`) of one or both switches, or the access ports were not assigned to the corresponding VLAN, causing tagged frames to be dropped when crossing the trunk.

**Solution:**
Check the database with `show vlan brief`, create the VLAN on both ends, and formally assign it to the access port:

```text
Switch(config)# vlan 10
Switch(config-vlan)# name ADMIN
Switch(config)# interface fa0/24
Switch(config-if)# switchport access vlan 10
```

---

## 6. Key Takeaways

1. **Port Security requires a static port:**
   A port must be explicitly configured as `switchport mode access` (or a static trunk) before applying *Port Security*. With a static mode set, the port no longer negotiates through DTP.

2. **Violation modes:**
   - **`Protect`:** silently drops frames from unauthorized MAC addresses. It does not increment counters or send alerts.
   - **`Restrict`:** drops unauthorized frames, increments the *Security Violation Count*, and generates a Syslog/SNMP message.
   - **`Shutdown` (default):** immediately places the interface in the `err-disabled` state, administratively shutting down the port.

3. **Port Security status:**
   The security engine needs an active link and traffic from the host to move from `Secure-down` to `Secure-up` and to register MAC addresses under the *Sticky* mechanism.

4. **EtherChannel consistency:**
   Both ends of an LACP bundle must use compatible modes (`active`/`active` or `active`/`passive`), and the trunk settings (native VLAN and allowed VLANs) must match on both switches.
