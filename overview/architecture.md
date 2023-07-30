---
description: A high-level overview of the different devices across the rover
---

# Architecture

The rover has three layers of code that each run in different environments. Each layer builds on the previous and is mostly independent of the other code in its own layer.

## The Firmware

At the bottom of the stack is our firmware -- code to drive the hardware. We have four subteams that need firmware code: HREI (Arm), EA (Science), Electrical, and Drive. Each subsystem gets its own Arduino microcontroller which has code specific to its subsystem and is completely independent of the other firmware.&#x20;

This code handles the nitty-gritty details of basic tasks. For example, what could be described as "drive forward" is implemented as a PWM signal on specific pins to drive specific motors. Once the firmware handles these details, they are never implemented again, and the higher-level API is used instead.

## The Onboard Computers

These devices are the brains of the rover, running on Raspberry Pis. We've chosen Dart to run our programs, but in the past, they have been written in Python or C++. They perform any complex task that doesn't fit within any one subsystem. Currently, we use these computers to:

* Coordinate all the subsystems' firmware
* Stream video from all attached cameras
* Drive the rover autonomously for the Autonomous Navigation mission

The **Subsystems computer** is the main controller of the rover since it ultimately drives the hardware. Any connections to the rover should start here, and no firmware should be read from or controlled by any other code. These programs don't have to run on different devices, but they should be separate programs.&#x20;

## The Dashboard

The Dashboard is our user-friendly application that connects to the onboard computers, streams data from them, and sends commands on the user's behalf. This is perhaps the most complex system "on" the rover, since it needs to interface with, ultimately, every other system, whereas most other systems are independent.&#x20;

While other teams configure their rover to expose a web server and host their dashboard _on_ the rover, we see all this complexity as reason enough to extract it to its own program. And not use JavaScript. Instead, we use Flutter, a cross-platform app framework, with a practical focus on a Windows application. The main goal of the dashboard, above all else, is to be simple to use and not require an in-depth understanding of the rover to operate.
