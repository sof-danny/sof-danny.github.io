---
layout: post
title: "A* Path Planning: Guided Search on a Grid"
date: 2024-02-15
category: motion-planning
tags: [robotics, path-planning, A-star, C++, grid, heuristic]
excerpt: "How A* finds the shortest path by balancing the cost already paid against a heuristic estimate of the cost remaining — implemented in C++ with obstacles and terrain costs."
---

Imagine you are in a city and want to drive from your house to a coffee shop. You could explore every possible street — that is Dijkstra's algorithm — but it is wasteful. Instead, you naturally favor roads that are *heading towards* the coffee shop. That is exactly what A* does: it is a **guided search**.

Full implementation: [`astarV3.cpp`](https://github.com/sof-danny/cpp_ppp/blob/main/robotics_algorithms/Astar/astarV3.cpp)

---

## The Core Idea

A* finds the shortest path by balancing two questions at every step:

- *"How far have I already travelled?"* — don't ignore the cost already paid
- *"How far do I still have to go?"* — use a heuristic to prioritize promising directions

For every cell being considered, A* computes:

$$
f(n) = g(n) + h(n)
$$

where $g(n)$ is the actual cost from start to this cell, $h(n)$ is the estimated cost from this cell to the goal (the heuristic), and $f(n)$ is the total estimated path cost through this cell. A* always expands the cell with the **lowest $f(n)$** next.

---

## The Heuristic $h(n)$

Two common choices on a grid:

**Manhattan distance** — for 4-connected grids (no diagonals):

$$
h = |x_1 - x_2| + |y_1 - y_2|
$$

**Euclidean distance** — for 8-connected grids (with diagonals):

$$
h = \sqrt{(x_1 - x_2)^2 + (y_1 - y_2)^2}
$$

The heuristic must never *overestimate* the true cost — if it does, A* can miss the optimal path. A heuristic with this property is called **admissible**.

---

## Cost of Movement

### Manhattan (4-connected)

Four neighbors, each costing **1** unit to enter. From position $(row, col)$:

<table>
  <thead>
    <tr><th>Direction</th><th>Offset</th><th>Cost</th></tr>
  </thead>
  <tbody>
    <tr><td>Up</td><td>(0, +1)</td><td>1</td></tr>
    <tr><td>Down</td><td>(0, -1)</td><td>1</td></tr>
    <tr><td>Left</td><td>(-1, 0)</td><td>1</td></tr>
    <tr><td>Right</td><td>(+1, 0)</td><td>1</td></tr>
  </tbody>
</table>

### Euclidean (8-connected)

Eight neighbors — cardinal moves cost **1**, diagonal moves cost $\sqrt{2} \approx 1.41$:

<table>
  <thead>
    <tr><th>Direction</th><th>Offset</th><th>Cost</th></tr>
  </thead>
  <tbody>
    <tr><td>Up</td><td>(0, +1)</td><td>1.00</td></tr>
    <tr><td>Down</td><td>(0, -1)</td><td>1.00</td></tr>
    <tr><td>Left</td><td>(-1, 0)</td><td>1.00</td></tr>
    <tr><td>Right</td><td>(+1, 0)</td><td>1.00</td></tr>
    <tr><td>Top-Left</td><td>(-1, +1)</td><td>1.41</td></tr>
    <tr><td>Top-Right</td><td>(+1, +1)</td><td>1.41</td></tr>
    <tr><td>Bottom-Left</td><td>(-1, -1)</td><td>1.41</td></tr>
    <tr><td>Bottom-Right</td><td>(+1, -1)</td><td>1.41</td></tr>
  </tbody>
</table>

---

## The Algorithm
```
1. Create OPEN list (nodes to explore) and CLOSED list (nodes already explored)
2. Add START to OPEN
3. Loop while OPEN is not empty:
   a. Pick node with lowest f(n) from OPEN → call it CURRENT
   b. If CURRENT == GOAL → path found, reconstruct and return
   c. Move CURRENT from OPEN to CLOSED
   d. For each NEIGHBOR of CURRENT:
      - Skip if obstacle
      - Skip if already in CLOSED
      - Compute tentative g = g(CURRENT) + move cost
      - If NEIGHBOR already in OPEN with worse g → update g, h, f, parent
      - If NEIGHBOR not in OPEN → add it, set parent = CURRENT
4. If OPEN is empty and goal never reached → no path exists
```

---

## Path Reconstruction

Once the goal is reached, we trace back through parent pointers to recover the path:
```
1. Create path list
2. step = goal
3. While step != start:
   - push step to path
   - step = parent[step]
4. Push start
5. Reverse
```

---

## C++ Implementation Highlights

### Displaying the Path on the Grid
```cpp
void Astar::displayPathOnGrid(Grid& grid,
                               std::vector<std::pair<int,int>> path)
{
    auto display = displayGrid(grid);
    for (const auto& p : path)
        display[p.first][p.second] = '*';

    display[start.x][start.y] = 'S';
    display[goal.x][goal.y]   = 'G';

    for (char i = 0; i < grid.rows; ++i) {
        for (char j = 0; j < grid.cols; ++j)
            std::cout << display[i][j] << " ";
        std::cout << "\n";
    }
}
```

### Grid Display Helper
```cpp
std::vector<std::vector<char>> displayGrid(Grid& grid) {
    std::vector<std::vector<char>> display(
        grid.rows, std::vector<char>(grid.cols));

    for (int i = 0; i < grid.rows; ++i) {
        for (int j = 0; j < grid.cols; ++j) {
            if      (grid.cells[i][j] == -1) display[i][j] = '#'; // obstacle
            else if (grid.cells[i][j] ==  5) display[i][j] = '&'; // light terrain
            else if (grid.cells[i][j] == 15) display[i][j] = '%'; // heavy terrain
            else                             display[i][j] = '.'; // free
        }
    }
    return display;
}
```

---

## Obstacles and Terrain Costs

### Adding Obstacles

Set cell values to `-1` and skip them during neighbor expansion:
```cpp
void createObstacle(Grid& grid) {
    for (int i = 2; i < 6; ++i)
        for (int j = 3; j < 6; ++j)
            grid.cells[i][j] = -1;
}
```

### Adding Terrain Costs (Hills / Mud)

In real navigation, not all paths are equal — a tarred road is easier than a muddy slope. Assign higher initial cell values to make those cells more costly to enter. A* naturally routes around them when a cheaper path exists:
```cpp
void addHillTerrain(Grid& grid) {
    for (int i = 4; i < 7; ++i)
        for (int j = 6; j < 8; ++j)
            grid.cells[i][j] = 15;  // higher cost = harder terrain
}
```

The higher initial value is added into $g(n)$ when the cell is entered, so A* naturally routes around it when a cheaper path exists.

---

## Sample Results

The figures below show two example runs. `S` is the start, `G` is the goal, `*` marks the planned path, `#` marks obstacles, and `%` marks high-cost terrain.

<img src="/assets/blog/motion-planning/astar/astar1.png" alt="A* path planning example 1" style="width:100%; max-width:600px; display:block; margin:0 auto; margin-bottom:16px;">

<img src="/assets/blog/motion-planning/astar/astar2.png" alt="A* path planning example 2" style="width:100%; max-width:600px; display:block; margin:0 auto;">

---

## Summary

<table>
  <thead>
    <tr><th>Concept</th><th>Key Idea</th></tr>
  </thead>
  <tbody>
    <tr><td>$f(n) = g(n) + h(n)$</td><td>Balance cost paid vs. cost estimated</td></tr>
    <tr><td>Heuristic</td><td>Manhattan (4-connected) or Euclidean (8-connected)</td></tr>
    <tr><td>Admissibility</td><td>Heuristic must never overestimate</td></tr>
    <tr><td>OPEN / CLOSED</td><td>Track explored and unexplored nodes</td></tr>
    <tr><td>Parent pointers</td><td>Enable path reconstruction from goal to start</td></tr>
    <tr><td>Terrain costs</td><td>Higher cell values → higher $g(n)$ → A* avoids them</td></tr>
  </tbody>
</table>

A* is the workhorse of grid-based path planning in robotics. Its combination of optimality guarantees and practical efficiency makes it the go-to choice before reaching for more sophisticated planners like RRT or PRM.

---

**Code:** [`astarV3.cpp`](https://github.com/sof-danny/cpp_ppp/blob/main/robotics_algorithms/Astar/astarV3.cpp)

---

*Next up: probabilistic planners — RRT and PRM for high-dimensional spaces where grid search becomes intractable.*