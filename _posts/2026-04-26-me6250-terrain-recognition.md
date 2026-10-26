---
title: "IMU-Based Terrain Recognition for Wearable Robotics"
permalink: /me6250-terrain-recognition/
date: 2026-04-26
share: false
excerpt: "A wearable sensing and time-series learning system for classifying walking terrain from four IMUs."
group: project
featured_order: 2
tags: [Wearable Robotics, Embedded Systems, C++, IMU, Time-Series Machine Learning]
---

Timeline: Spring 2026<br>
Location: Boston, MA, USA<br>
Skills: 

## Project overview

Terrain changes alter walking patterns. For a lower-limb exoskeleton, recognizing level ground, inclines, uneven terrain, or sand could provide context for adapting assistance. Our team built a wearable inertial-sensing prototype and compared three deep-learning approaches for classifying terrain from motion data.

The system combined four MPU-6050 inertial measurement units (IMUs), a Raspberry Pi 4, a TCA9548A I²C multiplexer, C++ data acquisition, and three time-series classifiers: a one-dimensional convolutional neural network (1D CNN), a long short-term memory network (LSTM), and a temporal convolutional network (TCN).

**Reported accuracy:** 47% for the 1D CNN, 94% for the LSTM, and 96% for the TCN. These are results from our project dataset and evaluation setup—not evidence of generalization to new subjects or all real-world walking conditions.

**My role:** I implemented the 1D CNN. Hardware integration, data collection, and embedded software were shared team efforts; teammates implemented the LSTM and TCN. We collaborated on the overall system, dataset, and evaluation.

<!-- Replace with a strong hardware or in-the-field photo. -->
![Wearable terrain-recognition prototype](/assets/images/projects/wearable-terrain/terrain-system.jpg)
*The wearable prototype combined bilateral thigh and ankle IMUs with a Raspberry Pi-based acquisition system.*

## System architecture

1. **Motion sensing:** Four MPU-6050 IMUs were attached at the thighs and ankles.
2. **Sensor multiplexing:** A TCA9548A I²C multiplexer exposed four sensors through one shared I²C bus.
3. **Embedded acquisition:** A Raspberry Pi 4 collected accelerometer and gyroscope readings. The report describes moving from approximately 80 Hz with Python to 250 Hz with C++.
4. **Dataset preparation:** Each timestamp contained 24 features: six accelerometer/gyroscope channels per IMU.
5. **Classification:** The team compared a 1D CNN, LSTM, and TCN across four terrain classes.
6. **Evaluation:** We compared accuracy and confusion matrices to understand which terrain types were confused.

![Terrain recognition system architecture](/assets/images/projects/wearable-terrain/system-architecture.png)
*Suggested diagram: four IMUs → TCA9548A multiplexer → Raspberry Pi 4 / C++ acquisition → sensor time series → preprocessing and temporal models → four terrain labels.*

## Embedded sensing and acquisition

### Four IMUs on one I²C bus

Each MPU-6050 provides three-axis accelerometer and three-axis gyroscope readings. Four sensors produce 24 channels per timestamp:

\[
\mathbf{x}_t = [a_x,a_y,a_z,g_x,g_y,g_z]_{\text{left thigh}} \oplus
[a_x,a_y,a_z,g_x,g_y,g_z]_{\text{right thigh}} \oplus
[a_x,a_y,a_z,g_x,g_y,g_z]_{\text{left ankle}} \oplus
[a_x,a_y,a_z,g_x,g_y,g_z]_{\text{right ankle}}.
\]

The multiplexer selects one sensor channel at a time. Because the sensors are read sequentially, their measurements are not perfectly simultaneous—a relevant limitation when comparing fast transients across body locations.

### Sampling rate and reliability

The final report describes an initial Python acquisition rate of about 80 Hz and a C++ implementation reaching 250 Hz. The motivation was to retain faster motion signatures, such as impacts and short-duration changes in foot motion. The report also describes a consistent two-minute collection period, an LED recording indicator, and exporting trial CSVs for post-processing.

**Implementation note:** The uploaded `terrain.cpp` appears to be a candidate version of the Pi data-collection program, but it has not been confirmed as the exact deployed version. This page therefore describes the acquisition approach reported by the team, rather than claiming every detail in this file matches the code running on the device.

![Sensor and embedded hardware detail](/assets/images/projects/wearable-terrain/hardware-detail.jpg)
*Suggested image: annotated close-up of the IMUs, multiplexer, Raspberry Pi, wiring, power source, and status LED.*

## Dataset and preprocessing

We collected data across four terrain categories:

- **Level ground**
- **Inclines / ramps**
- **Uneven terrain**, recorded while walking uphill on Acorn Street
- **Sand**

The dataset came from a single subject with bilateral thigh and ankle IMUs. Level and incline trials included multiple treadmill inclines and speeds, while uneven terrain and sand represented outdoor conditions.

For the 1D CNN, the report describes this pipeline:

1. Split the raw time series into training (80%) and test (20%) portions before windowing, to reduce direct temporal leakage between splits.
2. Compute per-feature normalization statistics on the training split and apply them to both splits.
3. Divide each split into 100-sample windows with a stride of one timestep.
4. Assign each window the majority terrain label.
5. Feed the model 24 features over 100 timesteps.

