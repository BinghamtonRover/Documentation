# Networking

All of our rover's programs (on the Pi, base station, and dashboard) communicate over the network using our custom `burt_network` library. This library provides a standardized way to send and receive data, ensuring all our programs can talk to each other.

## Core Technologies

Our networking stack is built on two main technologies:

1. **UDP (User Datagram Protocol)**: A lightweight, connectionless protocol for sending messages ("datagrams") over a network. It's faster than TCP but doesn't guarantee that messages will arrive or that they'll arrive in order. We build our own reliability on top of it where needed (see Heartbeats).

2. **Protobuf (Protocol Buffers)**: A language-agnostic format for serializing structured data. We define the "shape" of our data in `.proto` files, and the Protobuf compiler generates Dart code to create, read, and write that data. This ensures that the sender and receiver agree on the data structure, and it's very efficient. All our message definitions are in the [Protobuf repository](https://github.com/BinghamtonRover/Protobuf).

## The `burt_network` Library

This library, located in `Rover-Code/burt_network`, abstracts away the details of UDP and Protobuf. The most important class is `RoverSocket`, which all our on-rover programs use.

### `RoverSocket`

This class provides a simple interface for network communication:

- It automatically handles sending and receiving heartbeats to check if the dashboard is still connected.
- It wraps all outgoing messages in a `WrappedMessage`, which contains the Protobuf data itself, the name of the message type, and a timestamp.
- It unwraps incoming messages and directs them to the correct handler.

### Sending and Receiving Messages

To send a message, you simply call `sendMessage()`:
```dart
// Create a Protobuf message
final command = DriveCommand(throttle: 1.0);
// Send it over the socket
server.sendMessage(command);
```

To receive messages, you subscribe to the `messages` stream, usually filtering for a specific message type:
```dart
// From Rover-Code/autonomy/lib/src/autonomy_interface.dart
server.messages.onMessage(
  name: AutonomyCommand().messageName,
  constructor: AutonomyCommand.fromBuffer,
  callback: onCommand,
);
```
The `onMessage` helper function (an extension on `Stream<WrappedMessage>`) finds messages with the given name, uses the provided constructor to parse the binary data into a `AutonomyCommand` object, and then passes it to the `onCommand` callback.

This publish-subscribe model allows different parts of the code to listen for the messages they care about without having to parse every single packet that comes through the socket.
