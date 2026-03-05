---
layout: post
title: "GEM e2 Voronoi Navigation"
permalink: /projects/voronoi/
---

<img src="/assets/gifs/gem_e2/voronoi_cropped.gif" alt="Voronoi Navigation Demo" style="width:100%; max-width:600px; display:block; margin:0 auto;">

## Overview

This project implements a full autonomous summoning pipeline on the Polaris GEM e2 vehicle. Given a pre-mapped obstacle environment and a destination coordinate, the vehicle computes a collision-free path using Voronoi-based planning, smooths it with a Bézier curve, and executes it using a pure pursuit controller with GPS feedback. A parallel safety module continuously monitors the ZED2 camera feed and triggers emergency braking if a pedestrian is detected in the vehicle's path.

---

## Hardware & Software Stack

- **Platform:** Polaris GEM e2 autonomous vehicle
- **Localization:** NovAtel GNSS/INS (`/novatel/inspva`) — latitude, longitude, heading
- **Safety sensor:** ZED2 stereo camera
- **Control interface:** PACMod (via ROS topics)
- **Framework:** ROS (Python)
- **Key libraries:** SciPy (Bézier), OpenCV, dlib, `alvinxy` (GPS → local XY)

---

## System Architecture

The pipeline runs as two parallel ROS nodes:
```
[summon_voronoi.py]              [summon_control.py]
       │                                │
       ├── Load obstacle map (.npy)     ├── Subscribe /novatel/inspva
       ├── Get current GPS position     ├── Subscribe /mp0/BeizerPath
       ├── Voronoi path planning        ├── Pure pursuit steering
       ├── Bézier curve smoothing       ├── PID velocity control
       └── Publish /mp0/BeizerPath      └── Publish steer/accel/brake

                    [braking.py]
                         │
                         ├── Subscribe ZED2 RGB image
                         ├── HOG pedestrian detection
                         └── Emergency brake if person detected
```

---

## Stage 1 — Voronoi Path Planning

### Obstacle Map

The environment is represented as a pre-built binary occupancy grid stored as `voronoi_obstacle_map.npy`. Occupied cells are obstacles; free cells are drivable space. This map is loaded once at startup.

### GPS to Local XY

The NovAtel INS provides latitude, longitude, and azimuth heading. These are converted to a local Cartesian frame using `alvinxy.ll2xy()`, referenced to a fixed origin `(40.0928563, -88.2359994)`. The vehicle's rear axle offset is corrected by projecting along the heading direction:

$$
x_{curr} = x_{gps} - \delta \cos(\psi), \quad y_{curr} = y_{gps} - \delta \sin(\psi)
$$

where $\delta = 0.46$ m is the sensor-to-axle offset and $\psi$ is the yaw angle.

### Voronoi Planning

A `Voronoi` object is constructed from the obstacle map. Given the current vehicle position and the destination — both transformed to image coordinates — it computes a discrete path through the Voronoi diagram of the free space. Voronoi paths naturally stay maximally far from all obstacles, providing a safety margin by construction.

### Bézier Smoothing

The discrete Voronoi path is often jagged and unsuitable for a vehicle with limited steering range. It is smoothed using a Bézier curve (`bezier_curve()`), which fits a smooth parametric curve through the waypoints. The function iteratively increases control point repetition until the resulting curve is verified to be obstacle-free via `voronoi.are_waypoints_clear()`, enforcing a minimum clearance of 3.0 m from any obstacle.

The smoothed path (1000 sampled points) is published as an image message on `/mp0/BeizerPath` and a visualization is published on `/mp0/VoronoiPath`.

---

## Stage 2 — Pure Pursuit Control

### Path Subscription

The control node subscribes to `/mp0/BeizerPath` and converts the image message back to ground-plane coordinates using the inverse of the image-to-ground transform.

### Pure Pursuit Steering

At each control step (50 Hz), the node computes the vehicle's current pose `(curr_x, curr_y, curr_deviation)` and scans the path for two lookahead points:

- **Near lookahead** (`look_ahead = 5 m`) — used for steering computation
- **Far lookahead** (`far_look_ahead = 8 m`) — used to anticipate upcoming turns and pre-emptively slow down

The heading angle to the near lookahead point $\alpha$ is computed and fed into the pure pursuit steering formula:

$$
\delta = \arctan\left(\frac{2 L \sin(\alpha)}{d}\right)
$$

where $L = 1.75$ m is the wheelbase and $d$ is the distance to the lookahead point. The steering angle is clamped to $\pm 35°$ and mapped to a steering wheel angle via a quadratic fit:

$$
\theta_{steer} = -0.1084 \cdot \delta_{deg}^2 + 21.775 \cdot \delta_{deg}
$$

### PID Velocity Control

A PID controller (`kp=1.2, ki=0.2, kd=0.6`) tracks a desired velocity of 0.4 m/s. When the vehicle is approaching a turn (either lookahead angle exceeds 10°), the desired velocity is halved to 0.2 m/s to ensure safe cornering. Vehicle velocity is low-pass filtered with a 4th-order Butterworth filter (cutoff 1.2 Hz) before being used in control. The acceleration output is clamped to `[0.1, 0.4]`, with a higher minimum of 0.35 when the vehicle is stopped to overcome static friction.

The vehicle stops and brakes when it comes within 4.0 m of the destination.

---

## Stage 3 — Pedestrian Safety

In parallel, `braking.py` runs a HOG-based pedestrian detector (OpenCV's default people detector) on every ZED2 camera frame. If a pedestrian is detected with confidence above 0.13, the node publishes a full brake command (`f64_cmd=1.0`) to `/pacmod/as_rx/brake_cmd`, overriding the control node. When the frame is clear, the brake is released.

---

## Control Flow Summary
```
GPS + heading
      │
      ▼
Local XY pose  ──►  Image coordinates
                           │
                    Voronoi path planning
                           │
                    Bézier smoothing + clearance check
                           │
                    /mp0/BeizerPath (ROS Image)
                           │
                    Pure pursuit → steering angle
                    PID velocity → accel command
                           │
                    PACMod → steer / accel / brake / gear
                           │
                    ◄── ZED2 HOG safety check (parallel)
```

---

## Key Design Decisions

**Why Voronoi for planning?** Voronoi diagrams maximize clearance from obstacles by construction — the path always runs equidistant between the nearest obstacles on either side. This is ideal for a vehicle operating in a structured outdoor environment where the obstacle layout is known in advance.

**Why Bézier smoothing?** The GEM e2 has a finite steering range and cannot track sharp corners. Bézier curves produce $C^1$-continuous paths that respect the vehicle's kinematic constraints. The iterative clearance check ensures smoothing never cuts through obstacles.

**Why two lookahead distances?** The near lookahead drives immediate steering corrections; the far lookahead provides anticipation — the vehicle slows down before entering a turn rather than reacting to it mid-corner.

**Why HOG for pedestrian detection?** HOG is fast, runs on CPU, and requires no GPU or pretrained deep model — well-suited for a safety-critical real-time check that needs to run in parallel without competing for compute resources with the control loop.

---

## Links

- [Video Demo](https://drive.google.com/file/d/1ixq9S3DUYFpo9bVkba6YkmbjmpPkaKGe/view?usp=sharing)
- [Code](https://github.com/sof-danny/Gem-Autonomous-Vehicle-ROS-Projects/tree/main/Voronoi%20obstacle%20map%20and%20autonomous%20navigation/Code)