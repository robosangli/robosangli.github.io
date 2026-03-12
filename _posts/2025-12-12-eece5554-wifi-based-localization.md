---
title: "NU Localization via Wi-Fi through RSSI/BSSID Pair Training"
permalink: /eece5554-wifi-based-localization/
date: 2025-12-12
layout: single
share: false
excerpt: Localization via Wi-Fi through RSSI/BSSID Pair Training - Final Project for EECE 5554 (Northeastern University)

header:
  teaser: /assets/images/wifi-based-loc-data-collection.jpg
---

Timeline: Fall 2025<br>
Location: Boston, MA, USA<br>
Skills: 

For the final project in the Robotics Sensing & Navigation (EECE 5554) course at Northeastern University, I worked with two peers to develop a Wi-Fi localization system using RSSI/BSSID Pair Training.

I worked on the development, implementation, and evaluation of learning-based localization methods to estimate planar position from WiFi signal data. This involved,
- Designing regression models that take BSSID-indexed RSSI measurements as inputs and output estimated (x, y) coordinates						
- Implementing these models as a ROS2 prediction node that integrates with the system-wide sensing and filtering pipeline
- Performing data analysis and quantitative evaluation to assess localization accuracy and system efficacy

My other 2 team members, Jack and Ben, focused on complementary system components like the Sensing and Filtering aspects of the pipeline. This included the ROS2 Wi-Fi driver, custom message definitions, low-pass filtering of raw RSSI measurements, and Kalman filtering for temporal smoothing. My development of the core inference layer of the localization system is wrapped in a ROS2 Python Node with a regression training Python script.

<figure class="align-center">
<img src= "/assets/images/wifi-based-loc-system-overview.png">
<figcaption> System Pipeline Overview </figcaption>
</figure>

I followed a process of analysing the data, experimenting with various models, and producing the necessary results for filtering.

### Sensing Modality Data Challenges:
- High dimensionality & sparsity: Only a subset of all the access points for each Wi-Fi scan
- Measurement noise & temporal variability
- Nonlinear spatial relationships
Despite these challenges, Wi-Fi was chosen for its infrastructure-free and low-cost sensing capabilities.

<figure class="align-center">
<img src= "/assets/images/wifi-based-loc-data-collection.jpg">
<figcaption> Data Collection in EXP Highbay </figcaption>
</figure>

### Data Representation + Preprocessing
Wi-Fi scans recorded consisted of variable-length access point observations, which are incompatible with standard regression models. To address this, I used a fixed-dimensional representation indexed by BSSID. I used mean imputation to handle the missing RSSI values. As a result, the consistently observed access points were incorporated into the model. 
I also implemented a variance-based feature selection to remove access points with low informational content.

### Candidate Modeling Approaches
I implemented 3 approaches, each with increasing complexity.
- Lazy learning or fingerprinting: Evaluated as an initial baseline due to their simplicity but they scaled poorly with dataset size and did not provide reasonable estimates. Downstream probabilistic filtering was probably necessary to address accuracy concerns in the training dataset.
- Gaussian Process Regression (GPR): Trained a single multi-output GP and observed its ability to model nonlinear relationships while quantifying predictive uncertainty.
- Multi-output regression formulations using GPR: Two independent GPs, one for each spatial dimension (x, y), which yielded the highest predictive accuracy. 

### Results
Gaussian Process Regression, when combined with structural preprocessing and careful kernel design, was shown to be well-suited for Wi-Fi-based indoor localization. We successfully achieved 2-Dimensional localization with sufficiently accurate position estimates.

<figure class="align-center">
<img src= "/assets/images/wifi-based-loc-train-regressor.png">
<figcaption> Training Regressor </figcaption>
</figure>

<figure class="align-center">
<img src= "/assets/images/wifi-based-loc-training-complete.png">
<figcaption> Training completed </figcaption>
</figure>

<figure class="align-center">
<img src= "/assets/images/wifi-based-loc-play-rosbag-for-testing.png">
<figcaption> Play pre-collected ROSbag (for testing) - Position (2.5,2) </figcaption>
</figure>

<figure class="align-center">
<img src= "/assets/images/wifi-based-loc-echo-wifi.png">
<figcaption> Echo '/wifi' topic to confirm </figcaption>
</figure>

<figure class="align-center">
<img src= "/assets/images/wifi-based-loc-wifi-predictor.png">
<figcaption> Run Wifi Predictor </figcaption>
</figure>

Please note that GenAI was used to debug and help develop this regression, as this project was my first introduction to Machine Learning. I used it as an opportunity to learn as much as I could while keeping up with the rapid development timeline.

[Github Code Repo](https://github.com/robosangli/wifi_localization)
[Report](https://drive.google.com/file/d/1mV7eSKb1KLgF4JEs_iNp0PSw4UIq2qgF/view?usp=sharing)
[Presentation Slides](https://drive.google.com/file/d/1P3ir1rHqdzVZI9zGP4wqrHtJfiAFUdXk/view?usp=sharing)