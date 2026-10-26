---
title: "Class-Agnostic Maritime Obstacle Detection & Horizon Stabilization"
permalink: /cs5330-maritime-detection+stabilization/
date: 2026-08-03
share: false
excerpt: "A maritime perception research pipeline exploring synthetic camera-motion compensation, YOLOv8n obstacle detection, and Kalman-based temporal tracking."
# author_profile: true
group: project
featured_order: 1
tags:
  - C++
  - Python
  - OpenCV
  - Computer Vision
  - Object Detection
  - Geometric Image Transformation
  - Kalman Filtering
---

Timeline: Summer 2026<br>
Location: Boston, MA, USA<br>
Skills:

<!--
Before publishing:
- Keep the distinction between the Python offline research/evaluation pipeline and the smaller C++ stabilization prototype explicit.
- The public main branch's C++ target currently builds main.cpp, FakeIMU.cpp, and HorizonStabilizer.cpp only; it does not build the YOLO detector or Kalman tracker.
- The main-branch C++ demo uses a single synthetic sinusoidal roll and cv::getRotationMatrix2D/warpAffine. The report's pitch/roll homography and noisy/delayed orientation-estimate experiments are described in the Python evaluation methodology.
- Do not claim real-IMU, real-vessel, or end-to-end real-time deployment validation.
-->

# Class-Agnostic Maritime Obstacle Detection & Horizon Stabilization

**CS 5330 — Pattern Recognition & Computer Vision · Northeastern University · Team project**

Autonomous surface vessels (ASVs) must detect hazards from a camera whose viewpoint changes with vessel motion. This project explored a modular maritime-perception pipeline combining class-agnostic YOLOv8n obstacle detection, synthetic IMU-guided horizon stabilization, and Kalman-based temporal tracking.

**My primary ownership:** the synthetic camera-motion / IMU perturbation and horizon-stabilization component. I also contributed to dataset selection and overall architecture design. Dataset construction, detector training, and tracking experiments were collaborative team work.

