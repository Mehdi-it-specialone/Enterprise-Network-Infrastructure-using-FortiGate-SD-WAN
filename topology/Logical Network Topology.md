# Logical Network Topology

## Overview

The enterprise network is designed using a hierarchical architecture that separates the access, distribution, and core layers. The Headquarters (HQ) and Branch Office (BR) are interconnected through FortiGate firewalls using an IPsec Site-to-Site VPN.

The FortiGate firewalls also provide SD-WAN functionality, allowing the network to use two independent ISP connections with automatic failover.

---

## Logical Architecture

```text
                         ENTERPRISE USERS
                               │
                               │
                    ┌─────────────────────┐
                    │    ACCESS LAYER     │
                    │ Cisco Access Switch │
                    └──────────┬──────────┘
                               │
                               │ VLANs
                               │
                    ┌──────────▼──────────┐
                    │  DISTRIBUTION LAYER │
                    │ Cisco Distribution  │
                    │      Switches       │
                    └──────────┬──────────┘
                               │
                               │ Trunk
                               │
                    ┌──────────▼──────────┐
                    │      CORE-HQ        │
                    │  Cisco Layer 3 SW   │
                    │                     │
                    │ Inter-VLAN Routing  │
                    └──────────┬──────────┘
                               │
                         10.255.255.0/30
                               │
                               │
                    ┌──────────▼──────────┐
                    │   FORTIGATE HQ      │
                    │                     │
                    │ Firewall            │
                    │ NAT                 │
                    │ SD-WAN              │
                    │ IPsec VPN           │
                    └───────┬───────┬─────┘
                            │       │
                         ISP 1    ISP 2
                            │       │
                            └───┬───┘
                                │
                         ┌──────▼──────┐
                         │   CORE ISP   │
                         │              │
                         │ ISP Transit  │
                         └──────┬───────┘
                            │       │
                         ISP 1    ISP 2
                            │       │
                    ┌───────┴───────┴──────┐
                    │                      │
              ┌─────▼──────────────────────▼─────┐
              │          FORTIGATE BR             │
              │                                   │
              │ Firewall • NAT • SD-WAN • IPsec  │
              └────────────────┬──────────────────┘
                               │
                         10.255.254.0/30
                               │
                    ┌──────────▼──────────┐
                    │      CORE-BR        │
                    │   Layer 3 Switch    │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │    ACCESS LAYER     │
                    │   Branch Switches   │
                    └──────────┬──────────┘
                               │
                         BRANCH USERS
```

---

## Headquarters Architecture

The Headquarters network follows a three-layer hierarchical design:

* **Access Layer** – Provides connectivity to end-user devices.
* **Distribution Layer** – Aggregates the access switches and provides Layer 2 connectivity.
* **Core Layer** – Performs Layer 3 routing and Inter-VLAN routing.
* **FortiGate HQ** – Provides firewalling, NAT, SD-WAN and IPsec VPN services.

The Core Layer 3 switch is connected to the FortiGate through the dedicated HQ transit network:

```text
Core-HQ              FortiGate-HQ
10.255.255.2/30  ───  10.255.255.1/30
```

---

## Branch Office Architecture

The Branch Office uses a simplified architecture consisting of:

* FortiGate Branch firewall
* Layer 3 Core switch
* Access switches
* End-user devices

The Branch Core switch connects to the FortiGate through the dedicated Branch transit network:

```text
Core-BR              FortiGate-BR
10.255.254.2/30  ───  10.255.254.1/30
```

---

## VLAN Architecture

The enterprise network is segmented using dedicated VLANs to separate different departments and infrastructure services.

| VLAN     | Purpose                 | Network            |
| -------- | ----------------------- | ------------------ |
| VLAN 10  | Servers                 | 10.10.10.0/24      |
| VLAN 20  | IT                      | 10.10.20.0/24      |
| VLAN 30  | HR                      | 10.10.30.0/24      |
| VLAN 40  | Sales                   | 10.10.40.0/24      |
| VLAN 99  | Management              | 10.10.99.0/24      |
| VLAN 100 | Branch / Infrastructure | 10.20.100.0/24     |
| VLAN 999 | Native VLAN             | No user addressing |

