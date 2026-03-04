---
layout: default
title: PX4 ROS2 CBF Safety Filter
permalink: /projects/px4-cbf/
---

# PX4 + ROS2 Control Barrier Function Safety Filter

## <span id="px4-cbf"></span> PX4 + ROS2 Control Barrier Function Safety Filter

<img src="/assets/gifs/cbf_drone.gif" class="project-media">

### Overview
This project implements a **Control Barrier Function (CBF) safety filter** for a PX4-controlled drone using ROS2.  
The filter modifies the nominal control input to guarantee safety constraints such as obstacle avoidance while preserving the intended trajectory.

### Key Idea

Given a nominal control input \(u_{nom}\), the safety filter solves a small optimization problem:

\[
\min_u ||u - u_{nom}||^2
\]

subject to safety constraints defined by a **control barrier function**:

\[
\dot h(x,u) + \alpha h(x) \ge 0
\]

This ensures the system remains inside a safe set while minimally altering the desired control input.

### Features

- ROS2 node implementing a **real-time safety filter**
- Integration with **PX4 simulation**
- Quadratic program solved at each control step
- Enforces obstacle avoidance constraints

### Technology

- ROS2
- PX4 SITL
- Python / C++
- Control Barrier Functions

### Links

### Links

- **Code:** [GitHub Repository](https://github.com/sof-danny/px4-ros2-cbf-safety-filter)  
- **Documentation:** [Project README](https://github.com/sof-danny/px4-ros2-cbf-safety-filter/blob/main/README.md)