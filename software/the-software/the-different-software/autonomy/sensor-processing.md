# Sensor Processing

Raw sensor data can be noisy. To get more consistent navigation, we process the data from the GPS and IMU before using it. This is handled by the `ErrorCorrector` class.

## The `ErrorCorrector`

Instead of using raw values directly, the `RoverGps` and `RoverImu` services pass them through an `ErrorCorrector`. The corrector's behavior is controlled by two parameters:

* `maxSamples`: The number of recent readings to keep in a queue.
* `maxDeviation`: A threshold to reject outliers.

The `addValue()` method is where the logic happens. When a new value comes in, it's compared to the current calibrated average. If it's too different (i.e., greater than `maxDeviation`), it's ignored as an outlier. Otherwise, it's added to the queue (and the oldest value is removed).

The "official" value that the rest of the program uses is then recalculated by taking a weighted average of all the samples in the queue. Currently, this is a simple average, but this design allows for more complex weighting in the future.

This process of smoothing and outlier rejection prevents the rover from overreacting to single bad sensor readings, making its movement much more stable.
