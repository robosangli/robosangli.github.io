---
title: "EECE 5554 GPS-IMU Sensor Fusion for Vehicle Position Estimation"
permalink: /eece5554-car-navigation/
date: 2025-12-03
layout: single
share: false
excerpt: GPS-IMU Sensor Fusion for Vehicle Position Estimation for EECE 5554 (Northeastern University)

header:
  teaser: /assets/images/car-nav-testing-setup.png
---

Timeline: Fall 2025<br>
Location: Boston, MA, USA<br>
Skills: 

As part of the Robotics Sensing and Navigation (EECE 5554) course curriculum at Northeastern University, I worked with 2 peers on GPS-IMU sensor fusion to determine a moving car's path.

We first verified the working of each individual driver, gps_driver, and vn_driver to ensure the proper functioning of each individual component (GPS & IMU). Then we verified the fusion of the data streams from these two sensors.

<figure class="align-center">
<img src= "/assets/images/car-nav-testing-setup.png">
<figcaption> Car Setup Testing </figcaption>
</figure>

<figure class="align-center">
<img src= "/assets/images/car-nav-imu-placement-on-dash.png">
<figcaption> IMU Dash Setup </figcaption>
</figure>

Then we collected data in 4 environments: stationary, stationary with the engine idle, a circular path, and driving in Boston through traffic.

<video controls width="600">
  <source src="/assets/videos/car-nav-circular-path.mp4" type="video/mp4">
 Circular Path Data Collection
</video>

<video controls width="600">
  <source src="/assets/videos/car-nav-boston-traffic.mp4" type="video/mp4">
 Boston Traffc Data Collection (snippet)
</video>

We calculated the calibration from magnetometer data using least-squares. We then plotted the accelerometer and gyro data, along with their time integrals. With this data from the VN and GPS, we plotted the vehicle’s path. We ended our exploration with a brief discussion on the sources of error in this path/position estimation.

For more details please take a look at the [Github](https://github.com/robosangli/nuance_navigation) and the [Report](https://drive.google.com/file/d/1edk2zzMs9VqS_tcLH5IIqBwc4JlY6u4l/view?usp=sharing)