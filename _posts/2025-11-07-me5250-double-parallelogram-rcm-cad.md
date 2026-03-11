---
title: "ME 5250 Double Parallelogram RCM Mechanism"
permalink: /me5250-rcm-mechanism/
date: 2025-11-07
share: false
excerpt: The CAD images and animation video for a double parallelogram RCM mechanism for ME 5250 (Northeastern University)
---

Timeline: Fall 2025<br>
Location: Boston, MA, USA<br>
Skills: 3D CAD (Onshape)

Inspired by the parallelogram mechanism in robots like the Da Vinci system in Minimally Invasive Surgery, I explored the design and analysis of a double-parallelogram remote center of motion mechanism. This was for my first project in my Robot Mechanics and Control (ME 5250) course at Northeastern University.

To further understand this mechanism’s ability to rotate its end-effector around a fixed point without the presence of a physical revolute joint, I performed a survey of the existing literature. I first looked at 1-DOF RCM mechanisms formed by combining two parallelogram linkages, as seen below

<figure class="align-center">
<img src= "/assets/images/me5250-1dof.png">
<figcaption> 1-DOF Double parallelogram RCM </figcaption>
</figure>

I then moved on to 2-DOF RCM mechanisms formed by the serial connection of two parallelograms. 

<figure class="align-center">
<img src= "/assets/images/me5250-2dof+mobility.jpg">
<figcaption> 2-DOF Double parallelogram RCM + Mobility Analysis </figcaption>
</figure>

I performed a mobility analysis using Grubler’s formula to demonstrate 2 DOF while maintaining a total joint count of 9. Intuitively, this made sense as the translation/insertion motions along the tool axis (end-effector). 

I then used a simplified kinematic model of the mechanism to compute the Jacobian matrix of the linkage and identify the existing singularities (as shown in the report). 

<figure class="align-center">
<img src= "/assets/images/me5250-simplified-kinematics-model.jpg">
<figcaption> Simplified Kinematics Model </figcaption>
</figure>

This project introduced me to the complex mechanics of remote-center-of-motion mechanisms, particularly double-parallelogram mechanisms. For more details, please refer to the [Project Report](https://drive.google.com/file/d/10gav7T_xcN6B1vySFROKQxyaBcYVikXJ/view?usp=sharing).