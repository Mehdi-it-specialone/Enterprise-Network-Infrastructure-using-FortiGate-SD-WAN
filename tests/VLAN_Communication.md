# VLAN Communication Test

## Objective

Verify that devices connected to the same VLAN can communicate successfully through the access layer without requiring Layer 3 routing.

## Test Scenario

* Source Device: PC-IT-01
* Destination Device: PC-IT-02
* VLAN: VLAN 20 (IT Department)

## Procedure

1. Deploy two end devices connected to access switch ports assigned to VLAN 30.
2. Verify that both devices receive IP addresses belonging to the same subnet.
3. From PC-HR, execute an ICMP ping to PC-Assistant-HR.
4. Observe the connectivity results.

## Expected Result

* Both devices communicate successfully.
* ICMP replies are received.
* Packet loss equals **0%**.

## Test Result

**Status:** ✅ Passed

The communication between hosts belonging to VLAN 20 was successfully established, confirming that Layer 2 switching and VLAN segmentation were correctly configured.

## Evidence

> Insert the screenshot showing the successful ping between both devices.

Example image:

screenshots/VLAN_Communication.png
