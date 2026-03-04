---
layout: default
title: PX4 ROS2 CBF Safety Filter
permalink: /projects/px4-cbf/
---

# PX4 + ROS 2: CBF Safety Filter

Control Barrier Function (CBF) safety filter demo for UAV obstacle avoidance using **PX4 SITL**, **ROS 2 Humble**, and **gz-sim Harmonic**.

<div class="gif-table" markdown="1">

    | No CBF (Collision) | CBF Only (Zigzag) | CBF + Sliding (Smooth) |
    |:------------------:|:-----------------:|:----------------------:|
    | ![No CBF](/assets/px4-cbf/gifs/no_cbf_fast.gif) | ![CBF Only](/assets/px4-cbf/gifs/cbf_no_sliding_fast.gif) | ![CBF + Sliding](/assets/px4-cbf/gifs/cbf_sliding_fast.gif) |
    | Crashes at obstacle 1 | Safe but slow (87s) | Safe and fast (64s) |
    
</div>

## Problem
Autonomous drones often follow a **nominal controller** (e.g., trajectory tracking), but that controller may violate safety constraints such as minimum distance to obstacles.  
The goal is to enforce safety **without redesigning** the full controller.


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


## Features
- QP-based CBF implementation with formal safety guarantees  
- 12-obstacle course (30m) with varying radii  
- Tangential sliding to avoid local minima  
- Toggle CBF on/off to compare safe vs unsafe behavior  
- Logs barrier function \(h(x)\) to verify safety constraint satisfaction  

## CBF Mathematical Formulation

**Single-integrator dynamics** (position-controlled UAV):

\[
\dot{x} = u \quad \text{where } x = [p_x, p_y],\; u = [v_x, v_y]
\]

**Barrier function** for a circular obstacle:

\[
h(x) = \|x - x_{\text{obs}}\|^2 - d_{\text{safe}}^2
\]

- \(h>0\): safe region  
- \(h=0\): safety boundary  
- \(h<0\): unsafe (collision imminent)

**CBF constraint** (forward invariance of safe set):

\[
\dot{h} + \alpha h \ge 0
\qquad\Longleftrightarrow\qquad
\nabla h(x)^\top u \ge -\alpha h(x)
\]

**QP-CBF formulation**:

\[
\begin{aligned}
\min_{u,\delta}\quad & \|u-u_{\text{nom}}\|^2 + \lambda\delta^2 \\
\text{s.t.}\quad
& \nabla h_i(x)^\top u \ge -\alpha h_i(x) - \delta,\;\;\forall i \\
& \|u\| \le v_{\max} \\
& \delta \ge 0
\end{aligned}
\]

Where \(\delta\) is a relaxation variable to ensure feasibility. The QP solver (**OSQP via cvxpy**) finds the closest safe velocity to the nominal command.

## Results
We evaluate the safety filter by monitoring the barrier function \(h(x)\) during simulation.

<img src="/assets/gifs/px4-cbf/plots/barrier_function.png" class="project-media">
<p class="caption">
Barrier function evolution under different controllers. Safety is guaranteed when \(h(x) \ge 0\).
</p>

The figure above shows the evolution of the barrier function \(h(x)\) over time for three cases:

- **No CBF (red):** the nominal controller violates the safety constraint, causing the barrier function to become negative. This corresponds to the system entering an unsafe region.
- **CBF only (orange):** the control barrier function modifies the nominal input to maintain \(h(x) \ge 0\), enforcing safety.
- **CBF + sliding controller (blue):** combining the safety filter with a sliding-mode controller improves robustness and keeps the system further away from the safety boundary.

The dashed line represents the **safety boundary \(h(x)=0\)**.  
Maintaining \(h(x) \ge 0\) guarantees forward invariance of the safe set.

These results demonstrate that the safety filter successfully enforces safety constraints while minimally modifying the nominal control input.

![Trajectory Comparison](/assets/px4-cbf/plots/trajectory_comparison.png)



## Prerequisites
- Ubuntu 22.04 + ROS 2 Humble  
- gz-sim Harmonic (v8)  
- PX4 SITL built at `~/ros2_ws/src/PX4-Autopilot`  
- MicroXRCE-DDS-Agent  
- Python: `pip install cvxpy osqp matplotlib numpy`

## How to Run (minimal)
```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws/px4-ros2-cbf-safety-filter
colcon build
```

## Tech Stack
- ROS2
- PX4 SITL
- Python / C++
- Control Barrier Functions (CBFs)
- Quadratic Programming (QP)

## Links
- **Code:** [px4-ros2-cbf-safety-filter](https://github.com/sof-danny/px4-ros2-cbf-safety-filter)  
- **README:** [Documentation](https://github.com/sof-danny/px4-ros2-cbf-safety-filter/blob/main/README.md)