The Core Layer 3 switch provides the SVI gateways and performs Inter-VLAN routing.

---

## WAN and SD-WAN Architecture

Both Headquarters and Branch Office FortiGate firewalls use two independent WAN connections:

```text
                 FORTIGATE
                 /       \
              ISP 1     ISP 2
                │         │
                └────┬────┘
                     │
                  SD-WAN
                     │
              Health Check
                     │
              Automatic Failover
```

ISP1 is configured as the preferred WAN connection, while ISP2 provides redundancy.

The SD-WAN health-check mechanism continuously monitors WAN connectivity. If the primary ISP becomes unavailable or fails to meet the configured SLA, traffic is automatically redirected through the secondary ISP.

---

## Core ISP Transit

A dedicated Core ISP router is used to interconnect the ISP1 and ISP2 paths between Headquarters and the Branch Office.

The Core ISP uses the following point-to-point networks:

| Connection | Network       | Core ISP Address |
| ---------- | ------------- | ---------------- |
| HQ ↔ ISP1  | 172.16.1.0/30 | 172.16.1.1       |
| HQ ↔ ISP2  | 172.16.2.0/30 | 172.16.2.1       |
| BR ↔ ISP1  | 172.16.3.0/30 | 172.16.3.1       |
| BR ↔ ISP2  | 172.16.4.0/30 | 172.16.4.1       |

These networks provide the simulated ISP infrastructure used by the EVE-NG laboratory environment.

---

## IPsec Site-to-Site VPN

The Headquarters and Branch FortiGate firewalls establish an IPsec Site-to-Site VPN across the WAN infrastructure.

```text
┌─────────────────┐                    ┌─────────────────┐
│   FORTIGATE HQ  │                    │  FORTIGATE BR   │
│                 │                    │                 │
│  SD-WAN         │====================│  SD-WAN         │
│  IPsec VPN      │      IPsec         │  IPsec VPN      │
└─────────────────┘      Tunnel        └─────────────────┘
        │                                      │
     HQ LAN                                  BR LAN
```

The VPN provides secure communication between the internal networks of both sites while allowing the SD-WAN infrastructure to provide WAN redundancy.

---

## Traffic Flow

### Internal Traffic

```text
User
  │
Access Switch
  │
Distribution
  │
Core L3 Switch
  │
Inter-VLAN Routing
  │
Destination VLAN
```

### Internet Traffic

```text
User
  │
Core L3 Switch
  │
FortiGate
  │
SD-WAN
  │
ISP 1 / ISP 2
  │
Internet
```

### Headquarters-to-Branch Traffic

```text
HQ User
   │
Core-HQ
   │
FortiGate-HQ
   │
SD-WAN
   │
IPsec VPN
   │
FortiGate-BR
   │
Core-BR
   │
Branch User
```

---

## Design Objectives

The logical topology was designed to provide:

* **Network segmentation** through VLANs.
* **Inter-VLAN routing** through the Core Layer 3 switches.
* **Firewall protection** through FortiGate.
* **Internet connectivity** through SD-WAN.
* **WAN redundancy** using two ISP connections.
* **Automatic ISP failover** through SD-WAN health checks.
* **Secure HQ-to-Branch communication** through IPsec Site-to-Site VPN.
* **Hierarchical network organization** using Access, Distribution and Core layers.
* **Centralized routing** at the Layer 3 Core.

---

## Laboratory Environment

The complete architecture was implemented and tested using the **EVE-NG network simulator**, allowing the enterprise topology, Cisco switching infrastructure, FortiGate firewalls, ISP infrastructure, SD-WAN and IPsec VPN to be reproduced in a controlled laboratory environment.

### Related Documentation

* `../addressing/network addressing plan 1.pdf` – IP addressing and VLAN plan
* `../configurations/` – Device configurations
* `../tests/` – Network validation and testing
* `../report/` – Complete project report
