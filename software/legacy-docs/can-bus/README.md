---
description: Communication between the firmware
---

# CAN bus

## Overview

We use the CAN bus protocol to send and receive data across the firmware devices. CAN is a relatively simple protocol that offers the following advantages:&#x20;

* All data is shared along one bus, and all devices are connected to the bus. This simplifies the wiring as all communications occur along just two wires, and each firmware device can be connected in the same way
* Devices can set a filter to only handle messages with a given ID. An ID can therefore be used as not only a way to address a device but also to identify the purpose of a given packet
* CAN is extremely resilient against noisy and corrupted data. Devices on the bus can automatically detect corrupted packets as well as ensure that they send valid packets

Since we use Protobuf, all our data can fit within the small 8-byte limit imposed by CAN, and with CAN FD we can use 64. Instead of using IDs to just identify devices, we also encode the _direction_ of the data.&#x20;

## CAN IDs

Each Teensy can send data or receive commands. Even though all data goes through the Subsystems computer, by assigning IDs based on the Teensy they came from, we can automatically ascertain what _type_ of data the subsystem computer has received.&#x20;

| Device           | Send data | Receive command |
| ---------------- | --------- | --------------- |
| Electrical Board | `0x13`    | `0x63`          |
| Drive Board      | `0x14`    | `0x53`          |
| Arm Board        | `0x15`    | `0x23`          |
| Gripper Board    | `0x16`    | `0x33`          |
| Science Board    | `0x17`    | `0x43`          |

Note that CAN IDs are priority-based: lower IDs receive higher priority. For this reason, we reserve the first 16 IDs for future use, like a heartbeat mechanism. The IDs are otherwise "scattered" for historical reasons, and should be updated.

Because CAN uses a bus, all devices on the bus receive the same electrical signals, and by extension, packets. These IDs are used to tell each device which CAN messages to handle.

## CAN Frames

Each message comes in a packet called a [CAN frame](https://docs.kernel.org/networking/can.html#how-to-use-socketcan). The specific API can be found there, but we only use a frame's ID, payload, and content length. The length is especially important to pay attention to -- each CAN frame arrives with either 8 or 64 bytes, even if not all of those bytes were used when sending. These extra bytes are set to zero, which can cause issues for encodings like Protobuf where zeroes are valid parts of the data. The `length` field tells you how many bytes were actually sent so you can only use those parts of the data buffer.

## More Resources

* [The Wikipedia page](https://en.wikipedia.org/wiki/CAN\_bus) for the CAN bus protocol
* [The Linux SocketCAN](https://docs.kernel.org/networking/can.html) library and Spec
* [CSS Electronic's Guide ](https://www.csselectronics.com/pages/can-bus-simple-intro-tutorial)to CAN and [CAN FD](https://www.csselectronics.com/pages/can-fd-flexible-data-rate-intro)
