# Interfaces and Implementations

A common pattern across our codebase is the use of abstract "interfaces" with multiple concrete "implementations". This allows us to write flexible and testable code.

## The Core Idea

The pattern works by defining an `abstract class` (the interface) that declares a set of methods and properties a component must have, but doesn't provide the logic. Then, one or more concrete classes extend this interface and provide the actual implementation.

This decouples the code that *uses* a component from the specific details of *how* that component works.

## Use Cases

We use this pattern for two main purposes.

### Simulation and Testing

This is most prominent in the `autonomy` program. Nearly every component that interacts with hardware (like `DriveInterface`, `GpsInterface`, `ImuInterface`) has at least two implementations:
- A real implementation (e.g., `RoverDrive`) that communicates with the actual hardware.
- A simulated implementation (e.g., `DriveSimulator`) that mimics the hardware's behavior in software.

A top-level "collection" class (`RoverAutonomy` or `AutonomySimulator`) determines which set of implementations to use. This allows us to run the entire autonomy program on a development machine without any rover hardware, which helps with rapid development and testing.

### Platform-Specific Code

Some hardware only works on a specific operating system. For example, the CAN bus and RealSense camera libraries are only supported on Linux. To allow the rest of the program to run on Windows or macOS, we use interfaces.

- The real implementation (e.g., `CanFFI`, `RealSenseFFI`) uses Dart's FFI to call the native Linux libraries.
- A stub implementation (e.g., `CanStub`, `RealSenseStub`) provides empty methods that do nothing.

A factory constructor then chooses the correct implementation at runtime based on `Platform.isLinux`. This prevents the program from crashing when run on an unsupported OS. You can see this pattern in both the `subsystems` and `video` programs.
