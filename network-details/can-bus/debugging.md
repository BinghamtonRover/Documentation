# Debugging

While CAN works most of the time, there is a lot that can go wrong. Thankfully, most have simple fixes. Since both the Pis and the Teensy boards work with CAN, you may need to debug both. It is almost always simplest to hook up the subsystems Pi to the CAN bus and test there. Since all devices are connected to the same bus, try to connect one device at a time to isolate a known "good" device and test it against a malfunctioning device.

## Basic checklist

* Check your connections and ensure that the CAN\_HIGH and CAN\_LOW pins are properly connected. Run continuity tests to ensure that the CAN wire itself is working.
* Make sure at least two devices are powered on and connected to the bus. The CAN protocol requires devices to acknowledge messages and for senders to listen for those acknowledgments, so if no device is connected, the sending device assumes it is malfunctioning and may even shut down its CAN interface.
* Make sure the CAN bus is terminated on both "ends" of the bus, with a 120Ω resistor on each end. It can be tricky to decide what the "ends" of the bus may be, so you may need to fiddle with it. Objectively, the resistance between the CAN\_HIGH and CAN\_LOW pins should be 60Ω.
* When a device is disconnected from the Pi while the Pi is running, it may bug out the CAN chip. Run `ip set can0 down` and `ip set can0 up` to reset it.
* The CAN protocol only supports data up to 8 bytes. CAN FD supports up to 64 but must be explicitly enabled on all devices

## Debugging on the Pi

Linux has a great tool called `can-utils` which offers command-line support for reading and writing to the CAN bus. Simply open two terminals on the Pi and run `candump can0` on one and `cansend can0#deadbeef` on the other. You should see the message (`0xdeadbeef)` appear on the `candump` terminal. In general, when testing other devices, running `candump can0` will show all the data that appears on the bus. If no data appears, then the bus is empty or the Pi is not properly connected.

Verify that the `can` device has indeed been set up by running `ip link show`. If `can0` does not appear, follow the instructions at [setting-up-can-on-a-pi.md](setting-up-can-on-a-pi.md "mention").

## Debugging on the Teensy

If the Pi is able to send and receive messages to another device but not to a Teensy, then the issue is probably on the Teensy side. The issue may be:&#x20;

* The CAN transceiver chip isn't properly connected to the board
* The CAN bus may not be properly terminated. Adding new devices to the bus can change the resistance between the HIGH and LOW pins, so measure the resistance and verify that it is 60Ω. If not, add or move 120Ω resistors until it is
* Check your IDs. The firmware is configured to filter out all messages _except_ those with a given ID, so be sure your messages from `cansend` are addressed to the right ID.

Unfortunately, the Teensy and firmware don't offer convenient tools to debug the CAN connection. You must look into the [firmware-utilities.md](../../the-firmware/firmware-utilities.md "mention") package and its implementation for further debugging, but it has been rigorously tested and it is unlikely that the is there.
