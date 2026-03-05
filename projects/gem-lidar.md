---
layout: default
title: GEM Autonomous Vehicle — LiDAR-Based Box Localization
permalink: /projects/gem-lidar/
---

<img class="project-media" src="/assets/gifs/gem_e2/gem_e2_lidar_pose.gif" alt="Gem e2 demo">

<div class="callout">
LiDAR-based obstacle detection with grid density clustering, geometric waypoint planning, and pure pursuit GPS tracking on a Polaris GEM autonomous vehicle.
</div>

<div class="links">
  <a href="https://github.com/sof-danny/Gem-Autonomous-Vehicle-ROS-Projects/tree/main/Vehicle%20Lidar%20based%20positioning/Code" target="_blank">Code</a>
    <a href="https://drive.google.com/file/d/1_hzEcZu6WF5EkOcxCFEr3qmJTYHPZuDt/view?usp=sharing">Video</a>
</div>

---

## Overview

This project implements an autonomous navigation pipeline on a Polaris GEM e2 vehicle using a Velodyne LiDAR sensor. The vehicle detects two physical boxes in its environment, plans a path through the gap between them, and drives to the midpoint using GPS-guided pure pursuit control — all within a ROS framework.

---

## System Architecture

<div class="arch">
  <div class="arch-row">
    <div class="arch-box"><strong>Velodyne LiDAR</strong><br><small>PointCloud2 stream</small></div>
    <div class="arch-arrow">→</div>
    <div class="arch-box"><strong>Box Locator</strong><br><small>Grid density clustering</small></div>
    <div class="arch-arrow">→</div>
    <div class="arch-box"><strong>Waypoint Planner</strong><br><small>Midpoint + GPS transform</small></div>
    <div class="arch-arrow">→</div>
    <div class="arch-box"><strong>Pure Pursuit</strong><br><small>PACMod vehicle control</small></div>
  </div>
  <div class="arch-note">ROS pipeline on Polaris GEM e2 — Novatel GPS + Velodyne LiDAR</div>
</div>

---

## LiDAR Box Detection

The Velodyne point cloud is filtered to retain only points in the forward detection zone — within 2m laterally and at a minimum distance from the vehicle. The surviving points are projected onto a **12×12 occupancy grid** in the XY plane, and the two highest-density cells are identified as box centers.

$$
\text{density}(i,j) = \sum_{\mathbf{p} \in \text{cell}(i,j)} \mathbf{1}[-1.4 \le p_z \le 0]
$$

The two peak-density blocks are returned as the detected obstacle positions.

---

## Waypoint Planning

The planner targets the **midpoint between the two boxes** as the navigation goal, then linearly interpolates 15 waypoints from the vehicle's current position to that target:

$$
\mathbf{w}_i = \mathbf{p}_{\text{curr}} + \frac{i}{N-1}(\mathbf{p}_{\text{dest}} - \mathbf{p}_{\text{curr}}), \quad i = 0, \dots, N-1
$$

Waypoints are then rotated and translated from LiDAR-local frame into GPS world coordinates using the vehicle's heading from the Novatel INS.

---

## Results

<table style="width:100%; border-collapse:collapse; margin:1rem 0;">
  <thead>
    <tr style="background:#f4f4f4;">
      <th style="border:1px solid #ddd; padding:8px;">Component</th>
      <th style="border:1px solid #ddd; padding:8px;">Method</th>
    </tr>
  </thead>
  <tbody>
    <tr><td style="border:1px solid #ddd; padding:8px;">Obstacle Detection</td><td style="border:1px solid #ddd; padding:8px;">Velodyne LiDAR + grid density clustering</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Localization</td><td style="border:1px solid #ddd; padding:8px;">Novatel GPS/INS (NovatelPosition + Inspva)</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Path Planning</td><td style="border:1px solid #ddd; padding:8px;">Linear interpolation to gap midpoint</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Control</td><td style="border:1px solid #ddd; padding:8px;">Pure pursuit via PACMod interface</td></tr>
    <tr><td style="border:1px solid #ddd; padding:8px;">Platform</td><td style="border:1px solid #ddd; padding:8px;">Polaris GEM e2, ROS Noetic</td></tr>
  </tbody>
</table>

---

## Key Observations

- Grid density clustering is simple but effective for detecting large objects like boxes in sparse outdoor point clouds
- Linear waypoint interpolation works well for straight-line approach paths; curved paths would require a spline-based planner
- The coordinate frame transform (LiDAR → GPS world) is critical — a heading error directly shifts the target position