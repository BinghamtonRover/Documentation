# CAN bus

## Overview

CAN bus is a communication protocol commonly used in the automotive industry to communicate between different Electronics Control Units (ECUs). Before CAN was created, cars had thousands of meters of cable in them. CAN simplifies the wiring by allowing all devices to communicate with each other across a bus, significantly reducing the amount of wire needed. In addition, CAN has several other advantages:
* All data is shared along one bus, and all devices are connected to the bus. This simplifies the wiring as all communications occur along just two wires, and can be chained together in series
* CAN messages are sent as fixed-size packets, making it much easier to decode data
* CAN messages have a built-in checksum, making them very reliable under noisy conditions

## CAN IDs

All CAN messages are self-identifying, meaning when you receive a message, you will also receive information about what the message is or where it came from. For CAN 2.0, this ID is an 11 bit integer, and for CAN 2.0b, this is a 29 bit integer. Due to how few devices are on our network, we only use an 11 bit integer.

<details>

<summary>Additional Information About IDs</summary>

{% hint style="info" %} You will not need to know this for the Rover, however this may be interesting to learn. {% endhint %}

For most CAN networks, such as ones seen in modern vehicles, a 29 bit ID is used. While the 29 bits are technically just an integer, since there's a lot of data in it, it's used as a "canvas" for information. For example, the first 5 bits can be used as a type ID, which represents the type of data coming through (request, error, data, etc), the next 7 bits could represent a parameter ID, etc.

On the rover, we have our frame ID represents a number corresponding to a message type, but for more advanced networks with lots of devices, 29 bit IDs will be used as a blank canvas.

For examples of this, see the [J1939 Protocol](https://www.csselectronics.com/pages/j1939-explained-simple-intro-tutorial) and the [FRC CAN Bus Format](https://docs.wpilib.org/en/stable/docs/software/can-devices/can-addressing.html)

</details>

When CAN messages are sent over the bus, the messages with the lower ID are given a greater priority. For this reason, heartbeats and device broadcasts are given the lowest IDs, and drive and relay commands come right after, as these are the most important messages to be sent over our network.

{% hint style="info" %} Because of how message ID priority works, 2 separate devices cannot transmit a message with the same ID at the same time, otherwise, the bus may encounter an error. For more information, see the [arbitration process](https://electronics.stackexchange.com/questions/43139/transmission-of-different-messages-with-the-same-id-on-a-can-bus/43150#43150). {% endhint %}

## CAN Frames

Each message comes in a packet called a [CAN frame](https://docs.kernel.org/networking/can.html#how-to-use-socketcan). The specific API can be found there, but we only use a frame's ID, payload, and content length. The length is especially important to pay attention to -- each CAN frame arrives with either 8 or 64 bytes, even if not all of those bytes were used when sending. These extra bytes are set to zero, which can cause issues for encodings like Protobuf where zeroes are valid parts of the data. The `length` field tells you how many bytes were actually sent so you can only use those parts of the data buffer.

## More Resources

* [The Wikipedia page](https://en.wikipedia.org/wiki/CAN\_bus) for the CAN bus protocol
* [CSS Electronic's](https://www.csselectronics.com/pages/can-bus-simple-intro-tutorial) page on CAN bus.
