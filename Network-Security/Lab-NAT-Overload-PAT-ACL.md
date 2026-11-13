# Network Engineering Portfolio | CCNA 200-301

## Lab: NAT Overload (PAT) and Traffic Filtering with Extended ACLs

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

This lab documents the implementation, verification, and troubleshooting of **Network Address Translation with Overload (NAT Overload / PAT)** together with access control policies using **standard and extended ACLs** in a Cisco IOS environment.

The scenario simulates an enterprise edge router (`BorderR`) that provides Internet access to a private local network (`192.168.10.0/24`), while blocking web (HTTP) access from one specific host to an external server.

---

## 2. Topology and Interface Table

### Network Diagram

![NAT and ACL lab topology](Topologia-NAT-ACL.png)

```text
 [ PCLAN ] ---- Fa0/24 [ Switch ] Gig0/1 ---- Gig0/1 [ BorderR ] Gig0/0 ---- Fa0 [ Server-EXTERNO ]
192.168.10.50                                    192.168.10.1 | 203.0.113.1           203.0.113.100
                  <------- LAN 192.168.10.0/24 ------->      <---- WAN 203.0.113.0/24 ---->
                              (NAT inside)                          (NAT outside)
```

### Interface Table

| Device | Interface | IP Address | Subnet Mask | NAT Role / Function |
| :--- | :--- | :--- | :--- | :--- |
| **BorderR** | `GigabitEthernet0/1` | `192.168.10.1` | `255.255.255.0` (`/24`) | **NAT Inside** / LAN gateway |
| **BorderR** | `GigabitEthernet0/0` | `203.0.113.1` | `255.255.255.0` (`/24`) | **NAT Outside** / External (WAN) link |
| **PCLAN** | `FastEthernet0` | `192.168.10.50` | `255.255.255.0` (`/24`) | Internal host (Gateway: `192.168.10.1`) |
| **Server-EXTERNO** | `FastEthernet0` | `203.0.113.100` | `255.255.255.0` (`/24`) | External web / ICMP server (Gateway: `203.0.113.1`) |

---

## 3. Step-by-Step Configuration

### 3.1 Interface Configuration and NAT Roles

Definition of the IP addressing on the physical interfaces and assignment of the logical `inside` (private network) and `outside` (public network) zones.

```text
BorderR> enable
BorderR# configure terminal

! Interface toward the Local Network (LAN)
BorderR(config)# interface gigabitEthernet 0/1
BorderR(config-if)# description LINK_TO_LAN
BorderR(config-if)# ip address 192.168.10.1 255.255.255.0
BorderR(config-if)# ip nat inside
BorderR(config-if)# no shutdown
BorderR(config-if)# exit

! Interface toward the External Server / ISP (WAN)
BorderR(config)# interface gigabitEthernet 0/0
BorderR(config-if)# description LINK_TO_EXTERNAL
BorderR(config-if)# ip address 203.0.113.1 255.255.255.0
BorderR(config-if)# ip nat outside
BorderR(config-if)# no shutdown
BorderR(config-if)# exit
```

### 3.2 NAT Overload (PAT) Configuration

A **standard ACL** (`access-list 1`) identifies the private network segment that is allowed to be translated. It is then bound to the outside interface with the `overload` keyword.

```text
! 1. Define the internal traffic eligible for translation
BorderR(config)# access-list 1 permit 192.168.10.0 0.0.0.255

! 2. Enable PAT using the IP address of the outside interface Gig0/0
BorderR(config)# ip nat inside source list 1 interface gigabitEthernet 0/0 overload
```

### 3.3 Security Policy: Blocking HTTP with an Extended ACL

The named extended ACL `FILTRO-SEGURIDAD` denies only web traffic (TCP port 80 / `www`) from host `192.168.10.50` to server `203.0.113.100`, while permitting all other traffic (`icmp`, `https`, etc.).

```text
BorderR(config)# ip access-list extended FILTRO-SEGURIDAD
BorderR(config-ext-nacl)# deny tcp host 192.168.10.50 host 203.0.113.100 eq www
BorderR(config-ext-nacl)# permit ip any any
BorderR(config-ext-nacl)# exit

! Apply the ACL inbound on the LAN interface
BorderR(config)# interface gigabitEthernet 0/1
BorderR(config-if)# ip access-group FILTRO-SEGURIDAD in
BorderR(config-if)# exit
```

