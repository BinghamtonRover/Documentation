# Driving Strategies

The `DriveInterface` is our standard way of telling the rover to move, but we have a few different strategies for how to carry out those commands. The `RoverDrive` class acts as a wrapper that chooses the right strategy based on its configuration.

## `SensorDrive`

This is the main implementation we use on the rover. It relies on a feedback loop using the rover's sensors to move precisely.

This is handled by the `runFeedback()` utility method, which runs a `while (true)` loop. Inside the loop, it executes a given function (like `moveForward()`) and then checks a condition (like `isNear(target)`). The loop continues until the condition is true or the task is cancelled. This makes the movement precise, but it requires good sensor data.

## `TimedDrive`

This implementation is simpler and doesn't use sensors. When told to perform an action, it runs the motors for a pre-calibrated amount of time. These durations are defined in the `DriveConfig` class (e.g., `oneMeterDelay`, `turnDelay`).

This is less precise and is mostly used in situations where sensors might be unavailable or for simple hardware tests.

## `RoverDrive`

This class wraps the other two implementations and is what the rest of the autonomy code usually interacts with. It decides which strategy to use based on the `useGps` and `useImu` boolean flags passed to its constructor.

This allows for a mix-and-match approach. For example, when running a simulation without a real IMU, we can configure `RoverDrive` to use the precise `SensorDrive` for forward movements (using the simulated GPS) but fall back to the less precise `TimedDrive` for turns.
