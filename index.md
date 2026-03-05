---
layout: default
title: Home
---

<div class="hero">
  <img class="portrait" src="/assets/img/sam.jpeg" alt="Portrait of Samuel Folorunsho" />

  <h1>Samuel Folorunsho</h1>

  <p class="subtitle">Robotics & Autonomy</p>

  <p class="bio">
  Hello and welcome. I'm Samuel Folorunsho, a graduate student at the University of Illinois Urbana-Champaign (UIUC). 
  My work focuses on perception-driven autonomy, motion planning, safe control, and optimal trajectory generation 
  for multi-agent robotic systems. I design and implement end-to-end autonomy stacks using ROS2 and C++/Python, 
  and evaluate them across both physical robotic platforms and modern simulation environments. 
  Please see <a href="/about/">this page</a> for more info about me.
</p>

  <div class="contact">
    <a href="https://github.com/sof-danny" target="_blank" rel="noopener">GitHub</a>
    <a href="mailto:folorunshosamuel001@gmail.com">Email</a>
    <a href="https://www.linkedin.com/in/samuelofolorunsho/" target="_blank" rel="noopener">LinkedIn</a>
      <a href="/assets/cv/SamFolorunshoCV_Feb11_26_redacted.pdf" target="_blank" rel="noopener">Resume</a>
    <a href="https://scholar.google.com/citations?user=vsYAVNYAAAAJ" target="_blank" rel="noopener">Google Scholar</a>
  </div>

  <div class="badges">
      <span class="badge">Robotics</span>
      <span class="badge">Perception & Mapping</span>
      <span class="badge">Planning & Control</span>
      <span class="badge">Multi-Agent Systems</span>
      <span class="badge">Safe & Optimal Control</span>
      <span class="badge">Computer Vision</span>
  </div>

  <div class="badges">
      <span class="badge">ROS / ROS2</span>
      <span class="badge">C++ / Python</span>
      <span class="badge">MATLAB / Simulink</span>
      <span class="badge">Gazebo / Isaac Sim / MuJoCo</span>
  </div>



<h2 class="section-title">Featured Projects</h2>

<div class="grid">

  <div class="card">
    <img class="project-media" src="/assets/gifs/multi_agent.gif" alt="Multi agent inspection demo">
    <h3>Multi-Agent Infrastructure Inspection</h3>
    <p>Coordinated planning + task allocation for scalable infrastructure monitoring in simulation (Proprietary).</p>
    <div class="meta">
      <span class="tag">Multi-agent</span><span class="tag">Planning</span><span class="tag">ROS 2</span> <span class="tag">Gazebo</span>
    </div>
    <div class="links">
      <a href="https://drive.google.com/file/d/13R-W2K8bESukeauyLw2ZlrKn6mjwQMIj/view?usp=sharing">Video</a>
    </div>
  </div>

  <div class="card">
      <img class="project-media" src="/assets/gifs/gem_e2/ped_distance.gif" alt="Ped distance demo">
    <h3>GEM e2 Pedestrian Distance Control</h3>
    <p>PID-based pedestrian follower using bounding box height as a distance proxy.</p>
    <div class="meta">
      <span class="tag">Autonomous Driving</span><span class="tag">Stereo Camera</span><span class="tag">Navigation Safety</span>
    </div>
    <div class="links">
      <a href="/projects/gem_dist_to_ped/">Write-up</a>
      <a href="https://drive.google.com/file/d/1dnmHyaLZsYDYlAtzy72SAb3eZUjuYix9/view?usp=sharing">Video</a>
      <a href="https://github.com/sof-danny/Gem-Autonomous-Vehicle-ROS-Projects/tree/main/Controlling%20distance%20to%20a%20pedestrian/Code">Code</a>
    </div>
  </div>

  <div class="card">
       <img class="project-media" src="/assets/gifs/px4_cbf.gif" alt="PX4 CBF demo">
    <h3>PX4 + ROS 2 CBF Safety Filter</h3>
    <p>Constraint-enforcing safety filter for PX4-controlled drone via control barrier functions.</p>
    <div class="meta">
      <span class="tag">CBF</span><span class="tag">PX4</span><span class="tag">ROS 2</span>
    </div>
    <div class="links">
      <a href="https://github.com/sof-danny/px4-ros2-cbf-safety-filter">Code</a>
      <a href="/projects/px4-cbf/">Write-up</a>
        <a href="https://drive.google.com/file/d/1Plm4kVgCq5tP8E9XnPCEGSzsnyfAlZP6/view?usp=sharing">Video</a>
    </div>
  </div>

  <div class="card">
       <img class="project-media" src="/assets/gifs/vr_ur3.gif" alt="UR3 VR demo">
    <h3>VR Teleoperation for UR3</h3>
    <p>VR interface for UR3 end-effector control with a ROS bridge.</p>
    <div class="meta">
      <span class="tag">VR</span><span class="tag">Manipulation</span><span class="tag">ROS</span>
    </div>
    <div class="links">
      <a href="/papers/vr_ur3.pdf" target="_blank">Write-up</a>
        <a href="https://github.com/sof-danny/vr_ur3">Code</a>
      <a href="https://drive.google.com/file/d/1JQicGon3gMT7v2XJytUEQzuHEKDI03KX/view">Video</a>
    </div>
  </div>

  <div class="card">
       <img class="project-media" src="/assets/gifs/husky_cbf.gif" alt="Husky Depth CBF demo">
    <h3>Husky Depth-Camera CBF Safety</h3>
    <p>Perception-to-safety pipeline using depth sensing and barrier-style constraints.</p>
    <div class="meta">
      <span class="tag">Perception</span><span class="tag">CBF</span><span class="tag">Mobile Robot</span>
    </div>
    <div class="links">
      <!-- <a href="/projects/#husky-cbf">Write-up</a> -->
      <a href="https://drive.google.com/file/d/1FvPmmdOXzjMrN9dJuLUZ_tppwugBukiE/view?usp=sharing">Video</a>
    </div>
  </div>


 <div class="card">
      <img class="project-media" src="/assets/gifs/gem_e2/gem_e2_lidar_pose.gif" alt="Gem e2 demo">
    <h3>GEM e2 LiDAR Positioning</h3>
    <p>LiDAR-based box detection with grid density clustering, geometric waypoint planning through the gap, and pure pursuit GPS tracking on a GEM autonomous vehicle. </p>
    <div class="meta">
      <span class="tag">Autonomous Driving</span><span class="tag">LiDAR</span><span class="tag">Localization & Planning</span>
    </div>
    <div class="links">
        <a href="/projects/gem-lidar/">Write-up</a>
      <a href="https://drive.google.com/file/d/1_hzEcZu6WF5EkOcxCFEr3qmJTYHPZuDt/view?usp=sharing">Video</a>
      <a href="https://github.com/sof-danny/Gem-Autonomous-Vehicle-ROS-Projects/tree/main/Vehicle%20Lidar%20based%20positioning/Code">Code</a>
    </div>
  </div>


