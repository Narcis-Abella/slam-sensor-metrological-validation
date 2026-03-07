# Research Plan — Phase I: Metrological Validation of Visual-LiDAR SLAM Sensors

**Author:** Narcís Abella  
**Supervisor:** Dr. Antonio Gabino Salazar Martín  
**Institution:** IQS School of Engineering (Universitat Ramon Llull), Barcelona  
**Version:** 0.3 — Design phase, pending supervisor review  
**Date:** 2026-03-06

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

What remains poorly characterized is the combined effect: sensor model fidelity on end-to-end SLAM performance, measured across hardware and simulation simultaneously, under a controlled kinematic reference. Most studies treat sensor modeling and sim-to-real gap as separate problems, and few use an industrial manipulator as the trajectory reference. This study addresses both gaps in a single experimental framework — from static Allan Variance characterization to dynamic validation on YuMi arm trajectories, comparing real hardware, standard Gazebo simulation, and a metrological plugin in the same pipeline.

---

## 2. Scope and Strategic Reorientation

The original three-phase proposal (Abella, 2026) covered: (I) metrological sensor validation, (II) sim-to-real gap benchmarking on a mobile platform, and (III) massive-scale autonomous SLAM optimization. Based on supervisor feedback (A.G. Salazar, February 2026), this work is deliberately scoped to Phase I only.

Three reasons support keeping the scope to Phase I only.

First, a complete metrological validation of sensor models — with rigorous statistical testing against a sub-millimeter ground truth — is a contribution that stands on its own. Bundling it with a sim-to-real gap benchmark or a parameter optimization study would dilute the metrological result rather than strengthen it.

Second, the YuMi arm is the right tool for this specific question. Controlled trajectories, known kinematics, and ±0.02 mm repeatability are not replicable on a mobile base. That clean experimental setting is precisely what makes the sensor-level comparison credible.

Third, Phases II and III introduce additional variables — platform dynamics, uncontrolled environments, thousands of simulation runs — that each deserve their own experimental design. They are future work, not omissions.

To be explicit about scope: this study characterizes each sensor statically (noise coefficients via Allan Variance), validates the models dynamically (against YuMi ground truth trajectories), and tests whether a metrological Gazebo plugin using those coefficients produces trajectory estimates statistically indistinguishable from real hardware. It does not evaluate loop closure, map quality, or navigation performance — those require a mobile platform and belong to the next stage.

---

## 3. Related Work

### 3.1 IMU Noise Characterization

The IEEE standard for IMU noise characterization (e.g. IEEE Std 647) defines Allan Variance (or Allan Deviation, ADEV) as the primary tool for identifying stochastic noise processes in inertial sensors. The three key parameters are:

- ARW (Angle Random Walk): white noise floor of the gyroscope; identified by the slope of $-1/2$ on the log-log ADEV curve.
- Bias Instability: minimum of the ADEV curve; the best achievable gyroscope bias stability.
- RRW (Rate Random Walk): low-frequency drift; identified by slope $+1/2$. Not always identifiable from short logs; 10–12 h static recordings are required for robust estimation.

Furrer et al. (2016) established the simulation standard for inertial sensors in the RotorS framework, demonstrating that ARW and Bias Instability must be derived empirically from hardware logs — not assumed from datasheets — to produce drift predictions that match real flight data.

The **Six-Position Test (IEEE Std 1293)** [14] complements Allan Variance by isolating scale factor errors and gravitational bias — parameters that are not observable from static horizontal logs but are critical for tight-coupling in FAST-LIO2 [22] and GLIM [23].

### 3.2 LiDAR Characterization

Liu & Zhang (2021) highlighted the geometric degeneracy risks of solid-state LiDARs in environments with low structural variation. The Livox Mid-360's non-repetitive Rosetta scanning pattern provides full spherical coverage over time but creates short-term point cloud gaps that affect frame-to-frame ICP registration quality. Characterizing this temporal integration behavior — validating that the simulated Rosetta pattern matches the physical point cloud density over time — is a core contribution of this work.

