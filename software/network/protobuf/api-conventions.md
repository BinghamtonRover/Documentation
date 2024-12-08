---
description: How to read our API
---

# API Conventions

## Data and Commands

To keep our APi organized, we make a new `.proto` file for every subsystem or program in our rover. Each file defines at least two types of messages, data and commands. The data message is one big message containing all the data that can be sent _by_ the device, while the command message contains all the commands that could be sent _to_ the device. Because of how Protobuf serializes, we can fill in as many or as little of these fields as we want, so long as they are defined.

Developers are encouraged to nest data and commands when necessary. For example, the arm reports the same data on each of its motors, so we define a helper message:&#x20;

```protobuf
message MotorData { /* ... */ }
message ArmData {
  MotorData motor1;
  MotorData motor2;
  // ...
}
```

## Wrapped Messages

One of the biggest problems with Protobuf is that messages are not self-describing. In other words, if you have two messages in binary format, you can try to parse on as the other:

```dart
// In this example, A and B are Protobuf messages
final a = A();
final b = B.fromBuffer(a.writeToBuffer());  // this could work, or crash
```

On the CAN bus, each packet has an ID that denotes its content, so this is not an issue; messages are simply handled according to their ID. But in other contexts, we may not be able to know ahead of time what kind of message we are receiving. To get around this, we define a `WrappedMessage`

```protobuf
message WrappedMessage {
  string name = 1;
  bytes data = 2;
}
```

Every single message that is sent (except for CAN) must be wrapped in a `WrappedMessage` so the receiver can handle it accordingly. When a message is received, it can always be parsed as a `WrappedMessage` then handled according to its `name`.&#x20;

## Heartbeats and Handshakes

Like `WrappedMessage`, there are some message that are handled specially. When connecting to a device, you must send the `Connect()` message identifying yourself and who you'd like to connect to. When the other device receives your message, it'll respond with another `Connect()` message identifying itself. If you don't receive one, you can assume the connection was unsuccessful.&#x20;

This idea of sending a message and waiting for confirmation (a "handshake") is useful in general:

* The dashboard periodically re-send the `Connect()` handshake to all connected devices to ensure they are still reachable. We call these "heartbeats" and they are vital to monitoring the connection of the rover. For example, when the subsystems program stops receiving heartbeats, it knows to stop all operations and wait to come back online.
* Important commands, like starting the autonomy mission, are echoed back as handshakes to ensure the rover has really received it. While most commands like "drive forward" can be sent as long as the user is pushing the joystick, these commands need to be received the first time.

## Data containing zeroes

**Note**: This behavior has been undone with the new [explicit field presence](https://protobuf.dev/programming-guides/field_presence/) and should be updated

Protobuf is able to serialize large messages into small buffers by omitting all fields that were not set in the original format. When parsing, Protobuf assigns a default value to all unset fields. For example, unset numbers default to zero. This represents a serious problem: how do we tell the rover to stop if we can't send a speed of zero? Or, conversely, if all non-speed messages have speed set to zero implicitly, how does the rover ever drive?

To work around this, we reserve a field `bool set_x` for any numeric `x` that can be zero (or bool that can be false). The sender must set `set_x` to `true` _in the same message as `x`_. The receiving endpoint must then check `set_x` before using or ignoring `x`. For example:

```dart
DriveCommand(throttle: 1.0);  // Will not work
DriveCommand(throttle: 1.0, setThrottle: true);  // OK

DriveCommand(left: 1.0, right: 1.0);  // Will not work
DriveCommand(left: 1.0, setRight: true);  // Will not work
DriveCommand(left: 1.0, setLeft: true);  // OK
DriveCommand(right: 1.0, setRight: true);  // OK
DriveCommand(left: 1.0, right: 1.0, setLeft: true, setRight: true);  // OK
```

## Small messages

The firmware is limited by the [can-bus](../../legacy-docs/can-bus/ "mention") in how many bytes it can receive. Recall that Protobuf omits data that was left in its default value, so only data that was explicitly set is sent over the wire. The exact byte counts for each data type are listed [here](https://protobuf.dev/programming-guides/proto3/#scalar), but as a general rule, each field (with its `set_x` field) should get its own message. For example:&#x20;

```dart
// This command will only send over CAN FD, but not normal CAN
DriveCommand(left: 1.0, right: 1.0, setLeft: true, setRight: true);  // Too big
// This command may not even send over CAN FD!
DriveCommand(tilt: 3.0, swivel: 5.0, left: 1.0, setLeft: true, /* ... */);
// Instead, break up big commands into several smaller commands
DriveCommand(left: 1.0, setLeft: true);  // OK
DriveCommand(right: 1.0, setRight: true);  // OK
```
