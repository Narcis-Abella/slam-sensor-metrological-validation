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
- **For practice:** Labs and companies make design decisions (sensors, algorithms, parameter tuning) based on simulation. If the sensor models are naïve, those decisions can be systematically biased.
- **For research:** Many Sim2Real papers study algorithms and environments, but far fewer quantify the role of **sensor model fidelity itself** in the Reality Gap.

Several works already cover individual pieces of this puzzle:
- **IMUs:** [Furrer et al., 2016](docs/RESEARCH_PLAN.md#9-references) showed that you must derive Allan Variance coefficients from real logs, not datasheets, to match inertial drift in simulation.
- **LiDARs:** [Liu & Zhang, 2021](docs/RESEARCH_PLAN.md#9-references) analysed the degeneracy risks of solid‑state Livox sensors and their non‑repetitive scan patterns.
- **Dynamic metrology:** [Lin et al., 2022 (*Measurement*)](docs/RESEARCH_PLAN.md#9-references) validated dynamic measurement systems with a laser tracker as ground truth, using Allan analysis and fault‑tolerant sensor fusion.

What is missing — and what this study contributes — is a **single, metrologically grounded framework** that:
1. Uses a sub‑millimetre industrial robot (ABB YuMi) as kinematic ground truth.
2. Builds **state‑dependent sensor noise models** whose variance changes with time since power‑on and with the sensor’s motion.
3. Tests those models across **multiple tight‑coupled SLAM backends**, comparing real sensors, standard simulation and metrological simulation under the same trajectories.

The core theoretical contribution is an explicit covariance formula [(model M4)](https://github.com/Narcis-Abella/slam-sensor-metrological-validation?tab=readme-ov-file#51-key-contribution-kinematic-state-dependent-covariance-model-m4) where the noise variance depends both on thermal state and on kinematic state; the formula and its intuition are kept in this README, while the full derivation and justification live in the technical documents under `docs/`.

---

## 1. What this project is about

This repository contains the design of a **stand‑alone metrological study** on Visual–LiDAR SLAM. Its goal is simple to state:

> Quantitatively measure how close a simulator can get to real sensors if we model their noise and scanning behaviour with the same level of care used in instrumentation and metrology.

Concretely:
- We use an **ABB YuMi** collaborative robot as kinematic ground truth (±0.02 mm repeatability) and RobotStudio as an independent trajectory predictor.
- We characterise three sensor families: **IMUs**, **LiDARs** (2D mechanical and 3D solid‑state Livox Mid‑360) and an **RGB‑D camera** (RealSense D455).
- We build a family of **sensor noise models** whose variance depends on:
  - time since power‑on (thermal state), and
  - kinematic state of the sensor (velocity, acceleration, jerk).
- We then compare, for multiple SLAM systems, three kinds of data:
  1. **Real hardware logs** on the YuMi,
  2. **Standard simulation** (Gazebo defaults / manufacturer specs),
  3. **Metrological simulation** (our noise + scanning models).

 

The detailed scientific rationale and hypotheses live in [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md).

---

## 2. Scope and possible extensions

This study was originally conceived as the **sensor‑level step** of a broader research line on Sim‑to‑Real Visual–LiDAR SLAM (with possible extensions to a mobile‑platform Reality Gap benchmark and large‑scale SLAM optimisation in simulation).  
In the context of this degree project, however, **only the metrological validation with YuMi ground truth is in scope**. Any mobile‑platform benchmark or automatic optimisation stage is treated strictly as **possible future work**, not as a commitment of this project.

For a compact overview of the current study’s goals and contributions, see [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md), sections 1–2 and 6.

---

## 3. Hardware and ground truth at a glance

**Ground truth platform**
- ABB YuMi IRC5 dual‑arm cobot  
- Repeatability: ±0.02 mm  
- Trajectories programmed in RobotStudio and exported as time‑stamped poses.

**Sensors under test**

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
- **IMUs:** 10–12 h static logs per IMU. Allan Variance (ARW, Bias Instability, RRW) + Six‑Position Test (IEEE Std 1293).
- **LiDARs:** sensor facing a flat wall. We track how point‑to‑plane residuals drift over time (thermal ToF drift).
- **Camera:** RealSense D455 observing a static AprilTag board for several hours. We quantify pose drift of the board (thermal deformation of intrinsics and FPN).

These logs feed the **static** parts of the noise models (manufacturer vs. long‑term Allan vs. in‑session static).

### 4.2 Stage 2 — Dynamic sessions on YuMi

Four sessions, one per sensor configuration:

| Session | Sensor | Motion DOF | Notes |
|---------|--------|-----------|-------|
| A | WitMotion WT901C | 3D | IMU‑only reference |
| B | RealSense D455 | 3D | Visual–inertial (RGB‑D + IMU) |
| C | RPLiDAR A2M12 | 2D (yaw only) | Planar LiDAR SLAM |
| D | Livox Mid‑360 | 3D | 3D LiDAR‑IMU with Rosetta pattern |

Each session follows the same pattern:
- 60 s static → **Block MIX** (CW/CCW/CW) → cooldown → **Block CW** → cooldown → **Block CCW** → 60 s static.
- Three trajectory profiles (smooth / moderate / aggressive) per session (T1/T2/T3), re‑used across sensors.

The static 60 s segments inside dynamic runs are also used to characterise **thermal behaviour in working conditions**.

---

## 5. Noise models — from datasheet to kinematic residuals

The simulator will be run under four noise model configurations (see [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md) and [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) §4–5):

1. **M1 — Manufacturer:**  
   Values from datasheets or typical community practice.
2. **M2 — Static Allan:**  
   Coefficients (ARW, BI, RRW) from long static logs (Stage 1).
3. **M3 — In‑Session Static:**  
   Allan analysis on concatenated 60 s static windows within dynamic sessions (captures warm‑up and mounting effects).
4. **M4 — Kinematic‑Residual + Thermal:**  
   - First, we reconstruct the true kinematics of the sensor from YuMi ground truth (splines + derivatives + hand‑eye).  
   - Then we compute residuals (measured − true) for IMU, LiDAR and camera.  
   - Finally, we fit a model where the noise variance depends on **time since power‑on** and on **kinematic state**:
     - IMU: rotation, acceleration and jerk (g‑sensitivity, vibration).  
     - LiDAR: Time‑of‑Flight drift and spread under high rotational speed / acceleration.  
     - Camera: thermal drift of intrinsics and motion‑blur‑induced degradation.

### 5.1 Key contribution: kinematic-state-dependent covariance model (M4)

No prior work has combined thermal decay with explicit kinematic-state dependence in a single covariance model for SLAM sensor simulation. Our M4 model expresses the variance as:

$$
\sigma^2(t) =
\sigma^2_{\mathrm{static}}\bigl(t_{\mathrm{on}}(t)\bigr) +
c_v \|\omega_{\mathrm{true}}(t)\| +
c_a \|a_{\mathrm{true}}(t)\| +
c_j \|\dot{a}_{\mathrm{true}}(t)\|,
$$

where:
- $t_{\mathrm{on}}(t)$ is time since power-on
- $\sigma^2_{\mathrm{static}}(t_{\mathrm{on}})$ decays from cold start to steady state (exponential with thermal time constant $\tau_T$)
- $c_v, c_a, c_j$ are fitted from residuals over kinematic regimes S1–S3.

This captures g-sensitivity, structural vibration and motion blur as a function of
- Angular velocity $\|\omega\|$
- Acceleration $\|a\|$
- Jerk $\|\dot{a}\|$

Rather than assuming constant noise as in standard simulation. 

The model is grounded in validated practice: static Allan Variance [\[1\]](docs/RESEARCH_PLAN.md#9-references), exponential thermal settling for inertial sensors [\[11\]](docs/RESEARCH_PLAN.md#9-references), [\[12\]](docs/RESEARCH_PLAN.md#9-references), known g-sensitivity and motion-dependent noise in MEMS [\[11\]](docs/RESEARCH_PLAN.md#9-references), residual-based identification with ground truth [\[13\]](docs/RESEARCH_PLAN.md#9-references), and robot-arm validation [\[10\]](docs/RESEARCH_PLAN.md#9-references); the novelty is the unified formula and its use for SLAM simulation. See [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md) §3.6 and References for full citations.

---

## 6. SLAM backends and comparison logic

SLAM algorithms are treated as **measuring instruments**: we use several per modality to avoid conclusions that depend on a single implementation. The full list and matrix are in [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md); in resumen:

- **Cross‑modal baseline:** `GLIM` (factor graph, GPU, tight‑coupled LiDAR+IMU and visual‑inertial).
- **LiDAR 3D (Mid‑360):** `FAST-LIO2`, `Point-LIO`, `GLIM`.
- **LiDAR 2D (RPLiDAR):** `Cartographer 2D`, `KISS-ICP 2D`, `GLIM` (planar 3D, experimental).
- **Visual / Visual–inertial (D455):** `ORB-SLAM3`, `GLIM`, `R3LIVE` (donde sea viable).
- **Loose‑coupled baseline (opcional):** `RTAB-Map` en 2D, 3D y RGB‑D para contrastar tight vs. loose coupling.

Para cada backend:
- Ajustamos una configuración **nominal** en datos reales (con M2/M3) y la congelamos.
- Derivamos dos variantes de sensibilidad (LOW/HIGH) escalando covarianzas clave.
- Ejecutamos todas estas configuraciones sobre:
  - datos reales (YuMi), y  
  - simulaciones con M1–M4.

Las trayectorias se comparan contra el YuMi usando `evo` (ATE/RPE con alineación de Umeyama). El diseño estadístico (tests, corrección de Bonferroni, tamaños de efecto) está en [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md) §2–3.

---

## 7. How to read this repository

- **Quick understanding of the idea:**  
  - This `README.md`  
  - [`docs/RESEARCH_PLAN.md`](docs/RESEARCH_PLAN.md) — sections 1–2, 3.6 (M4 justification), and 6  
- **Exact experimental protocol (qué se hace con el YuMi):**  
  - [`docs/EXPERIMENTAL_DESIGN.md`](docs/EXPERIMENTAL_DESIGN.md)  
- **Mathematical/statistical detail (Allan, residuales, tests):**  
  - [`docs/METHODOLOGY.md`](docs/METHODOLOGY.md)  
- **Which SLAMs and noise models are tested and how:**  
  - [`docs/SLAM_BACKENDS.md`](docs/SLAM_BACKENDS.md)  
- **Hardware constraints and payload analysis:**  
  - [`docs/HARDWARE_PAYLOAD.md`](docs/HARDWARE_PAYLOAD.md)  
- **Current status and decisions taken so far:**  
  - [`PROGRESS_LOG.md`](PROGRESS_LOG.md)
- **Extended reference pool (for manuscript and experimentation):**  
  - [`docs/REFERENCES_POOL.md`](docs/REFERENCES_POOL.md)

---

## 8. Execution timeline (planned)

| Period | Milestone |
|--------|-----------|
| Q2 2026 | Stage 1 (static characterisation) + YuMi trajectory validation |
| Q3 2026 | Dynamic sessions A–D data collection |
| Q4 2026 | M4 model fitting + SLAM evaluation matrix |
| Q1 2027 | Manuscript submission to *Measurement* / IEEE TIM |

---

## 9. Known limitations

- **Ground truth bound:** YuMi repeatability (±0.02 mm) bounds the lower limit of ground truth confidence; absolute volumetric accuracy is not characterised.
- **No spinning mechanical LiDAR:** LIO-SAM and similar algorithms optimised for repetitive-pattern LiDARs are not included; extension possible if a Velodyne/Ouster becomes available.
- **Thermal model scope:** Thermal models assume quasi-steady-state operation; cold-start transients &lt;5 min (typical MEMS warm-up) are not fully modelled.

---

## 10. License

MIT License. See [LICENSE](LICENSE) for details.

Research conducted at IQS School of Engineering (Universitat Ramon Llull), Barcelona.
