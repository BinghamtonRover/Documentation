---
description: How the different layers, as described in the Architecture section, interact
---

# Network

Our structure adheres to perhaps the most important principle of programming: **abstraction**. Each device or program has one high-level responsibility and delegates everything else to other programs. In particular, we standardize our communication on _protocols_, not implementation. No device needs to concern itself with the details of another to do its job.

For example, the dashboard may need to drive the rover forward, but it should not need to know about the Drive subsystem. Rather it should send a command to the subsystems program which then forwards the command to the drive firmware. The dashboard does not know how to communicate with the drive firmware, and the subsystems program does not know how to drive.&#x20;

Describing our programs in terms of communication rather than functionality allows us to take advantage of another important principle, **modularity. For** example, we often test Autonomy on a smaller version of the rover, the Tank. Had we intermixed responsibilities, Autonomy would need to change to accommodate the tank's drive system, but since we specify the `DriveCommand`payload, we made a new, modified version of Subsystems that receives the same `DriveCommand`data and controls the tank motors instead.&#x20;

Our protocols also helped us switch from Python to Dart. By ensuring our new programs worked with the same messages as the old ones, we could migrate incrementally and run in a mixed Python-Dart environment without modifying the Dashboard.

## Firmware to Onboard Computers: Serial (USB)

We use standard [serial](https://en.wikipedia.org/wiki/Serial_communication) over USB to communicate with our firmware. All firmware on the rover is connected to the device running the Subsystems Program (called the **Subsystems Computer**). While this doesn't allow for a bus setup like CAN, it simplifies our network layout by forcing us to consider simple one-to-one connections. We've found this adequately supports our use case without limiting our functionality.&#x20;

The Subsystems program represents **encapsulation** in that all communication with the firmware must happen through it. While this m

More details can be found in the [serial](../network/serial/ "mention") page.

## Onboard Computers to the Dashboard: UDP (Ethernet/Wi-Fi)

We use the [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol) standard to send packets across an Ethernet or wireless connection between the onboard computers and the dashboard. Each device is assigned an IP address, and each program is assigned a socket on a unique port. Each packet is then sent from one socket to another.&#x20;

The onboard computers are connected via Ethernet to each other and to the antenna, which bridges the connection wirelessly to the base station, where the user's PC (running the dashboard) is connected via Ethernet to the base station antenna. In this way, you can think of the antennas as one long Ethernet cable.

More information can be found in the [udp](../network/udp/ "mention") page.
