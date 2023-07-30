# UDP

To simplify network setup, the computers in the rover use static IP addresses -- that is, hard-coded and unchanging. The user's PC, on the other hand, is allowed to connect via [DHCP](https://en.wikipedia.org/wiki/Dynamic\_Host\_Configuration\_Protocol), which allows the router to assign the device an IP address so users don't have to. This means that, while the dashboard always knows the IP addresses of the onboard computers, they do not know the IP address of the dashboard.&#x20;

To get around this, we use a simple "handshake" protocol. The dashboard sends a `Connect()` message to each onboard computer (the "server"). When a server receives a `Connect()` message, it saves the IP address of the packet.&#x20;
