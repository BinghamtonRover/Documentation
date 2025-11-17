# Autonomous Driving

The autonomy program navigates the rover from point A to point B without human intervention. It's designed to be modular and testable, allowing for complex behaviors to be built from simple, reusable components.

The architecture uses pathfinding algorithms, a stack-based state machine for task execution, and an interface-based design that allows for easy simulation and testing.

## Core Concepts

The autonomy code is built around a few core concepts that work together. These are explained in more detail in the following pages:

* [Program Structure](./autonomy/program-structure.md): The code is organized around a set of core services, each with a specific responsibility. This modular structure makes the code easier to understand, test, and extend.

* [The State Machine](./autonomy/state-machine.md): A stack-based state machine manages the rover's behavior. It allows for complex tasks to be broken down into smaller, manageable states, making the code easier to understand and maintain.

* [Task Execution](./autonomy/task-execution.md): An orchestrator manages high-level tasks, like navigating to a GPS coordinate. It uses the state machine to plan, execute, and adapt its plan as the environment changes.

* [Pathfinding](./autonomy/pathfinding.md): How the rover uses the A* algorithm to find a path to its destination.

* [Driving Strategies](./autonomy/driving-strategies.md): The different ways the rover can be commanded to move, either with or without sensor feedback.

* [Obstacle Detection](./autonomy/obstacle-detection.md): How the rover detects and keeps track of obstacles in its environment.

* [Sensor Processing](./autonomy/sensor-processing.md): How raw GPS and IMU data is processed to get more consistent readings.