# Table of contents

* [Software Documentation](README.md)

## Overview

* [Architecture](overview/architecture.md)
* [Data format](overview/data-format.md)
* [Network](overview/network.md)
* [Our GitHub organization](overview/our-github-organization.md)

## Writing Code

* [Code conventions (Dart)](writing-code/code-conventions-dart.md)
* [Code conventions (C++)](writing-code/code-conventions-c++.md)
* [Documenting code](writing-code/documenting-code.md)
* [Using Git](writing-code/using-git/README.md)
  * [Basic workflow](writing-code/using-git/basic-workflow.md)
  * [Pull Requests](writing-code/using-git/pull-requests.md)
  * [Git Submodules](writing-code/using-git/git-submodules.md)
  * [Common problems](writing-code/using-git/common-problems.md)
* [Reviewing Code (admins)](writing-code/reviewing-code-admins.md)

## Network

* [Protobuf](network/protobuf/README.md)
  * [API Conventions](network/protobuf/api-conventions.md)
  * [Working with Proto files](network/protobuf/working-with-proto-files.md)
  * [Generating code](network/protobuf/generating-code.md)
  * [Debugging Protobuf](network/protobuf/debugging-protobuf.md)
* [UDP](network/udp/README.md)
  * [Setting up the network](network/udp/setting-up-the-network.md)

## The Firmware

* [Introduction](the-firmware/introduction.md)
* [Setting up your IDE](the-firmware/setting-up-your-ide.md)
* [Firmware Utilities](the-firmware/firmware-utilities.md)
* [TMC Motor Drivers](the-firmware/tmc-motor-drivers.md)
* [The different firmware](the-firmware/the-different-firmware/README.md)
  * [Arm](the-firmware/the-different-firmware/arm.md)
  * [Science](the-firmware/the-different-firmware/science.md)
  * [Drive](the-firmware/the-different-firmware/drive.md)
  * [Electrical](the-firmware/the-different-firmware/electrical.md)
  * [Antenna](the-firmware/the-different-firmware/antenna.md)

## The Software

* [Introduction](the-software/introduction.md)
* [Pi Setup](the-software/pi-setup/README.md)
  * [UDev Rules](the-software/pi-setup/udev-rules.md)
  * [Configuring systemd](the-software/pi-setup/configuring-systemd.md)
* [Burt Network](the-software/burt-network.md)
* [The different software](the-software/the-different-software/README.md)
  * [Subsystems](the-software/the-different-software/subsystems-computer.md)
  * [Video](the-software/the-different-software/video/README.md)
    * [Multithreading](the-software/the-different-software/video/multithreading.md)
    * [OpenCV streaming](the-software/the-different-software/video/opencv-streaming.md)
    * [RealSense Streaming](the-software/the-different-software/video/realsense-streaming.md)
    * [Lidar Streaming](the-software/the-different-software/video/lidar-streaming.md)
  * [Autonomy](the-software/the-different-software/autonomy.md)
  * [Computer Vision](the-software/the-different-software/computer-vision.md)
  * [Base Station](the-software/the-different-software/base-station.md)

## Dashboard

* [Introduction](dashboard/introduction.md)
* [MVVM Architecture](dashboard/mvvm-architecture.md)
* [Design Philosophies](dashboard/design-philosophies.md)

## Appendix

* [Team Documentation](appendix/team-documentation.md)
* [Open Source Contributions](appendix/open-source-contributions.md)
* [Alternative Design Choices](appendix/alternative-design-choices/README.md)
  * [ROS for Networking](appendix/alternative-design-choices/ros-for-networking.md)
  * [Web Server on the Rover](appendix/alternative-design-choices/web-server-on-the-rover.md)
  * [C++ for Software](appendix/alternative-design-choices/c++-for-software.md)
  * [Python for Software](appendix/alternative-design-choices/python-for-software.md)
  * [CAN Bus for firmware](appendix/alternative-design-choices/can-bus-for-firmware.md)

## Legacy docs

* [CAN bus](legacy-docs/can-bus/README.md)
  * [Setting up CAN on a Pi](legacy-docs/can-bus/setting-up-can-on-a-pi.md)
  * [Debugging CAN](legacy-docs/can-bus/debugging.md)
