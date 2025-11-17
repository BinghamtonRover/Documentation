---
description: How programs on the rover are structured.
---

# Program Structure

Our rover's software follows a few architectural patterns to promote modularity and testability. This section explains these core patterns, which are used across our Dart programs in the `Rover-Code` directory. Understanding these concepts will make it easier to understand the specific documentation for each program.

This documentation is split into the following pages:

- [Services and Collections](./services.md): An explanation of the `Service` class and the "collection" pattern used for dependency management and program lifecycle.
- [Interfaces and Implementations](./interfaces.md): How we use abstract classes to support different hardware, platforms, or simulated environments.
- [Networking](./networking.md): An overview of how our programs communicate using the `burt_network` library.

Following these patterns helps us develop and test features on a computer without needing access to the physical rover, which speeds up development.
