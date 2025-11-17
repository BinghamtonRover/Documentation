# Pathfinding with A*

We use the A* pathfinding algorithm to find a path from the rover's current location to a destination. The logic is powered by the `a_star` pub.dev package, with our custom implementation defined in the `AutonomyAStarState` class. This class represents a single "node" or state in the search space.

## The Grid and State

Instead of a traditional 2D grid, our search space is defined by the rover's physical state: its GPS coordinates and its orientation (the direction it's facing, like North, North-East, etc.).

From any given state, the `expand()` method in `AutonomyAStarState` generates all possible next moves:
* Move forward one meter.
* Turn left or right by 90 degrees (a full turn).
* Turn left or right by 45 degrees (a quarter turn).

## Costs and Heuristics

A* decides which path to explore by calculating a cost for each potential state. The goal is to find the path with the lowest total cost.

* **Cost**: The `getCost()` method defines the cost of each move. Moving forward is the cheapest, while quarter turns and full turns are progressively more expensive. This encourages the algorithm to find paths with fewer turns.
* **Heuristic**: The `heuristic()` method estimates the remaining distance to the goal. We use the "octile distance," which is a standard heuristic for grids that allow movement in 8 directions (straight and diagonal). It's a good estimate that helps A* make smart choices about which path to explore next.

## Path Optimization

Once A* finds a path, the `RoverPathfinder` runs an optimization pass. The `optimizePath()` function looks for sequences of identical quarter turns (e.g., two 45-degree left turns in a row) and combines them into a single, more efficient 90-degree turn.

## Obstacles

The pathfinder is aware of obstacles recorded by the `DetectorInterface`. When the `expand()` method generates potential moves, it checks if a move would land the rover on a known obstacle. If it does, that state is considered invalid and is pruned from the search. This ensures the final path avoids any obstacles the rover knows about.
