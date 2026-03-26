# Metrological Validation of Visual-LiDAR SLAM Sensors

<div align="justify">

**Institution:** IQS School of Engineering (Universitat Ramon Llull), Barcelona  
**Author:** Narcís Abella  
**Supervisor:** Dr. Antonio Gabino Salazar Martín  
**Status:** Design phase — pending experimental execution  
**Last updated:** 2026-03-06

</div>

---

## 0. Why this study matters

<div align="justify">

Robotics researchers rely heavily on simulation to design and tune SLAM 
systems before deployment. The dominant response to the sim-to-real gap — 
domain randomisation — has a well-documented failure mode: robots learn to 
exploit simulator inconsistencies rather than solve the actual task (Muratore 
et al., 2022; Aljalbout et al., *Annual Review*, 2026). The root cause is 
not insufficient randomisation — it is that sensor models are never validated 
against real hardware. A robot trained on an unvalidated sensor model doesn't 
learn robustness; it learns to assume the simulator's artefacts.

This study asks a very concrete question:

> *If we take sensor modelling as seriously as metrology does — using 
> a calibrated robot arm as ground truth and detailed noise models — 
> can we demonstrate statistically that simulation is sufficiently 
> faithful to real hardware for design decisions?*

The word *demonstrate* is deliberate. Current sim-to-real validation 
practice compares RMSE values or applies significance tests designed 
to detect differences — not to demonstrate sufficiency. A 
non-significant result (p > 0.05) does not mean the simulation is 
good enough; it may simply mean the sample was too small to detect a 
real gap. No existing study applies formal equivalence testing with 
pre-specified margins to sensor model validation in robotics. This 
study does.

This matters for two reasons:
- **For practice:** labs and companies make design decisions (sensor 
  selection, algorithm tuning, parameter choices) based on simulation. 
  If the sensor models are naïve, those decisions can be systematically 
  biased — and there is currently no rigorous way to know how biased.
- **For research:** many Sim2Real papers study algorithms and 
  environments, but far fewer quantify the role of sensor model 
  fidelity itself in the Reality Gap, and none do so with formal 
  equivalence testing.

