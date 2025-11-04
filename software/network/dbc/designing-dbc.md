---
description: How we make a DBC file
---

# Designing a DBC File

## DBC Syntax

Here's what a basic message looks like in DBC syntax:
```
BO_ 0 Device_Broadcast: 2 Vector__XXX
	SG_ Device_Value : 0|5@1+ (1,0) [0|15] "" Vector__XXX
	SG_ FW_Version_Major : 5|4@1+ (1,0) [1|15] "" Vector__XXX
	SG_ FW_Version_Minor : 9|4@1+ (1,0) [0|15] "" Vector__XXX
```

It's not hard to notice that this is extremely difficult for a human to understand. This is much easier for a computer to understand, so many software tools have been made to make designing a DBC file easier.

## CSS DBC Editor

A simple but useful DBC editor is [CSS Electronic's DBC Editor](https://www.csselectronics.com/pages/dbc-editor-can-bus-database), which can run directly in a browser.

This online editor has a lot of features, such as viewing messages, and information about its individual signals. It also has the ability to preview signal data. Although this is much nicer to look at than the syntax seen above, it can still be a challenge to make an entire DBC using this editor, since it can be time consuming to determine:
1. The exact length of the message based on the signals
2. Minimum and maximum values of a signal
3. The precision (scale) of a signal based on its min and max values

## Custom YAML Format

In order to simplify the creation of DBC messages, we use a custom YAML format, which is read by a Python script which generates a DBC file, which is what is often used in industry.

Here's an example of a message made with our custom CAN YAML format:
```yaml
type_aliases:
  - boolean:
      length: 1
      is_signed: false
      scale: 1
      min: 0
      max: 1
      offset: 0

messages:
  - Drive_Set_Speeds:
      id: 0x101
      size: 7
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
            offset: 0
            min: -1.0
            max: 1.0
        - Right_Speed:
            length: 16
            is_signed: true
            offset: 0
            min: -1.0
            max: 1.0
        - Throttle:
            length: 16
            is_signed: false
            offset: 0
            min: 0.0
            max: 1.0
```

### YAML Specification

Our custom YAML format is designed to be human-readable and easy to work with. A Python script parses this file to generate the final `.dbc` file.

#### Top-Level Fields

- `type_aliases` (optional): A list of reusable signal definitions to avoid repetition.
- `messages` (required): A list of all the CAN messages.

#### Message Definition

Each message in the `messages` list has a name and the following fields:

- `id` (required): The CAN message ID. Can be in decimal or hex, but in most cases should be in hex.
- `signals` (required): A list of signals contained within the message.
- `size` (optional): The size of the message in bytes. If omitted or set to `"auto"`, the size is automatically calculated by summing the bit lengths of all its signals and rounding up to the nearest byte.
- `comment` (optional, but recommended): A description for the message.
- `is_fd` (optional): Set to `true` if this is a CAN-FD message, allowing a payload of up to 64 bytes. Defaults to `false`.

#### Signal Definition

Each signal in the `signals` list has a name and the following fields:

- `length` (required): The length of the signal in bits.
- `is_signed` (required): `true` if the signal is a signed integer, `false` otherwise.
- `type_alias` (optional): The name of an alias from `type_aliases`. The signal will inherit the properties of the alias, but any properties explicitly defined in the signal will override the alias's properties.
- `comment` (optional): A description for the signal.
- `unit` (optional): The physical unit of the signal's value (e.g., "V", "RPM", "°C").

### Automatic Calculation

To simplify defining signals, the generator script can automatically calculate the `scale`, `offset`, `min`, and `max` properties based on the information you provide.

You must provide one of the following combinations for the script to work:

1. `min` and `max`:
    - The script will calculate the `scale` and `offset` for you. This is the most common and recommended method.
    - `scale = (max - min) / (2^length - 2)`
    - `offset` is derived from the `min` and `scale`.

2. `scale` and `offset`:
    - The script will calculate the `min` and `max` values the signal can represent.

3. `scale`, `offset`, `min`, and `max`:
    - If you provide all four, no calculations are performed. This is useful if you need to override the script's calculations for a specific reason.

If you do not provide enough information (e.g., only `min`), the script will terminate with an error.
