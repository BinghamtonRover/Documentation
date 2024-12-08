---
description: How the different layers, as described in the Architecture section, interact
---

# Network

Our structure adheres to perhaps the most important principle of programming: **abstraction**. Each device or program has one high-level responsibility, and each layer is run on a different platform, but no device needs to concern itself with the details of another in order to work.&#x20;

For example, the dashboard may need to drive the rover forward, but it should not need to know about the Drive subsystem. Rather it should send a command to the subsystems program which then forwards the command to the drive firmware. The dashboard does not know how to communicate with the drive firmware, and the subsystems program does not know how to drive.&#x20;

All software running in the same layer (ie, environment) can communicate on the same protocol. Each layer only needs to communicate with the layer immediately before it.&#x20;

## Firmware to Onboard Computers: USB (Serial)

We use standard serial over USB to communicate with our firmware. We used to use CAN, but switching has made integration with the electrical teams way simpler, and released us from harsh limits on message sizes.&#x20;

All firmware on the rover is connected to the device running the Subsystems Program (called the **Subsystems Computer**). While this doesn't allow for a bus setup like CAN does, it simplifies our network layout by forcing us to consider simple one-to-one connections. We've found this adequately supports our use case without limiting our functionality.&#x20;

## Onboard Computers to the Dashboard: UDP (Ethernet/Wi-Fi)

We use the [UDP](https://en.wikipedia.org/wiki/User_Datagram_Protocol) standard to send packets of data across an Ethernet or wireless connection between the onboard computers and the dashboard. Each device is assigned an IP address, and each individual socket is assigned a port. A program may use multiple sockets. Each packet is then routed to a combination of an IP address and a port.&#x20;

The onboard computers are connected via Ethernet to each other and to the antenna, which bridges the connection wirelessly to the base station, where the user's PC (running the dashboard) is connected via Ethernet to the base station antenna. In this way, you can think of the antennas as one long Ethernet cable.

More information can be found in the [udp](../network/udp/ "mention") document.
