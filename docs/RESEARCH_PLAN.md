# Research Plan — Phase I: Metrological Validation of Visual-LiDAR SLAM Sensors

**Author:** Narcís Abella  
**Supervisor:** Dr. Antonio Gabino Salazar Martín  
**Institution:** IQS School of Engineering (Universitat Ramon Llull), Barcelona  
**Version:** 0.5 — Design phase, pending supervisor review  
**Date:** 2026-03-23

---

## Table of Contents

1. [Motivation and Scientific Rationale](#1-motivation-and-scientific-rationale)
2. [Scope and Strategic Reorientation](#2-scope-and-strategic-reorientation)
3. [Related Work](#3-related-work)
4. [Research Hypothesis](#4-research-hypothesis)
5. [Experimental Platform](#5-experimental-platform)
6. [Phase I Contributions](#6-phase-i-contributions)
7. [Planned SLAM Backends](#7-planned-slam-backends)
8. [Connection to Future Phases](#8-connection-to-future-phases)
9. [References](#9-references)

---

## 1. Motivation and Scientific Rationale

Standard robotic simulations typically rely on generic Gaussian noise models and idealized scanning geometries. Several sensor-specific effects are not captured by these models.

In MEMS IMUs, **Angle Random Walk (ARW)** and **Bias Instability** determine how fast orientation error accumulates during motion. **Rate Random Walk (RRW)** is a slower noise process that only becomes visible in logs longer than a few hours — yet it sets the long-term drift floor in any tight-coupled system. For solid-state LiDARs, the non-repetitive Rosetta scanning pattern of the Livox Mid-360 creates short-term point cloud gaps that increase geometric degeneracy risk in low-variation environments; traditional planar LiDAR models miss this entirely. Finally, thermal drift in IMU biases and camera intrinsics is not static: it evolves over the operational cycle, and a simulator that ignores it will consistently underestimate localization error in real deployments.

Furrer et al. (2016) showed that Allan Variance-derived coefficients — not datasheet values — are required to replicate IMU drift in simulation. Liu & Zhang (2021) characterized the degeneracy risks of Livox-type sensors. Filipenko & Afanasyev (2018) found that overly optimistic sensor models in simulation produce systems that appear robust until deployed on hardware.

What remains poorly characterized is the combined effect: sensor model fidelity on end-to-end SLAM performance, measured across hardware and simulation simultaneously, under a controlled kinematic reference — and validated using formal statistical equivalence testing rather than informal RMSE comparison. Most studies treat sensor modeling and sim-to-real gap as separate problems, few use an industrial manipulator as trajectory reference, and none apply equivalence testing with pre-specified margins to determine whether simulation is sufficiently faithful for design decisions. This study addresses all three gaps in a single experimental framework — from static Allan Variance characterization to dynamic validation on YuMi arm trajectories, comparing real hardware, standard Gazebo simulation, and a metrological plugin in the same pipeline.

---

## 2. Scope and Strategic Reorientation

The original three-phase proposal (Abella, 2026) covered: (I) metrological sensor validation, (II) sim-to-real gap benchmarking on a mobile platform, and (III) massive-scale autonomous SLAM optimization. Based on supervisor feedback (A.G. Salazar, February 2026), this work is deliberately scoped to Phase I only.

Three reasons support keeping the scope to Phase I only.

A complete metrological validation of sensor models — with rigorous statistical equivalence testing against a sub-millimeter ground truth — is a contribution that stands on its own. Bundling it with a sim-to-real gap benchmark or a parameter optimization study would dilute the metrological result rather than strengthen it. **The YuMi arm** is the right tool for this specific question: controlled trajectories, known kinematics, and sub-millimeter path repeatability (0.10 mm per ISO 9283) are not replicable on a mobile base, and that clean experimental setting is what makes the sensor-level comparison credible. **Phases II and III** introduce additional variables — platform dynamics, uncontrolled environments, thousands of simulation runs — that each deserve their own experimental design; they are future work, not omissions.

To be explicit about scope: this study characterizes each sensor statically (noise coefficients via Allan Variance), validates the models dynamically (against YuMi ground truth trajectories), and tests whether a metrological Gazebo plugin using those coefficients produces trajectory estimates statistically equivalent to real hardware — as formally demonstrated by TOST with pre-specified margins. It does not evaluate loop closure, map quality, or navigation performance — those require a mobile platform and belong to the next stage.

---

## 3. Related Work

### 3.1 IMU Noise Characterization

The IEEE standard for IMU noise characterization (e.g. IEEE Std 647) defines Allan Variance (or Allan Deviation, ADEV) as the primary tool for identifying stochastic noise processes in inertial sensors. The three main parameters are:

- ARW (Angle Random Walk): white noise floor of the gyroscope; identified by the slope of $-1/2$ on the log-log ADEV curve.
- Bias Instability: minimum of the ADEV curve; the best achievable gyroscope bias stability.
- RRW (Rate Random Walk): low-frequency drift; identified by slope $+1/2$. Not always identifiable from short logs; 10–12 h static recordings are required for robust estimation.

Furrer et al. (2016) established the simulation standard for inertial sensors in the RotorS framework, demonstrating that ARW and Bias Instability must be derived empirically from hardware logs — not assumed from datasheets — to produce drift predictions that match real flight data.

The **Six-Position Test (IEEE Std 1293)** [14] complements Allan Variance by isolating scale factor errors and gravitational bias — parameters that are not observable from static horizontal logs but are critical for tight-coupling in FAST-LIO2 [22] and GLIM [23].

### 3.2 LiDAR Characterization

Liu & Zhang (2021) highlighted the geometric degeneracy risks of solid-state LiDARs in environments with low structural variation. The Livox Mid-360's non-repetitive Rosetta scanning pattern provides full spherical coverage over time but creates short-term point cloud gaps that affect frame-to-frame ICP registration quality. Characterizing this temporal integration behavior — validating that the simulated Rosetta pattern matches the physical point cloud density over time — is a core contribution of this work.

For static LiDAR characterization, we propose a **planar orthogonal residual method** (instead of absolute distance measurement): the sensor is fixed facing a flat wall, and temporal variation is evaluated by computing the orthogonal error of successive point clouds against a reference plane fitted at t=0. This isolates thermal ToF drift without introducing ICP registration errors.

Regarding vibration-induced effects, Schlager et al. (2021) [45] tested an Ouster OS1-64 spinning LiDAR on a shaker table at 6–2000 Hz, finding point dropout but without a parametric noise model. Brazeal et al. (2021) [46] simulated Risley prism misalignment in the Livox Mid-40 and found that range accuracy is largely unaffected while angular accuracy degrades — but this was not validated experimentally. No study has empirically characterized whether Livox Mid-360 range noise statistics change under mechanical vibration or kinematic stress. This study provides the first such characterization.

### 3.3 Camera Characterization

Handa et al. (2014) demonstrated that even with perfect ground truth, SLAM algorithms fail in synthetic environments due to incorrect noise modeling and lack of photorealistic texture. For the RealSense D455, the primary concerns are:

- **Fixed-Pattern Noise (FPN)** in the infrared projector — stable over short periods but temperature-dependent.
- **Thermal deformation of intrinsic parameters** — focal length and principal point drift as the projector warms up.

Monitoring 6-DOF pose estimation of a static AprilTag pattern over several hours directly quantifies these effects. If the simulator assumes a perfect camera and the real camera exhibits thermal intrinsic drift, we have a documented, quantified source of Reality Gap — which provides direct motivation for Phase II.

### 3.4 Sim-to-Real Validation Framework

Kadian et al. (2019) introduced "Sim2Real Predictivity" as a formal metric, investigating whether simulation performance correlates with real-world performance. Wang et al. (2020) used the TartanAir dataset to argue that the Reality Gap remains significant for visual odometry algorithms dependent on fine-grained texture features. In parallel, a large body of work on **domain randomization** for perception and control shows that aggressively varying textures, lighting, object materials and camera parameters in simulation can improve sim-to-real robustness — for instance in depth sensing for grasping [23] and agile visual navigation [39].

In this work we deliberately do not adopt domain randomization. Instead, we control the environment (especially in Session D) so that real and simulated scenes share simple, well-characterised patterns and focus exclusively on sensor model fidelity at the signal level — comparing raw IMU streams, point cloud statistics, and estimated trajectories between three conditions: (i) real hardware, (ii) standard Gazebo simulation, (iii) metrological simulation with empirically derived parameters.

A structural limitation of all existing sim-to-real validation work — including the studies above — is methodological: comparisons use RMSE or significance tests designed to detect differences, not to demonstrate sufficiency. VIO-DualProNet [40] and AirIMU [41] show that state-dependent covariance improves VIO performance, but neither study asks whether the simulation with improved covariance is statistically equivalent to real hardware within task-relevant bounds. Jongeneel et al. (2024, RA-L) [43] provide the most rigorous quantitative sim-to-real comparison in the contact-rich manipulation literature and still rely on trajectory-level RMSE rather than equivalence testing. This study adopts a different validation logic: formal Two One-Sided Tests (TOST) with a pre-specified equivalence margin δ, borrowed from pharmaceutical bioequivalence testing (Schuirmann, 1987) [31] and applied here for the first time to sim-to-real sensor validation in robotics.

### 3.5 Ground Truth Quality in SLAM Benchmarks

Standard SLAM benchmarks provide high-quality reference trajectories. For instance, the EuRoC MAV dataset [33] and the TUM-VI benchmark [34] use optical motion capture systems (Vicon/OptiTrack), while the Hilti Challenge [35] and the Newer College Dataset [36] employ survey-grade laser trackers or structured ICP maps. TUM-VI explicitly characterizes IMU noise via Allan Variance prior to data collection.

However, these datasets are recorded on mobile platforms or hand-held rigs. In such setups, the true kinematic state (velocity, acceleration, jerk) must be derived by numerically differentiating the ground truth pose, which amplifies noise. In contrast, an industrial manipulator like the YuMi provides a direct, analytically differentiable kinematic reference programmed in advance. This isolates the sensor noise from ground-truth differentiation artifacts.

Regarding dynamic ground truth for metrology, Lin et al. (2022, *Measurement*) [7] validated a 6-DOF dynamic measurement system using a laser tracker as ground truth, with IMU error analysis via Allan Variance and a fault-tolerant Kalman filter for sensor fusion. Kuti et al. (2024) [10] used a robot arm as high-precision reference to validate IMU orientation under dynamic trajectories. Ferguson et al. (2023) [42] used an AUBO i5 collaborative arm to estimate IMU intrinsic parameters from residuals via nonlinear least squares. Their work focuses on deterministic calibration parameters (bias, scale factor) — not on stochastic noise model coefficients as explicit functions of kinematic state for simulation purposes, which is what this study contributes.

### 3.6 Justification and Differentiation of the M4 Covariance Model

The concept of state-dependent noise covariance for inertial sensors is not new. Learning-based approaches already demonstrate its value: VIO-DualProNet (Solodar & Klein, *Engineering Applications of AI*, 2024) [40] uses dual DNNs to estimate per-timestep IMU covariance matrices, achieving 25% improvement over constant-covariance baselines. AirIMU (Qiu et al., arXiv:2310.04874, 2023) [41] jointly learns noise correction and per-frame covariance via a CNN encoder. The physical motivation — that vibration, acceleration, and thermal state affect sensor noise — is established.

The contribution of M4 is therefore not conceptual novelty but **metrological rigour and physical interpretability**. M4 differs from prior work in three specific ways.

First, M4 is **explicit and parametric** rather than learned from a black box. Each coefficient has a physical interpretation — $c_\omega$ captures g-sensitivity, $c_j$ captures vibration-induced noise — and can be independently validated against physical models. This makes the model auditable, which is a requirement for publication in metrological journals (Measurement, IEEE TIM).

Second, M4 combines **measured temperature $T(t)$ and the full kinematic state vector simultaneously**. Existing thermal calibration literature (Joerger et al., 2021; Petovello et al., 2020) models temperature-dependent bias and scale factor but not noise variance. Existing kinematic noise models cover acceleration and angular rate but not jerk, angular acceleration, and temperature simultaneously. No prior work combines both in a single parametric noise variance formula.

Third, M4 coefficients are fitted from **dynamic residuals against a sub-millimetre industrial manipulator**, not from flight data or uncontrolled platform trajectories. This provides the ground truth quality necessary for the TOST to have statistical power within margins that are meaningful for system design. Without a model of this fidelity, the equivalence test cannot distinguish a good simulator from a bad one within the chosen δ.

M4 is therefore best understood as a **necessary enabler of the validation framework** rather than as the primary contribution. The primary contribution is the TOST-based equivalence testing pipeline; M4 is what gives that pipeline sufficient resolution to be informative.

This modeling effort **differs from** continuous-time calibration frameworks such as Kalibr. Furgale et al. [37] and Rehder et al. [38] established a robust methodology to optimize spatial and temporal extrinsics alongside IMU biases using a continuous-time spline representation. While they model biases using random walks and optimize parameters against visual ground truth, they assume a constant measurement noise covariance. M4 goes a step further: it captures how the *variance* of the noise itself scales with the kinematic state, rather than just estimating the bias states or fixed extrinsics.

The sensor-specific coefficient pruning ($c_v = 0$ for IMU; full vector for LiDAR) and the held-out generalization check (fit on T1+T2, evaluate on T3) are described in [`docs/METHODOLOGY.md`](METHODOLOGY.md) §5.4.1 and §5.6.

### 3.7 Summary and Novelty Statement

The novelty of this work is primarily **methodological**, not conceptual. Each individual component has precedent:

- State-dependent IMU covariance: VIO-DualProNet [40], AirIMU [41] (learned); Kalibr + RotorS (static Allan coefficients).
- Robot arm as dynamic sensor reference: Kuti et al. (2024) [10]; Ferguson et al. (2023) [42].
- Multi-backend SLAM evaluation: GEOBENCH 2025, SLAM Hive, M2DGR.
- Livox Mid-360 scan pattern simulation: Vultaggio et al. (ISPRS 2023) [44], Livox-SDK official plugin (Gazebo Classic, EOL January 2025).
- Allan Variance for simulation parameterization: Furrer et al. (2016) [1], RotorS pipeline.

What has not been done — and what this study contributes:

**Primary contribution:** The first application of formal statistical equivalence testing (TOST) with pre-specified, system-derived margins to sim-to-real sensor model validation in robotics. The claim is not "simulation matches hardware" but "simulation is equivalent to hardware within bounds that matter for design decisions" — a fundamentally different and stronger statement that existing validation practice cannot make.

**Secondary contribution:** An explicit, physics-motivated parametric noise model (M4) that combines measured temperature and full kinematic state vector with interpretable coefficients — fitted from dynamic residuals against a sub-millimetre robot arm reference. Unlike learning-based state-dependent covariance methods, M4 is auditable, physically justified per coefficient, and directly injectable into Gazebo simulation plugins.

**Software contribution:** A CPU-based Gazebo Fortress plugin for the Livox Mid-360 non-repetitive (Rosetta) scan pattern with empirically-derived noise model, fully compatible with ROS 2 Humble. While a third-party plugin exists for Gazebo Harmonic (RobotecAI), it requires NVIDIA OptiX/CUDA for ray tracing, making it inaccessible for generic hardware. Our Fortress plugin requires no dedicated GPU, targets the current ROS 2 LTS (Humble, supported until 2027), and is the first to inject parametric state-dependent noise (M4). Community demand for a Fortress solution has been documented since April 2023 (gz-sim issue #1958) [47] without resolution.

**Experimental contribution:** First empirical characterization of whether Livox Mid-360 range noise statistics change under mechanical vibration and kinematic stress — a gap confirmed absent in the literature (Schlager et al. [45] tested Ouster only; Brazeal et al. [46] simulated Risley prism misalignment only, not validated experimentally).

The combination of these four contributions — formal equivalence testing, metrologically-fitted parametric noise model, Fortress plugin, and vibration characterization — constitutes the novelty of the work as an integrated system, even where individual components have precedent.

---

## 4. Research Hypothesis

Standard hypothesis testing (t-test, Wilcoxon) asks whether two distributions differ significantly. For sim-to-real validation, that is the wrong question: a non-significant result does not demonstrate that simulation is good enough — it may simply reflect insufficient statistical power. The correct question is whether the difference between simulation and real hardware is small enough to be practically irrelevant. Two One-Sided Tests (TOST) answer that question by testing whether the mean difference falls within a pre-specified equivalence margin δ. Equivalence is declared only when both one-sided tests reject — i.e., when the data provide positive evidence that the difference is smaller than δ in both directions.

**Primary hypothesis (H0):**
> A simulation plugin that injects empirically derived Allan Variance coefficients (ARW, Bias Instability, RRW) and enforces the correct Livox Rosetta scanning topology will produce sensor output — and resulting SLAM trajectory estimates — that are statistically equivalent to real hardware within a pre-specified margin $\delta$, as measured by ATE and RPE against the YuMi kinematic reference.

Formal test: **TOST (Two One-Sided Tests)** for equivalence [Schuirmann, 1987; Lakens, 2017 — References 31–32; Wellek, 2010 — Reference 48]. The margin $\delta$ (e.g. 5 mm ATE or 10% of reference ATE) is pre-specified before data collection. Equivalence is declared if the 90% confidence interval for the mean difference (Metrological sim − Real) lies entirely within $[-\delta, +\delta]$.

**Success criterion:** (i) **Equivalence:** TOST declares Metrological sim equivalent to Real within margin $\delta$. (ii) **Control:** Standard simulation differs significantly from Real (p < 0.05 via paired t-test or Wilcoxon), confirming that the metrological approach is necessary. If equivalence is not declared, this is also publishable — it indicates that even empirically derived parameters are insufficient.

**Pre-registration (Session D):** Session D (RealSense D455, visual-inertial) is expected to show the largest Reality Gap of all sessions, due to visual SLAM's dependence on texture and illumination that standard Gazebo does not capture. This outcome is expected and publishable — it quantifies the limitation that motivates Phase II.

**Secondary hypothesis (H1):**
> This is an exploratory hypothesis: While comparing three IMUs of different quality grades under identical kinematic conditions, we hypothesize that IMU quality (ARW, Bias Instability) has a systematic effect on tight-coupled SLAM performance (ATE/RPE). However, this effect is conditioned by the varying multi-modal fusion architectures (IMU-only, 2D LiDAR, 3D LiDAR, Visual-Inertial) present in each session, which prevents perfect isolation of the IMU variable.

**Secondary hypothesis (H2):**
> Systematic CW/CCW asymmetry is observable in gyroscope bias — i.e., bias drift differs between clockwise and counter-clockwise rotation at identical angular velocities — and this asymmetry is captured more accurately by the metrological model than by standard simulation.

---

## 5. Experimental Platform

### 5.1 Ground Truth System

**ABB YuMi IRC5** (dual-arm collaborative robot):
- Pose repeatability: $\pm0.02$ mm (ISO 9283 RP) — applies only to static points (waypoints, start/end poses).
- Linear path repeatability: $0.10$ mm (ISO 9283 RT) — relevant for dynamic trajectory comparison.
- Linear path accuracy: $1.36$ mm (ISO 9283 AT, worst-case at 1.5 m/s; lower at our ~0.1 m/s).
- Payload capacity: ~500 g per arm
- Programming: RobotStudio (offline trajectory definition and playback)
- Reference frame: mechanical flange — requires hand-eye calibration to sensor optical/inertial center (see [METHODOLOGY.md](METHODOLOGY.md))

> For dynamic trajectory comparisons (ATE), the bounding mechanical uncertainty is path repeatability (0.10 mm) and path accuracy (up to 1.36 mm). Pose repeatability ($\pm0.02$ mm) applies only to static points. This distinction must be stated explicitly in any publication.

RobotStudio provides the executed trajectory as the independent predictor of robot motion, separate from any SLAM estimate. This dual role — kinematic ground truth *and* independent simulation reference — is the main methodological strength of this setup.

### 5.2 Sensor Suite

See [`docs/HARDWARE_PAYLOAD.md`](HARDWARE_PAYLOAD.md) for full specification, weight breakdown, and session viability assessment.

Sessions are ordered by modality complexity (A = IMU-only, B = 2D LiDAR, C = 3D LiDAR, D = visual-inertial):

| ID | Sensor | IMU chip | IMU rate | LiDAR type | Session |
|----|--------|----------|----------|------------|---------|
| S1 | WitMotion WT901C | MPU9250 | 200 Hz | — | A |
| S2 | RPLiDAR A2M12 | — | — | Mechanical 2D, 360° | B |
| S3 | Livox Mid-360 | ICM40609 | 200 Hz | Solid-state, 360° | C |
| S4 | Intel RealSense D455 | BMI055 | 400 Hz | — | D |

IMU quality ranking (reference for H1 analysis):
1. WitMotion WT901C (MPU9250) — mid-high grade; literature-characterized chip; reference IMU of the study
2. Livox Mid-360 (ICM40609) — mid grade; factory-calibrated LiDAR-IMU extrinsics
3. RealSense D455 (BMI055) — low-mid grade; notoriously noisy for standalone navigation; factory-calibrated camera-IMU extrinsics

### 5.3 Software Stack

- **ROS 2 Humble** — middleware and data logging
- **RobotStudio** — trajectory programming and ground truth export
- **Python / imu_utils / kalibr** — Allan Variance computation and calibration
- **evo** [29] — trajectory evaluation (ATE, RPE) with Umeyama alignment
- **Gazebo Fortress** — simulation environment (standard and metrological plugins)

---

## 6. Phase I Contributions

Upon completion, Phase I will produce the following verifiable outputs, listed in order of methodological significance:

### 6.1 Statistical Equivalence Validation Framework (TOST)

The primary methodological output: a complete TOST-based pipeline for declaring metrological simulation statistically equivalent to real hardware within pre-specified, system-derived margins. Includes:

- Pre-specified equivalence margins δ per sensor modality and trajectory type, derived from system design requirements before data collection.
- 90% confidence intervals for mean ATE/RPE difference (Metrological sim − Real) per backend and trajectory type.
- Holm-Bonferroni correction for primary endpoints; exploratory endpoints reported without correction.
- Explicit statement of test, normality assessment, effect size, and result interpretation above the ground truth uncertainty floor.

This framework is the first application of formal equivalence testing to sim-to-real sensor model validation in robotics.

### 6.2 Open-Source Gazebo Fortress Plugin for Livox Mid-360

A Gazebo Fortress (gz-sim) plugin implementing the non-repetitive Rosetta scan pattern of the Livox Mid-360, compatible with ROS 2 Humble. Features:

- Faithful non-repetitive scan pattern topology (no existing gz-sim solution; all prior plugins target Gazebo Classic, EOL January 2025).
- Empirically-derived noise injection from M4 coefficients fitted against YuMi ground truth.
- Released as standalone open-source contribution on GitHub prior to paper submission.

### 6.3 M4 Parametric Noise Model and Coefficients

An explicit, physics-motivated parametric covariance model with identifiable coefficients, fitted from dynamic residuals against YuMi ground truth:

$$\sigma^2(t) = f\bigl(T(t),\, \|v(t)\|,\, \|\omega(t)\|,\, \|a(t)\|,\, \|\dot{\omega}(t)\|,\, \|\dot{a}(t)\|\bigr)$$

- Thermal component $\sigma^2_{\mathrm{static}}(T)$ characterising noise variance evolution with measured temperature from cold start to steady state.
- Kinematic coefficients fitted on T1+T2, evaluated on held-out T3 to verify generalization.
- Sensor-specific coefficient pruning ($c_v = 0$ for IMU; full vector for LiDAR) with physical justification.
- Reported with confidence intervals; candidates that fail to generalize explicitly discarded.

### 6.4 Allan Variance Characterization Dataset (Static)

- Allan Deviation curves for all three IMUs (WT901C, BMI055, ICM40609) under identical temperature-documented conditions.
- Numerical coefficients with units and confidence intervals: ARW [°/√h], Bias Instability [°/h], RRW [°/h$^{3/2}$] where identifiable.
- Six-Position Test (IEEE Std 1293) per IMU: scale factor and gravitational bias per axis.
- LiDAR planar orthogonal residual characterization (Mid-360 and RPLiDAR A2M12) over temperature.
- Camera thermal intrinsic drift quantification (RealSense D455, AprilTag static protocol).

### 6.5 Dynamic Validation Dataset

- Three trajectory types × three repetition blocks × four sensor sessions = structured dataset with full kinematic reference from YuMi RobotStudio export.
- First empirical characterization of Livox Mid-360 range noise statistics under mechanical vibration and kinematic stress — gap confirmed absent in the literature.
- Cooling periods with documented thermal recovery curves; settling time verification in T3.
- Dataset published alongside plugin and analysis code.

### 6.6 Multi-Backend SLAM Evaluation Matrix

- Three-condition dataset: real hardware / standard Gazebo simulation (M1) / metrological Gazebo simulation (M4).
- ATE and RPE computed with `evo` [29] (Umeyama alignment) for all conditions, trajectory types, and backends.
- CW/CCW asymmetry analysis per sensor (secondary H2).
- SLAM backends: FAST-LIO2, Point-LIO, GLIM, Cartographer 2D, KISS-ICP, ORB-SLAM3, OpenVINS; optional RTAB-Map loose-coupled baseline.

---

## 7. Planned SLAM Backends

Phase I is not a benchmark of SLAM algorithms per se, but SLAM backends are required as "measuring instruments" to evaluate sensor model fidelity. To avoid conclusions that depend on a single estimator, multiple backends are used per modality. Using multiple backends also strengthens the statistical argument: if metrological simulation is declared equivalent to real hardware across FAST-LIO2, GLIM, and ORB-SLAM3 simultaneously, the conclusion is not an artefact of a single estimator's tolerance to noise model mismatch.

- Cross-modal baseline: GLIM [23] (factor-graph, GPU-accelerated, tight-coupled LiDAR+IMU and visual-inertial).
- LiDAR 3D (Mid-360): FAST-LIO2 [22] (IESKF, LiDAR+IMU), Point-LIO [27] (dense point-based), GLIM [23].
- LiDAR 2D (RPLiDAR): Cartographer 2D [25] (graph-based SLAM), KISS-ICP 2D [26] (odometry), GLIM [23] in planar 3D mode where feasible.
- Visual / Visual-inertial (RealSense D455): ORB-SLAM3 [24] (feature-based VIO), GLIM [23] (visual-inertial), OpenVINS [28] (EKF-based VIO). R3LIVE is excluded (camera+LiDAR fusion; Session D is RGB-D+IMU only).
- **Loose-coupled baseline (optional):** RTAB-Map [30] (2D, 3D LiDAR, RGB-D) in a reduced subset of experiments to contrast tight vs. loose coupling.

Noise injected in simulation follows four configurations (see `SLAM_BACKENDS.md`): M1 (manufacturer), M2 (static Allan), M3 (in-session static Allan), M4 (kinematic-residual). For each SLAM backend, a nominal configuration is tuned once on real YuMi data and then frozen; low/high sensitivity variants (scaled covariances) are used for robustness checks.

The goal is to show that the conclusions about sensor model fidelity — particularly for the kinematic-residual model — are consistent across different SLAM architectures (Kalman filter vs. factor graph vs. pose graph), different sensing modalities (IMU-only, 2D LiDAR, 3D LiDAR, RGB-D), and reasonable variations in SLAM parameter tuning.

---

## 8. Possible Future Work (outside this project)

The experimental design and software architecture in this plan were originally inspired by a **broader three-phase research line** (metrological validation, mobile-platform Reality Gap benchmark, and large-scale SLAM optimisation). In the context of this degree project, however, only the metrological validation with YuMi ground truth is in scope. Any mobile-platform benchmark or automatic optimisation stage should be understood strictly as potential future work that could build on the outputs of this study — not as a commitment that will be executed within this project.

A natural extension of the metrological framework developed here is its application to **six-axis force/torque (F/T) sensors** for contact-rich manipulation. The same YuMi platform, the same statistical methodology (Allan Variance + state-dependent covariance + TOST), and the same ground truth infrastructure could be applied to characterize F/T sensor noise under dynamic loading — a gap confirmed in the literature and of increasing relevance given the emergence of force-aware foundation models (ForceVLA, FD-VLA). This is noted here as a future direction, not a commitment of the present project.

---

## 9. References

[1] F. Furrer, M. Burri, M. Achtelik, and R. Siegwart, "RotorS — A Modular Extensive Multirotor Micro Aerial Vehicle Simulation Platform," in *Robot Operating System (ROS): The Complete Reference (Volume 1)*, Springer, 2016.

[2] J. Liu and F. Zhang, "Livox-Mapping: A LiODOM for Livox LiDARs," arXiv:2104.10831, 2021.

[3] M. Filipenko and I. Afanasyev, "Comparison of Various SLAM Systems for Mobile Robot in an Indoor Environment," in *2018 International Conference on Intelligent Systems (IS)*, IEEE, 2018.

[4] A. Handa, T. Whelan, J. McDonald, and A. J. Davison, "A Benchmark for RGB-D Visual Odometry, 3D Reconstruction and SLAM," in *ICRA 2014*, IEEE, 2014.

[5] A. Kadian et al., "Sim2Real Predictivity: Does Evaluation in Simulation Predict Real-World Performance?", *IEEE Robotics and Automation Letters*, vol. 5, no. 4, 2020.

[6] W. Wang et al., "TartanAir: A Dataset to Push SLAM to New Limits," in *IROS 2020*, IEEE/RSJ, 2020.

[7] J. Lin et al., "An Accurate 6-DOF Dynamic Measurement System with Laser Tracker for Large-Scale Metrology," *Measurement*, vol. 204, 2022. [DOI: 10.1016/j.measurement.2022.112052](https://www.sciencedirect.com/science/article/pii/S0263224122012489)

[8] R. Mur-Artal and J. D. Tardós, "ORB-SLAM2: An Open-Source SLAM System for Monocular, Stereo, and RGB-D Cameras," *IEEE Transactions on Robotics*, vol. 33, no. 5, 2017.

[9] N. Abella, "Comprehensive Validation and AI-Driven Optimization of Visual-LiDAR SLAM for Mobile Robotics — A Three-Stage Research Framework," IQS School of Engineering, February 2026. *(Original three-phase proposal)*

[10] J. Kuti, T. Piricz, and P. Galambos, "A Robust Method for Validating Orientation Sensors Using a Robot Arm as a High-Precision Reference," *Sensors*, vol. 24, no. 24, 8179, 2024. [https://doi.org/10.3390/s24248179](https://www.mdpi.com/1424-8220/24/24/8179)

[11] M. Joerger et al., "Development of Stochastic IMU Error Models for INS/GNSS Integration," Virginia Tech, 2021. [PDF](https://www.aoe.vt.edu/content/dam/aoe_vt_edu/people/faculty/joerger/publications/Development%20of%20Stochastic%20IMU%20Error%20Models%20for%20INS-GNSS%20Integration_2021.pdf)

[12] M. G. Petovello et al., "Real-Time Estimation of Temperature Time Derivative in Inertial Measurement Unit by Finite-Impulse-Response Exponential Regression on Updates," *Sensors*, vol. 20, no. 5, 1299, 2020. [https://doi.org/10.3390/s20051299](https://www.mdpi.com/1424-8220/20/5/1299)

[13] D. Cheng, J. Kang, and X. Xiong, "Simultaneous Calibration of Noise Covariance and Kinematics for State Estimation of Legged Robots via Bi-level Optimization," arXiv:2510.11539, 2025. [https://arxiv.org/abs/2510.11539](https://arxiv.org/html/2510.11539v1)

[14] Six-Position Test for IMU calibration — see IEEE calibration standards (e.g. IEEE Std 1293) or Joerger et al. [11] for MEMS procedures.

[15] S. Umeyama, "Least-squares estimation of transformation parameters between two point patterns," *IEEE Transactions on Pattern Analysis and Machine Intelligence*, vol. 13, no. 4, pp. 376–380, 1991.

[16] ABB, "IRB 14000 YuMi — Product specification," ABB Robotics. [https://new.abb.com/products/robotics/industrial-robots/irb-14000-yumi](https://new.abb.com/products/robotics/industrial-robots/irb-14000-yumi) *(Repeatability ±0.02 mm per datasheet.)*

[17] H. Hou, "Modeling inertial sensors errors using Allan variance," *Proc. IEEE/ION PLANS*, 2004. [Semantic Scholar](https://www.semanticscholar.org/paper/Modeling-inertial-sensors-errors-using-Allan-Hou/6a1b4db58f429843a36d2f9f815d753a265c3dda)

[18] A. Carlson, K. A. Skinner, R. Vasudevan, and M. Johnson-Roberson, "Sensor Transfer: Learning Optimal Sensor Effect Image Augmentation for Sim-to-Real Domain Adaptation," arXiv:1809.06256, 2018. [https://arxiv.org/abs/1809.06256](https://arxiv.org/abs/1809.06256)

[19] National Institute of Standards and Technology (NIST), "3D Ground-Truth Systems for Object/Human Recognition and Tracking," NIST Report, 2019 (approx.). [PDF](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=913818)

[20] Intel Corporation, "IMU Calibration Tool for Intel RealSense Depth Cameras (D435i, D455, L515)," Whitepaper rev. 1.4, July 2020. [PDF](https://www.realsenseai.com/wp-content/uploads/2020/07/IMU_Calibration_Tool_for_Intel_RealSense-Depth_Cameras_Whitepaper.pdf)

[21] "Procedure and Accuracy Assessment Results of Livox MID-360 Scanners," *ISPRS Arch. Photogramm. Remote Sens. Spatial Inf. Sci.*, Vol. XLVIII-1/W6-2025, 2025. [https://doi.org/10.5194/isprs-archives-XLVIII-1-W6-2025-147-2025](https://isprs-archives.copernicus.org/articles/XLVIII-1-W6-2025/147/2025/)

[22] W. Xu, Y. Cai, D. He, J. Lin, and F. Zhang, "FAST-LIO2: Fast Direct LiDAR-Inertial Odometry," *IEEE Transactions on Robotics*, vol. 38, no. 4, pp. 2053–2073, 2022. [DOI: 10.1109/TRO.2022.3141876](https://ieeexplore.ieee.org/document/9697912)

[23] K. Koide, M. Yokozuka, S. Oishi, and A. Banno, "Globally Consistent and Tightly Coupled 3D LiDAR Inertial Mapping," in *Proc. IEEE International Conference on Robotics and Automation (ICRA)*, Philadelphia, 2022, pp. 5622–5628. [arXiv: 2202.00242](https://arxiv.org/abs/2202.00242)

[24] C. Campos, R. Elvira, J. J. Gómez Rodríguez, J. M. M. Montiel, and J. D. Tardós, "ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM," *IEEE Transactions on Robotics*, vol. 37, no. 6, pp. 1874–1890, 2021. [DOI: 10.1109/TRO.2021.3075644](https://ieeexplore.ieee.org/document/9440682)

[25] W. Hess, D. Kohler, H. Rapp, and D. Andor, "Real-Time Loop Closure in 2D LIDAR SLAM," in *Proc. IEEE International Conference on Robotics and Automation (ICRA)*, Stockholm, 2016, pp. 1271–1278. [DOI: 10.1109/ICRA.2016.7487258](https://ieeexplore.ieee.org/document/7487258)

[26] I. Vizzo, T. Guadagnino, B. Mersch, L. Wiesmann, J. Behley, and C. Stachniss, "KISS-ICP: In Defense of Point-to-Point ICP – Simple, Accurate, and Robust Registration If Done the Right Way," *IEEE Robotics and Automation Letters*, vol. 8, no. 2, pp. 1029–1036, 2023. [arXiv: 2209.15397](https://arxiv.org/abs/2209.15397)

[27] D. He, W. Xu, N. Chen, F. Kong, C. Yuan, and F. Zhang, "Point-LIO: Robust High-Bandwidth LiDAR-Inertial Odometry," *Advanced Intelligent Systems*, vol. 5, no. 7, 2023. [DOI: 10.1002/aisy.202200459](https://doi.org/10.1002/aisy.202200459)

[28] P. Geneva, K. Eckenhoff, W. Lee, Y. Yang, and G. Huang, "OpenVINS: A Research Platform for Visual-Inertial Estimation," in *Proc. IEEE International Conference on Robotics and Automation (ICRA)*, Paris, 2020. [https://github.com/rpng/open_vins](https://github.com/rpng/open_vins)

[29] M. Grupp, "evo: Python package for the evaluation of odometry and SLAM," 2017. [https://github.com/MichaelGrupp/evo](https://github.com/MichaelGrupp/evo)

[30] M. Labbé and F. Michaud, "RTAB-Map as an Open-Source Lidar and Visual SLAM Library for Large-Scale and Long-Term Online Operation," *Journal of Field Robotics*, vol. 36, no. 3, pp. 416–446, 2018. [DOI: 10.1002/rob.21831](https://doi.org/10.1002/rob.21831)

[31] D. J. Schuirmann, "A comparison of the Two One-Sided Tests Procedure and the Power Approach for assessing the equivalence of average bioavailability," *Journal of Pharmacokinetics and Biopharmaceutics*, vol. 15, no. 6, pp. 657–680, 1987. [DOI: 10.1007/BF01068419](https://doi.org/10.1007/BF01068419)

[32] D. Lakens, "Equivalence Tests: A Practical Primer for t Tests, Correlations, and Meta-Analyses," *Social Psychological and Personality Science*, vol. 8, no. 4, pp. 355–362, 2017. [DOI: 10.1177/1948550617697177](https://doi.org/10.1177/1948550617697177)

[33] M. Burri et al., "The EuRoC micro aerial vehicle datasets," *The International Journal of Robotics Research*, vol. 35, no. 10, pp. 1157–1163, 2016.

[34] D. Schubert et al., "The TUM VI Benchmark for Evaluating Visual-Inertial Odometry," in *Proc. IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)*, 2018.

[35] M. Helmberger et al., "The Hilti SLAM Challenge Dataset," *IEEE Robotics and Automation Letters*, vol. 7, no. 3, pp. 7518–7525, 2022.

[36] M. Ramezani et al., "The Newer College Dataset for Visual-LiDAR Egocentric Mapping," in *Proc. IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)*, 2020.

[37] P. Furgale, J. Rehder, and R. Siegwart, "Unified Temporal and Spatial Calibration for Multi-Sensor Systems," in *Proc. IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)*, 2013.

[38] J. Rehder, J. Nikolic, T. Schneider, T. Hinzmann, and R. Siegwart, "Extending kalibr: Calibrating the extrinsics of multiple IMUs and of individual axes," in *Proc. IEEE International Conference on Robotics and Automation (ICRA)*, 2016.

[39] A. Loquercio, E. Kaufmann, R. Ranftl, A. Dosovitskiy, V. Koltun, and D. Scaramuzza, "Deep Drone Racing: From Simulation to Reality with Domain Randomization," *IEEE Transactions on Robotics*, vol. 36, no. 1, pp. 1–14, 2020. [Preprint](https://rpg.ifi.uzh.ch/docs/TRO19_Loquercio.pdf)

[40] A. Solodar and I. Klein, "VIO-DualProNet: Visual-inertial odometry with learning based process noise covariance," *Engineering Applications of Artificial Intelligence*, 2024. [DOI: 10.1016/j.engappai.2024.006249](https://www.sciencedirect.com/science/article/abs/pii/S0952197624006249)

[41] S. Qiu et al., "AirIMU: Learning Uncertainty Propagation for Inertial Odometry," arXiv:2310.04874, 2023. [https://arxiv.org/abs/2310.04874](https://arxiv.org/abs/2310.04874)

[42] C. Ferguson, T. Ertop, M. Herrell, and R. J. Webster III, "Unified Robot and Inertial Sensor Self-Calibration," *Robotica*, vol. 41, no. 5, pp. 1590–1616, 2023. [DOI: 10.1017/S0263574722001679](https://pmc.ncbi.nlm.nih.gov/articles/PMC10508886/)

[43] M. Jongeneel et al., "Evaluating the Sim-to-Real Gap for Contact-Rich Robotic Manipulation," submitted to *IEEE Robotics and Automation Letters*, 2024. [HAL: hal-04673156](https://hal.science/hal-04673156v1/file/RAL_2024_Evaluating_the_Sim_to_Real_Gab_for_Contact_Rich_Robotic_Manipulation.pdf)

[44] B. Vultaggio, F. Giacomini, and A. Vitti, "Simulation of Low-Cost MEMS-LiDAR and Analysis of Its Effect on the Performances of State-of-the-Art SLAMs," *ISPRS Archives*, vol. XLVIII-1/W1-2023, pp. 539–546, 2023. [DOI: 10.5194/isprs-archives-XLVIII-1-W1-2023-539-2023](https://isprs-archives.copernicus.org/articles/XLVIII-1-W1-2023/539/2023/)

[45] B. Schlager et al., "Experimental Evaluation of Vibration Influence on a Resonant MEMS Scanning System for Automotive Lidars," *IEEE Open Journal of Intelligent Transportation Systems*, vol. 2, pp. 87–99, 2021. [DOI: 10.1109/OJITS.2021.3053853](https://ieeexplore.ieee.org/document/9380978/)

[46] R. Brazeal, B. Wilkinson, and R. Hochmair, "A Rigorous Observation Model for the Risley Prism-Based Livox Mid-40 Lidar Sensor," *Sensors*, vol. 21, no. 14, 4722, 2021. [DOI: 10.3390/s21144722](https://www.mdpi.com/1424-8220/21/14/4722)

[47] gazebosim/gz-sim, "Custom lidar sensor — Feature Request," GitHub issue #1958, opened April 2023. [https://github.com/gazebosim/gz-sim/issues/1958](https://github.com/gazebosim/gz-sim/issues/1958)

[48] S. Wellek, *Testing Statistical Hypotheses of Equivalence and Noninferiority*, 2nd ed., Chapman & Hall/CRC, 2010.
