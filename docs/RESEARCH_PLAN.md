# Research Plan

* **Author:** Narcís Abella
* **Supervisor:** Dr. Antonio Gabino Salazar Martín
* **Institution:** IQS School of Engineering (Universitat Ramon Llull), Barcelona
* **Version:** 0.6 (pending supervisor review)
* **Date:** 2026-03-28

---

## Table of Contents

1. [Motivation and Scientific Rationale](#1-motivation-and-scientific-rationale)
2. [Scope Definition](#2-scope-definition)
3. [Related Work](#3-related-work)
4. [Research Hypothesis](#4-research-hypothesis)
5. [Experimental Platform](#5-experimental-platform)
6. [Expected Contributions and Deliverables](#6-expected-contributions-and-deliverables)
7. [Possible Future Work](#7-possible-future-work-outside-this-project)
8. [References](#8-references)

---

## 1. Motivation and Scientific Rationale

Standard robotic simulations typically rely on generic Gaussian noise models and idealized scanning geometries. Several sensor-specific effects are not captured by these models.

In MEMS IMUs, **Angle Random Walk (ARW)** and **Bias Instability** determine how fast orientation error accumulates during motion. **Rate Random Walk (RRW)** is a slower noise process that only becomes visible in logs longer than a few hours; it sets the long-term drift floor in any tight-coupled system. For solid-state LiDARs, the non-repetitive Rosetta scanning pattern of the Livox Mid-360 creates short-term point cloud gaps that increase geometric degeneracy risk in low-variation environments; traditional planar LiDAR models miss this entirely. Finally, thermal drift in IMU biases and camera intrinsics is not static: it evolves over the operational cycle, and a simulator that ignores it will consistently underestimate localization error in real deployments.

Furrer et al. (2016) support simulation pipelines that rely on empirically tuned inertial noise parameters instead of pure datasheet defaults [[10]](REFERENCES_CONSOLIDATED.md). Liu & Zhang (2021) characterize geometric degeneracy risks in Livox-type sensing [[24]](REFERENCES_CONSOLIDATED.md). Filipenko & Afanasyev (2018) provide early empirical evidence that simulation performance can degrade on real hardware [[63]](REFERENCES_CONSOLIDATED.md).

- The field's most comprehensive recent survey on the sim-to-real gap (Aljalbout et al., 2026) identifies unvalidated sensor models as a core unresolved source (distinct from dynamics parameterisation, collision sensing, and actuator models). Domain randomisation and high-fidelity physics do not resolve this if the sensor statistics are structurally wrong.
- Muratore et al. (2022) formalise the *Simulation Optimization Bias*: a policy trained on a slightly faulty simulator maximises reward by exploiting model errors, not by solving the task. An autonomy stack tuned in simulation with an unvalidated sensor model learns to compensate for artefacts that don't exist on real hardware.

What remains poorly characterized is the sensor signal level itself: no prior work formally tests whether a simulation plugin produces sensor output statistically equivalent to real hardware within pre-specified, system-derived margins. Most studies compare simulation and hardware at the trajectory level, after an estimator has partially absorbed and corrected sensor noise differences, or rely on informal RMSE without pre-specified equivalence bounds. None apply formal equivalence testing directly to sensor residuals (point-to-plane residuals for LiDAR, angular rate and acceleration residuals for IMU, AprilTag pose drift for camera) under a controlled kinematic reference. This study closes that gap: static Allan Variance characterization, dynamic residual extraction against a sub-millimetre manipulator reference, and TOST equivalence testing applied directly at the sensor signal level before any estimator stage.

---

## 2. Scope Definition

Based on supervisor feedback (A.G. Salazar, February 2026), this work is deliberately scoped to metrological sensor characterization and TOST-based equivalence validation only.

Three reasons support this scope.

A complete metrological validation of sensor models, with rigorous statistical equivalence testing against a sub-millimeter ground truth, is a contribution that stands on its own. Bundling it with a sim-to-real gap benchmark or a parameter optimization study would dilute the metrological result rather than strengthen it. **The YuMi arm** is the right tool for this specific question: controlled trajectories, known kinematics, and sub-millimeter path repeatability (0.10 mm per ISO 9283) are not replicable on a mobile base, and that clean experimental setting is what makes the sensor-level comparison credible. Mobile-platform benchmarking and large-scale SLAM optimization each introduce additional variables (platform dynamics, uncontrolled environments, thousands of simulation runs) that deserve their own experimental design; they are future work, not omissions from this study.

To be explicit about scope: this study characterizes each sensor statically (noise coefficients via Allan Variance), validates the models dynamically (against YuMi ground truth trajectories), and tests whether a metrological Gazebo plugin using those coefficients produces sensor output statistically equivalent to real hardware at the signal level, as formally demonstrated by TOST with pre-specified margins.

**Scope of variables:**

| Variable | IMU | LiDAR 2D | LiDAR 3D | RGB-D |
| -------- | --- | -------- | -------- | ----- |
| Thermal drift | ✅ | ✅ | ✅ | ✅ |
| Kinematic state-dependent noise (M4) | ✅ | ✅ | ✅ | ✅ |
| Range-dependent noise | - | ✅ | ✅ | ✅ |
| Incidence angle | - | ✅ | ❌ declared | - |
| Scan pattern geometry | - | - | ✅ | - |
| Cross-axis sensitivity | ✅ | - | - | - |
| IR fixed-pattern noise | - | - | - | ✅ |
| Motion blur | - | - | - | ✅ |
| Quantization | ✅ | ✅ | ✅ | ✅ |
| Vibration (external) | ❌ FW | ❌ FW | ❌ FW | ❌ FW |
| Material reflectivity | ❌ FW | ❌ FW | ❌ FW | ❌ FW |
| Ambient light | ❌ FW | ❌ FW | ❌ FW | ❌ FW |
| Magnetic interference | ❌ FW | - | - | - |
| Power supply noise | ❌ FW | ❌ FW | ❌ FW | ❌ FW |
| Aging / long-term drift | ❌ FW | ❌ FW | ❌ FW | ❌ FW |
| Rolling shutter | - | - | - | ❌ FW |
| Multi-return / crosstalk | - | ❌ FW | ❌ FW | - |

✅ In scope | ❌ FW Out of scope, future work | ❌ declared Out of scope, contributor declared in uncertainty budget | - Not applicable

This study characterizes sensor noise variables that are intrinsic to the sensor's physical measurement principle and controllable under 6DOF kinematic ground truth in a laboratory setting. External environmental variables - including material reflectivity, ambient light interference, magnetic fields, power supply noise, and mechanical vibration from operational platforms - are explicitly out of scope and identified as structured directions for future work. The kinematic state-dependent noise model (M4) characterizes the effect of controlled 6DOF motion on sensor output and represents a lower bound on dynamic operational noise; real deployment platforms will exceed this floor due to mechanical vibration sources not modeled here. This limitation is declared explicitly in the uncertainty budget.

---

## 3. Related Work

### 3.1 IMU Noise Characterization

The inertial metrology literature commonly uses Allan Variance (or Allan Deviation, ADEV) as a primary tool for identifying stochastic IMU noise processes [[9]](REFERENCES_CONSOLIDATED.md). The three main parameters are:

- ARW (Angle Random Walk): white noise floor of the gyroscope; identified by the slope of $-1/2$ on the log-log ADEV curve.
- Bias Instability: minimum of the ADEV curve; the best achievable gyroscope bias stability.
- RRW (Rate Random Walk): low-frequency drift; identified by slope $+1/2$. Not always identifiable from short logs; 10–12 h static recordings are used here as a conservative protocol for robust estimation.

Furrer et al. (2016) provide strong simulation-context precedent for empirically calibrated inertial parameters in MAV pipelines, compared with generic defaults [[10]](REFERENCES_CONSOLIDATED.md).

The **Six-Position Test (IEEE Std 1293)** [[83]](REFERENCES_CONSOLIDATED.md) complements Allan Variance by isolating scale factor errors and gravitational bias (parameters not observable from static horizontal logs but critical for tight-coupled LiDAR-inertial estimators).

### 3.2 LiDAR Characterization

Liu & Zhang (2021) highlighted the geometric degeneracy risks of solid-state LiDARs in environments with low structural variation. The Livox Mid-360's non-repetitive Rosetta scanning pattern provides full spherical coverage over time but creates short-term point cloud gaps that affect frame-to-frame ICP registration quality. Characterizing this temporal integration behavior (validating that the simulated Rosetta pattern matches the physical point cloud density over time) is a core contribution of this work.

For static LiDAR characterization, we propose a **planar orthogonal residual method** (instead of absolute distance measurement): the sensor is fixed facing a flat wall, and temporal variation is evaluated by computing the orthogonal error of successive point clouds against a reference plane fitted at t=0. This isolates thermal ToF drift without introducing ICP registration errors.

Regarding vibration-induced effects, Schlager et al. [[26]](REFERENCES_CONSOLIDATED.md) and Brazeal et al. [[27]](REFERENCES_CONSOLIDATED.md) provide relevant but indirect precedents (different setups and/or simulation-focused analyses). No study has empirically characterized whether Livox Mid-360 range noise statistics change under controlled kinematic conditions — specifically, whether velocity, acceleration, and jerk at the sensor level produce measurable changes in point-to-plane residuals.

### 3.3 Camera Characterization

Handa et al. (2014) demonstrated that even with perfect ground truth, SLAM algorithms fail in synthetic environments due to incorrect noise modeling and lack of photorealistic texture. For the RealSense D455, the primary concerns are:

- **Fixed-Pattern Noise (FPN)** in the infrared projector, stable over short periods but temperature-dependent.
- **Thermal deformation of intrinsic parameters** (focal length and principal point drift as the projector warms up).

Monitoring 6-DOF pose estimation of a static AprilTag pattern over several hours directly quantifies these effects. If the simulator assumes a perfect camera and the real camera exhibits thermal intrinsic drift, we have a documented, quantified source of sim-to-real gap and direct evidence for why sensor-level equivalence testing is needed.

### 3.4 Sim-to-Real Validation Framework

Kadian et al. (2019) introduced "Sim2Real Predictivity" as a formal metric, investigating whether simulation performance correlates with real-world performance. Wang et al. (2020) used the TartanAir dataset to argue that the sim-to-real gap remains significant for visual odometry algorithms dependent on fine-grained texture features. In parallel, a large body of work on **domain randomization** for perception and control shows that aggressively varying textures, lighting, object materials and camera parameters in simulation can improve sim-to-real robustness, for instance in sensor-effect image adaptation [[60]](REFERENCES_CONSOLIDATED.md) and agile visual navigation [[65]](REFERENCES_CONSOLIDATED.md).

In this work we deliberately do not adopt domain randomisation. The problem with randomisation-based approaches is not their breadth but their unverifiability: without a formal test of sensor model equivalence, there is no way to know whether the real world falls inside the training distribution or whether the policy has instead learned to exploit simulation artefacts (Muratore et al., 2022, *Simulation Optimization Bias*). We instead control the environment so that real and simulated scenes share simple, well-characterised patterns, and focus exclusively on sensor model fidelity at the signal level.

A structural limitation of much sim-to-real validation work (including the studies above) is methodological: comparisons often use RMSE or significance tests designed to detect differences, not to demonstrate sufficiency. VIO-DualProNet [[18]](REFERENCES_CONSOLIDATED.md) and AirIMU [[19]](REFERENCES_CONSOLIDATED.md) show that state-dependent covariance can improve VIO performance, but they do not frame simulation-vs-hardware validation as a pre-specified equivalence test. A contact-rich manipulation record on HAL [[57]](REFERENCES_CONSOLIDATED.md) also uses descriptive trajectory-level metrics. This study adopts a different validation logic: formal Two One-Sided Tests (TOST) with a pre-specified equivalence margin δ, borrowed from bioequivalence testing [[1]](REFERENCES_CONSOLIDATED.md) and, to the best of our knowledge, applied here at sensor-residual level in robotics.

### 3.5 Ground Truth Quality: Prior Benchmarks and Their Limitations

Standard SLAM benchmarks provide high-quality reference trajectories. For instance, the EuRoC MAV dataset [[77]](REFERENCES_CONSOLIDATED.md) and the TUM-VI benchmark [[78]](REFERENCES_CONSOLIDATED.md) use high-quality external references, while the Hilti Challenge [[79]](REFERENCES_CONSOLIDATED.md) and the Newer College Dataset [[80]](REFERENCES_CONSOLIDATED.md) rely on carefully engineered survey/mapping-grade references. These datasets are methodologically valuable context, but not direct proofs of this project's exact AVAR protocol choices.

However, these datasets are recorded on mobile platforms or hand-held rigs. In such setups, the true kinematic state (velocity, acceleration, jerk) must be derived by numerically differentiating the ground truth pose, which amplifies noise. In contrast, an industrial manipulator like the YuMi provides a direct, analytically differentiable kinematic reference programmed in advance. This isolates the sensor noise from ground-truth differentiation artifacts.

Regarding dynamic ground truth for metrology, Lin et al. (2022, *Measurement*) [[42]](REFERENCES_CONSOLIDATED.md) validated a 6-DOF dynamic measurement system using a laser-tracker reference. Kuti et al. (2024) [[40]](REFERENCES_CONSOLIDATED.md) used a robot arm as a high-precision reference to validate IMU orientation under dynamic trajectories. Ferguson et al. (2023) [[41]](REFERENCES_CONSOLIDATED.md) addressed robot/IMU self-calibration. Together they support the general feasibility of manipulator-assisted inertial validation, while this study targets stochastic noise variance characterization under controlled kinematics.

### 3.6 Justification and Differentiation of the M4 Covariance Model

The concept of state-dependent noise covariance for inertial sensors is not new. Learning-based approaches already demonstrate its value: VIO-DualProNet [[18]](REFERENCES_CONSOLIDATED.md) and AirIMU [[19]](REFERENCES_CONSOLIDATED.md) estimate covariance-related uncertainty dynamically. The physical motivation - that kinematic/thermal state can affect sensor noise - is established in prior literature, although implementation details and assumptions differ by method.

The contribution of M4 is therefore not conceptual novelty but **metrological rigour and physical interpretability**. M4 differs from prior work in three specific ways.

First, M4 is **explicit and parametric** rather than learned from a black box. Each coefficient has a physical interpretation ($c_\omega$ captures g-sensitivity, $c_j$ captures vibration-induced noise) and can be independently validated against physical models. This makes the model auditable, which is a requirement for publication in metrological journals (Measurement, IEEE TIM).

Second, M4 combines **measured temperature $T(t)$ and the full kinematic state vector simultaneously**. Existing thermal calibration literature [[12]](REFERENCES_CONSOLIDATED.md), [[13]](REFERENCES_CONSOLIDATED.md) and dynamic covariance literature [[18]](REFERENCES_CONSOLIDATED.md), [[19]](REFERENCES_CONSOLIDATED.md) motivate this direction, but do not provide this exact parametric combination for our use case.

Third, M4 coefficients are fitted from **dynamic residuals against a sub-millimetre industrial manipulator**, not from flight data or uncontrolled platform trajectories. This provides the ground truth quality necessary for the TOST to have statistical power within margins that are meaningful for system design. Without a model of this fidelity, the equivalence test cannot distinguish a good simulator from a bad one within the chosen δ.

M4 is therefore best understood as a **necessary enabler of the validation framework** rather than as the primary contribution. The primary contribution is the TOST-based equivalence testing pipeline; M4 is what gives that pipeline sufficient resolution to be informative.

This modeling effort **differs from** continuous-time calibration frameworks such as Kalibr. Furgale et al. [[49]](REFERENCES_CONSOLIDATED.md) and Rehder et al. [[50]](REFERENCES_CONSOLIDATED.md) established a robust methodology to optimize spatial and temporal extrinsics alongside IMU biases using a continuous-time spline representation. While they model biases using random walks and optimize parameters against visual ground truth, they assume a constant measurement noise covariance. M4 goes a step further: it captures how the *variance* of the noise itself scales with the kinematic state, rather than just estimating the bias states or fixed extrinsics.

The sensor-specific coefficient pruning ($c_v = 0$ for IMU; full vector for LiDAR) and the held-out generalization check (fit on T1+T2, evaluate on T3) are described in [`docs/METHODOLOGY.md`](METHODOLOGY.md) §5.4.1 and §5.6.

### 3.7 Summary and Novelty Statement

The following claims are made to the best of our knowledge, based on a systematic review of the literature conducted in March 2026.

The novelty of this work is primarily **methodological**, not conceptual. Each individual component has precedent:

- State-dependent IMU covariance: VIO-DualProNet [[18]](REFERENCES_CONSOLIDATED.md), AirIMU [[19]](REFERENCES_CONSOLIDATED.md) (learned); Kalibr-style calibration context.
- Robot/arm-assisted inertial validation and calibration context: Kuti et al. (2024) [[40]](REFERENCES_CONSOLIDATED.md); Ferguson et al. (2023) [[41]](REFERENCES_CONSOLIDATED.md).
- Livox Mid-360 scan-pattern simulation context: Vultaggio et al. (ISPRS 2023) [[25]](REFERENCES_CONSOLIDATED.md), plus existing ecosystem plugins and issue tracking [[81]](REFERENCES_CONSOLIDATED.md), [[82]](REFERENCES_CONSOLIDATED.md).
- Allan Variance for simulation parameterization context: El-Sheimy et al. (2008) [[9]](REFERENCES_CONSOLIDATED.md), with simulation-pipeline precedent in RotorS [[10]](REFERENCES_CONSOLIDATED.md).

What has not been done, and what this study contributes:

**Primary contribution:** To the best of our knowledge, an early application of formal statistical equivalence testing (TOST) with pre-specified, system-derived margins to sim-to-real sensor noise model validation in robotics, applied directly at the sensor signal level (residuals). The claim is not "simulation matches hardware" but "simulation is equivalent to hardware within bounds that matter for design decisions," a materially different statement from descriptive trajectory-only validation.

**Secondary contribution:** An explicit, physics-motivated parametric noise model (M4) that combines measured temperature and full kinematic state vector with interpretable coefficients, fitted from dynamic residuals against a sub-millimetre robot arm reference. Unlike learning-based state-dependent covariance methods, M4 is auditable, physically justified per coefficient, and directly injectable into Gazebo simulation plugins.

**Software contribution:** A CPU-based Gazebo Fortress plugin for the Livox Mid-360 non-repetitive (Rosetta) scan pattern with empirically-derived noise model, fully compatible with ROS 2 Humble. While a third-party plugin exists for Gazebo Harmonic (RobotecAI), it typically depends on NVIDIA OptiX/CUDA and does not provide this project's CPU-first Humble/Fortress workflow. Community demand for a Fortress-oriented solution has been documented since April 2023 (gz-sim issue #1958) [[81]](REFERENCES_CONSOLIDATED.md). We position this as a practical implementation gap fill, not an exclusivity claim.

**Experimental contribution:** Empirical characterization of whether Livox Mid-360 range noise statistics change under controlled kinematic conditions (velocity, acceleration, and jerk at the sensor level), addressing a gap where available prior work is indirect or setup-mismatched (for example, different hardware or simulation-only analyses) [[26]](REFERENCES_CONSOLIDATED.md), [[27]](REFERENCES_CONSOLIDATED.md).

The combination of these four contributions - formal equivalence testing, metrologically-fitted parametric noise model, Fortress plugin, and kinematic-stress characterization - constitutes the novelty of the work as an integrated system, even where individual components have precedent. The closest prior work, Ngo et al. (2021), proposes explicit + implicit sensor evaluation for radar sim-to-real but relies on descriptive distance metrics (no pre-specified equivalence bounds) and lacks a controlled kinematic ground truth. This study provides both.

---

## 4. Research Hypothesis

Standard hypothesis testing (t-test, Wilcoxon) asks whether two distributions differ significantly. For sim-to-real validation, that is the wrong question: a non-significant result does not demonstrate that simulation is good enough. It may simply reflect insufficient statistical power. The correct question is whether the difference between simulation and real hardware is small enough to be practically irrelevant. Two One-Sided Tests (TOST) answer that question by testing whether the mean difference falls within a pre-specified equivalence margin δ. Equivalence is declared only when both one-sided tests reject (that is, when the data provide positive evidence that the difference is smaller than δ in both directions).

**Primary hypothesis (H0):**
> A simulation plugin that injects empirically derived noise coefficients (M2: Allan Variance ARW/BI/RRW; M4: state-dependent thermal and kinematic covariance) and enforces the correct Livox Rosetta scanning topology will produce sensor output statistically equivalent to real hardware within pre-specified, sensor-type-specific margins δ, as measured directly by sensor residuals against the YuMi kinematic reference.

Formal test: **TOST (Two One-Sided Tests)** for equivalence [[1]](REFERENCES_CONSOLIDATED.md)–[[3]](REFERENCES_CONSOLIDATED.md). The margin δ is pre-specified per sensor type before data collection: mm for LiDAR point-to-plane residuals; deg/s for IMU gyroscope residuals; m/s² for IMU accelerometer residuals; mm/deg for camera AprilTag pose drift. Equivalence is declared if the 90% confidence interval for the mean residual difference (Metrological sim − Real) lies entirely within $[-\delta, +\delta]$.

**Success criterion:** (i) **Equivalence:** TOST declares Metrological sim equivalent to Real within margin $\delta$. (ii) **Control:** Standard simulation differs significantly from Real (p < 0.05 via paired t-test or Wilcoxon), confirming that the metrological approach is necessary. If equivalence is not declared, this is also publishable: it indicates that even empirically derived parameters are insufficient.

**Pre-registration (Session D):** Session D (RealSense D455) is expected to show the largest sensor residual gap of all sessions, due to the D455's sensitivity to thermal intrinsic drift and illumination conditions that standard Gazebo does not capture. This outcome is expected and publishable: it quantifies a sensor-level limitation that can be addressed in follow-on studies outside the present scope.

**Secondary hypothesis (H1, exploratory):**
> While comparing three IMUs of different quality grades under identical kinematic conditions, we hypothesize that IMU quality (ARW, Bias Instability) has a systematic effect on IMU residual magnitude within the dynamic sessions. This effect is conditioned by the varying sensor configurations across sessions A–D, which prevents perfect isolation of the IMU variable. Trajectory-level consequences of IMU quality differences are out of scope for this study.

**Secondary hypothesis (H2):**
> Systematic CW/CCW asymmetry is observable in gyroscope bias (that is, bias drift differs between clockwise and counter-clockwise rotation at identical angular velocities), and this asymmetry is captured more accurately by the metrological model than by standard simulation.

---

## 5. Experimental Platform

### 5.1 Ground Truth System

**ABB YuMi IRC5** (dual-arm collaborative robot):

Key ground truth parameters: path repeatability $RT = 0.10$ mm, path accuracy $AT \leq 1.36$ mm (ISO 9283 worst-case); $RP = \pm 0.02$ mm applies to static points only. Full specification in [docs/HARDWARE_PAYLOAD.md](HARDWARE_PAYLOAD.md) §1.

> For dynamic residual comparisons, the bounding mechanical uncertainty is path repeatability (0.10 mm) and path accuracy (up to 1.36 mm). Pose repeatability ($\pm0.02$ mm) applies only to static points. This distinction must be stated explicitly in any publication.

RobotStudio provides the executed trajectory as the independent predictor of robot motion. This dual role as kinematic ground truth *and* independent simulation reference is the main methodological strength of this setup.

### 5.2 Sensor Suite

See [`docs/HARDWARE_PAYLOAD.md`](HARDWARE_PAYLOAD.md) for full specification, weight breakdown, and session viability assessment.

Sessions are ordered by modality complexity (A = IMU-only, B = 2D LiDAR, C = 3D LiDAR, D = visual-inertial):

| ID | Sensor | IMU chip | IMU rate | LiDAR type | Session |
| -- | ------ | -------- | -------- | ---------- | ------- |
| S1 | WitMotion WT901C | MPU9250 | 200 Hz | - | A |
| S2 | RPLiDAR A2M12 | - | - | Mechanical 2D, 360° | B |
| S3 | Livox Mid-360 | ICM40609 | 200 Hz | Solid-state, 360° | C |
| S4 | Intel RealSense D455 | BMI055 | 400 Hz | - | D |

IMU quality ranking (reference for H1 analysis):

1. WitMotion WT901C (MPU9250): mid-high grade; literature-characterized chip; reference IMU of the study
2. Livox Mid-360 (ICM40609): mid grade; factory-calibrated LiDAR-IMU extrinsics
3. RealSense D455 (BMI055): low-mid grade; notoriously noisy for standalone navigation; factory-calibrated camera-IMU extrinsics

### 5.3 Software Stack

- **ROS 2 Humble**: middleware and data logging
- **RobotStudio**: trajectory programming and ground truth export
- **Python / imu_utils / kalibr**: Allan Variance computation and calibration
- **Python (NumPy/SciPy/Open3D/Pandas)**: residual extraction, statistical analysis, and reporting
- **Gazebo Fortress**: simulation environment (standard and metrological plugins)

---

## 6. Expected Contributions and Deliverables

Upon completion, this study will produce the following verifiable outputs, listed in order of methodological significance:

### 6.1 Statistical Equivalence Validation Framework (TOST)

The primary methodological output: a complete TOST-based pipeline for declaring metrological simulation statistically equivalent to real hardware within pre-specified, system-derived margins, applied at the sensor signal level. Includes:

- Pre-specified equivalence margins δ per sensor type in physical signal units (mm point-to-plane for LiDAR; deg/s for IMU gyroscope; m/s² for IMU accelerometer; mm/deg for camera AprilTag pose drift), derived from system design requirements before data collection.
- 90% confidence intervals for mean residual difference (Metrological sim − Real) per session (A–D) and sensor type.
- Holm-Bonferroni correction for primary endpoints; exploratory endpoints (T1/T3 residual comparisons, CW/CCW asymmetry, M1/M3 secondary rungs) reported without correction.
- Explicit statement of test selection, normality assessment, effect size, and result interpretation above the ground truth uncertainty floor.

To the best of our knowledge, this framework is among the first formal equivalence-testing applications to sensor-noise-model validation at the signal level in robotics simulation.

### 6.2 Gazebo Fortress Reference Implementation for Livox Mid-360

A Gazebo Fortress (gz-sim) reference implementation that reproduces the non-repetitive Rosetta scan pattern of the Livox Mid-360 and supports the metrological validation workflow in ROS 2 Humble. Features:

- Faithful non-repetitive scan pattern topology for a CPU-first Humble/Fortress workflow (existing alternatives are partial, target different Gazebo releases, or rely on different compute assumptions).
- Empirically-derived noise injection from M4 coefficients fitted against YuMi ground truth.
- Reproducible configuration and release artifacts to support independent replication of the validation pipeline.

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
- Empirical characterization of Livox Mid-360 range-noise behavior under controlled kinematic stress, targeting a gap that remains weakly covered by direct prior evidence.
- Cooling periods with documented thermal recovery curves; settling time verification in T3.
- Dataset published alongside plugin and analysis code.

---

## 7. Possible Future Work (outside this project)

A natural extension of the metrological framework developed here is its application to **six-axis force/torque (F/T) sensors** for contact-rich manipulation. The same YuMi platform, the same statistical methodology (Allan Variance + state-dependent covariance + TOST), and the same ground truth infrastructure could be applied to characterize F/T sensor noise under dynamic loading (a gap confirmed in the literature and of increasing relevance given the emergence of force-aware foundation models such as ForceVLA and FD-VLA). This is noted here as a future direction, not a commitment of the present project.

---

## 8. References

> **Canonical source:** [`docs/REFERENCES_CONSOLIDATED.md`](REFERENCES_CONSOLIDATED.md) is the single source of truth for all bibliography entries, tags, and relevance notes.
>
> Inline numeric citations in this document are retained for narrative continuity and should be resolved through the consolidated list (see `prev. [N]` mappings where applicable).