- **Repository:** [maritime_obstacle_detection](https://github.com/robosangli/maritime_obstacle_detection)
- **Team:** Ananda Sangli, Aryan Alpeshkumar Patel, Julee Chung
- **Core tools:** Python, C++, OpenCV, YOLOv8n, ONNX Runtime, NumPy, Kalman filtering
- **Research questions:** Can geometric stabilization recover detection performance under camera motion? Can temporal state estimation reduce bounding-box jitter and identity fragmentation?

## Project demonstration

The project produced several useful demonstrations and evaluation artifacts. The most informative order for a reader is:

1. **Input video** — original maritime footage.
2. **Synthetic camera video** — the same footage with controlled oscillatory image rotation.
3. **Stabilized video** — correction driven by the synthetic orientation estimate.
4. **No filter vs. Kalman** — side-by-side tracking comparison on the same detector stream.
5. **Detector evaluation** — confidence-threshold curves, precision–recall curve, and normalized confusion matrix.

<!-- Replace the paths below with the actual filenames after copying compressed, portfolio-ready media into images/projects/maritime/. -->
<div class="video-grid">
  <figure>
    <video controls preload="metadata" playsinline width="100%">
      <source src="{{ '/images/projects/maritime/input.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support embedded video.
    </video>
    <figcaption>Input footage — the unperturbed reference sequence.</figcaption>
  </figure>
  <figure>
    <video controls preload="metadata" playsinline width="100%">
      <source src="{{ '/images/projects/maritime/simulated-camera.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support embedded video.
    </video>
    <figcaption>Synthetic camera motion — controlled oscillatory rotation applied to the input.</figcaption>
  </figure>
  <figure>
    <video controls preload="metadata" playsinline width="100%">
      <source src="{{ '/images/projects/maritime/stabilized.mp4' | relative_url }}" type="video/mp4">
      Your browser does not support embedded video.
    </video>
    <figcaption>Stabilized output — inverse rotation applied using the simulated orientation estimate.</figcaption>
  </figure>
</div>

> **Evaluation boundary:** The camera motion and IMU-like orientation were simulated. These videos demonstrate the method under controlled conditions; they are not evidence of performance using synchronized physical IMU telemetry or a deployed ASV.

## 1. Problem formulation

Wave-induced pitch and roll change the apparent horizon and shift obstacles within the image. Small, distant hazards can become harder to detect consistently, while frame-to-frame detection variation can make tracked boxes jitter or fragment into new identities.

The project investigated two complementary mechanisms:

- **Pre-inference geometric stabilization:** compensate for estimated camera rotation before object detection.
- **Post-detection temporal filtering:** use a constant-velocity Kalman filter and IoU-based association to smooth bounding-box trajectories and coast through short detection gaps.

We formulated detection as a single `obstacle` class rather than retaining a fine-grained taxonomy of vessel, buoy, swimmer, and other hazard categories. This makes the detector class-agnostic, but it does not provide semantic hazard classification or a complete collision-avoidance policy.

## 2. Architecture and data flow

The experiments use one frozen detector so the effect of stabilization and tracking can be isolated without changing model weights.

<!-- Recommended figure: create a vector diagram with two lanes. Lane 1: image path. Lane 2: synthetic orientation path. Mark Python evaluation and C++ prototype as separate implementations, not a single deployed binary. -->
<figure>
  <img src="{{ '/images/projects/maritime/system-architecture.png' | relative_url }}" alt="Architecture of the maritime perception experiments: dataset preparation, YOLOv8n detection, synthetic camera perturbation, noisy orientation estimate, optional stabilization, and optional Kalman tracking">
  <figcaption>Architecture to draw for the portfolio. Separate the Python offline evaluation harness from the C++ stabilization prototype. The Python experiments evaluate the detector and tracker; the public C++ CMake target currently compiles only the video, synthetic roll, and affine stabilization path.</figcaption>
</figure>

### Offline Python evaluation path

1. Convert USVTrack tracking annotations and LaRS panoptic annotations into a unified YOLO-format dataset.
2. Map retained object annotations to a single `obstacle` class.
3. Train YOLOv8n and export the selected detector checkpoint to ONNX.
4. Run per-dataset validation and select a confidence operating point.
5. For the stabilization experiments, apply controlled synthetic camera motion, generate a noisy/delayed orientation estimate, optionally stabilize the frame, and evaluate detections.
6. For the tracking experiments, compare greedy IoU association without a motion model against Kalman-based state estimation and short-gap coasting.

### C++ stabilization prototype

The public `main` branch contains a smaller OpenCV application built from `main.cpp`, `FakeIMU.cpp`, and `HorizonStabilizer.cpp`. It reads a video, generates a deterministic sinusoidal roll estimate, applies an inverse in-plane rotation, writes a stabilized video, and displays the output. The CMake target does **not** currently compile the YOLO detector or Kalman tracker, so I describe this as a stabilization prototype rather than a complete real-time perception stack.

## 3. My contribution: synthetic motion and horizon stabilization

I took primary responsibility for the synthetic camera-motion / IMU perturbation and horizon-stabilization component. The intent was to create a repeatable experiment in which the image perturbation was known, but the correction stage received an imperfect orientation estimate rather than ground truth.

### 3.1 Controlled synthetic motion in the research experiments

The Python research experiments model time-varying roll and pitch with sinusoidal components:

| Parameter | Primary experiment setting |
|---|---:|
| Effective frame rate | 5 Hz (USVTrack stride-5 sequence) |
| Assumed horizontal field of view | 80° |
| Roll amplitude | ±30° |
| Roll frequency | 0.25 Hz |
| Pitch amplitude | ±15° |
| Pitch frequency | 0.15 Hz |
| Orientation-estimate bias | 0.5° |
| Orientation-estimate Gaussian noise | 1.0° standard deviation |
| Orientation-estimate latency | 1 frame |

The report describes the image rotation as a rotational homography:

\[
H = K R K^{-1}
\]

where \(K\) is the camera intrinsic matrix and \(R\) is the rotation associated with the simulated camera attitude. The correction is driven by the noisy, delayed synthetic estimate—not the exact angle used to generate the perturbation. Ground-truth bounding boxes are transformed consistently with the image geometry used for each condition.

The datasets do not contain synchronized per-frame pitch/roll telemetry, so the synthetic model enables controlled testing but cannot establish performance under real sensor noise, vibration, calibration error, or camera/IMU synchronization.

### 3.2 C++ implementation visible on the public `main` branch

The C++ prototype is deliberately simpler than the Python experimental model:

- `FakeIMU::getRoll(t)` returns a deterministic sinusoidal roll signal.
- `HorizonStabilizer::stabilize()` computes the image center, constructs a 2 × 3 rotation matrix with `cv::getRotationMatrix2D(center, -rollDegrees, 1.0)`, and applies it with `cv::warpAffine`.
- Border pixels are handled with `cv::BORDER_REPLICATE`.
- `main.cpp` reads a video, computes time from `frameNumber / fps`, applies stabilization, overlays the synthetic roll value, and writes the output video.

This is a useful isolated demonstration of the image-correction component. It does not itself model pitch, bias, Gaussian noise, or latency; those are part of the Python research experiments described above.

<!-- Use the actual stabilized output here, ideally with a synchronized input/perturbed/stabilized frame triptych. -->
<figure>
  <img src="{{ '/images/projects/maritime/stabilization-comparison.png' | relative_url }}" alt="Triptych showing original input, synthetically rotated camera frame, and stabilized frame">
  <figcaption>Qualitative stabilization result. Use the same source moment across all panels, and label the synthetic perturbation clearly.</figcaption>
</figure>

## 4. Dataset construction and detector

The team combined two datasets to broaden maritime visual diversity while preserving sequential data for temporal experiments.

- **USVTrack:** sequential frames and tracking annotations; used as the primary temporal domain.
- **LaRS:** diverse panoptic maritime scenes; contributes additional obstacle appearances and environments.
- **Unified label space:** retained object annotations are mapped to YOLO class `0: obstacle`; obstacle-free images use empty label files.
- **Static-obstacle scope:** LaRS's shoreline-spanning static-obstacle category is excluded because its region-level bounding box does not fit the project's object-level detection formulation. The resulting detector therefore emphasizes dynamic obstacles.

USVTrack is resampled at a stride of five frames. Five complete sequences are held out for validation to reduce temporal leakage. Because the validation set is USVTrack-heavy, results are reported separately for USVTrack and LaRS instead of relying only on a blended metric.

### Detector training and inference

The team trained a YOLOv8n detector at 640 × 640 resolution, with batch size 32 and up to 100 epochs using early stopping (patience 20). The report records stopping at epoch 68 and the best checkpoint at epoch 48. The detector was exported to ONNX (opset 12) and frozen for the downstream ablation experiments.

The experiments use CPU inference through ONNX Runtime. For the primary motion ablation, confidence threshold \(\tau_{\mathrm{conf}}=0.15\) was selected to favor recall for small and distant obstacles. The report notes that the F1-confidence curve peaks around 0.77 near threshold 0.34, but a lower threshold retains more persistent detections for the target use case.

<!-- FIGURE: detector evaluation artifacts you already have. Prefer the PR curve and normalized confusion matrix; put confidence-threshold curve nearby or in a compact evaluation gallery. -->
<div class="image-gallery">
  <figure>
    <img src="{{ '/images/projects/maritime/box-pr-curve.png' | relative_url }}" alt="Precision-recall curve for the class-agnostic YOLOv8n obstacle detector">
    <figcaption>Precision–recall curve for the frozen obstacle detector.</figcaption>
  </figure>
  <figure>
    <img src="{{ '/images/projects/maritime/confidence-threshold.png' | relative_url }}" alt="Detector metrics as the confidence threshold changes">
    <figcaption>Confidence-threshold sweep used to choose a recall-favoring operating point.</figcaption>
  </figure>
  <figure>
    <img src="{{ '/images/projects/maritime/confusion-matrix-normalized.png' | relative_url }}" alt="Normalized confusion matrix for obstacle detection">
    <figcaption>Normalized confusion matrix. In the report, approximately 85% of obstacles are detected and 15% are missed as background.</figcaption>
  </figure>
</div>

## 5. Results

### 5.1 Detector performance and domain shift

On the combined validation set, the report records **mAP@50 = 0.756**, **mAP@50–95 = 0.440**, **precision = 0.860**, and **recall = 0.700**. Per-dataset performance reveals a substantial generalization gap:

| Validation domain | mAP@50 | Recall |
|---|---:|---:|
| USVTrack | 0.918 | 0.849 |
| LaRS | 0.482 | 0.453 |

This difference is a central limitation, not just a secondary metric: strong performance in the USVTrack domain did not transfer equally to LaRS. The result suggests that visual domain shift remains a significant challenge for a detector intended to operate across different waterways and conditions.

### 5.2 RQ1 — Effect of horizon stabilization

Under the strongest tested synthetic perturbation (roll ±30°, pitch ±15°), the same frozen detector was evaluated on clean frames, perturbed frames, stabilized frames, and the combined stabilized-plus-Kalman configuration.

| Condition | Stabilization | Kalman tracker | Recall |
|---|---:|---:|---:|
| Clean reference | No | No | 0.891 |
| Perturbed baseline | No | No | 0.561 |
| Stabilized | Yes | No | 0.723 |
| Kalman only | No | Yes | 0.566 |
| Stabilized + Kalman | Yes | Yes | 0.726 |

Camera perturbation reduced recall by 0.330 relative to the clean reference. Stabilization recovered 0.162 recall over the perturbed baseline—approximately half the motion-induced loss. The amplitude sweep showed the benefit increased with motion severity: the recall gap between stabilized and unstabilized conditions grew from approximately 0.006 at ±5° roll to 0.162 at ±30° roll.

<!-- FIGURE: make a clean chart from the table. Do not use the Kalman result as evidence of improved detection; its value is temporal smoothing. -->
<figure>
  <img src="{{ '/images/projects/maritime/stabilization-recall-results.png' | relative_url }}" alt="Recall across clean, perturbed, stabilized, Kalman-only, and combined experimental conditions">
  <figcaption>Stabilization recovers a substantial portion of the recall lost under strong synthetic camera motion. The results are from controlled offline experiments, not a real-IMU field test.</figcaption>
</figure>

### 5.3 RQ2 — Filter-free tracking vs. Kalman tracking

The filter-free baseline uses greedy intersection-over-union (IoU) association between consecutive detections. It has no motion prediction or coasting, so a missed detection terminates the track and a later detection can receive a new identity.

The Kalman tracker uses a constant-velocity state based on bounding-box center, width, height, and center velocity. It predicts the state forward, associates detections using greedy IoU matching, requires two hits to confirm a track, and permits a one-frame coast through a missed detection.

The custom evaluation reported:
- **32% reduction** in centroid-acceleration jitter on the unstabilized stream.
- **43% reduction** on the stabilized stream.
- In one qualitative sequence, **71 identities** for filter-free tracking versus **60** for Kalman tracking.

These results support smoother trajectories and reduced identity fragmentation in the evaluated sequence. They are custom project metrics, not standardized multi-object tracking benchmark scores. The Kalman-only condition barely changed recall (0.561 to 0.566), which is expected because tracking operates after detection and cannot recover an object that was never detected.

<!-- Use both versions of your tracking video if they show meaningfully different sequences. Avoid two nearly identical clips; select one as the main comparison and link the other as an additional sequence. -->
<figure>
  <video controls preload="metadata" playsinline width="100%">
    <source src="{{ '/images/projects/maritime/no-filter-vs-kalman.mp4' | relative_url }}" type="video/mp4">
    Your browser does not support embedded video.
  </video>
  <figcaption>Filter-free association versus Kalman tracking on the same detector stream. Point out flickering IDs, box jitter, and the short missed-detection interval the tracker coasts through.</figcaption>
</figure>

<!-- Optional second tracking clip if it shows a different sequence or a useful failure case. -->
<details>
  <summary>Additional tracking example</summary>
  <video controls preload="metadata" playsinline width="100%">
    <source src="{{ '/images/projects/maritime/demo-no-filter-vs-kalman-s99.mp4' | relative_url }}" type="video/mp4">
    Your browser does not support embedded video.
  </video>
  <p>Use this additional sequence only if it adds a distinct observation or failure case. Confirm the exact meaning of “s99” before using it in the caption.</p>
</details>

## 6. Engineering decisions and trade-offs

**Class-agnostic detection.** Collapsing retained object classes simplified the label space across two datasets and focused the model on obstacle presence. The trade-off is that the detector does not distinguish hazard types, and excluding the shoreline-spanning static category limits coverage of fixed hazards.

**Frozen model for ablations.** Holding the detector fixed isolates the effects of stabilization and tracking. It also means these results evaluate modules around one trained detector rather than an end-to-end jointly optimized system.

**Imperfect simulated orientation.** Bias, noise, and latency make the correction input less idealized than ground truth. However, the synthetic model does not reproduce the full error characteristics of a physical IMU, nor does it validate camera-to-IMU calibration or timestamp synchronization.

**Separate metrics for separate failure modes.** Recall evaluates obstacle detection. Centroid-acceleration jitter and identity counts describe temporal behavior. Smoother tracks do not imply that previously missed obstacles have been detected.

**Prototype versus integrated system.** The public C++ target demonstrates a video-based synthetic-roll correction component. The detector and Kalman evaluation results were produced by the Python offline pipeline. End-to-end real-time performance was not established.

## 7. Limitations and next steps

1. **Synthetic telemetry:** validate with synchronized physical camera and IMU recordings to measure the effects of sensor noise, drift, latency, vibration, and calibration.
2. **Restricted motion model:** the rotational homography / in-plane rotation does not reproduce translation, heave, motion blur, spray, glare, or all real image-warping effects.
3. **Domain shift:** LaRS performance is substantially lower than USVTrack performance; evaluate on more held-out environments and conditions.
4. **Obstacle scope:** excluding the static shoreline-spanning category leaves some fixed hazards outside the current detector's scope.
5. **Custom tracking metrics:** add standardized tracking metrics and report them alongside the custom jitter and identity-fragmentation analysis.
6. **Deployment:** integrate detector inference and tracking into the C++ runtime path, then measure end-to-end latency, throughput, memory, and behavior on the intended compute target.

The most valuable next experiment would combine real synchronized camera/IMU data with calibration, run the detector on the stabilized stream, and evaluate both detection quality and end-to-end latency on a target platform.

## 8. What I learned / what this demonstrates

This project combined dataset engineering, detector training and export, geometric image transformations, synthetic sensor modeling, controlled ablations, and temporal state estimation. The experiments showed that the modules address different failure modes: stabilization recovered part of the detection loss under simulated camera motion, while Kalman tracking reduced jitter and identity fragmentation.

My primary individual contribution was the synthetic camera-motion / IMU perturbation and horizon-stabilization component. Dataset selection and the overall architecture were collaborative, as were the broader detector and tracking workflows.

## Resources

- [Project repository](https://github.com/robosangli/maritime_obstacle_detection)
- [USVTrack dataset paper](https://arxiv.org/abs/2506.18737)
- [LaRS dataset paper](https://openaccess.thecvf.com/content/ICCV2023/html/Zust_LaRS_A_Diverse_Panoptic_Maritime_Obstacle_Detection_Dataset_and_Benchmark_ICCV_2023_paper.html)
- [Ultralytics YOLOv8](https://github.com/ultralytics/ultralytics)

---

## Media and figure checklist

Create `images/projects/maritime/` in the **portfolio repository** and copy optimized assets there. Suggested mapping:

| Asset in this page | Your source artifact | Why it belongs |
|---|---|---|
| `input.mp4` | Input video | Establishes the original footage |
| `simulated-camera.mp4` | Synthetic camera / oscillation video | Shows the controlled perturbation |
| `stabilized.mp4` | Stabilized video | Shows the output of your component |
| `system-architecture.png` | New vector diagram | Explains how the modules connect and distinguishes Python experiments from the C++ prototype |
| `stabilization-comparison.png` | Frame samples from input, perturbed, and stabilized video | A quick static visual for readers who do not play videos |
| `box-pr-curve.png` | Box PR curve | Summarizes detector precision/recall behavior |
| `confidence-threshold.png` | Confidence-threshold plot | Explains why the experiment uses a recall-favoring threshold |
| `confusion-matrix-normalized.png` | Normalized confusion matrix | Makes missed obstacles visible |
| `stabilization-recall-results.png` | New chart from the ablation table | Quantifies stabilization's contribution |
| `no-filter-vs-kalman.mp4` | “no filter vs kalman” video | Primary tracking comparison |
| `demo-no-filter-vs-kalman-s99.mp4` | “demo_nofilter vs kalman(s99)” video | Optional second sequence; caption only after verifying what `s99` denotes |
| `trajectory-usv-s68.png` or `.mp4` | “trajectory usv s68” artifact | Use as a trajectory/jitter figure only after checking its axes, units, and exact content |

Use descriptive filenames and keep videos short and compressed. If a video is too large for GitHub Pages, host it through a suitable video platform or link it as an external demo rather than committing a large binary.