For static LiDAR characterization, we propose a **planar orthogonal residual method** (instead of absolute distance measurement): the sensor is fixed facing a flat wall, and temporal variation is evaluated by computing the orthogonal error of successive point clouds against a reference plane fitted at t=0. This isolates thermal ToF drift without introducing ICP registration errors.

### 3.3 Camera Characterization

Handa et al. (2014) demonstrated that even with perfect ground truth, SLAM algorithms fail in synthetic environments due to incorrect noise modeling and lack of photorealistic texture. For the RealSense D455, the primary concerns are:

- **Fixed-Pattern Noise (FPN)** in the infrared projector — stable over short periods but temperature-dependent.
- **Thermal deformation of intrinsic parameters** — focal length and principal point drift as the projector warms up.

Monitoring 6-DOF pose estimation of a static AprilTag pattern over several hours directly quantifies these effects. If the simulator assumes a perfect camera and the real camera exhibits thermal intrinsic drift, we have a documented, quantified source of Reality Gap — which provides direct motivation for Phase II.

### 3.4 Sim-to-Real Validation Framework

Kadian et al. (2019) introduced "Sim2Real Predictivity" as a formal metric, investigating whether simulation performance correlates with real-world performance. Wang et al. (2020) used the TartanAir dataset to argue that the Reality Gap remains significant for visual odometry algorithms dependent on fine-grained texture features.

Our approach differs: rather than evaluating SLAM system performance across environments, we evaluate **sensor model fidelity** at the signal level — comparing raw IMU streams, point cloud statistics, and estimated trajectories between three conditions: (i) real hardware, (ii) standard Gazebo simulation, (iii) metrological simulation with empirically derived parameters.

### 3.5. Related Work with Laser-Tracker Ground Truth

