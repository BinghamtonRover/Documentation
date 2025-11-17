# Task Execution and Replanning

High-level tasks are managed by the `RoverOrchestrator` and the state machine. The `onCommand()` method in the orchestrator is the entry point. It receives an `AutonomyCommand` from the dashboard and kicks off the appropriate task.

## The Execution Flow

The `AutonomyCommand` specifies a task, such as `GPS_ONLY` or `VISUAL_MARKER`. For a `GPS_ONLY` task, the orchestrator starts the state machine, which then takes over the execution flow:

1. **State Initialization**: The orchestrator pushes an initial state, usually a `SequenceState` that prepares the rover and then pushes a `PathingState`.
2. **Path Planning (`PathingState`)**: This state finds a path to the destination. If successful, it transitions to `NavigationState`.
3. **Path Following (`NavigationState`)**: This state executes the path step-by-step, pushing low-level driving and turning states as needed.

Throughout this process, the `RoverOrchestrator` periodically sends an `AutonomyData` status message back to the dashboard. This message includes the rover's current state (e.g., `PATHING`, `DRIVING`), the list of known obstacles, and the current path being followed.

## The Main Loop

There is no blocking `while` loop for task execution. Instead, the `RoverOrchestrator` starts a `PeriodicTimer` when a command begins. This timer calls the `controller.update()` method of the state machine at a regular interval, which drives all the state transitions and actions.

## Replanning

The rover can change its plan if something unexpected happens. The `NavigationState` is responsible for triggering a replan by checking for certain conditions on every `update()` call:

* If a new obstacle is detected and blocks the current path.
* If the rover's actual position deviates too far from the planned path.

If either of these happens, `NavigationState` will transition back to `PathingState`, which will compute a new route based on the latest information.
