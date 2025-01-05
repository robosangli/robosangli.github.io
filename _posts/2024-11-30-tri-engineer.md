---
title: "TRI System Integration Engineer"
permalink: /tri-engineer/
date: 2024-11-30
share: false
excerpt: My current work as a System Integration Engineer at Toyota Research Institute helping build a battery-electric vehicle.
---

Timeline: September 2024 - Present<br>
Location: Los Altos, CA, USA
Skills: SOLIDWORKS, Laser Cutting, Perception & Localization

I continue to assist in developing a new off-road battery electric vehicle with my previous responsibilities.

### Vehicle Compute Stack
<figure>
<img src= "/assets/images/tri-delrin-tablesaw-compute-stack.PNG">
<figcaption> I used the in-house tablesaw to cut 1/4" Delrin plates to fabricate the first iteration of the vehicle's compute stack to house onboard computers</figcaption>
</figure>
I utilized Solidworks guides to create 2D Drawings and image trace sketches. I also utilized the Solidworks Hole Wizard to automatically create screw clearances, adjusting for real-world inconsistencies from CAD data.

I also selected appropriate high-strength (class 10.9 & 8) zinc-plated steel fasteners (to prevent galling) like flange nuts & 100° countersink screws to accommodate for steel-aluminum interactions and nut-to-bolt tensile strengths.

As seen above, I fabricated a prototype shelf utilizing a table saw and laser cutter for 1/4" Delrin plates and stainless steel standoffs between these plates and shelves for easy modular installation. Since the plates were larger than the laser cutter bed, the bottom holes were manually drilled (to avoid rotation-recut alignment issues with the laser cutter).

I also communicated with the sheet-metal fabricator for Aluminum counterparts to replace the Delrin in the long-term with deep laser engraving post the anodizing finish.

### Wheel-speed sensor integration
I studied existing documentation & in-vehicle wiring for 4 individual in-hub wheel-speed sensors. As a result, I identified the correct brake ECU pins to integrate the sensors & transfer output to the low-level computer. 

I fabricated long-run sensor wires and integrated them into brake ECU to identify the correct CAN channel by monitoring bus traffic through a CAN-USB interface and monitoring software (Vector CANalyzer).

I am currently integrating these wheel-speed sensors to facilitate high-level software and autonomous control of the vehicle.

### Perception & Localization
I was introduced to the planar calibration difficulties with vehicle visual-inertial sensors and AprilTag boards for initial calibrations for computer vision systems.

I was also introduced to the framework used to label lane boundaries dealing with muliple frames (camera frame, imu frame, body frame, road frame, ground frame) extrinsic & intrinsic parameters, and tangential & radial distortions.

I set up software on my local system in terms of vision-based model predictive control, SAM2 (Segment Anything Model 2) for object segmentation, MATLAB, AWS data buckets (for rosbag data), and lane-boundary labeling scripts.

I am currently studying GPS RTK (real-time kinematics) technology to better understand the onboard INS (inertial navigational system) and the ground truth calculations for vehicles.

Here’s an exciting publication co-authored by my mentor using camera-based lane-mark measurements for  vehicle localization and road mapping, 
[A framework for joint vehicle localization and road mapping using onboard sensors - ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S0967066124002715)

### Other tasks + Support
I successfully integrated a human-machine interface keypad and buttons after testing on dSPACE layouts, thereby establishing analog detection emergency-stop capabilities and BMS (battery management system) signals. The resulting documentation will be utilized as vehicle operations for all future users.

I supported in the development & preparation for the vehicle's first driving tests in terms of wiring refabrications, rerouting, and other electrical clean-ups to prevent communication errors.

I also supported the team in loading the vehicle’s battery (pack + enclosure) through a 2 post-lift. We made adjustments of the rear shock absorber springs and re-aligned the wheels to compensate for the battery's weight.

I supported the team on vehicle drive tests incorporating torque limits (400Nm & 500Nm) before adding water for battery cooling. We discovered issues during this mechanical shakedown.

I also reviewed Simulink vehicle interface software merge requests made by the team for vehicle software integration across various low-level components like the BMS (Battery Management System), HMI (Human-Machine Interface) devices, and inverter-motor-gearbox modules.


​[Toyota Research Institute: Human Interactive Driving](https://www.tri.global/our-work/human-interactive-driving)