# Rover CAN Protocol

## Overview

The Rover uses a custom protocol over the CAN bus, built on top of CAN 2.0. While there are some established standards for CAN devices, such as CAN Open, J1939, and OBD2, none of these are very practical for use on the rover.

## Design Considerations

When designing the protocol for the rover's CAN network, several features were required:
1. Every device can know whether it's connected to the rover
2. The rover can determine which devices are connected
3. Easy to integrate/convert into existing Protobuf definitions
4. Does not have a small limitation on the number of message types

## CAN Type and Bitrate

The rover CAN protocol uses CAN 2.0A (11-bit frame IDs) with a bitrate of 500,000 bits/second.

## Message IDs

Each device on the bus is assigned a unique ID. The device IDs for each firmware application are shown below.

| Device             | Device ID |
| ----------------   | --------- |
| Subsystems (Rover) | `0x00`    |
| Drive              | `0x01`    |
| Relay              | `0x02`    |
| Arm                | `0x03`    |
| Science            | `0x04`    |
| Gripper            | `0x05`    |

When a message in the rover CAN protocol is sent across the bus, it is split into 2 parts, the device id, and the message type. The first 4 bits of the Frame ID are the device id. The remaining 8 bits are the message ID. For all devices, a message ID of `0x00` is a broadcast message, which the device sends when it receives a heartbeat message from the rover, containing the device type and firmware version information.

For example, a drive message with an id of 1 would be `0x101`, wheras a science message with an id of 5 would be `0x405`.

This can be calculated in code by using the following: `frame_id = (device_id << 8) | message_id`

In the rover's DBC file, the messages for each device already have the device ID incorporated into it, so this will not be necessary.

## Heartbeats and Device Broadcasts

### Overview

Our devices advertise themselves on the bus in a Heartbeat -> Broadcast system. Heartbeats are sent by the rover, and when a firmware application receives a heartbeat, it will send a device broadcast, acting as a confirmation that it received the heartbeat, along with letting subsystems and other devices know it is active and running.

### Message Timeline

The Heartbeat -> Broadcast following this sequence:
1. Rover sends a heartbeat
2. Device receives heartbeat
3. Device sends device broadcast, containing its device type and firmware version
4. Rover receives device broadcast

When the subsystems does not receive a broadcast from a device after a certain amount of time, it will assume the device has disconnected, and will no longer send commands to that device.

When a firmware device does not receive a heartbeat after a certain amount of time, it will assume the subsystems has disconnected, and should stop any moving components.

### Heartbeats

Heartbeat messages are id `0x01`, and contain the following data:
* Rover status enum

### Device Broadcasts

Device broadcasts are represented by the ID `(Device_ID << 8) | 0x00` and contain the following data:
* Device type enum value
* Firmware version major
* Firmware version minor