At 250 Hz, a 100-sample window spans approximately 0.4 seconds. The small stride creates heavily overlapping windows, increasing redundancy and training cost. Majority-label assignment can also obscure windows that cross terrain transitions.

A further generalization concern is that lab-based level and incline data may contain treadmill- or pace-specific cues, while sand and uneven-terrain data were collected outdoors. Accuracy should be interpreted in light of the collection protocol and single-subject dataset.

## Model comparison

### 1D CNN — my implementation

A 1D CNN applies learned filters along the time axis to detect local patterns such as peaks, oscillations, and transitions. Convolutional computation can be parallelized across timesteps.

The report describes `TerrainCNN1D` as three convolutional blocks followed by a classifier head:

| Stage | Reported configuration |
|---|---|
| Input | 24 channels × 100 timesteps |
| Block 1 | 64 filters, kernel size 5, batch normalization, ReLU, max pooling, dropout |
| Block 2 | 128 filters, kernel size 5, batch normalization, ReLU, max pooling, dropout |
| Block 3 | 256 filters, kernel size 3, then global average pooling |
| Classifier | Fully connected layer with 128 units, then four output classes |
| Optimization | Adam, learning rate 1e-3, cross-entropy, batch size 128 |
| Regularization | Dropout and early stopping on validation loss |

The model builds progressively richer local motion features and pools the temporal representation into a compact vector for four-class classification.

**Reported accuracy: 47%.** The 1D CNN performed substantially worse than the LSTM and TCN in this experiment. Its confusion matrix showed errors across multiple classes; level walking was frequently classified as incline, and uneven terrain was often confused with incline or sand.

This result is an important part of the engineering story: a straightforward temporal CNN was not sufficient for this dataset, despite the strong performance of the other models. The outcome motivates closer attention to temporal inductive bias, dataset structure, and collection conditions.

![1D CNN confusion matrix](/assets/images/projects/wearable-terrain/1d-cnn-confusion-matrix.png)
*Use the original plot or regenerate the 1D CNN confusion matrix from the notebook.*

### LSTM — team comparison

The LSTM is a recurrent architecture that maintains hidden and cell states as it processes a sequence. Our teammate implemented the LSTM. The report describes a single-layer, unidirectional LSTM with 64 hidden units, dropout, and a four-class linear output layer.

**Reported accuracy: 94%.** It performed strongly on level and incline walking and also classified sand and uneven terrain well.

![LSTM confusion matrix](/assets/images/projects/wearable-terrain/lstm-confusion-matrix.png)
*LSTM confusion matrix from the final report.*

### TCN — team comparison

The TCN uses causal, dilated convolutions to build a large temporal receptive field. The report describes five temporal blocks with two convolutional layers per block, weight normalization, causal-padding cleanup, and ReLU activations. The TCN was implemented by our other teammate and used a sequence-based training setup rather than the same sliding-window formulation described for the 1D CNN.

**Reported accuracy: 96%.** It performed best in the reported comparison, with strong per-class results in the final report.

![TCN confusion matrix](/assets/images/projects/wearable-terrain/tcn-confusion-matrix.png)
*TCN confusion matrix from the final report.*

### Results at a glance

| Model | Temporal mechanism | Reported accuracy |
|---|---|---:|
| 1D CNN — my implementation | Local temporal convolutions and pooling over 100-sample windows | 47% |
| LSTM — teammate implementation | Recurrent hidden and cell states | 94% |
| TCN — teammate implementation | Causal dilated temporal convolutions | 96% |

These are the accuracies reported in the final report, but the notebook contains multiple CNN experiment versions. An earlier run reports approximately 100% accuracy on a much smaller test set after creating overlapping windows and then randomly splitting those windows; that approach can place highly similar windows in both training and test sets. A later run reports 47% accuracy on a much larger test set, but its preprocessing shuffles individual timestamp rows before constructing windows, disrupting the original temporal sequence. The later score matches the final report, but neither pipeline should be presented as a clean, definitive estimate of CNN generalization without revisiting the split and windowing design.

The LSTM and TCN also use different data-splitting/training formulations, and the notebook's cached LSTM evaluation output is absent. The 94% / 96% / 47% figures are therefore best presented as the team's reported exploratory comparison, with this methodological caveat clearly stated—not as a controlled benchmark proving that architecture alone caused the performance gap.

## Testing outside the lab

The uploaded presentation demo is approximately 39 seconds long at 4K resolution and 30 fps. It visibly shows a treadmill walking trial and outdoor walking on sand; it is strong evidence that the project moved beyond a notebook into physical data collection. For the website, make a compressed 1080p or 720p web version rather than embedding the 125 MB original. A single short demo is enough near the top of the page; additional clips can be shown selectively below.

The photos and videos can make this page much more compelling than a standard ML write-up. Recommended media sequence:

