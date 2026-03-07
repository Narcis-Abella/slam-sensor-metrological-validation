# Metrological Validation of Visual-LiDAR SLAM Sensors

**Institution:** IQS School of Engineering (Universitat Ramon Llull), Barcelona  
**Author:** Narcís Abella  
**Supervisor:** Dr. Antonio Gabino Salazar Martín  
**Status:** 🟡 Design phase — pending experimental execution  
**Last updated:** 2026-03-06

---

## 0. Why this study matters

Robotics researchers and practitioners rely heavily on simulation to design and tune SLAM systems before moving to real hardware. In most simulators, sensors are modelled with very simple, constant Gaussian noise. On a robot that moves fast, heats up and vibrates, those assumptions are wrong — and systems that look robust in simulation can fail once they leave the lab.

This study asks a very concrete question:

> *If we take sensor modelling as seriously as metrology does — using a calibrated robot arm as ground truth and detailed noise models — how close can simulation really get to real sensors?*

This is important for two reasons:
- For practice: labs and companies make design decisions (sensors, algorithms, parameter tuning) based on simulation. If the sensor models are naïve, those decisions can be systematically biased.
- For research: many Sim2Real papers study algorithms and environments, but far fewer quantify the role of sensor model fidelity itself in the Reality Gap.

Several works already cover individual pieces of this puzzle:
- IMUs: [Furrer et al., 2016](docs/RESEARCH_PLAN.md#9-references) showed that you must derive Allan Variance coefficients from real logs, not datasheets, to match inertial drift in simulation.
- LiDARs: [Liu & Zhang, 2021](docs/RESEARCH_PLAN.md#9-references) analysed the degeneracy risks of solid‑state Livox sensors and their non‑repetitive scan patterns.
- Dynamic metrology: [Lin et al., 2022 (*Measurement*)](docs/RESEARCH_PLAN.md#9-references) validated dynamic measurement systems with a laser tracker as ground truth, using Allan analysis and fault‑tolerant sensor fusion.

What is missing — and what this study contributes — is a single, metrologically grounded framework that:
1. Uses a sub‑millimetre industrial robot (ABB YuMi) as kinematic ground truth.
2. Builds state‑dependent sensor noise models whose variance changes with time since power‑on and with the sensor’s motion.
3. Tests those models across multiple tight‑coupled SLAM backends, comparing real sensors, standard simulation and metrological simulation under the same trajectories.

The core theoretical contribution is an explicit covariance formula [(model M4)](#m4-target-formula) where the noise variance depends both on thermal state and on kinematic state; the formula and its intuition are in this README, while candidate formulations and full justification live under `docs/`.

---

## 1. What this project is about

This repository contains the design of a stand‑alone metrological study on Visual–LiDAR SLAM. Its goal is simple to state:

> Quantitatively measure how close a simulator can get to real sensors if we model their noise and scanning behaviour with the same level of care used in instrumentation and metrology.

Concretely:
- We use an ABB YuMi collaborative robot as kinematic ground truth (±0.02 mm repeatability) and RobotStudio as an independent trajectory predictor.
- We characterise three sensor families: IMUs, LiDARs (2D mechanical and 3D solid‑state Livox Mid‑360) and an RGB‑D camera (RealSense D455).
- We build a family of sensor noise models whose variance depends on:
  - time since power‑on (thermal state), and
  - kinematic state of the sensor (linear and angular velocity, acceleration, jerk, angular acceleration).
- We then compare, for multiple SLAM systems, three kinds of data:
  1. Real hardware logs on the YuMi,
  2. Standard simulation (Gazebo defaults / manufacturer specs),
  3. Metrological simulation (our noise + scanning models).

 

The detailed scientific rationale and hypotheses live in [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md).

---

## 2. Scope and possible extensions

This study was originally conceived as the sensor‑level step of a broader research line on Sim‑to‑Real Visual–LiDAR SLAM (with possible extensions to a mobile‑platform Reality Gap benchmark and large‑scale SLAM optimisation in simulation).  
In the context of this degree project, however, only the metrological validation with YuMi ground truth is in scope. Any mobile‑platform benchmark or automatic optimisation stage is treated strictly as possible future work, not as a commitment of this project.

For a compact overview of the current study’s goals and contributions, see [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md), sections 1–2 and 6.

---

## 3. Hardware and ground truth at a glance

Ground truth platform
- ABB YuMi IRC5 dual‑arm cobot  
- Repeatability: ±0.02 mm  
- Trajectories programmed in RobotStudio and exported as time‑stamped poses.

Sensors under test

| ID | Sensor | Type | Role |
|----|--------|------|------|
| S1 | WitMotion WT901C (MPU9250) | IMU, 200 Hz | Reference IMU (Session A) |
| S2 | Intel RealSense D455 (BMI055) | RGB‑D + IMU | Visual–inertial sensor (Session B) |
| S3 | Livox Mid‑360 (ICM40609) | 3D LiDAR + IMU | Primary 3D LiDAR (Session D) |
| S4 | RPLiDAR A2M12 | 2D LiDAR (mechanical) | 2D SLAM baseline (Session C) |

Geometry, payload and viability details are in [`docs/HARDWARE_PAYLOAD.md`](docs/HARDWARE_PAYLOAD.md).

---

## 4. High‑level experimental design

The experimental design has two stages; full details live in [`docs/EXPERIMENTAL_DESIGN.md`](docs/EXPERIMENTAL_DESIGN.md).

### 4.1 Stage 1 — Static characterisation

Performed without the robot arm:
- IMUs: 10–12 h static logs per IMU. Allan Variance (ARW, Bias Instability, RRW) + Six‑Position Test (IEEE Std 1293).
- LiDARs: sensor facing a flat wall. We track how point‑to‑plane residuals drift over time (thermal ToF drift).
- Camera: RealSense D455 observing a static AprilTag board for several hours. We quantify pose drift of the board (thermal deformation of intrinsics and FPN).

These logs feed the static parts of the noise models (manufacturer vs. long‑term Allan vs. in‑session static).

### 4.2 Stage 2 — Dynamic sessions on YuMi

Four sessions, one per sensor configuration:

| Session | Sensor | Motion DOF | Notes |
|---------|--------|-----------|-------|
| A | WitMotion WT901C | 3D | IMU‑only reference |
| B | RealSense D455 | 3D | Visual–inertial (RGB‑D + IMU) |
| C | RPLiDAR A2M12 | 2D (yaw only) | Planar LiDAR SLAM |
| D | Livox Mid‑360 | 3D | 3D LiDAR‑IMU with Rosetta pattern |

Each session follows the same pattern:
- 60 s static → Block MIX (CW/CCW/CW) → cooldown → Block CW → cooldown → Block CCW → 60 s static.
- Three trajectory profiles (smooth / moderate / aggressive) per session (T1/T2/T3), re‑used across sensors.

The static 60 s segments inside dynamic runs are also used to characterise thermal behaviour in working conditions.

---

## 5. Noise models — from datasheet to kinematic residuals

The simulator will be run under four noise model configurations (see [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md) and [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) §4–5):

1. M1 — Manufacturer: values from datasheets or typical community practice.
2. M2 — Static Allan: coefficients (ARW, BI, RRW) from long static logs (Stage 1).
3. M3 — In‑Session Static: Allan analysis on concatenated 60 s static windows within dynamic sessions (captures warm‑up and mounting effects).
4. M4 — Kinematic‑Residual + Thermal:
   - First, we reconstruct the true kinematics of the sensor from YuMi ground truth (splines + derivatives + hand‑eye).  
   - Then we compute residuals (measured − true) for IMU, LiDAR and camera.  
   - Finally, we fit a model where the noise variance depends on time since power‑on and on kinematic state:
     - IMU: rotation, acceleration, jerk, angular acceleration (g‑sensitivity, vibration).  
     - LiDAR: velocity, rotation, acceleration, jerk (motion blur, ToF drift, vibration).  
     - Camera: thermal drift of intrinsics; velocity and rotation for motion blur.

### 5.1 Key contribution: kinematic-state-dependent covariance model (M4)

Goal: An explicit variance formula with identifiable coefficients — fitted from residuals against YuMi ground truth — that depends on thermal state (via time since power-on) and kinematic state (linear and angular velocity, acceleration, jerk, angular acceleration). *Known limitation:* temperature also depends on ambient conditions; only time since power-on is modelled.

The aim is to demonstrate empirically which formulation fits the data, not to deduce it a priori. Several candidate formulas are proposed in [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md#54-candidate-formulas-for-the-kinematic-term-to-validate-empirically) §5.4; the experimental study will validate or discard each.

<a id="m4-target-formula"></a>

$$
\sigma^2(t) = f\bigl( t_{\mathrm{on}}(t),\, \|v(t)\|,\, \|\omega(t)\|,\, \|a(t)\|,\, \|\dot{\omega}(t)\|,\, \|\dot{a}(t)\| \bigr)
$$

where:
- $t_{\mathrm{on}}(t)$ is time since power-on (thermal state).
- $\|v(t)\|$ is linear velocity magnitude.
- $\|\omega(t)\|$ is angular velocity magnitude (rotation).
- $\|a(t)\|$ is linear acceleration magnitude.
- $\|\dot{\omega}(t)\|$ is angular acceleration magnitude.
- $\|\dot{a}(t)\|$ is jerk magnitude (rate of change of linear acceleration).

The model is grounded in: static Allan Variance [\[1\]](docs/RESEARCH_PLAN.md#9-references), exponential thermal settling [\[11\]](docs/RESEARCH_PLAN.md#9-references), [\[12\]](docs/RESEARCH_PLAN.md#9-references), g-sensitivity and motion-dependent noise in MEMS [\[11\]](docs/RESEARCH_PLAN.md#9-references), residual-based identification with ground truth [\[13\]](docs/RESEARCH_PLAN.md#9-references), and robot-arm validation [\[10\]](docs/RESEARCH_PLAN.md#9-references). See [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md) §3.6 and References for full citations.

---

## 6. SLAM backends and comparison logic

SLAM algorithms are treated as measuring instruments: we use several per modality to avoid conclusions that depend on a single implementation. The full list and matrix are in [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md); in summary:

- Cross‑modal baseline: `GLIM` (factor graph, GPU, tight‑coupled LiDAR+IMU and visual‑inertial).
- LiDAR 3D (Mid‑360): `FAST-LIO2`, `Point-LIO`, `GLIM`.
- LiDAR 2D (RPLiDAR): `Cartographer 2D`, `KISS-ICP 2D`, `GLIM` (planar 3D, experimental).
- Visual / Visual–inertial (D455): `ORB-SLAM3`, `GLIM`, `OpenVINS`. (R3LIVE excluded — camera+LiDAR fusion out of scope for Session B.)
- Loose‑coupled baseline (optional): `RTAB-Map` in 2D, 3D and RGB‑D to contrast tight vs. loose coupling.

For each backend:
- We tune a nominal configuration on real data (with M2/M3) and freeze it.
- We derive two sensitivity variants (LOW/HIGH) by scaling key covariances.
- We run all these configurations on:
  - real data (YuMi), and  
  - simulations with M1–M4.

Trajectories are compared against the YuMi using `evo` (ATE/RPE with Umeyama alignment). The statistical design (tests, Bonferroni correction, effect sizes) is in [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) §2–3.

---

## 7. How to read this repository

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

---

## 8. Execution timeline (planned)

| Period | Milestone |
|--------|-----------|
| Until June 2026 | Stage 1 (static characterisation); preparation (mounts, sync, hand-eye protocol). |
| June 2026 | YuMi dynamic sessions A–D (after exams). Order of sessions at author’s discretion. |
| From July 2026 | M4 model fitting, Gazebo plugin development (incl. non-repetitive scan pattern), SLAM evaluation matrix. |
| Q4 2026 – Q1 2027 | Statistical analysis, manuscript submission to *Measurement* / IEEE TIM. |

---

## 9. Known limitations

- Ground truth bound: YuMi repeatability (±0.02 mm) bounds the lower limit of ground truth confidence; absolute volumetric accuracy is not characterised.
- No spinning mechanical LiDAR: LIO-SAM and similar algorithms optimised for repetitive-pattern LiDARs are not included; extension possible if a Velodyne/Ouster becomes available.
- Thermal model scope: Thermal models assume quasi-steady-state operation; cold-start transients &lt;5 min (typical MEMS warm-up) are not fully modelled.

---

## 10. License

MIT License. See [LICENSE](LICENSE) for details.

Research conducted at IQS School of Engineering (Universitat Ramon Llull), Barcelona.