Lin et al. (2022, *Measurement*) validated a 6-DOF dynamic measurement system using a laser tracker as ground truth, with IMU error analysis via Allan Variance and a fault-tolerant Kalman filter for sensor fusion [Lin et al., 2022](https://www.sciencedirect.com/science/article/pii/S0263224122012489). Their work focuses on large-scale metrology and single-axis/turntable calibration, without LiDAR or visual sensors, and does not model kinematic-state-dependent noise for SLAM evaluation.

### 3.6. Justification and differentiation of the M4 covariance model

The M4 model rests on established foundations: (i) **Allan Variance** (IEEE standard; Furrer et al., 2016) provides the static baseline — ARW, Bias Instability, RRW — that must be taken from empirical logs rather than datasheets for faithful simulation; (ii) **thermal drift** of IMU biases and variance is well documented (Joerger et al., 2021; MDPI Sensors, 2020), and exponential settling with a time constant $\tau_T$ is a standard way to model warm-up from cold start to steady state in inertial sensors; (iii) **g-sensitivity and motion-dependent noise** in MEMS (vibration, acceleration, angular rate) are known to affect bias and effective variance, so expressing additive terms proportional to $\|\omega\|$, $\|a\|$ and $\|\dot{a}\|$ is a direct parametrisation of these effects; (iv) **residual-based identification** (measured minus true, with ground truth from a high-precision reference) is the standard approach in calibration and in data-driven covariance tuning (e.g. Cheng et al., 2025, bi-level estimator-in-the-loop). Validation using a **robot arm as ground truth** for orientation and trajectory has been demonstrated (Kuti et al., 2024, *Sensors*). What is new is the **combination**: a single, explicit formula that adds a thermal term $\sigma^2_{\mathrm{static}}(t_{\mathrm{on}})$ and kinematic terms $c_v\|\omega\| + c_a\|a\| + c_j\|\dot{a}\|$, with all coefficients fitted from residuals under dynamic motion on a sub-millimetre repeatability manipulator (YuMi), and then used to drive SLAM sensor simulation. No prior work in the literature presents this unified, state-dependent covariance model for IMU/LiDAR/camera in the context of multi-backend SLAM evaluation.

### 3.7. Summary and Novelty Statement

- Gap: No prior work combines (a) kinematic-state-dependent IMU noise models, (b) multi-backend SLAM evaluation, and (c) sub-millimetre robotic ground truth.
- Our contribution: Phase I fills this gap by providing metrological characterisation and simulation models that are validated against real hardware and multiple SLAM backends.
- Explicit novelty: Prior work has characterised IMU noise via Allan Variance in static conditions (Furrer et al., 2016) or LiDAR accuracy in controlled environments (Liu & Zhang, 2021), and some studies have validated IMU dynamic performance with laser trackers (Lin et al., 2022, *Measurement*). No study has combined: (1) kinematic-state-dependent noise models derived from dynamic ground truth, (2) validation across multiple tight-coupled SLAM backends, and (3) statistical comparison against a sub-millimetre industrial manipulator reference. This combination is the novelty of Phase I.

---

## 4. Research Hypothesis

Primary hypothesis (H0):
> A simulation plugin that injects empirically derived Allan Variance coefficients (ARW, Bias Instability, RRW) and enforces the correct Livox Rosetta scanning topology will produce sensor output — and resulting SLAM trajectory estimates — that are statistically equivalent to real hardware within a pre-specified margin $\delta$, as measured by ATE and RPE against the YuMi kinematic reference.

Formal test: **TOST (Two One-Sided Tests)** for equivalence [Schuirmann, 1987; Lakens, 2017 — see References 31–32]. The margin $\delta$ (e.g. 5 mm ATE or 10% of reference ATE) is pre-specified before data collection. Equivalence is declared if the 90% confidence interval for the mean difference (Metrological sim − Real) lies entirely within $[-\delta, +\delta]$.

Success criterion: (i) **Equivalence:** TOST declares Metrological sim equivalent to Real within margin $\delta$. (ii) **Control:** Standard simulation differs significantly from Real (p < 0.05 via paired t-test or Wilcoxon), confirming that the metrological approach is necessary. If equivalence is not declared, this is also publishable — it indicates that even empirically derived parameters are insufficient.

Secondary hypothesis (H1):
> The quality of the IMU (ARW, Bias Instability) has a measurable and systematic effect on tight-coupling SLAM performance (ATE/RPE), quantifiable across three IMUs of different quality grades under identical dynamic conditions.

Secondary hypothesis (H2):
> Systematic CW/CCW asymmetry is observable in gyroscope bias — i.e., bias drift differs between clockwise and counter-clockwise rotation at identical angular velocities — and this asymmetry is captured more accurately by the metrological model than by standard simulation.

---

## 5. Experimental Platform

### 5.1 Ground Truth System

**ABB YuMi IRC5** (dual-arm collaborative robot):
- Repeatability: ±0.02 mm
- Payload capacity: ~500 g per arm
- Programming: RobotStudio (offline trajectory definition and playback)
- Reference frame: mechanical flange — requires hand-eye calibration to sensor optical/inertial center (see [METHODOLOGY.md](METHODOLOGY.md))

> Important distinction: The YuMi's ±0.02 mm figure is *repeatability*, not *volumetric accuracy*. The lower bound of our ground truth confidence is bounded by repeatability, not absolute accuracy. This must be stated explicitly in any publication.

RobotStudio provides the executed trajectory as the independent predictor of robot motion, separate from any SLAM estimate. This dual role — kinematic ground truth *and* independent simulation reference — is a key methodological strength of this setup.

### 5.2 Sensor Suite

See [`docs/HARDWARE_PAYLOAD.md`](HARDWARE_PAYLOAD.md) for full specification, weight breakdown, and session viability assessment.

| ID | Sensor | IMU chip | IMU rate | LiDAR type | Session |
|----|--------|----------|----------|------------|---------|
| S1 | WitMotion WT901C | MPU9250 | 200 Hz | — | A |
| S2 | Intel RealSense D455 | BMI055 | 400 Hz | — | B |
| S3 | Livox Mid-360 | ICM40609 | 200 Hz | Solid-state, 360° | D |
| S4 | RPLiDAR A2M12 | — | — | Mechanical 2D, 360° | C |

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

Upon completion, Phase I will produce the following verifiable outputs:

### 6.1 Allan Variance Characterization (Static)

- Allan Deviation curves (ADEV) for all three IMUs under identical conditions (temperature-documented, 10–12 h logs)
- **Numerical coefficients** with units and confidence intervals: ARW [°/√h], Bias Instability [°/h], RRW [°/h$^{3/2}$] where identifiable
- Six-Position Test results (IEEE Std 1293) for each IMU: scale factor and gravitational bias
- LiDAR planar orthogonal residual characterization (Mid-360 and RPLiDAR A2M12) over temperature
- Camera thermal intrinsic drift quantification (RealSense D455, AprilTag static protocol)

### 6.2 Dynamic Validation Dataset

- Three trajectory types (smooth / moderate / aggressive) × three repetition blocks (CW/CCW/mixed) × four sensor sessions = structured dataset with full kinematic reference
- Cooling periods between blocks with documented thermal recovery curves
- Settling time verification in T3 (aggressive): vibration spectrum during inter-waypoint pauses

### 6.3 Comparative Evaluation

- Three-condition dataset: real hardware / standard Gazebo simulation / metrological Gazebo plugin
- ATE and RPE computed with `evo` [29] (Umeyama alignment) for all conditions and trajectory types
- Statistical test results (paired t-test or Wilcoxon, $\alpha = 0.05$) with explicit statement of the test, normality assessment, and effect size
- CW/CCW asymmetry analysis per sensor (secondary H2)

### 6.4 Open-Source Software

- **Gazebo plugin for non-repetitive scan patterns:** One of the expected contributions is a Gazebo plugin that models **non-repetitive scanning topology** (Livox Mid-360 Rosetta pattern, and optionally Mid-70) so that simulation can reproduce the temporal integration and point-cloud statistics of solid-state Livox sensors. This complements the existing IMU/LiDAR noise injection and is required for a faithful comparison in Session D.
- Metrological Gazebo plugin: custom IMU noise injection (ARW + BI + RRW) and Livox Rosetta scanning topology emulation, extended with state-dependent variance as a function of time since power-on and kinematic state (velocity, acceleration, jerk).
- Allan Variance processing scripts (compatible with `imu_utils` and `kalibr`)
- `evo` [29]-based comparison pipeline with reproducible configuration
- SLAM evaluation matrix across multiple backends (GLIM [23], FAST-LIO2 [22], ORB-SLAM3 [24], Cartographer [25], KISS-ICP [26], Point-LIO [27], OpenVINS [28]; optional RTAB-Map [30]) under four noise model configurations (manufacturer, static Allan, in-session static, kinematic-residual with explicit thermal dependence) and three SLAM parameter regimes (low/nominal/high).

---

## 7. Planned SLAM Backends

Phase I is not a benchmark of SLAM algorithms per se, but SLAM backends are required as \"measuring instruments\" to evaluate sensor model fidelity. To avoid conclusions that depend on a single estimator, multiple backends are used per modality:

- Cross-modal baseline: GLIM [23] (factor-graph, GPU-accelerated, tight-coupled LiDAR+IMU and visual-inertial).
- LiDAR 3D (Mid-360): FAST-LIO2 [22] (IESKF, LiDAR+IMU), Point-LIO [27] (dense point-based), GLIM [23].
- LiDAR 2D (RPLiDAR): Cartographer 2D [25] (graph-based SLAM), KISS-ICP 2D [26] (odometry), GLIM [23] in planar 3D mode where feasible.
- Visual / Visual-inertial (RealSense D455): ORB-SLAM3 [24] (feature-based VIO), GLIM [23] (visual-inertial), OpenVINS [28] (EKF-based VIO). R3LIVE is excluded (camera+LiDAR fusion; Session B is RGB-D+IMU only).
- **Loose-coupled baseline (optional):** RTAB-Map [30] (2D, 3D LiDAR, RGB-D) in a reduced subset of experiments to contrast tight vs. loose coupling.

Noise injected in simulation follows four configurations (see `SLAM_BACKENDS.md`): M1 (manufacturer), M2 (static Allan), M3 (in-session static Allan), M4 (kinematic-residual). For each SLAM backend, a nominal configuration is tuned once on real YuMi data and then frozen; low/high sensitivity variants (scaled covariances) are used for robustness checks.

The goal is to show that the conclusions about sensor model fidelity — particularly for the kinematic-residual model — are consistent across:

- Different SLAM architectures (Kalman filter vs. factor graph vs. pose graph),
- Different sensing modalities (IMU-only, 2D LiDAR, 3D LiDAR, RGB-D),
- And reasonable variations in SLAM parameter tuning.

---

## 8. Possible Future Work (outside this project)

The experimental design and software architecture in this plan were originally inspired by a **broader three-phase research line** (metrological validation, mobile-platform Reality Gap benchmark, and large-scale SLAM optimisation). In the context of this degree project, however, only the metrological validation with YuMi ground truth is in scope. Any mobile-platform benchmark or automatic optimisation stage should be understood strictly as potential future work that could build on the outputs of this study — not as a commitment that will be executed within this project.

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

[14] Six-Position Test for IMU calibration — see IEEE calibration standards (e.g. IEEE Std 952) or Joerger et al. [11] for MEMS procedures.

[15] S. Umeyama, "Least-squares estimation of transformation parameters between two point patterns," *IEEE Transactions on Pattern Analysis and Machine Intelligence*, vol. 13, no. 4, pp. 376–380, 1991.

[16] ABB, "IRB 14000 YuMi — Product specification," ABB Robotics. [https://new.abb.com/products/robotics/industrial-robots/irb-14000-yumi](https://new.abb.com/products/robotics/industrial-robots/irb-14000-yumi) *(Repeatability ±0.02 mm per datasheet.)*

[17] H. Hou, "Modeling inertial sensors errors using Allan variance," *Proc. IEEE/ION PLANS*, 2004. [Semantic Scholar](https://www.semanticscholar.org/paper/Modeling-inertial-sensors-errors-using-Allan-Hou/6a1b4db58f429843a36d2f9f815d753a265c3dda) *(Classic procedure for extracting ARW/BI/RRW from AVAR; supports §1–2 and M2/M3 methodology.)*

[18] A. Carlson, K. A. Skinner, R. Vasudevan, and M. Johnson-Roberson, "Sensor Transfer: Learning Optimal Sensor Effect Image Augmentation for Sim-to-Real Domain Adaptation," arXiv:1809.06256, 2018. [https://arxiv.org/abs/1809.06256](https://arxiv.org/abs/1809.06256) *(Explicitly models sensor effects — not only scene — as source of sim-to-real gap; supports motivation in §1 and §3.4.)*

[19] National Institute of Standards and Technology (NIST), "3D Ground-Truth Systems for Object/Human Recognition and Tracking," NIST Report, 2019 (approx.). [PDF](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=913818) *(Survey of ground-truth systems for perception evaluation; contextualises use of robot arm as kinematic reference.)*

[20] Intel Corporation, "IMU Calibration Tool for Intel RealSense Depth Cameras (D435i, D455, L515)," Whitepaper rev. 1.4, July 2020. [PDF](https://www.realsenseai.com/wp-content/uploads/2020/07/IMU_Calibration_Tool_for_Intel_RealSense-Depth_Cameras_Whitepaper.pdf) *(Official procedure for D455 IMU calibration; supports §2.1 and static characterisation.)*

[21] "Procedure and Accuracy Assessment Results of Livox MID-360 Scanners," *ISPRS Arch. Photogramm. Remote Sens. Spatial Inf. Sci.*, Vol. XLVIII-1/W6-2025, 2025. [https://doi.org/10.5194/isprs-archives-XLVIII-1-W6-2025-147-2025](https://isprs-archives.copernicus.org/articles/XLVIII-1-W6-2025/147/2025/) *(Direct accuracy assessment of Mid-360; supports sensor choice and HARDWARE_PAYLOAD.)*

**SLAM backends and evaluation tools:**

[22] W. Xu, Y. Cai, D. He, J. Lin, and F. Zhang, "FAST-LIO2: Fast Direct LiDAR-Inertial Odometry," *IEEE Transactions on Robotics*, vol. 38, no. 4, pp. 2053–2073, 2022. [DOI: 10.1109/TRO.2022.3141876](https://ieeexplore.ieee.org/document/9697912)

[23] K. Koide, M. Yokozuka, S. Oishi, and A. Banno, "Globally Consistent and Tightly Coupled 3D LiDAR Inertial Mapping," in *Proc. IEEE International Conference on Robotics and Automation (ICRA)*, Philadelphia, 2022, pp. 5622–5628. [arXiv: 2202.00242](https://arxiv.org/abs/2202.00242)

[24] C. Campos, R. Elvira, J. J. Gómez Rodríguez, J. M. M. Montiel, and J. D. Tardós, "ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM," *IEEE Transactions on Robotics*, vol. 37, no. 6, pp. 1874–1890, 2021. [DOI: 10.1109/TRO.2021.3075644](https://ieeexplore.ieee.org/document/9440682)

[25] W. Hess, D. Kohler, H. Rapp, and D. Andor, "Real-Time Loop Closure in 2D LIDAR SLAM," in *Proc. IEEE International Conference on Robotics and Automation (ICRA)*, Stockholm, 2016, pp. 1271–1278. [DOI: 10.1109/ICRA.2016.7487258](https://ieeexplore.ieee.org/document/7487258)

[26] I. Vizzo, T. Guadagnino, B. Mersch, L. Wiesmann, J. Behley, and C. Stachniss, "KISS-ICP: In Defense of Point-to-Point ICP – Simple, Accurate, and Robust Registration If Done the Right Way," *IEEE Robotics and Automation Letters*, vol. 8, no. 2, pp. 1029–1036, 2023. [arXiv: 2209.15397](https://arxiv.org/abs/2209.15397)

[27] D. He, W. Xu, N. Chen, F. Kong, C. Yuan, and F. Zhang, "Point-LIO: Robust High-Bandwidth LiDAR-Inertial Odometry," *Advanced Intelligent Systems*, vol. 5, no. 7, 2023. [DOI: 10.1002/aisy.202200459](https://doi.org/10.1002/aisy.202200459)

[28] P. Geneva, K. Eckenhoff, W. Lee, Y. Yang, and G. Huang, "OpenVINS: A Research Platform for Visual-Inertial Estimation," in *Proc. IEEE International Conference on Robotics and Automation (ICRA)*, Paris, 2020. [https://github.com/rpng/open_vins](https://github.com/rpng/open_vins)

[29] M. Grupp, "evo: Python package for the evaluation of odometry and SLAM," 2017. [https://github.com/MichaelGrupp/evo](https://github.com/MichaelGrupp/evo)

[30] M. Labbé and F. Michaud, "RTAB-Map as an Open-Source Lidar and Visual SLAM Library for Large-Scale and Long-Term Online Operation," *Journal of Field Robotics*, vol. 36, no. 3, pp. 416–446, 2018. [DOI: 10.1002/rob.21831](https://doi.org/10.1002/rob.21831)

**Equivalence testing (TOST):**

[31] D. J. Schuirmann, "A comparison of the Two One-Sided Tests Procedure and the Power Approach for assessing the equivalence of average bioavailability," *Journal of Pharmacokinetics and Biopharmaceutics*, vol. 15, no. 6, pp. 657–680, 1987. [DOI: 10.1007/BF01068419](https://doi.org/10.1007/BF01068419) *(Foundational TOST procedure; standard in bioequivalence and adopted for measurement equivalence.)*

[32] D. Lakens, "Equivalence Tests: A Practical Primer for t Tests, Correlations, and Meta-Analyses," *Social Psychological and Personality Science*, vol. 8, no. 4, pp. 355–362, 2017. [DOI: 10.1177/1948550617697177](https://doi.org/10.1177/1948550617697177) *(Practical primer on equivalence testing; explains why p > 0.05 does not demonstrate equivalence.)*
