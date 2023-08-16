---
description: Communication between the onboard computers
---

# UDP

UDP stands for the User Datagram Protocol and represents a higher-level communications protocol compared to CAN and Serial. UDP is built on the Ethernet/IP layer, so its logic should be very familiar to computer users.&#x20;

## Overview

In UDP, each device is assigned an IP address, and each device may open multiple UDP sockets, each with its own port number. Messages are then sent as packets or groups of packets addressed to a socket on a given address and port. Similar to CAN IDs, a socket number may be used to signify the purpose of the message, but these are less flexible since each socket can only interface with one port.

In our systems, we run multiple programs on different devices. In theory, this means we could reuse port numbers across devices. However, when testing, we would like the flexibility to run them on the same device as well. To ensure two programs don't try to access the same ports, we have assigned each program a unique port number, regardless of which device it is run on.

<table><thead><tr><th>Device/Program</th><th>IP address</th><th width="117">Port</th><th>Purpose</th></tr></thead><tbody><tr><td>Subsystems</td><td>192.168.1.20</td><td>8001</td><td>Communicate with Dashboard</td></tr><tr><td>Video</td><td>192.168.1.30</td><td>8002</td><td>Sends metadata about cameras and receives commands</td></tr><tr><td>Video </td><td>192.168.1.30</td><td>8003</td><td>Sends video frames</td></tr><tr><td>Autonomy</td><td>192.168.1.30</td><td>8004</td><td>Communicates with the dashboard</td></tr><tr><td>Autonomy</td><td>192.168.1.30</td><td>8005</td><td>Communicates with the subsystems</td></tr></tbody></table>

Because each program has its own port, they become _modular_ -- any other program running on the same ports could be used instead. For example, this allowed us to switch our Python implementation with Dart code one program at a time, where the new Dart code could still interface with the old Python code using the above sockets.

This also allows us to make our testing setup more modular. For example, we use a mini rover, dubbed the Tank, to test our autonomy platform. To get the tank to drive instead of the rover, we simply run a modified subsystems program on port 8001 to mimic the subsystems computer.

## UDP vs TCP

UDP isn't the only protocol that works over the IP layer. TCP is a very common alternative to UDP. They both use packets, IP addresses, and port numbers. The main difference between the two is that TCP monitors its connection and the status of its packets whereas UDP does not. This allows TCP to guarantee that packets are delivered and received in the order they were sent, with intelligent error handling along the way.&#x20;

UDP on the other hand, sends packets out into the void with absolutely no promises on whether they were delivered, or whether any device was even available to receive them. This makes UDP much less reliable but reduces a lot of overhead which is great for reducing our traffic bandwidth. It also gives us much more flexibility in situations where the rover is out of range, which is a very real and common scenario with our setup.

In general, we always assume that any packet sent over the network - particularly over the wireless bridge -- may drop. However, to be practical, we assume that _some_ packets will eventually deliver unless the rover is really out of range. Therefore, care should be taken when sending important data or commands, and non-sensitive data or commands can be sent repeatedly or periodically to ensure it is eventually delivered.&#x20;

## Static vs Dynamic IPs

To simplify network setup, the computers in the rover use static IP addresses -- that is, hard-coded and unchanging. The user's PC, on the other hand, is allowed to connect via [DHCP](https://en.wikipedia.org/wiki/Dynamic\_Host\_Configuration\_Protocol), which allows the router to assign the device an IP address so users don't have to. We also allow the dashboard to choose random port numbers. This means that, while the dashboard always knows the addresses and ports of the onboard computers, the onboard computers do not know how to reach the dashboard's sockets.

To get around this, we use a simple "handshake" protocol. The dashboard sends a `Connect()` message to each onboard computer (the "server"). When a server receives a `Connect()` message, it saves the IP address of the packet. Then, when the server needs to send a message to the dashboard, it can refer to the saved IP address. The servers also echo this `Connect()` message back to the dashboard to confirm delivery. This handshake mechanism is discussed in more detail in [#heartbeats-and-handshakes](../protobuf/#heartbeats-and-handshakes "mention").

## More Resources

* [The Wikipedia page on UDP](https://en.wikipedia.org/wiki/User\_Datagram\_Protocol)
* [This article on UDP vs TCP](https://www.spiceworks.com/tech/networking/articles/tcp-vs-udp/)

