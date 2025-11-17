# Obstacle Detection

Detecting and avoiding obstacles is handled by services that implement the `DetectorInterface`. The system can get obstacle information from multiple sources and uses a management strategy to decide how to handle them.

## Detection Sources

* **Lidar (`RoverDetector`)**: This service processes `LidarPointCloud` data. The `_queueObstacles()` method filters the raw data by comparing the cartesian and polar coordinates for each point. If they are consistent, it's considered a valid point. This helps filter out noise and false readings.
* **Network (`NetworkDetector`)**: This service listens for `AutonomyData` messages from other programs. This allows a separate computer vision service, for example, to report obstacles it has detected, which are then fed into our pathfinding.

## Obstacle Management

When an obstacle is detected, its GPS coordinates are recorded by the `PathfindingInterface`.

### Temporary vs. Locked Obstacles

We handle obstacles in two ways to make the system more flexible:

* **Temporary Obstacles**: When an obstacle is first seen, it's marked as temporary. The pathfinder will avoid it, but we know it might have been a bad sensor reading. The `RoverDetector.findObstacles()` method contains logic to clear out temporary obstacles that are behind the rover or no longer in its field of view.
* **Locked Obstacles**: An obstacle gets "locked" when it is confirmed to be in the way. The `findAndLockObstacles()` method in the `RoverOrchestrator` is responsible for this. If a temporary obstacle is found to be intersecting with the `currentPath`, it gets promoted to a locked obstacle. Locked obstacles are treated as permanent for the rest of the task, preventing the rover from repeatedly trying to plan a path through something it knows is there.
