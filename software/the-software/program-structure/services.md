# Services and Collections

A common pattern in our software is the "Service" and "Collection" model. This is our approach to dependency injection and managing the lifecycle of different parts of a program.

## The `Service` Class

Most major components in our Dart programs are a `Service`. A `Service` is a simple abstract class from `package:burt_network` that defines a common lifecycle for our components:

- `Future<bool> init()`: Initializes the service. This could involve opening a socket, connecting to hardware, or setting up data streams. It returns whether it was successful.
- `Future<void> dispose()`: Cleans up the service's resources. This is for closing sockets, canceling stream subscriptions, and generally preventing memory leaks.

This contract ensures that all our components can be started and stopped in a predictable way.

## What Makes a Good Service?

A service should be designed with a single responsibility in mind. Think of a service as a manager for a specific domain of the rover's functionality.

**What should be in a service?**
- **A single responsibility:** A service should manage one main thing, like a piece of hardware (e.g., `GpsReader`), a network connection (`RoverSocket`), or a logical group of devices (`FirmwareManager`).
- **Lifecycle management:** The `init` method should acquire any resources the service needs (like opening ports or starting timers), and `dispose` must release all of them to prevent resource leaks.
- **Data handling:** Services often listen to streams of data, parse them, and either forward them or make them available to other parts of the program.

**What should NOT be in a service?**
- **High-level logic:** A service's job is to provide data or perform an action, not to make complex decisions. For example, the `GpsReader` provides coordinates, but the `autonomy` program's orchestrator decides how to act on those coordinates.
- **Multiple responsibilities:** A service that reads from the GPS should not also be responsible for controlling the arm. These should be separate services.
- **Simple data objects:** Classes that just hold data (like Protobuf messages) or files with simple utility functions are not services. A class should only be a service if it has a lifecycle and an ongoing responsibility.

## The "Collection" Pattern

Instead of using singletons or global variables, we group related services into a "collection" class. This collection acts as a central place to access all the services a program needs. All our main programs (`subsystems`, `video`, `autonomy`) have a `Collection` class that extends `Service`.

The collection's `init()` method is responsible for initializing all its services, and its `dispose()` method cleans them all up. This provides a clear sense of ownership and makes the program's lifecycle easy to manage. Any service can be given a reference to the collection, allowing it to access any other service it needs without having to manually pass references everywhere. This pattern also makes testing easier, as we can substitute fake or simulated services in a test environment.
