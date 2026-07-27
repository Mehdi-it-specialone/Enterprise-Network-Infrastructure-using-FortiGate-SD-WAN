# Internet Access Test

## Objective

Verify that devices from different VLANs can successfully access public Internet resources through the FortiGate SD-WAN firewall using the configured outbound NAT policy.

## Test Scenario

* **Device 1:** PC-IT (VLAN 20 – IT Department)
* **Device 2:** PC-HR (VLAN 30 – HR Department)

The following public websites were tested:

* https://www.google.com
* https://www.facebook.com
* https://www.youtube.com
* https://www.yahoo.com
* https://aws.amazon.com

## Procedure

1. Verify that the FortiGate SD-WAN members are operational.
2. Confirm that the default route points to the SD-WAN zone.
3. From **PC-IT** and **PC-HR**, open a web browser.
4. Access each of the listed public websites.
5. Verify that all websites load successfully.

## Expected Result

* Both devices have Internet connectivity.
* All tested websites are accessible.
* Traffic is translated through the FortiGate firewall using NAT.
* Internet traffic is forwarded through the active SD-WAN WAN member.

## Test Result

**Status:** ✅ Passed

Both workstations successfully accessed all tested public websites, confirming that Internet connectivity, NAT, default routing, and SD-WAN forwarding are operating correctly across multiple VLANs.

## Evidence
![Internet Access Test](../screenshots/Internet_Connectivity.png)

*Figure 3 – Successful Internet access from PC-IT and PC-HR to multiple public websites through the FortiGate SD-WAN firewall.*
