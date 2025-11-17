# The Stack-Based State Machine

We use a stack-based state machine to manage the rover's behavior. This lets us break down complex tasks into a series of smaller, simpler states. The `StateController` class manages the stack, and the main loop is driven by a `PeriodicTimer` in the `RoverOrchestrator` that calls `controller.update()`.

## The `StateController`

The `StateController` holds a stack of states. The state on top of the stack is the "active" state. It has a few key methods:

* `pushState(state)`: Pushes a new state onto the stack. For example, `NavigationState` will push a `SensorForwardState` to handle one step of a path.
* `popState()`: Removes the top state. A state calls `controller.popState()` on itself when it's finished with its task.
* `transitionTo(state)`: A shortcut for popping the current state and pushing a new one. `PathingState` uses this to switch to `NavigationState` once a path is found.

## Key States

Here are some of the most important states we use for navigation:

* `PathingState`: Its main job is to call `collection.pathfinder.getPath()` to compute a path. If it succeeds, it transitions to `NavigationState`. If not, it pops itself, and the task fails.
* `NavigationState`: This is the workhorse for following a path. On each `update()`, it looks at the next step in the path and pushes the correct driving state (e.g., `SensorForwardState`). It also checks for replanning conditions on every update. If the rover is too far off the path, or if an obstacle is found, it will transition back to `PathingState` to compute a new route.
* `SequenceState`: Executes a list of child states in order. This is useful for setup tasks, like having the rover first turn to a specific orientation and then start pathing.
