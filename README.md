Design and Implementation of a Highly Available Enterprise Network Infrastructure using FortiGate SD-WAN, Site-to-Site IPsec VPN, VLAN Segmentation and Cisco Layer 3 Switching in EVE-NG

## Overview

This repository contains our Bachelor's Final Year Project developed for the Systems and Network Engineering program.

The project consists of designing, implementing, and validating a highly available enterprise network infrastructure connecting a Headquarters (HQ) and a Branch office using FortiGate SD-WAN and Site-to-Site IPsec VPN.

The complete infrastructure was designed and tested in the EVE-NG network simulator.

---

## Project Objectives

- Build a multi-site enterprise network
- Provide high availability using dual Internet providers
- Implement SD-WAN failover
- Secure communications with Site-to-Site IPsec VPN
- Segment the network using VLANs
- Configure Layer 3 switching
- Validate routing and connectivity

---

## Technologies

- FortiGate Firewall
- SD-WAN
- IPsec VPN
- Cisco IOS
- Layer 3 Switching
- VLAN
- Static Routing
- High Availability
- EVE-NG

---

## Network Architecture

- Headquarters
- Branch Office
- Dual ISP Connectivity
- SD-WAN
- IPsec VPN
- Layer 3 Core Switches
- Access Switches
- Enterprise VLANs

---

## Addressing Plan

### Headquarters

| VLAN | Network |
|-------|---------|
| Servers | 10.10.10.0/24 |
| IT | 10.10.20.0/24 |
| HR | 10.10.30.0/24 |
| Sales | 10.10.40.0/24 |
| Management | 192.168.99.0/24 |
| Transit | 10.255.255.0/30 |

### Branch

| VLAN | Network |
|-------|---------|
| Users | 10.20.20.0/24 |
| Management | 192.168.199.0/24 |
| Transit | 10.255.254.0/30 |

---

## Features

- SD-WAN Failover
- Site-to-Site IPsec VPN
- VLAN Segmentation
- Inter-VLAN Routing
- Static Routing
- Enterprise Addressing
- High Availability
- Internet Redundancy

---

## Validation Tests

- VLAN Communication
- Inter-VLAN Routing
- Internet Access
- Site-to-Site VPN
- Routing Table Verification
- SD-WAN Dashboard
- HQ ↔ Branch Connectivity
- Traceroute
- ISP Failover
- ISP Recovery

---

## Repository Structure

```
report/
presentation/
topology/
addressing/
configurations/
screenshots/
tests/
```

---

## Author

- El Mahdi Zahir

Bachelor's Final Year Project

Systems & Network Engineering

Sup Management