Several works already cover individual pieces of this puzzle:
- **IMUs:** [Furrer et al., 2016](docs/RESEARCH_PLAN.md#9-references) 
  showed that Allan Variance coefficients must be derived from real 
  hardware logs — not datasheets — to replicate inertial drift in 
  simulation.
- **LiDARs:** [Liu & Zhang, 2021](docs/RESEARCH_PLAN.md#9-references) 
  characterised the degeneracy risks of solid-state Livox sensors and 
  their non-repetitive scan patterns.
- **Dynamic metrology:** [Lin et al., 2022 
  (*Measurement*)](docs/RESEARCH_PLAN.md#9-references) validated a 
  dynamic measurement system with a laser tracker as ground truth using 
  Allan analysis and fault-tolerant sensor fusion.
- **State-dependent covariance:** learning-based approaches 
  (VIO-DualProNet, AirIMU) already model IMU noise covariance as a 
  function of kinematic state — demonstrating that the concept is 
  physically motivated and practically useful.

What is missing — and what this study contributes — is a single, 
metrologically grounded framework that:
1. Uses a sub-millimetre industrial robot (ABB YuMi, path repeatability 
   0.10 mm) as kinematic ground truth for dynamic sensor 
   characterisation.
2. Builds an explicit, physically interpretable parametric noise model 
   (M4) whose variance depends on measured temperature and kinematic 
   state — fitted from residuals against the YuMi reference, not 
   learned from black-box data.
3. Implements M4 in a Gazebo Fortress plugin for the Livox Mid-360 
   non-repetitive scan pattern — a combination with no existing 
   open-source solution for ROS 2 Humble.
4. Validates the complete pipeline using **formal statistical 
   equivalence testing (TOST)** with pre-specified margins derived from 
   system requirements: metrological simulation is declared equivalent 
   to real hardware only if the 90% confidence interval for the mean 
   ATE difference falls within those margins — not from post-hoc 
   inspection of RMSE values.
5. Tests the full pipeline across multiple tight-coupled SLAM backends 
   to ensure conclusions are not an artefact of a single estimator's 
   tolerance to noise model mismatch.

M4 is not proposed as a conceptually new type of noise model — the 
idea of state-dependent covariance already exists. Its contribution 
here is different: it provides sufficient simulation fidelity for the 
equivalence test to have statistical power. Without a model of this 
precision, TOST cannot distinguish a good simulator from a bad one 
within margins that are meaningful for system design. The formula, 
candidate formulations, and physical justification are in 
[`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) §5.

The study does not assume that M4 is necessary. The four noise model configurations (M1–M4) form an epistemic ladder: from current practice (M1, datasheet) to metrologically correct but low-cost (M2, static Allan) to operationally refined (M3, in-session static) to fully state-dependent (M4, kinematic-residual and thermal). The TOST is applied at every rung. If M2 already achieves statistical equivalence within δ, that is the primary result: it means that a rigorous static Allan characterisation — accessible to any laboratory with a vibration-isolated bench and a 10–12 hour log — resolves a problem the community has spent decades either ignoring or over-complicating. M4 then serves as a positive control: without it, a passing result at M2 could be an artefact of the SLAM backends' insensitivity to high-frequency noise rather than genuine sensor model fidelity. The hierarchy is not redundant — it is what makes any result at any rung interpretable.

</div>

---

## 1. What this project is about

<div align="justify">

This repository contains the design of a stand‑alone metrological study on Visual–LiDAR SLAM. Its goal is simple to state:

> Quantitatively measure how close a simulator can get to real sensors if we model their noise and scanning behaviour with the same level of care used in instrumentation and metrology.

Concretely:
- We use an ABB YuMi collaborative robot as kinematic ground truth (path repeatability 0.10 mm; path accuracy up to 1.36 mm; pose repeatability ±0.02 mm at static points only) and RobotStudio as an independent trajectory predictor.
- We characterise three sensor families: IMUs, LiDARs (2D mechanical and 3D solid‑state Livox Mid‑360) and an RGB‑D camera (RealSense D455).
- We build a family of sensor noise models whose variance depends on:
  - measured temperature (thermal state), and
  - kinematic state of the sensor (linear and angular velocity, acceleration, jerk, angular acceleration).
- We then compare, for multiple SLAM systems, three kinds of data:
  1. Real hardware logs on the YuMi,
  2. Standard simulation (Gazebo defaults / manufacturer specs),
  3. Metrological simulation (our noise + scanning models).

The three conditions are compared using formal statistical equivalence 
testing (TOST): we declare metrological simulation equivalent to real 
hardware only if the 90% confidence interval for the mean difference 
in ATE falls within a pre-specified margin δ derived from system 
requirements — not from post-hoc inspection of the data.

The detailed scientific rationale and hypotheses live in [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md).

</div>

---

## 2. Scope and possible extensions

<div align="justify">

This study was originally conceived as the sensor‑level step of a broader research line on Sim‑to‑Real Visual–LiDAR SLAM (with possible extensions to a mobile‑platform Reality Gap benchmark and large‑scale SLAM optimisation in simulation).  
In the context of this degree project, however, only the metrological validation with YuMi ground truth is in scope. Any mobile‑platform benchmark or automatic optimisation stage is treated strictly as possible future work, not as a commitment of this project.

For a compact overview of the current study’s goals and contributions, see [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md), sections 1–2 and 6.

</div>

---

## 3. Hardware and ground truth at a glance

<div align="justify">

Ground truth platform
- ABB YuMi IRC5 dual‑arm cobot  
- Path repeatability 0.10 mm; path accuracy up to 1.36 mm; pose repeatability ±0.02 mm at static points only.  
- Trajectories programmed in RobotStudio and exported as time‑stamped poses.

Sensors under test (sessions A–D by modality: IMU-only → 2D LiDAR → 3D LiDAR → visual‑inertial)

</div>

| ID | Sensor | Type | Role |
|----|--------|------|------|
| S1 | WitMotion WT901C (MPU9250) | IMU, 200 Hz | Reference IMU (Session A) |
| S2 | RPLiDAR A2M12 | 2D LiDAR (mechanical) | 2D SLAM baseline (Session B) |
| S3 | Livox Mid‑360 (ICM40609) | 3D LiDAR + IMU | Primary 3D LiDAR (Session C) |
| S4 | Intel RealSense D455 (BMI055) | RGB‑D + IMU | Visual–inertial sensor (Session D) |

<div align="justify">

Geometry, payload and viability details are in [`docs/HARDWARE_PAYLOAD.md`](docs/HARDWARE_PAYLOAD.md).

</div>

---

## 4. High‑level experimental design

<div align="justify">

The experimental design has two stages; full details live in [`docs/EXPERIMENTAL_DESIGN.md`](docs/EXPERIMENTAL_DESIGN.md).

</div>

### 4.1 Stage 1 — Static characterisation

<div align="justify">

Performed without the robot arm:
- IMUs: 10–12 h static logs per IMU. Allan Variance (ARW, Bias Instability, RRW) + Six‑Position Test (IEEE Std 1293).
- LiDARs: sensor facing a flat wall. We track how point‑to‑plane residuals drift over time (thermal ToF drift).
- Camera: RealSense D455 observing a static AprilTag board for several hours. We quantify pose drift of the board (thermal deformation of intrinsics and FPN).

These logs feed the static parts of the noise models (manufacturer vs. long‑term Allan vs. in‑session static).

</div>

### 4.2 Stage 2 — Dynamic sessions on YuMi

<div align="justify">

Four sessions, one per sensor configuration:

</div>

| Session | Sensor | Motion DOF | Notes |
|---------|--------|-----------|-------|
| A | WitMotion WT901C | 3D | IMU‑only reference |
| B | RPLiDAR A2M12 | 2D (yaw only) | Planar LiDAR SLAM |
| C | Livox Mid‑360 | 3D | 3D LiDAR‑IMU with Rosetta pattern |
| D | RealSense D455 | 3D | Visual–inertial (RGB‑D + IMU) |

<div align="justify">

Each session follows the same pattern:
- 60 s static → Block MIX (CW/CCW/CW) → cooldown → Block CW → cooldown → Block CCW → 60 s static.
- Three trajectory profiles (smooth / moderate / aggressive) per session (T1/T2/T3), re‑used across sensors.

The static 60 s segments inside dynamic runs are also used to characterise thermal behaviour in working conditions.

</div>

---

## 5. Noise models — from datasheet to kinematic residuals

<div align="justify">

The simulator will be run under four noise model configurations (see [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md) and [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) §4–5):

1. M1 — Manufacturer: values from datasheets or typical community practice.
2. M2 — Static Allan: coefficients (ARW, BI, RRW) from long static logs (Stage 1).
3. M3 — In‑Session Static: Allan analysis on concatenated 60 s static windows within dynamic sessions (captures warm‑up and mounting effects).
4. M4 — Kinematic‑Residual + Thermal:
   - First, we reconstruct the true kinematics of the sensor from YuMi ground truth (splines + derivatives + hand‑eye).  
   - Then we compute residuals (measured − true) for IMU, LiDAR and camera.  
   - Finally, we fit a model where the noise variance depends on measured temperature and on kinematic state:
     - IMU: rotation, acceleration, jerk, angular acceleration (g‑sensitivity, vibration).  
     - LiDAR: velocity, rotation, acceleration, jerk (motion blur, ToF drift, vibration).  
     - Camera: thermal drift of intrinsics; velocity and rotation for motion blur.

</div>

### 5.1 Model M4: parametric state-dependent covariance for simulation fidelity

<div align="justify">

M4 is not proposed as a conceptually new noise model — state-dependent covariance estimation already exists in the literature via learning-based approaches (VIO-DualProNet, AirIMU). The contribution of M4 is different: it provides an explicit, physics-motivated parametric formula with identifiable coefficients that can be (a) fitted from residuals against a sub-millimetre kinematic reference, (b) interpreted physically per coefficient, and (c) injected into a Gazebo plugin with sufficient fidelity for TOST to have statistical power. Without a model of this precision, the equivalence test cannot distinguish between a good simulator and a bad one within meaningful margins.

The aim is to demonstrate empirically which formulation fits the data, not to deduce it a priori. Several candidate formulas are proposed in [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md#54-candidate-formulas-for-the-kinematic-term-to-validate-empirically) §5.4; the experimental study will validate or discard each.

<a id="m4-target-formula"></a>

$\sigma^2(t) = f\bigl( T(t), \|v(t)\|, \|\omega(t)\|, \|a(t)\|, \|\dot{\omega}(t)\|, \|\dot{a}(t)\| \bigr)$

where:
- $T(t)$ is measured temperature (thermal state).
- $\|v(t)\|$ is linear velocity magnitude.
- $\|\omega(t)\|$ is angular velocity magnitude (rotation).
- $\|a(t)\|$ is linear acceleration magnitude.
- $\|\dot{\omega}(t)\|$ is angular acceleration magnitude.
- $\|\dot{a}(t)\|$ is jerk magnitude (rate of change of linear acceleration).

The model is grounded in: static Allan Variance [\[1\]](docs/RESEARCH_PLAN.md#9-references), exponential thermal settling [\[11\]](docs/RESEARCH_PLAN.md#9-references), [\[12\]](docs/RESEARCH_PLAN.md#9-references), g-sensitivity and motion-dependent noise in MEMS [\[11\]](docs/RESEARCH_PLAN.md#9-references), residual-based identification with ground truth [\[13\]](docs/RESEARCH_PLAN.md#9-references), and robot-arm validation [\[10\]](docs/RESEARCH_PLAN.md#9-references). See [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md) §3.6 and References for full citations.

</div>

---

## 6. SLAM backends and comparison logic

<div align="justify">

SLAM algorithms are treated as measuring instruments: we use several per modality to avoid conclusions that depend on a single implementation. Using multiple backends also strengthens the statistical argument: if metrological simulation is declared equivalent to real hardware across FAST-LIO2, GLIM, and ORB-SLAM3 simultaneously, the conclusion is not an artefact of a single estimator's tolerance to noise model mismatch. The full list and matrix are in [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md); in summary:

- **Cross-modal baseline:** `GLIM` (factor graph, GPU, tight-coupled LiDAR+IMU and visual-inertial).
- **LiDAR 3D (Mid-360):** `FAST-LIO2`, `Point-LIO`, `GLIM`.
- **LiDAR 2D (RPLiDAR):** `Cartographer 2D`, `KISS-ICP 2D`, `GLIM`   (planar 3D, experimental).
- **Visual / Visual–inertial (D455):** `ORB-SLAM3`, `GLIM`, `OpenVINS`. (R3LIVE excluded — camera+LiDAR fusion out of scope for   Session D.)
- **Loose-coupled baseline (optional):** `RTAB-Map` in 2D, 3D and RGB-D to contrast tight vs. loose coupling.

For each backend:
- We tune a nominal configuration on real data (with M2/M3) and freeze it.
- We derive two sensitivity variants (LOW/HIGH) by scaling key covariances.
- We run all configurations on real data (YuMi) and on simulations with M1–M4.

Trajectories are compared against the YuMi using `evo` (ATE/RPE with Umeyama alignment). Metrological simulation (M4) is declared equivalent to real hardware only if the 90% confidence interval for 
the mean ATE difference lies entirely within the pre-specified equivalence margin δ — applied independently per backend and per trajectory type. The statistical design (TOST, Bonferroni correction, effect sizes) is in [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) §2–3.

</div>

---

## 7. How to read this repository

<div align="justify">

- Quick understanding of the idea:  
  - This `README.md`  
  - [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md) — sections 1–2, 3.6 (M4 justification), and 6  
- Exact experimental protocol (what is done with the YuMi):  
  - [`docs/EXPERIMENTAL_DESIGN.md`](docs/EXPERIMENTAL_DESIGN.md)  
- Mathematical/statistical detail (Allan, residuals, tests):  
  - [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md)  
- Which SLAMs and noise models are tested and how:  
  - [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md)  
- Hardware constraints and payload analysis:  
  - [`docs/HARDWARE_PAYLOAD.md`](docs/HARDWARE_PAYLOAD.md)  
- Current status and decisions taken so far:  
  - [`PROGRESS_LOG.md`](PROGRESS_LOG.md)
- Extended reference pool (for manuscript and experimentation):  
  - [`docs/REFERENCES_POOL.md`](docs/REFERENCES_POOL.md)

</div>

---

## 8. Execution timeline (planned)

| Period | Milestone |
|--------|-----------|
| Until June 2026 | Stage 1 (static characterisation); preparation (mounts, sync, hand-eye protocol). Mid-360 warm-up protocol and cross-session hand-eye verification procedure defined and documented. |
| June 2026 | YuMi dynamic sessions A–D (after exams). Order of sessions at author's discretion. |
| From July 2026 | **Gazebo Fortress plugin for Mid-360 non-repetitive scan pattern** (standalone open-source deliverable, ROS 2 Humble compatible — no existing solution); M4 model fitting from YuMi residuals; SLAM evaluation matrix with TOST equivalence analysis. |
| Q4 2026 – Q1 2027 | Statistical analysis, manuscript submission to *Measurement* / IEEE TIM. Plugin released independently on GitHub prior to paper submission. |

---

## 9. Known limitations

<div align="justify">

- **Ground truth bound:** For dynamic trajectories the floor is set by path repeatability (0.10 mm) and path accuracy (up to 1.36 mm). Pose repeatability (±0.02 mm) applies only to static points. Absolute volumetric accuracy is not characterised.

- **No spinning mechanical LiDAR:** LIO-SAM and similar algorithms optimised for repetitive-pattern LiDARs are not included; extension possible if a Velodyne/Ouster becomes available.

- **Thermal model scope:** Thermal models assume quasi-steady-state operation; cold-start transients &lt;5 min (typical MEMS warm-up) are not fully modelled.

- **Cable routing effects:** USB 3.0 (RealSense D455) and Ethernet (Livox Mid-360) cables routed along the YuMi arm introduce variable external forces on the flange during T2/T3 dynamic trajectories. Mitigated by careful cable chain routing prior to each session; residual effect is documented as a contributor to the ground truth uncertainty budget and declared as an explicit limitation in the experimental report.

- **Mid-360 internal self-heating:** The sensor's internal detector temperature may differ from the externally measured ambient temperature by a non-negligible gradient during the first 15–20 minutes of operation. Protocol requires a minimum 20-minute powered warm-up before any static characterisation log or dynamic session begins. This is documented in [`docs/EXPERIMENTAL_DESIGN.md`](docs/EXPERIMENTAL_DESIGN.md) §2.3.

- **Hand-eye cross-session verification:** Hand-eye calibration is redone per session, but microscopic mount displacement between sessions (dismount and remount) cannot be fully excluded. A fixed verification pose at the start of each dynamic session — reprojection error check against a reference AprilTag board fixed in the environment — serves as an acceptance criterion before data collection begins. Results archived in `data/calibration/`.

- **TOST margin selection:** The equivalence margin $\delta$ must be defined *a priori* based on specific system requirements, and agreed upon before any empirical data analysis. Defining $\delta$ after inspecting data (or as a percentage of measured outcomes) would invalidate the statistical test. Results will be reported for the chosen $\delta$ and sensitivity to alternative margins will be discussed.

</div>

---

## 10. License

<div align="justify">

MIT License. See [LICENSE](LICENSE) for details.

Research conducted at IQS School of Engineering (Universitat Ramon Llull), Barcelona.

</div>
