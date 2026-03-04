---
layout: default
title: PX4 ROS2 CBF Safety Filter
permalink: /projects/px4-cbf/
---

# PX4 + ROS 2: CBF Safety Filter

Control Barrier Function (CBF) safety filter demo for UAV obstacle avoidance using **PX4 SITL**, **ROS 2 Humble**, and **gz-sim Harmonic**.

<table class="gif-table">
  <thead>
    <tr>
      <th align="center">No CBF (Collision)</th>
      <th align="center">CBF Only (Zigzag)</th>
      <th align="center">CBF + Sliding (Smooth)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="center"><img src="/assets/gifs/px4-cbf/gifs/no_cbf_fast.gif" alt="No CBF"></td>
      <td align="center"><img src="/assets/gifs/px4-cbf/gifs/cbf_no_sliding_fast.gif" alt="CBF Only"></td>
      <td align="center"><img src="/assets/gifs/px4-cbf/gifs/cbf_sliding_fast.gif" alt="CBF + Sliding"></td>
    </tr>
    <tr>
      <td align="center">Crashes at obstacle 1</td>
      <td align="center">Safe but slow (87s)</td>
      <td align="center">Safe and fast (64s)</td>
    </tr>
  </tbody>
</table>



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

$$
\dot{x} = u \quad \text{where } x = [p_x, p_y],\; u = [v_x, v_y]
$$

**Barrier function** for a circular obstacle:

$$
h(x) = \|x - x_{\text{obs}}\|^2 - d_{\text{safe}}^2
$$

- $h>0$: safe region  
- $h=0$: safety boundary  
- $h<0$: unsafe (collision imminent)

**CBF constraint** (forward invariance of safe set):

$$
\dot{h} + \alpha h \ge 0
\qquad\Longleftrightarrow\qquad
\nabla h(x)^\top u \ge -\alpha h(x)
$$

**QP-CBF formulation**:

$$
\begin{aligned}
\min_{u,\delta}\quad & \|u-u_{\text{nom}}\|^2 + \lambda\delta^2 \\
\text{s.t.}\quad
& \nabla h_i(x)^\top u \ge -\alpha h_i(x) - \delta,\;\;\forall i \\
& \|u\| \le v_{\max} \\
& \delta \ge 0
\end{aligned}
$$

Where $\delta$ is a relaxation variable to ensure feasibility. The QP solver (**OSQP via cvxpy**) finds the closest safe velocity to the nominal command.

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


<img src="/assets/gifs/px4-cbf/plots/trajectory_comparison.png" class="project-media">
<p class="caption">
Comparison of trajectory for different scenario.
</p>

## Comparison Summary
<table style="width:100%; border-collapse:collapse; margin:1rem 0;">
  <thead>
    <tr style="background:#f4f4f4;">
      <th style="border:1px solid #ddd; padding:8px;">Metric</th>
      <th style="border:1px solid #ddd; padding:8px;">CBF OFF</th>
      <th style="border:1px solid #ddd; padding:8px;">CBF Only</th>
      <th style="border:1px solid #ddd; padding:8px;">CBF + Sliding</th>
    </tr>
  </thead>
  <tbody>
    <tr><td style="border:1px solid #ddd; padding:8px;">Min barrier h(x)</td><td style="border:1px solid #ddd; padding:8px;">-0.975</td><td style="border:1px solid #ddd; padding:8px;">0.036</td><td style="border:1px solid #ddd; padding:8px;">0.491</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Min distance (m)</td><td style="border:1px solid #ddd; padding:8px;">1.505</td><td style="border:1px solid #ddd; padding:8px;">1.810</td><td style="border:1px solid #ddd; padding:8px;">1.932</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Safety violated</td><td style="border:1px solid #ddd; padding:8px; color:red;">YES</td><td style="border:1px solid #ddd; padding:8px; color:green;">NO</td><td style="border:1px solid #ddd; padding:8px; color:green;">NO</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Collision</td><td style="border:1px solid #ddd; padding:8px; color:red;">YES</td><td style="border:1px solid #ddd; padding:8px; color:green;">NO</td><td style="border:1px solid #ddd; padding:8px; color:green;">NO</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Reached target</td><td style="border:1px solid #ddd; padding:8px; color:red;">NO (crash)</td><td style="border:1px solid #ddd; padding:8px; color:green;">YES</td><td style="border:1px solid #ddd; padding:8px; color:green;">YES</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Flight time (s)</td><td style="border:1px solid #ddd; padding:8px;">7.5</td><td style="border:1px solid #ddd; padding:8px;">87.4</td><td style="border:1px solid #ddd; padding:8px;">64.2</td></tr>
  </tbody>
</table>



Key observations:
- **CBF Only**: Safe but inefficient (zigzags around obstacles, 36% slower)
- **CBF + Sliding**: Safe and efficient (smooth curves, better safety margin)



## Prerequisites
- Ubuntu 22.04 + ROS 2 Humble  
- gz-sim Harmonic (v8)  
- PX4 SITL built at `~/ros2_ws/src/PX4-Autopilot`  
- MicroXRCE-DDS-Agent  
- Python: `pip install cvxpy osqp matplotlib numpy`

## How to Run

### Terminal 1: Agent
`./tools/start_agent.sh`

### Terminal 2: PX4 + gz-sim with obstacles
`./tools/spawn_drone.sh`

### Terminal 3: Controller
`source /opt/ros/humble/setup.bash`
`source install/setup.bash`

### WITH CBF (safe):
`ros2 launch cbf_demo_bringup cbf_demo.launch.py use_cbf:=true`

### WITHOUT CBF (collision):
`ros2 launch cbf_demo_bringup cbf_demo.launch.py use_cbf:=false`

### Terminal 4: RViz2
`rviz2 -d $(ros2 pkg prefix cbf_demo_bringup)/share/cbf_demo_bringup/config/cbf_viz.rviz`

## Tech Stack
- ROS2
- PX4 SITL
- Python / C++
- Control Barrier Functions (CBFs)
- Quadratic Programming (QP)

## Links
- **Code:** [px4-ros2-cbf-safety-filter](https://github.com/sof-danny/px4-ros2-cbf-safety-filter)  
- **README:** [Documentation](https://github.com/sof-danny/px4-ros2-cbf-safety-filter/blob/main/README.md)

## References
- Ames et al., "Control Barrier Functions: Theory and Applications", 2019
- Federica Ferraguti et al. "Safety and Efficiency in Robotics: the Control Barrier Functions Approach", 2022
- [CBF Tutorial Repo](https://github.com/sof-danny/px4-ros2-cbf-safety-filter/tree/main/cbf/code)