1. Hardware overview: Raspberry Pi, multiplexer, IMUs, wiring, and power source.
2. Treadmill trials: controlled level and incline collection.
3. Normal outdoor walking.
4. Sand.
5. Cobbled / uneven street: the Acorn Street trial used for the uneven-terrain category.
6. Poster presentation and a team photo.

<div class="video-grid">
  <figure>
    <video controls playsinline preload="metadata" width="100%">
      <source src="/assets/videos/wearable-terrain/treadmill-trial.mp4" type="video/mp4">
    </video>
    <figcaption>Treadmill data collection under controlled walking conditions.</figcaption>
  </figure>
  <figure>
    <video controls playsinline preload="metadata" width="100%">
      <source src="/assets/videos/wearable-terrain/outdoor-walking.mp4" type="video/mp4">
    </video>
    <figcaption>Outdoor walking trial.</figcaption>
  </figure>
  <figure>
    <video controls playsinline preload="metadata" width="100%">
      <source src="/assets/videos/wearable-terrain/sand-trial.mp4" type="video/mp4">
    </video>
    <figcaption>Walking on sand.</figcaption>
  </figure>
  <figure>
    <video controls playsinline preload="metadata" width="100%">
      <source src="/assets/videos/wearable-terrain/acorn-st-trial.mp4" type="video/mp4">
    </video>
    <figcaption>Uneven-terrain trial on Acorn Street.</figcaption>
  </figure>
</div>

*All media paths above are placeholders. Use compressed videos and confirm that the Jekyll theme supports the HTML video elements.*

## What we learned

- **Sensor placement and acquisition matter.** Four IMUs provided a 24-channel view of lower-limb motion, and faster C++ acquisition was intended to retain more rapid movement dynamics.
- **Temporal architecture matters—but evaluation design matters too.** The TCN and LSTM outperformed the 1D CNN. Architecture, training formulation, and dataset characteristics all need consideration before attributing the gap to one factor.
- **Cross-environment data can create confounds.** If terrain labels correlate with collection location, treadmill speed, or gait pattern, a classifier may learn those cues instead of terrain properties that generalize.
- **A single-subject dataset is an early prototype.** More participants, repeated trials, varied speeds, and terrain collected across environments are needed to assess subject-independent performance.
- **This was classification research, not a deployed assistive controller.** The project explored a sensing component that could inform future assistance strategies; it did not demonstrate a safety-validated controller adapting torque from model predictions.

## My contributions and team collaboration

**Individual model ownership**
- Implemented the 1D CNN terrain-classification model and participated in the model-comparison work.

**Shared team contributions**
- Integrated wearable IMU hardware and the Raspberry Pi-based acquisition setup.
- Collected and prepared the terrain dataset.
- Collaborated on overall system design, experimentation, evaluation, and the final technical presentation.

**Other model implementations**
- LSTM: Zach Locher.
- TCN: Jack Ager.

This reflects the role assignments on the project poster. Hardware, data collection, and embedded software should be described as collaborative rather than attributed solely to one person.

## Sensor data: what is worth sharing?

Raw IMU files are useful for reproducibility, but a large collection of raw CSVs would be too much for the main portfolio page and would distract from the engineering story. I recommend showing a compact visualization instead:

- A 2–5 second excerpt of synchronized accelerometer and gyroscope signals from all four IMUs.
- A clear legend identifying sensor location and channel (for example, ankle accelerometer magnitude or thigh gyroscope axes).
- A short caption explaining the sampling rate, units, terrain label, and whether the plot is raw or normalized.
- Optionally, a small sanitized sample CSV and a link to a public dataset/code repository if you have permission to share the data.

Do not upload the entire raw dataset just to demonstrate that data exists. If you later publish it, document the column schema, sensor/channel mapping, units, timestamps, trial labels, train/test split rules, and any participant privacy constraints. The current notebook uses CSV rows containing an IMU index, three accelerometer channels, three gyroscope channels, and a time value; the model pipeline pivots these into 24 channels per synchronized timestamp.

## Project resources

- **Final report:** Add a link to the published PDF.
- **Poster:** Add a link to the final project poster.
- **Notebook:** [Open the Google Colab notebook](https://colab.research.google.com/drive/1UxJPnoZLc0qfIXGXgtOh3FRdGlN1gXQl?usp=sharing).
- **Source code:** Add the project repository URL once confirmed.
- **Demo media:** Add a short, compressed hardware or outdoor-testing video.

## Limitations and next steps

1. **Validate generalization across subjects.** Collect data from more participants and evaluate on held-out subjects.
2. **Improve collection diversity.** Record multiple terrain classes in both controlled and outdoor settings to reduce correlations between label and collection environment.
3. **Revisit windowing and class balance.** Quantify the effects of overlapping windows, class trimming, and transition-window labeling.
4. **Measure online behavior.** Evaluate inference latency and prediction stability on the Raspberry Pi, not only offline accuracy.
5. **Confirm the deployed acquisition code.** The uploaded `terrain.cpp` may be an older version; compare it with the actual Pi version before presenting implementation details as verified deployment facts.
6. **Integrate cautiously with assistance control.** A future controller would need confidence handling, transition logic, and safety validation before predictions influence assistive torque.