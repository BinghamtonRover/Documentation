---
description: A high-level overview of the different devices across the rover
---

# Architecture

The rover has three layers of code that run in different environments. Each layer builds on the previous to provide extra functionality, but . The following diagram shows an overview, and the following sections explain in more detail.&#x20;

<figure><picture><source srcset="../.gitbook/assets/Rover wiring diagram (12).png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/Software components diagram.png" alt="A diagram that shows the software components and how the integrate. The Dashboard is connected to the base station antenna, which connects over WiFi to the rover&#x27;s antenna. The antenna connects via Ethernet to the main rover programs: Lidar, Video, Computer Vision, Autonomy, and Subsystems. The Subsystems program connects via USB to the firmware: Drive, Electrical, Arm, and Science"></picture><figcaption><p>All the different software components on the rover and how they communicate</p></figcaption></figure>

## The Firmware

At the bottom of the stack is our firmware -- code to drive the hardware. We have four subteams that need firmware code: HREI (Arm), EA (Science), Electrical, and Drive. Each subsystem gets its own Arduino microcontroller which has code specific to its subsystem and is completely independent of the other firmware. Off the rover, we have a base station setup that also has a small firmware setup to control the antenna.

This code handles the nitty-gritty details of basic tasks. For example, what could be described as "drive forward" is implemented as a signal on specific pins to drive specific motors. Once the firmware handles these details, they are never implemented again, and the higher-level API is used instead.

## The Onboard Computers

These devices are the brains of the rover, running on Raspberry Pis. We've chosen Dart to run our programs, but in the past, they have been written in Python or C++. They perform any complex task that doesn't fit within any one subsystem. Currently, we use these computers to:

* Coordinate all the firmware
* Stream and analyze video from all attached cameras and sensors
* Drive the rover autonomously for the Autonomous Navigation mission

The **Subsystems program** is the main controller of the rover since it ultimately drives the hardware. Any connections to the rover should start here, and no firmware on the rover should be read from or controlled by any other code. These programs are separate, but they don't necessarily have to run on separate _devices_.&#x20;

## The Dashboard

The Dashboard is our user-friendly application that connects to the onboard computers, streams data from them, and sends commands on the user's behalf. This is perhaps the most complex system "on" the rover, since it needs to interface with, ultimately, every other system, whereas most other systems are independent.&#x20;

The Dashboard is meant to be the ultimate testing, debugging, and operating tool for all our needs. If you need to test any device, it should be possible to do so via the Dashboard. So, aside from connecting to the Subsystems, Video, and Autonomy programs over UDP, the Dashboard can also connect to any firmware device over USB.&#x20;

While other teams configure their rover to expose a web server and host their dashboard _on_ the rover, we see all this complexity as reason enough to extract it to its own program. And not use JavaScript. Instead, we use Flutter, a cross-platform app framework, with a practical focus on a Windows application. Since Flutter uses Dart, we get to share a lot of code with the rover — particularly, our networking protocols.&#x20;
