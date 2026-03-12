---
title: "ME 5250 Camera Orbit Trajectory through Co-robot Arm"
permalink: /me5250-orbit-trajectory/
date: 2025-12-12
share: false
excerpt: The animation video for the Camera Orbit Trajectory through a 6-DOF Co-robot Arm for ME 5250 (Northeastern University)

header:
  teaser: /assets/videos/me5250-project2-animation.mp4
---

Timeline: Fall 2025<br>
Location: Boston, MA, USA<br>
Skills: Python, Forward Kinematics (DH Parameters), Inverse Kinematics (Newton-Damped Least-Squares), Trajectory Planning

<!-- <figure class="align-center">
<img src= "/assets/images/tri-brake-ecu-to-actuator-wire.png">
<figcaption> A wire I fabricated to connect the brake ECU to the actuator, thereby setting up the vehicle's brake-by-wire technology</figcaption>
</figure> -->

Inspired by industrial inspections performed by co-robot arms equipped with a visual sensor that captures data from multiple angles of the object, I created an animation of the end-effector's camera orbit trajectory circling a target point. This fulfills the inspection criteria by being aimed at the object’s center while maintaining consistent lighting, stable viewpoint geometry, and uniform feature coverage. This was for the second project in my Robot Mechanics and Control (ME 5250) course at Northeastern University.

To set up the Kinematics, I used the official UR10e DH Parameters. I then implemented Forward Kinematics using DH Transform matrices. Using the equation, I wrote code to compute the end-effector given the joint angles.
I also implemented a Newton-Raphson-based inverse kinematics solver. The code uses the Damped Least Squares method to avoid singularities, and the desired end-effector pose is used to compute the required joint angles.
While specifying the trajectory, for an initial resolution of 1mm, 1600 waypoints were needed. This was reduced to 400 waypoints to reduce animation runtime.

<video controls width="600">
  <source src="/assets/videos/me5250-project2-animation.mp4" type="video/mp4">
 6DOF UR10e animation for Camera Orbit Trajectory
</video>

The animation produced is an attempt at this trajectory. While the trajectory demonstrates smooth 3D motion and full 6-DOF motion with no singularities for the most part, it is difficult to identify the continuous orientation control. This is apparent when observing the RGB end-effector frame. Only a few irregular motions are observed due to singularities.

The code I used is shared on the [Github Code Repo](https://github.com/robosangli/co-robot-camera-orbit-trajectory).

In the future, I hope to improve 3D motion smoothness by implementing more robust FK and IK solvers. It would also be interesting to explore collision avoidance and extend this study to dynamics and control.

[Project Report](https://drive.google.com/file/d/1kW-ZbZ1q9C_i6x_wlm8V1jHffvTgfrrN/view?usp=drive_link)