---

## 4. Operational Verification

### 4.1 NAT Roles and Statistics (`show ip nat statistics`)

Confirms that the `inside` and `outside` interfaces match the physical topology and that dynamic translations are being processed.

```text
BorderR# show ip nat statistics
Total translations: 4 (0 static, 4 dynamic, 4 extended)
Outside Interfaces: GigabitEthernet0/0
Inside Interfaces: GigabitEthernet0/1
Hits: 7  Misses: 8
Expired translations: 4
```

### 4.2 Active Translation Table (`show ip nat translations`)

Validates that ICMP/TCP requests from the private IP `192.168.10.50` are dynamically translated to the public IP `203.0.113.1`, differentiated by port number / identifier:

```text
BorderR# show ip nat translations
Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.1:17    192.168.10.50:17   203.0.113.100:17   203.0.113.100:17
icmp 203.0.113.1:18    192.168.10.50:18   203.0.113.100:18   203.0.113.100:18
```

### 4.3 Security Filter Validation (`show access-lists FILTRO-SEGURIDAD`)

After trying to open `http://203.0.113.100` in the browser from `PCLAN` (failed) and running a `ping` (successful), the match counters confirm that the rule blocks port 80 without affecting ICMP connectivity:

```text
BorderR# show access-lists FILTRO-SEGURIDAD
Extended IP access list FILTRO-SEGURIDAD
    10 deny tcp host 192.168.10.50 host 203.0.113.100 eq www (12 match(es))
    20 permit ip any any (16 match(es))
```

---

## 5. Troubleshooting Case Studies

### Problem 1: Subnet Mask Error on the External Server

**Symptom:**
When configuring IP `203.0.113.100` with a `/30` mask (`255.255.255.252`), the system returns the error `Invalid IP for this subnet mask entered`.

**Root cause:**
A `/30` mask splits the network into blocks of 4 addresses (`0-3`, `4-7`, and so on). In a `/30` scheme, `.100` is a **network address** (the block `203.0.113.100` to `.103`), so it cannot be assigned to a host, and it does not belong to the `203.0.113.0/30` segment where the router interface sits.

**Solution:**
Set the network prefix on both `BorderR` (`Gig0/0`) and `Server-EXTERNO` to `/24` (`255.255.255.0`), making both addresses valid within the same broadcast domain.

---

### Problem 2: `Destination host unreachable` from the LAN

**Symptom:**
Running `ping 203.0.113.100` from `PCLAN` returns `Reply from 192.168.10.1: Destination host unreachable`.

**Root cause:**
The router had no direct connectivity to the server's subnet and no default static route (`0.0.0.0/0`) toward the external network.

**Solution:**
Make sure the outside interface (`Gig0/0`) is up (`no shutdown`) and in the same subnet as the server, or add a default route (`ip route 0.0.0.0 0.0.0.0 <Next_Hop_IP>`).

---

### Problem 3: Inverted NAT `Inside` and `Outside` Roles

**Symptom:**
`show ip nat statistics` showed the LAN interface as `Outside` and the WAN interface as `Inside`.

**Root cause:**
The `ip nat inside` and `ip nat outside` commands were assigned the wrong way around with respect to the physical topology.

**Solution:**
Explicitly correct each interface according to its physical connection (`Gig0/1` as `inside` toward the switch and `Gig0/0` as `outside` toward the external server), then clear the previous sessions with `clear ip nat translation *`.

---

## 6. Key Takeaways

1. **Order of operations (inbound ACL vs. NAT):**
   When an ACL is applied inbound (`in`) on the `inside` interface, the router evaluates the packet **before** NAT translation. The extended ACL must therefore reference the real private source IP (`192.168.10.50`), not the translated public IP.

2. **Extended ACL placement:**
   Following Cisco best practices, extended ACLs should be placed as **close to the source** of the traffic as possible (`Gig0/1` inbound), so unwanted packets are dropped before consuming routing or NAT processing.

3. **Final explicit permit:**
   Every ACL ends with an implicit `deny ip any any`. Omitting the `permit ip any any` statement would block all legitimate traffic (such as ICMP or DNS) from the local network.

4. **A NAT ACL is not a security filter:**
   The standard `access-list 1` only defines *which* addresses are eligible for translation. Blocking or permitting traffic is the job of a separate ACL applied to an interface, such as `FILTRO-SEGURIDAD`.