<div class="card">
    <img class="project-media" src="/assets/gifs/tuav/multiple_trajectory.gif" alt="TUAV demo">
  <h3>Tethered UAV Nonlinear Backstepping Control</h3>
  <p>Nonlinear cascade controller for a tethered UAV connected to a ground winch via a flexible catenary tether — achieving centimeter-scale hover accuracy and robust disturbance rejection under stochastic wind.</p>
  <div class="meta">
    <span class="tag">Nonlinear Control</span>
    <span class="tag">UAV</span>
    <span class="tag">Backstepping</span>
    <span class="tag">Lyapunov Stability</span>
  </div>
  <div class="links">
    <a href="https://arxiv.org/pdf/2412.09502" target="_blank">Paper</a>
    <a href="https://github.com/AUVSL/TUAV_system_control" target="_blank">Code</a>
  </div>
</div>


<div class="card">
    <img class="project-media" src="/assets/gifs/gem_e2/voronoi_cropped.gif" alt="Voronoi demo">
  <h3>GEM e2 Voronoi Navigation</h3>
  <p>LiDAR-based obstacle mapping with Voronoi diagram path planning and GPS-guided pure pursuit control for autonomous summoning on a Polaris GEM vehicle.</p>
  <div class="meta">
    <span class="tag">Autonomous Driving</span>
    <span class="tag">LiDAR</span>
    <span class="tag">Path Planning</span>
    <span class="tag">Voronoi</span>
  </div>
  <div class="links">
    <a href="https://github.com/sof-danny/Gem-Autonomous-Vehicle-ROS-Projects/tree/main/Voronoi%20obstacle%20map%20and%20autonomous%20navigation/Code" target="_blank">Code</a>
      <a href="https://drive.google.com/file/d/1ixq9S3DUYFpo9bVkba6YkmbjmpPkaKGe/view?usp=sharing" target="_blank">Video</a>
      <a href="/projects/voronoi/" target="_blank">Write-up</a>
  </div>
</div>

<div class="card">
  <img class="project-media" src="/assets/gifs/ur3/robot_safety.gif" alt="Human Robot Colab demo">
  <h3>Safety System for Human-Robot Interaction</h3>
  <p>Vision-based safety system for a UR3 robotic arm using HSV color blob detection to identify human presence and automatically halt motion.</p>
  <div class="meta">
    <span class="tag">Safety</span>
    <span class="tag">Perception</span>
    <span class="tag">Human-Robot Interaction</span>
    <span class="tag">Camera</span>
  </div>
  <div class="links">
    <a href="https://github.com/sof-danny/Human_Robot_Interaction/tree/main/ece598hri-fa25-mps/ws_hri" target="_blank">Code</a>
    <a href="https://drive.google.com/file/d/1TekHGW16r6IefxefWsiNgXtQxbZbI4ud/view?usp=sharing" target="_blank">Video</a>
    <a href="/papers/ur3_safety.pdf" target="_blank">Write-up</a>
  </div>
</div>

  
</div>

