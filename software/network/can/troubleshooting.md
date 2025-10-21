# CAN Troubleshooting

While CAN is a relatively simple protocol, there are a lot of places for things to go wrong.

## No messages are being received

### Check Wiring

Check the following parts of your wiring:
1. All devices are plugged in
2. Measure the resistance across CAN High and CAN Low (do not do this while the bus is powered on), the resistance should be 60 Ohms, if it is not, make sure there is a 120 Ohm resistor only on the start and end of the bus
3. CAN H and CAN L are consistent. By convention, yellow wires are CAN H, and green wires are CAN L

### Check Your Filters

First, check whether the IDs of the messages you're trying to receive are being blocked by a filter. For the Teensy devices, this is done through the Mailbox configuration.

### Bus Bitrate

For all devices on the bus, the bitrate they are reading/writing to have to match. The 2 common bitrates used on the rover are 1,000,000 bits/sec and 500,000 bits/sec. If even one device doesn't match the bitrate, it can cause issues across the entire bus.
