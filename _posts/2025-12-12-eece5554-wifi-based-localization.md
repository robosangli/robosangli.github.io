---
title: "EECE 5554 Localization via Wi-Fi through RSSI/BSSID Pair Training"
permalink: /eece5554-wifi-based-localization/
date: 2025-12-12
share: false
excerpt: Localization via Wi-Fi through RSSI/BSSID Pair Training - Final Project for EECE 5554 (Northeastern University)
---

Timeline: Fall 2025<br>
Location: Boston, MA, USA<br>
Skills: 

For the final project in the Robotics Sensing & Navigation (EECE 5554) course at Northeastern University, I worked with two peers to develop a Wi-Fi localization system using RSSI/BSSID Pair Training.

I worked on the development, implementation, and evaluation of learning-based localization methods to estimate planar position from WiFi signal data. This involved,
Designing regression models that take BSSID-indexed RSSI measurements as inputs and output estimated (x, y) coordinates						
Implementing these models as a ROS2 prediction node that integrates with the system-wide sensing and filtering pipeline
Performing data analysis and quantitative evaluation to assess localization accuracy and system efficacy

My other 2 team members, Jack and Ben, focused on complementary system components like the Sensing and Filtering aspects of the pipeline. This included the ROS2 Wi-Fi driver, custom message definitions, low-pass filtering of raw RSSI measurements, and Kalman filtering for temporal smoothing. My development of the core inference layer of the localization system is wrapped in a ROS2 Python Node with a regression training Python script.

<!-- add pipeline image -->
<!-- <figure class="align-center">
<img src= "/assets/images/tri-brake-ecu-to-actuator-wire.png">
<figcaption> A wire I fabricated to connect the brake ECU to the actuator, thereby setting up the vehicle's brake-by-wire technology</figcaption>
</figure> -->

[more details to come soon]

I followed a process of analysing the data, experimenting with various models, and producing the necessary results for filtering.

Sensing Modality Data Challenges:
High dimensionality & sparsity: Only a subset of all the access points for each Wi-Fi scan
Measurement noise & temporal variability
Nonlinear spatial relationships
Despite these challenges, Wi-Fi was chosen for its infrastructure-free and low-cost sensing capabilities.

<!-- add highbay image -->
<!-- <figure class="align-center">
<img src= "/assets/images/tri-brake-ecu-to-actuator-wire.png">
<figcaption> A wire I fabricated to connect the brake ECU to the actuator, thereby setting up the vehicle's brake-by-wire technology</figcaption>
</figure> -->

Data Representation + Preprocessing
Wi-Fi scans recorded consisted of variable length access point observations, which is incompatible with standard regression models. To address this, I used a fixed-dimensional representation indexed by BSSID. I used mean imputation to handle the missing RSSI values. As a result, the consistently observed access points were contributed to the model. 
I also implemented a variance-based feature selection to remove access points with low informational content.

Candidate Modeling Approaches
I implemented 3 approaches, each with increasing complexity.
Lazy learning or fingerprinting: Evaluated as an initial baseline due to their simplicity but they scaled poorly with dataset size and did not provide reasonable estimates. Downstream probabilistic filtering was probably necessary to address accuracy concerns in the training dataset.
Gaussian Process Regression (GPR): Trained a single multi-output GP and observed its ability to model nonlinear relationships while quantifying predictive uncertainty.
Multi-output regression formulations using GPR: Two independent GPs, one for each spatial dimension (x, y), which yielded the highest predictive accuracy. 

Results
Gaussian Process Regression, when combined with structural preprocessing and careful kernel design, was shown to be well-suited for Wi-Fi-based indoor localization. We successfully achieved 2-Dimensional localization with sufficiently accurate position estimates.

[screenshot of working position estimate -> can’t seem to find anything on computer
Re-run the code to get this with random testing bags]

[github code link] - have personal repo archive/somehow merge so all commit history included?? -update readme?

Please note that GenAI was used to debug and help develop this regression, as this project was my first introduction to Machine Learning. I used it as an opportunity to learn as much as I could while keeping up with the rapid development timeline.

[Document link]

<!-- <video controls width="600">
  <source src="/assets/videos/me5250-project2-animation.mp4" type="video/mp4">
 6DOF UR10e animation for Camera Orbit Trajectory
</video> -->