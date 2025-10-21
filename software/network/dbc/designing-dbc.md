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

// TODO: Describe automatic calculation of precision/scale, min/max, and offset.
// Mention which properties are mandatory (length, signedness, etc) and what can be
// omitted in which situations
