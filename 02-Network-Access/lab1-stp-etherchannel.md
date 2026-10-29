# Laboratory 03: Spanning Tree Protocol (STP) and EtherChannel (LACP)
**Author:** Rafael Andrés Ramírez Benavides  
**Role:** Junior Network Engineer

## 📋 General Information
* **Objective:** Configure and verify EtherChannel (LACP - IEEE 802.3ad) between access and distribution layer switches, implementing Spanning Tree convergence (Rapid PVST+) and port security mechanisms (PortFast, BPDU Guard) to prevent Layer 2 loops.
* **Technologies/Protocols:** LACP (Link Aggregation Control Protocol), IEEE 802.1w (Rapid PVST+), PortFast, BPDU Guard, Cisco IOS CLI.
* **Topology:** Aggregated trunk connection between Distribution Switch (DWS1) and Access Switches (ASW1 / ASW2).

---

## 🛠️ 1. Topology Diagram and Requirements
* Trunk link aggregation using two physical interfaces per link (e.g., Gi0/1 and Gi0/2) grouped into a logical channel (`Port-channel 1`).
* LACP configuration in active mode (`channel-group 1 mode active`).
* Enabling Rapid PVST+ as the active spanning tree protocol.
* Edge port protection using PortFast and BPDU Guard against accidental loops caused by unauthorized switches.

---

## ⚙️ 2. Key Configuration (Cisco IOS CLI)

### EtherChannel (LACP) Configuration on the Switch
```cisco
Switch(config)# interface range GigabitEthernet0/1 - 2
Switch(config-if-range)# channel-group 1 mode active
Switch(config-if-range)# no shutdown
Switch(config-if-range)# exit

Switch(config)# interface Port-channel 1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 99
Switch(config-if)# description === EtherChannel Trunk Link to DWS1 ===
Verifying Port-Channel Status
Cisco CLI
Switch# show etherchannel summary
Key evidence: The protocol should display as LACP and the channel status as SU (Layer 2, Up in use).

🔍 3. Spanning Tree Protocol (STP) Verification
To check Root Bridge selection and the status of aggregated interfaces:

Cisco CLI
Switch# show spanning-tree vlan 10
Key evidence: Verify that the Port-Channel ports are in the FWRD (Forwarding) state and that paths toward the root bridge are optimal.

🔒 4. Layer 2 Security (PortFast and BPDU Guard)
Cisco CLI
Switch(config)# interface range FastEthernet0/1 - 10
Switch(config-if-range)# spanning-tree portfast
Switch(config-if-range)# spanning-tree bpduguard enable

📈 5. Demonstrated Skills and Takeaways
High-availability link aggregation to double bandwidth and mitigate single points of failure.

Implementation of Rapid PVST+ for rapid convergence during link failures.

User port hardening via BPDU Guard to prevent topology attacks or misconfigurations.
