---
layout: default
title: PX4 ROS2 CBF Safety Filter
permalink: /projects/px4-cbf/
---

# PX4 + ROS2 Control Barrier Function Safety Filter

<img src="/assets/gifs/cbf_drone.gif" class="project-media" alt="PX4 ROS2 CBF safety filter demo">

<div class="callout">
<b>One-liner:</b> A real-time safety filter that minimally modifies a nominal controller to enforce obstacle-avoidance constraints for a PX4 drone via Control Barrier Functions (CBFs).
</div>

## Problem
Autonomous drones often follow a **nominal controller** (e.g., trajectory tracking), but that controller may violate safety constraints such as minimum distance to obstacles.  
The goal is to enforce safety **without redesigning** the full controller.

## Approach
This project implements a **CBF-based safety filter** that sits between the nominal controller and PX4.

At each control step, given a nominal input \(u_{\text{nom}}\), we solve a small optimization problem:

\[
\min_{u}\;\; \lVert u - u_{\text{nom}} \rVert^2
\]

subject to safety constraints defined by a barrier function \(h(x)\) (safe set: \(h(x)\ge 0\)):

\[
\dot{h}(x,u) + \alpha h(x) \ge 0
\]

This guarantees forward invariance of the safe set under standard CBF conditions while staying as close as possible to the nominal input.

## System Architecture
<div class="arch">
  <div class="arch-row">
    <span class="arch-box">Planner / Nominal Controller</span>
    <span class="arch-arrow">→</span>
    <span class="arch-box"><b>CBF Safety Filter (QP)</b><br>(ROS2 node)</span>
    <span class="arch-arrow">→</span>
    <span class="arch-box">PX4 SITL / Vehicle</span>
  </div>
  <div class="arch-note">
    State feedback (pose/velocity) is used to evaluate barrier constraints and solve the QP in real time.
  </div>
</div>

## Key Features
- **Real-time safety filter** implemented as a ROS2 node
- **QP solved per control cycle** to enforce safety constraints
- Integrates with **PX4 SITL**
- Supports obstacle-avoidance constraints using a distance-based barrier function

## Results (what to show here)
If you have any of these, add them (even one is enough):
- Minimum distance to obstacle maintained (plot)
- Tracking performance before vs after filtering (plot)
- QP solve time statistics (mean / p95)
- Failure cases and how constraints prevent them

<div class="callout">
<b>TODO (recommended):</b> Add 1 plot + 1 table: (1) minimum distance vs time, (2) QP solve-time stats.
</div>

## How to Run (minimal)
1. Build / source ROS2 workspace  
2. Launch PX4 SITL  
3. Start the safety filter node and run a scenario

(Replace this with your actual commands once you confirm the current launch flow in your repo.)

## Tech Stack
- ROS2
- PX4 SITL
- Python / C++
- Control Barrier Functions (CBFs)
- Quadratic Programming (QP)

## Links
- **Code:** [px4-ros2-cbf-safety-filter](https://github.com/sof-danny/px4-ros2-cbf-safety-filter)  
- **README:** [Documentation](https://github.com/sof-danny/px4-ros2-cbf-safety-filter/blob/main/README.md)