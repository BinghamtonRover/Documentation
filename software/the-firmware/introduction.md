# Introduction

All other subteams on the rover project use custom-built hardware and custom-designed PCBs to power their systems. HREI uses a robotic arm, EA has what we call the Science Chamber, etc. These devices are powered by [Teensy](https://www.pjrc.com/store/teensy41.html) v4.1 microcontrollers running on the [Arduino](https://www.arduino.cc/) platform, with code written in C++.&#x20;

Our firmware is custom-designed for each subsystem. When working on the firmware, you should regularly meet with the subteam you're designing for and your team lead to discuss requirements and get feedback from the subteam. For example, our code often has exact millimeter-precision measurements and small delays down to the millisecond to get the behavior we need. That's not to say all code is written from scratch though. We have a shared framework that handles basic firmware needs, and we share hardware code when possible.&#x20;
