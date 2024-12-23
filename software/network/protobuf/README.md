---
description: How we send data and commands
---

# Protobuf

## Introduction

As described in the [data-format.md](../../overview/data-format.md "mention") page, we use [Protocol Buffers](https://protobuf.dev) ("Protobuf") to keep our data in sync across our devices. Protobuf has the advantage that we can declare what our data looks like in `.proto` files and use the Protobuf compiler to generate code to serialize and deserialize that data in any language.&#x20;

An API helps programmers structure communications across a system. Usually you think of an API as a set of functions with inputs and outputs, but our rover is a two-way stream: data flows freely in both directions. Our API defines how that data will be structured and how to interpret it.&#x20;

Say you need to drive forward at 100% throttle. That may be simple enough for the drive firmware, but the dashboard needs to be able to communicate this intent, and the subsystems program needs to know where to send it. Using our API, you can simply write

```dart
DriveCommand(left: 1.0, right: 1.0)
```

This section describes how we structure our [Protobuf repository](https://github.com/BinghamtonRover/Protobuf) and our API in general. The next pages describe conventions, the Protobuf tooling, and some Protobuf quirks to keep in mind.&#x20;

## Generated code

We choose to push the generated code to GitHub so only a handful of members need to deal with code generation. Note that you should not try to read the generated files, only the Proto files.

* Dart code can be found in the [`burt_network`](https://github.com/BinghamtonRover/Dart-Networking) package
* C++ code can be found in the [`firmware-utilities`](https://github.com/BinghamtonRover/Firmware-Utilities) package

## Reading Protobuf files

A sample Protobuf file might look like this

```protobuf
syntax = "proto3";

import "address.proto";

message User {
  int32 id = 1;
  string name = 2;
  Address address = 3;
  repeated User friends = 4;
  bytes payload = 5;
}
```

Which is equivalent to the following Dart code, for example:

```dart
import "address.dart";

class User {
  int id;
  String name;
  Address address;
  List<User> friends;
  Uint8List payload;
}
```

A `message` is a new type of data, like a `class` or a `struct`. A message contains fields, and each field has a type, name, and _unique_ ID. IDs do not have to be in order, and are only used by the Protobuf compiler, not us. See [here](https://protobuf.dev/programming-guides/proto3/#scalar) for a list of all the built-in types, but you can also use a type you defined. You can denote a list with `repeated`, and a list of bytes `(char*)` with `bytes`.  The [working-with-proto-files.md](working-with-proto-files.md "mention") page describes more detail about the syntax and code generation.

## More Resources

* [The official Protobuf website](https://protobuf.dev)
* [The syntax for `.proto` files](https://protobuf.dev/programming-guides/proto3)
* [Our Protobuf repository](https://github.com/BinghamtonRover/Protobuf)

