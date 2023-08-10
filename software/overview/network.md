---
description: How the different layers, as described in the Architecture section, interact
---

# Network

Our structure adheres to perhaps the most important principle of programming: **abstraction**. Each device or program has one high-level responsibility, and each layer is run on a different platform, but no device needs to concern itself with the details of another in order to work.&#x20;

For example, the dashboard may need to drive the rover forward, but it should not need to know about the Drive subsystem. Rather it should send a command to the subsystems program which then forwards the command to the drive firmware. The dashboard does not know how to communicate with the drive firmware, and the subsystems program does not know how to drive.&#x20;

All software running in the same layer (ie, environment) can communicate on the same protocol. Each layer only needs to communicate with the layer immediately before it.&#x20;

## Firmware to Onboard Computers: CAN bus

Each Teensy device is connected to a central [CAN bus](https://en.wikipedia.org/wiki/CAN\_bus) that ultimately connects to the Subsystems computer. We use the standard CAN protocol which only supports 8 bytes per packet but we are experimenting with CAN FD support which will let us send up to 64. Each CAN packet has an ID that identifies which device it is sent to.&#x20;

Normally, a CAN ID is also used to identify the purpose of the packet, since 8 bytes wouldn't be enough, but we actually don't need to use it for that, since our Protobuf data can always serialize to less than 8 bytes. Instead, we just use IDs to identify the devices themselves, as well as the direction of the data flow (Pi -> Teensy vs Teensy -> Pi)

More information can be found in the [can-bus](../network-details/can-bus/ "mention") document.

## Onboard Computers to the Dashboard: UDP

We use the [UDP](https://en.wikipedia.org/wiki/User\_Datagram\_Protocol) standard to send packets of data across an Ethernet or wireless connection between the onboard computers and the dashboard. Each device is assigned an IP address, and each individual socket is assigned a port. A program may use multiple sockets. Each packet is then routed to a combination of an IP address and a port.&#x20;

The onboard computers are connected via Ethernet to each other and to the antenna, which bridges the connection wirelessly to the base station, where the user's PC (running the dashboard) is connected via Ethernet to the base station antenna. In this way, you can think of the antennas as one long Ethernet cable.

More information can be found in the [udp](../network-details/udp/ "mention") document.
