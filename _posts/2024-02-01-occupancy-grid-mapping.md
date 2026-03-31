---
layout: post
title: "Occupancy Grid Mapping: Probabilistic Environment Representation"
date: 2024-02-01
category: mapping
tags: [robotics, perception, mapping, C++, occupancy-grid, ray-tracing]
excerpt: "How robots build a probabilistic map of their environment from noisy sensor data — using log-odds updates, ray tracing, and Bresenham's line algorithm."
---

Occupancy grid mapping is one of the most fundamental techniques in robotics — it lets a robot build a probabilistic map of its environment from noisy sensor data. This post walks through the full algorithm from scratch, implemented in C++.

The full implementation is available here:
- [`grid_mapV2.cpp`](https://github.com/sof-danny/cpp_ppp/blob/main/robotics_algorithms/Occupancy_map/grid_mapV2.cpp)
- [`bresenham_general.cpp`](https://github.com/sof-danny/cpp_ppp/blob/main/robotics_algorithms/Occupancy_map/bresenham_general.cpp)

---

## What is an Occupancy Map?

An occupancy map divides the environment into an $n \times n$ grid of discrete cells. The goal is to estimate the probability that each cell is **free** or **occupied** based on sensor data. Rather than making a binary decision — occupied or not — we maintain a continuous belief over each cell and update it over time as new sensor data arrives.

The map takes two inputs: **sensor data** (scan ranges, angles, and increments from a range sensor) and the **robot pose** (assumed known). A typical sensor data structure looks like:
```cpp
sensor_data = {angle_min: -90°, angle_increment: 45°, ranges: {1.2, 3.5, 2.3}}
robot_pose  = {x, y, theta}
```

---

## Why Log-Odds Instead of Raw Probabilities?

The naive approach — using Bayes' rule to directly update probabilities — works in theory but breaks down in practice. Repeated multiplication of small probabilities causes **numerical instability**, and it is computationally expensive.

Instead, we use the **log-odds representation**:

$$
l = l_{old} + \log\frac{p(m)}{1 - p(m)}
$$

We define two fixed update values ahead of time:

- $l_{free} = -0.4$ — added when a beam passes *through* a cell (evidence it is free)
- $l_{occupied} = +0.85$ — added at the *endpoint* of a beam (evidence it is occupied)

So the update is simply:

$$
l \mathrel{+}= \begin{cases} -0.4 & \text{beam passes through cell} \\ +0.85 & \text{beam endpoint lands on cell} \end{cases}
$$

This keeps all values as simple floating-point additions and avoids numerical issues entirely.

---

## Computing the Endpoint

Each beam in the sensor scan has a range — the distance to the first obstacle it hit. The **endpoint** is the world coordinate where that beam terminated:
```cpp
std::pair<double, double> compute_endpoint(
    double robot_x, double robot_y,
    double theta, double range, double angle)
{
    double x_ep = robot_x + (range * cos(theta + angle));
    double y_ep = robot_y + (range * sin(theta + angle));
    return {x_ep, y_ep};
}
```

The endpoint is where the beam *might* have hit an obstacle — "might" because sensor noise means we can never be fully certain.

---

## Ray Tracing with Bresenham's Line Algorithm

Once we have the endpoint, we need to know which grid cells lie along the path from the robot to the endpoint. Every cell along the path gets a **free** update; the endpoint cell gets an **occupied** update. This is ray tracing.

We use **Bresenham's line algorithm** — originally designed for drawing lines on pixel grids, it turns out to be exactly what we need for tracing rays through a discrete occupancy grid.

### How Bresenham Works

Given a start $(x_0, y_0)$ and end $(x_1, y_1)$, the algorithm finds all intermediate integer grid cells. The key insight is to track an **integer error** instead of a floating-point slope, keeping everything in integer arithmetic:
```
2 * dy added to error each step
Compare error to dx as threshold
If error >= dx: step y, subtract 2*dx from error
Always step x
```

A worked example from $(0,0)$ to $(4,3)$ with slope $= 0.75$, $2dy = 6$, $dx = 4$, $2dx = 8$:
```
x=1: error = 0+6 = 6.   6 ≥ 4? YES → y=1, error = 6-8 = -2
x=2: error = -2+6 = 4.  4 ≥ 4? YES → y=2, error = 4-8 = -4
x=3: error = -4+6 = 2.  2 ≥ 4? NO  → y stays 2
x=4: error = 2+6 = 8.   8 ≥ 4? YES → y=3, error = 8-8 = 0
```

### Two Important Fixes

The basic algorithm has two limitations that must be addressed for our use case:

**1. Direction** — the basic form only works left-to-right. Fix: track sign direction for both axes:
```cpp
int sx = (x1 > x0) ? 1 : -1;
int sy = (y1 > y0) ? 1 : -1;
```

**2. Steep lines** — when $dy > dx$, the algorithm breaks. Fix: detect steep lines and swap the primary axis of iteration:
```cpp
bool steep = (abs(y1 - y0) > abs(x1 - x0));
```

With both fixes, the algorithm correctly handles all 8 octants of movement. The core loop:
```cpp
for (int i = 0; i <= steps; ++i) {
    cells.push_back({x0, y0});

    if (steep) {
        y0 += sy;
        error += 2 * dx;
        if (error >= threshold) {
            error -= 2 * dy;
            x0 += sx;
        }
    } else {
        x0 += sx;
        error += 2 * dy;
        if (error >= threshold) {
            error -= 2 * dx;
            y0 += sy;
        }
    }
}
```

where `steps = steep ? abs(y1-y0) : abs(x1-x0)`.

---

## World to Grid Conversion

The sensor and robot pose are in world coordinates (meters). Before passing them to Bresenham, we convert to grid cell indices:
```cpp
std::pair<int,int> world_to_cell(
    double world_x, double world_y,
    double gridMin_X, double gridMin_Y,
    double offset, double resolution)
{
    int cellX = (int)((world_x - gridMin_X - offset) / resolution);
    int cellY = (int)((world_y - gridMin_Y - offset) / resolution);
    return {cellX, cellY};
}
```

`gridMin` is the bottom-left corner of the grid in world coordinates. `resolution` is the physical size of one cell — with `resolution = 0.2`, a $10 \text{m} \times 10 \text{m}$ environment maps to a $50 \times 50$ cell grid.

---

## The Full Update Loop

With all the pieces in place, the update is a two-level loop:

**Outer loop** — iterate over each scan (robot moves between scans):
```cpp
for (size_t i = 0; i < scans.size(); ++i) {
    int num_rays = scans[i].ranges.size();
    RobotPose pose_ = pose[i];
    // inner loop here
}
```

**Inner loop** — iterate over each beam (ray) within the scan:
```cpp
for (int j = 0; j < num_rays; ++j) {
    auto angle = map.compute_angle(
        scans[i].angle_min, scans[i].angle_increment, j);

    auto [x_ep, y_ep] = map.compute_endpoint(
        pose_.x, pose_.y, pose_.theta,
        scans[i].ranges[j], angle);

    auto [cellEP_x,  cellEP_y]  = world_to_cell(x_ep,    y_ep,
        grid.minX, grid.minY, cell.offset, cell.resolution);
    auto [cellPos_x, cellPos_y] = world_to_cell(pose_.x, pose_.y,
        grid.minX, grid.minY, cell.offset, cell.resolution);

    auto points = bresenham_general(
        cellPos_x, cellPos_y, cellEP_x, cellEP_y);

    // Update cells along the ray (free)
    for (int k = 0; k < (int)points.size() - 1; ++k) {
        auto [x, y] = points[k];
        grid.cells[x][y] += prob.l_free;        // -0.4
    }

    // Update endpoint cell (occupied)
    auto [ex, ey] = points.back();
    grid.cells[ex][ey] += prob.l_occupied;      // +0.85
}
```

---

## Visualizing the Map

Once all scans are processed, we can print the map to the terminal using log-odds → character mappings:

**Simple version** — three states:
```cpp
char cellToChar(float log_odds) {
    if (log_odds >  0.5) return '#';   // occupied
    if (log_odds < -0.5) return '.';   // free
    return '?';                         // unknown
}
```

**Density version** — finer granularity using probability conversion:
```cpp
float toProbability(float log_odds) {
    return 1.0f - (1.0f / (1.0f + std::exp(log_odds)));
}

char toDensityChar(float log_odds) {
    float p = toProbability(log_odds);
    if (p > 0.9)  return '#';   // very occupied
    if (p > 0.7)  return 'o';   // probably occupied
    if (p > 0.55) return '+';   // slightly occupied
    if (p > 0.45) return ' ';   // unknown
    if (p > 0.3)  return '-';   // slightly free
    if (p > 0.1)  return '.';   // probably free
    return ' ';                  // very free
}
```

Printing:
```cpp
for (const auto& row : grid.cells) {
    for (const auto& c : row) {
        std::cout << toDensityChar(c) << " ";
    }
    std::cout << std::endl;
}
```

---

## Summary

The full occupancy grid pipeline is:
```
Sensor scan + robot pose
        │
        ▼
Compute endpoint (world coords)
        │
        ▼
Convert robot pos + endpoint → grid cells
        │
        ▼
Bresenham ray trace → list of cells
        │
        ├── Intermediate cells → l += l_free  (-0.4)
        └── Endpoint cell     → l += l_occupied (+0.85)
        │
        ▼
Repeat for all beams, all scans
        │
        ▼
Render map (log-odds → character)
```

The log-odds update is fast, numerically stable, and naturally handles sensor uncertainty. The Bresenham ray tracer is exact, integer-only, and handles all 8 movement directions. Together they form the core of one of the most widely-used mapping algorithms in mobile robotics.

---

**Code:** [`grid_mapV2.cpp`](https://github.com/sof-danny/cpp_ppp/blob/main/robotics_algorithms/Occupancy_map/grid_mapV2.cpp) · [`bresenham_general.cpp`](https://github.com/sof-danny/cpp_ppp/blob/main/robotics_algorithms/Occupancy_map/bresenham_general.cpp)