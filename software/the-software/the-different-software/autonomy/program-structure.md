# Program Structure

The autonomy code is organized around a central `AutonomyInterface` class. This class acts as a container for all the services the rover needs to drive itself. All services inherit from a base `Service` class, which defines an `init()` and `dispose()` method for setup and teardown.

## The `AutonomyInterface`

The `AutonomyInterface` defines the program's main components. The two most important implementations are:

* `RoverAutonomy`: This class pieces together all the "real" service implementations that interact with hardware.
* `AutonomySimulator`: This class pieces together all the simulated services that mimic the hardware's behavior.

For example, `RoverAutonomy` instantiates `RoverGps`, which listens for real GPS data over the network. `AutonomySimulator` instantiates `GpsSimulator`, which just holds a variable for the rover's position in a virtual space.

```dart
// From rover.dart
class RoverAutonomy extends AutonomyInterface {
  @override
  late GpsInterface gps = RoverGps(collection: this);
  @override
  late ImuInterface imu = RoverImu(collection: this);
  // ... and so on
}

// From simulator.dart
class AutonomySimulator extends AutonomyInterface {
  @override
  late GpsInterface gps = GpsSimulator(collection: this);
  @override
  late ImuInterface imu = ImuSimulator(collection: this);
  // ... and so on
}
```

This structure allows the high-level task logic to remain completely unaware of whether it's running on a real rover or in a simulation, which is great for testing.