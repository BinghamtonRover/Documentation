---
description: Guidelines to follow when writing Protobuf code
---

# Working with Proto files

## Introduction

Protobuf is not just a syntax, it's a toolchain. You write `.proto` files and compile them using `protoc`, the Protobuf compiler. The compiler takes options over the command line which tells it what language to generate for and outputs the right code.&#x20;

When making modifications, be sure to follow the process in the [Broken link](broken-reference "mention")section, including documentation! Since Protobuf is shared across all devices, it is important to centralize documentation in the Proto files themselves.

## Creating files

Each endpoint should have its own `.proto` file. An endpoint is any program that runs on the network. For example, the arm and gripper comprise one subsystem, but each has a separate Arduino board attached to the CAN bus with different data and commands, so we designate them as separate endpoints. Conversely, the video and autonomy program run on the same device but have totally different functions, so they have different protobuf files.&#x20;

Below is a breakdown of a typical Proto file. More details can be found in the [Protobuf file spec](https://protobuf.dev/programming-guides/proto3).&#x20;

```protobuf
syntax = "proto3";  // <-- This line is required for every file

import "x.proto";  // Must be in the same directory 
import "google/protobuf/timestamp.proto";  // built-in Protobuf types

/// An enum allows you to define a closed set of values for a type.
///
/// The convention for enum members is SCREAMING_CASE. 
enum Color {
  COLOR_UNDEFINED = 0;  // <-- see the below for details
  RED = 1;
  GREEN = 2;
  BLUE = 3;
}

/// A message is like a class or struct, and may contain fields: 
/// - All fields must have a unique ID, which may be any number
/// - All fields must have a type, (including types we defined)
/// - Nested messages are valid protobuf, but do not use them
/// - For a list, use `repeated`
///
/// The convention for field names is snake_case
message Pallate {
  string name = 1;
  repeated Color paints = 2;
  int32 paint_count = 3;  // int32 is the standard integer type
}
```

## Guidelines

Here are a few guidelines to keep in mind when making changes to an existing Protobuf file:&#x20;

* IDs are arbitrary, but not useless. They are the basis of how Protobuf serializes and deserializes data. Mixing up IDs can be very dangerous, as the following example demonstrates

```protobuf
message Before {
  int32 a = 1;  // field a is serialized first
  int32 b = 2;
}

message After {
  int32 a = 2;
  int32 b = 1;  // field b is serialized first
}
```

When compiled, trying to parse a serialized `Before` using an `After` will result in the fields being swapped! If the types of the fields are compatible, this will fail silently and do the wrong thing. If not, the program will crash at runtime. **Avoid changing IDs once the field is in use**.&#x20;

* Related to the IDs issue, the Protobuf files are used in every single endpoint, but they are not references, they're copies. Meaning any changes you make must be manually propogated across all code that needs it, and communication between two endpoints that are out-of-sync may crash, fail silently, or damage something. Be conservative when changing code and **be sure that all developers and repositories are up-to-date** when you do.&#x20;
* If you find yourself writing a message that repeats fields, consider breaking them out into a new message and using that as the type of your fields. For example, a `MotorData` message can simplify a large `ArmData` message into just a few fields.&#x20;
* Similarly, if you are using data that is defined in other endpoints, consider breaking it out into a new message in another file so both endpoints can import it. For example, the `Coordinates` message type is common for IK, GPS, and Autonomy.
* When writing code that will be used by the firmware, **avoid strings at all costs**. Not only are strings a bottleneck in C/C++, but they are more complex to use, and do not mix well with our C++ toolchain. See [#nanopb](generating-code.md#nanopb "mention") for more details.
* Take care when defining enums:&#x20;
  * Due to the limitations of C, Protobuf requires that enum members must be **globally** unique across all Protobuf files. Avoid generic names like `OK` or `CONNECTED` because other enums won't be able to use them. Instead, add a prefix: eg, `DEVICE_OK` or `CAMERA_CONNECTED`.
  * All enums should have an `ENUM_NAME_UNDEFINED = 0` member. This is needed because if a message was not given a value for an enum field, it is assigned 0 which will crash the program if no 0 member was defined. Since enum members must have unique names, be sure to prefix it properly, even if it looks ugly.&#x20;

## More Resources

* [The Proto file syntax](https://protobuf.dev/programming-guides/proto3)
* [The Protobuf style guide](https://protobuf.dev/programming-guides/style/)
* [Unexpected Enum behavior](https://protobuf.dev/programming-guides/enum/)
* [Default and omitted fields](https://protobuf.dev/programming-guides/field\_presence/)
* [Protobuf practices](https://protobuf.dev/programming-guides/dos-donts/) and [API practices](https://protobuf.dev/programming-guides/api)
