---
description: How we design our DBC files from Protobuf Messages
---

# Designing a DBC Message from Protobuf

This guide walks through the process of converting a Protobuf message into one or more CAN messages for our custom DBC file.

Prerequisites: Before starting, you should be familiar with:
1. [CAN Database (DBC)](README.md): Why we use DBC and its core concepts.
2. [Designing a DBC File](designing-dbc.md): The syntax of our custom YAML format.

## Analyzing the Protobuf Message

First, examine the source Protobuf message defined in `drive.proto`:

```proto
message DriveCommand {
	// The max speed, as a percentage of the rover's possible speed.
	float throttle = 1;
	// Speed of the left wheels, as a percentage of [throttle].
	float left = 2;
	// Speed of the right wheels, as a percentage of [throttle].
	float right = 3;
	// Indicates that [left] = 0 is valid, even though 0 usually means no value.
	bool set_left = 4;
	// Indicates that [right] = 0 is valid, even though 0 usually means no value.
	bool set_right = 5;
	// Indicates that [throttle] = 0 is valid, even though 0 usually means no value.
	bool set_throttle = 6;
	float front_swivel = 7;
	float front_tilt = 8;
	float rear_swivel = 9;
	float rear_tilt = 10;
	ProtoColor color = 13;
	BoolState blink = 14;
}
```

This message contains fields for wheel speeds, module angles, and LED control.

## Segment into Logical CAN Messages

Sending one large CAN message for all these fields is inefficient, as CAN 2.0 can only fit 8 bytes per message. A better approach is to segment the fields into logical groups. For `DriveCommand`, we can identify three distinct groups:

1. Wheel Speeds: `throttle`, `left`, `right`, and the `set_*` booleans.
2. Servo Angles: `front_swivel`, `front_tilt`, `rear_swivel`, `rear_tilt`.
3. LED Control: `color` and `blink`.

This leads to three separate CAN messages:
- `Drive_Set_Speeds`
- `Drive_Set_Swivel`
- `Drive_Set_LED`

## Design the CAN Signals

After defining the messages, we design the signals for each one. This involves choosing an appropriate data type, range, and bit length.

### Choosing a Bit Length

The number of bits for a signal is a trade-off between precision and data efficiency. More bits give higher precision but use more of the limited 8-byte payload in a standard CAN message.

**Integers and Enums**

For whole numbers or enums, you need enough bits to represent all possible values.
- `bits = ceil(log2(number_of_values))`

For example, the `ProtoColor` enum has 5 options. It needs `ceil(log2(5)) = 3` bits. It's often practical to round up to 4 bits for easier debugging or future expansion.

Another important consideration for integer signals is the resolution of the sensor providing the data. If a sensor provides data with a specific bit resolution (e.g., a 10-bit analog signal for the voltage sensor), it is often appropriate to use that exact bit length for the corresponding signal in the DBC message. This ensures that the full precision of the sensor is captured without wasting bits. This approach is used for many fields in our `Drive_Motor_Data` message.

**Floating-Point Numbers**

For floats, we convert a continuous range (e.g., -90.0 to +90.0) into a discrete set of integer steps. The bit length determines the size of these steps, which is the signal's resolution.
- `resolution = (max_value - min_value) / (2^num_bits - 1)`

Consider a swivel angle with a range of -90 to +90 degrees (a total range of 180 degrees):
- With 8 bits (255 steps), the resolution is `180 / 255 ≈ 0.7` degrees per step. This may be too coarse for precise steering.
- With 12 bits (4095 steps), the resolution is `180 / 4095 ≈ 0.044` degrees per step. This offers a good level of precision for motor control.
- With 16 bits (65535 steps), the resolution is `180 / 65535 ≈ 0.0027` degrees per step. This is likely more precise than necessary and would use bits that could be allocated elsewhere.

For our purposes, 12 or 16 bits is often a suitable choice for floating-point values.

## Designing and Writing the YAML

Now, let's go through the process of translating the relevant fields from the `DriveCommand` Protobuf message into the `Drive_Set_Speeds` CAN message.

Here are the Protobuf fields we are working with:

```proto
message DriveCommand {
  // ...

  float throttle = 1;

  float left = 2;

  float right = 3;

  bool set_left = 4;

  bool set_right = 5;

  bool set_throttle = 6;

  // ...
}
```

First, let's handle the boolean flags: `set_left`, `set_right`, and `set_throttle`. These are simple on/off flags. For a CAN signal, a 1-bit unsigned integer is the most efficient representation. We can use our `boolean` type alias for this, which simplifies the definition.

Next, we'll translate the `throttle` field. In the Protobuf, it's a float. For the CAN signal, we need to decide on a bit length. As determined in the 'Choosing a Bit Length' section, a 16-bit unsigned integer provides high precision for a `0.0` to `1.0` range. So, we'll define a signal named `Throttle` with `length: 16`, `is_signed: false`, `min: 0.0`, and `max: 1.0`.

The `left` and `right` speed fields are also floats, but with a range from `-1.0` to `1.0`. We'll use 16-bit signed integers for them to handle the negative range, which gives a similarly high resolution.

Assembling these signals into the message, and assigning a unique CAN ID (e.g., `0x101`), gives us the final YAML definition:

```yaml
  - Drive_Set_Speeds:
      id: 0x101  # Make sure this ID is unique!
      comment: "Command to set Drive Speeds"
      signals:
        - Should_Set_Left:
            comment: "Whether or not to update the left speeds"
            type_alias: boolean
        - Should_Set_Right:
            comment: "Whether or not to update the right speeds"
            type_alias: boolean
        - Should_Set_Throttle:
            comment: "Whether or not to update the throttle"
            type_alias: boolean
        - Left_Speed:
            length: 16
            is_signed: true
            min: -1.0
            max: 1.0
        - Right_Speed:
            length: 16
            is_signed: true
            min: -1.0
            max: 1.0
        - Throttle:
            length: 16
            is_signed: false
            min: 0.0
            max: 1.0
```

## Example Implementations for Other Messages

The same methodology applies to our other messages.

- `Drive_Set_LED`: The `color` enum has 5 values, so it needs at least 3 bits (we use 4). The `blink` signal is a boolean, so it only needs 1 bit.
- `Drive_Set_Swivel`: The four angle signals are floats. As calculated earlier, 12 bits provides a good resolution of ~0.044 degrees for a -90 to +90 degree range.

Here are their final definitions for reference:

```yaml
  - Drive_Set_LED:
      id: 0x102
      comment: "Command to set the Drive LED color"
      size: 1
      signals:
        - Color:
            comment: "The enum value of the color"
            length: 4
            is_signed: false
            min: 0
            max: 8
        - Blink:
            comment: "Whether or not to blink"
            type_alias: boolean

  - Drive_Set_Swivel:
      id: 0x103
      size: 7
      comment: "Command to set the swivel angles of drive"
      signals:
        - Set_Front_Swivel:
            type_alias: boolean
        - Set_Front_Tilt:
            type_alias: boolean
        - Set_Rear_Swivel:
            type_alias: boolean
        - Set_Rear_Tilt:
            type_alias: boolean
        - Front_Swivel:
            length: 12
            is_signed: true
            min: -90
            max: 90
            unit: "°"
        - Front_Tilt:
            length: 12
            is_signed: true
            min: -90
            max: 90
            unit: "°"
        - Rear_Swivel:
            length: 12
            is_signed: true
            min: -90
            max: 90
            unit: "°"
        - Rear_Tilt:
            length: 12
            is_signed: true
            min: -90
            max: 90
            unit: "°"
```
