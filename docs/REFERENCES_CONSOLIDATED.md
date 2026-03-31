# References - Consolidated

* **Version:** 0.6
* **Date:** 2026-03-31
* **Status:** Single source of truth - supersedes RESEARCH_PLAN.md §9, REFERENCES_POOL.md, and EXTENDED_LITERATURE_REVIEW.md

**Sources merged:**

- `docs/RESEARCH_PLAN.md` §9, refs [1]–[48]
- `docs/REFERENCES_POOL.md` (pool refs 1–63, post-audit 2026-03-23)
- `docs/EXTENDED_LITERATURE_REVIEW.md` (120+ papers audited 2026-03-23)
- `Context Document for Repository Rewrite` (NEW-01–NEW-11, 2026-03-28)

**Scope (active):** Sensor metrology and statistical equivalence testing at residual level. SLAM-heavy and broad sim-to-real references are retained only as archived context.

**Tag legend:** `[TOST]` `[AVAR]` `[IMU]` `[LIDAR]` `[CAMERA]` `[ROBOT-GT]` `[HAND-EYE]` `[SYNC]` `[SIM2REAL]` `[SLAM]` `[PLUGIN]` `[METROLOGY]`

**Cross-reference:** The legacy numbering from RESEARCH_PLAN.md §9 is noted as *(prev. [N])* where applicable.

---

## Active Reference Priority

Use these groups as the default citation pool for active documentation (`README`, `METHODOLOGY`, `EXPERIMENTAL_DESIGN`, `HARDWARE_PAYLOAD`, `RESEARCH_PLAN`):

- `[METROLOGY]`, `[TOST]`, `[AVAR]`
- `[ROBOT-GT]`, `[LIDAR]`, `[CAMERA]`, `[SYNC]`
- Standards and uncertainty references (ISO/IEEE/GUM)

Use `[SLAM]` and SLAM-heavy `[SIM2REAL]` entries only when explicitly needed for archived context or prior-art framing.

---

## A. Equivalence Testing

**[1]** D. J. Schuirmann, "A comparison of the Two One-Sided Tests Procedure and the Power Approach for assessing the equivalence of average bioavailability," *Journal of Pharmacokinetics and Biopharmaceutics*, vol. 15, no. 6, pp. 657–680, 1987. DOI: 10.1007/BF01068419 *(prev. [31])*
`[TOST]`
> Defines the TOST procedure: equivalence declared only when both one-sided null hypotheses (difference exceeds ±δ) are rejected at α = 0.05, equivalently when the 90% CI lies entirely within [−δ, +δ]. The foundational reference for all TOST cited in this study; mandatory citation for any metrology journal submission.

---

**[2]** D. Lakens, "Equivalence Tests: A Practical Primer for t Tests, Correlations, and Meta-Analyses," *Social Psychological and Personality Science*, vol. 8, no. 4, pp. 355–362, 2017. DOI: 10.1177/1948550617697177. URL: [Publisher page](https://journals.sagepub.com/doi/10.1177/1948550617697177) *(prev. [32])*
`[TOST]`
> The most-cited modern guide for researchers implementing TOST in practice. Covers margin selection, 90% CI computation, effect size reporting, and interpretation. Cite when describing the TOST procedure in METHODOLOGY §3.4 alongside Schuirmann [1].

---

**[3]** S. Wellek, *Testing Statistical Hypotheses of Equivalence and Noninferiority*, 2nd ed., Chapman & Hall/CRC, 2010. ISBN: 978-1439808184. *(prev. [48])*
`[TOST]`
> Reference textbook for equivalence testing, covering parametric and non-parametric variants, sample size, power, and noninferiority designs. Reviewers at *Measurement* and *IEEE TIM* will expect this citation; use alongside [1] and [2].

---

**[4]** C. M. Robinson, "Model validation using equivalence tests," *Ecological Modelling*, vol. 176, no. 3–4, pp. 349–358, 2004. DOI: 10.1016/j.ecolmodel.2004.01.013. URL: [Publisher page](https://www.sciencedirect.com/science/article/pii/S0304380004000985)
`[TOST]`
> First paper proposing TOST as a model validation tool. Argues that classical NHST has the null hypothesis backwards for validation: the burden of proof should lie with the model to demonstrate equivalence, not with the data to prove a difference. Provides the methodological precedent for our primary contribution outside pharma.

---

**[5]** A. Ramert, "Equivalence Testing," STAT COE Best Practice Report 12-2020, Air Force Institute of Technology (AFIT), 2020.
`[TOST]`
> Applies TOST to defense M&S validation, using weapon trajectory simulation vs. live-fire data as the example. Explicitly states: "Model validation is an ideal application for equivalence testing." Validates our choice to apply TOST for sensor noise model validation; cite as a domain-bridge reference in METHODOLOGY §3.4 and README §6.1.

---

**[6]** J. Wu, U. Sander, C. Flannagan, M. Zhao, and J. Bargman, "Practical Equivalence Testing and Its Application in Synthetic Pre-Crash Scenario Validation," arXiv:2505.12827, 2025.
`[TOST]` `[SIM2REAL]`
> Applies practical equivalence testing to synthetic pre-crash scenario validation in an automotive context. Use as an adjacent-domain precedent for equivalence-style validation, but treat it as contextual evidence for this study because its statistical framing and task domain differ from residual-level robotics sensor metrology.

---

**[7]** G. Shieh, "Exact Power and Sample Size Calculations for the Two One-Sided Tests of Equivalence," *PLoS ONE*, vol. 11, no. 9, e0162093, 2016. DOI: 10.1371/journal.pone.0162093
`[TOST]`
> Derives exact (not approximate) power functions and sample size formulas for TOST. Required for the formal pre-data power analysis: given the planned n ≈ 40–60 repetitions per block, σ derived from literature, and chosen δ, this formula verifies that 80% power at α = 0.05 is achievable before YuMi sessions begin. Resolves the P0 item in `internal/PENDING_2026-03-27.md`.

---

**[8]** S. A. Pardo, *Equivalence and Noninferiority Tests for Quality, Manufacturing and Test Engineers*, CRC Press, 2014. ISBN: 978-1466586888.
`[TOST]` `[METROLOGY]`
> The only textbook on equivalence testing written for engineers rather than pharma statisticians. Covers applications in manufacturing quality, sensor testing, and instrumentation. Use in Discussion to frame TOST as "a 40-year-old standard methodology, well-established outside robotics," strengthening the novelty claim that this study is the first to apply it to sensor simulation validation.

---

## B. IMU Noise, Allan Variance, and State-Dependent Covariance

**[9]** N. El-Sheimy, H. Hou, and X. Niu, "Analysis and Modeling of Inertial Sensors Using Allan Variance," *IEEE Transactions on Instrumentation and Measurement*, vol. 57, no. 1, pp. 140–149, 2008. DOI: 10.1109/TIM.2007.908635
`[AVAR]` `[IMU]`
> Seminal paper (750+ citations) establishing Allan Variance as the standard tool for IMU noise characterization, covering ARW, Bias Instability, and RRW identification. The primary IEEE TIM reference for AVAR methodology; more widely cited than Furrer [10] for this specific topic. Mandatory citation for any submission to *IEEE TIM* or *Measurement*.

---

**[10]** F. Furrer, M. Burri, M. Achtelik, and R. Siegwart, "RotorS - A Modular Gazebo MAV Simulator Framework," in *ROS: The Complete Reference (Vol. 1)*, Springer, 2016. *(prev. [1])*
`[AVAR]` `[IMU]` `[SIM2REAL]`
> Establishes a widely used MAV simulation framework and motivates empirical sensor parameterization in simulation pipelines. Use as sim-infrastructure context; do not use as a primary source for Allan-variance methodology or for strict claims about ARW/BI estimation requirements.

---

**[11]** H. Hou and N. El-Sheimy, "Inertial Sensors Errors Modeling Using Allan Variance," in *Proc. ION GPS/GNSS*, 2003.
`[AVAR]` `[IMU]`
> Early and relevant AVAR background for separating IMU noise terms (ARW, BI, RRW). Use as foundational context, but avoid turning this source into a strict rule for fixed logging duration without additional direct support.

---

**[12]** N. Gallon, M. Joerger, and B. Pervan, "Development of Stochastic IMU Error Models for INS/GNSS Integration," in *Proc. ION GNSS+*, 2021. DOI: 10.33012/2021.17962.
`[AVAR]` `[IMU]`
> Covers stochastic IMU error modeling in navigation integration contexts. Use as supporting evidence for stochastic-modeling practice; treat direct transfer to the M4 thermal functional form as indirect unless corroborated by more specific thermal-identification sources.

---

**[13]** A. Kozlov and A. Tarygin, "Real-Time Estimation of Temperature Time Derivative in Inertial Measurement Unit by Finite-Impulse-Response Exponential Regression on Updates," *Sensors*, vol. 20, no. 5, 1299, 2020. DOI: 10.3390/s20051299. URL: [Publisher page](https://www.mdpi.com/1424-8220/20/5/1299) *(prev. [12])*
`[AVAR]` `[IMU]`
> Studies real-time thermal-state estimation signals in IMU pipelines. Use as thermal-behavior context; avoid claiming that it directly derives the exact exponential model form used in M4 without additional derivation evidence.

---

**[14]** K. A. Lethander and C. N. Taylor, "Conservative Estimation of Inertial Sensor Errors Using Allan Variance Data," *NAVIGATION: Journal of the Institute of Navigation*, vol. 70, no. 3, navi.563, 2023. DOI: 10.33012/navi.563. URL: [Publisher page](https://navi.ion.org/content/70/3/navi.563)
`[AVAR]` `[IMU]`
> Derives conservative upper-bound error estimates from AVAR data, showing that static AVAR provides a lower bound on operational noise (static < in-session < dynamic). Provides the physical justification for the M1 < M2 < M3 < M4 hierarchy: each rung closes the gap between the static lower bound and the true operational noise.

---

**[15]** Z. Miao, Z. Shen, Y. Bi, and X. Zhu, "Online Estimation of Allan Variance Coefficients Based on a Neural-Extended Kalman Filter," *Sensors*, vol. 15, no. 2, pp. 2496–2524, 2015. DOI: 10.3390/s150202496. URL: [Publisher page](https://www.mdpi.com/1424-8220/15/2/2496)
`[AVAR]` `[IMU]`
> Extends AVAR to real-time online coefficient estimation via a neural-extended Kalman filter. Demonstrates that AVAR coefficients can drift in operational conditions, supporting the distinction between M2 (static log) and M3 (in-session static windows). Cite if reviewers question why M3 differs from M2.

---

**[16]** G. Yan, W. Duan, H. Lu, X. Zhao, and T. Yin, "Effective Bias Warm-Up Time Reduction for MEMS Gyroscopes Based on Active Suppression of the Coupling Stiffness," *Microsystems & Nanoengineering*, vol. 5, art. 18, 2019. DOI: 10.1038/s41378-019-0057-2
`[IMU]`
> Characterizes MEMS gyroscope warm-up times (~2000 s without compensation) and proposes active suppression methods. Provides quantitative physical justification for the 20-minute powered warm-up protocol applied to the Mid-360 (ICM-40609) and WT901C (MPU9250) before any session; cite in EXPERIMENTAL_DESIGN §2.1 and §2.3.

---

**[17]** L. Chen, P. Li, Z. Ge, W. Pan, and X. Xi, "A Temperature Drift Suppression Method of Mode-Matched MEMS Gyroscope Based on Mode Reversal and Multiple Regression," *Micromachines*, vol. 13, no. 10, 1557, 2022. DOI: 10.3390/mi13101557. URL: [Publisher page](https://www.mdpi.com/2072-666X/13/10/1557)
`[IMU]`
> Models and suppresses temperature-induced drift in MEMS gyroscopes. Shows that thermal drift manifests as both bias offset and variance increase, with distinct time constants for different operating regimes. Supports the M4 thermal component design and the rationale for measuring temperature during sessions.

---

**[18]** A. Solodar and I. Klein, "VIO-DualProNet: Visual-inertial odometry with learning based process noise covariance," *Engineering Applications of Artificial Intelligence*, vol. 133, 2024. DOI: 10.1016/j.engappai.2024.108466. URL: [Publisher page](https://www.sciencedirect.com/science/article/pii/S0952197624003617) *(prev. [40])*
`[IMU]` `[SLAM]`
> Uses dual deep neural networks to estimate per-timestep IMU covariance matrices in a VIO system; achieves 25% improvement over constant-covariance baselines. **Primary prior art for the M4 concept.** Differentiator: VIO-DualProNet learns a black-box covariance from uncontrolled data; M4 is an explicit parametric formula with physically interpretable coefficients fitted from sub-mm ground truth. Cite in RESEARCH_PLAN §3.6 and README §5.1.

---

**[19]** S. Qiu, C. Chen, Y. Liu, C. Wang, Y. Zhao, and S. Shen, "AirIMU: Learning Uncertainty Propagation for Inertial Odometry," arXiv:2310.04874, 2023.
`[IMU]` `[SLAM]`
> CNN-based per-frame covariance estimation for IMU pre-integration; models how noise uncertainty propagates through the integration. **Second primary prior art for M4.** Same differentiator as [18]: learning-based black-box vs. M4's parametric, auditable formula. Cite in RESEARCH_PLAN §3.6 and README §5.1.

---

**[20]** D. Cheng, J. Kang, and X. Xiong, "Simultaneous Calibration of Noise Covariance and Kinematics for State Estimation of Legged Robots via Bi-level Optimization," arXiv:2510.11539, 2025. *(prev. [13])*
`[IMU]` `[ROBOT-GT]`
> Simultaneously optimizes covariance parameters and kinematic parameters for legged robot state estimation. Uses legged robot proprioception as the reference rather than an absolute ground truth manipulator. Differentiates from M4 in that our study uses a sub-mm industrial arm (not leg odometry) and targets noise *variance* as an explicit function of kinematic state rather than optimizing it jointly with kinematics.

---

**[21]** K. Khosoussi and I. Shames, "Joint State and Noise Covariance Estimation," arXiv:2502.04584, 2024.
`[IMU]`
> Provides theoretical observability bounds for jointly estimating system state and noise covariance. Establishes conditions under which state-dependent covariance can be identified from measurements. Useful for framing the identifiability analysis of M4 coefficients (PENDING_2026-03-27 P0 item on M4 identifiability protocol); cite in METHODOLOGY §5.4.

---

**[22]** P. Li, T. Qin, and S. Shen, "A Resilient Method for Visual–Inertial Fusion Based on Covariance Tuning," *Sensors*, vol. 22, no. 24, 9836, 2022. DOI: 10.3390/s22249836.
`[IMU]` `[SLAM]`
> Demonstrates that dynamically tuning IMU covariance at runtime improves VIO robustness against sensor degradation and dynamic motion. Supports the argument that state-dependent covariance (M4) is practically useful in SLAM estimation, not merely a modeling curiosity.

---

**[23]** S. Choi, D. Park, S.-Y. Hwang, and T.-W. Kim, "Statistical Uncertainty Learning for Robust Visual-Inertial State Estimation," arXiv:2510.01648, 2025.
`[IMU]` `[SLAM]`
> DNN-based uncertainty-learning prior art in robust VIO. Use as contextual evidence that learning-based uncertainty modeling is active; avoid broad dominance claims or overly specific camera+IMU output assumptions unless supported directly in full-text details.

---

## C. LiDAR Characterization

**[24]** J. Liu and F. Zhang, "LiODOM for Livox LiDARs: A Study on Geometric Degeneracy," arXiv:2111.03393, 2021. *(prev. [2])*
`[LIDAR]`
> Characterized degeneracy risks in solid-state Livox LiDARs caused by the non-repetitive Rosetta scan pattern: short-term point cloud gaps in low-structural-variation environments increase geometric degeneracy risk for frame-to-frame ICP. Justifies the need for a simulation plugin that faithfully reproduces the Rosetta topology, not merely a uniform Gaussian point cloud.

---

**[25]** B. Vultaggio, F. Giacomini, and A. Vitti, "Simulation of Low-Cost MEMS-LiDAR and Analysis of Its Effect on the Performances of State-of-the-Art SLAMs," *ISPRS Archives*, vol. XLVIII-1/W1-2023, pp. 539–546, 2023. DOI: 10.5194/isprs-archives-XLVIII-1-W1-2023-539-2023 *(prev. [44])*
`[LIDAR]` `[PLUGIN]` `[SIM2REAL]`
> Built a Gazebo **Classic** plugin implementing the Mid-360 Rosetta scan pattern. Benchmarked KISS-ICP, CT-ICP, LIO-SAM, and FAST-LIO2 with generic Gaussian noise - without metrologically derived parameters. **Primary prior art for our Fortress plugin**: shows the scan pattern simulation is feasible and valuable, but (a) targets Gazebo Classic (EOL Jan 2025), (b) uses no hardware-derived noise, and (c) cannot be used with ROS 2 Humble.

---

**[26]** B. Schlager et al., "Experimental Evaluation of Vibration Influence on a Resonant MEMS Scanning System for Automotive Lidars," *IEEE Transactions on Industrial Electronics*, vol. 69, no. 3, pp. 3099–3108, 2022. DOI: 10.1109/TIE.2021.3065608 *(prev. [45])*
`[LIDAR]`
> Evaluates vibration influence in an automotive LiDAR-related scanning setup and supports the broader relevance of vibration-aware analysis. Use as indirect gap-context evidence only; avoid "only paper" wording and avoid claiming direct coverage of the Mid-360 use case.

---

**[27]** R. Brazeal, B. Wilkinson, and R. Hochmair, "A Rigorous Observation Model for the Risley Prism-Based Livox Mid-40 Lidar Sensor," *Sensors*, vol. 21, no. 14, 4722, 2021. DOI: 10.3390/s21144722. URL: [Publisher page](https://www.mdpi.com/1424-8220/21/14/4722) *(prev. [46])*
`[LIDAR]`
> Simulated Risley prism misalignment under vibration in the Livox Mid-40. Conclusion: range accuracy is largely unaffected by prism misalignment while angular accuracy degrades. **Limitation**: purely simulation-based, not experimentally validated on a shaker or kinematic platform. The experimental validation of this claim for the Mid-360 is a direct contribution of this study; cite as the gap we fill.

---

**[28]** J. Dong, L. Yao, Y. Cai, B. Huang, J. Zheng, and L. Wang, "Vibration-aware LiDAR-Inertial Odometry based on Point-wise Post-Undistortion Uncertainty," arXiv:2507.04311, 2025.
`[LIDAR]` `[SLAM]`
> Models undistortion residual uncertainty under platform vibration in a LiDAR-inertial odometry system. Treats intrinsic LiDAR range noise as a fixed constant - exactly the gap this study addresses. Confirms the problem is recognized; differentiates by showing we characterize intrinsic noise statistics (not undistortion residuals) under controlled kinematic stress.

---

**[29]** D. M. K. Felix, A. Bry, M. Thomas, and J. Betz, "Lidar Variability: A Novel Dataset and Comparative Study of Solid-State and Spinning Lidars," arXiv:2507.04321, 2025.
`[LIDAR]`
> Systematic comparison of solid-state and spinning LiDARs across multiple operating conditions. Supports the selection of Mid-360 as the primary 3D LiDAR and the exclusion of spinning LiDARs (LIO-SAM) as a design decision. Cite in HARDWARE_PAYLOAD when explaining the sensor selection rationale.

---

**[30]** J. Fryskowska-Skibniewska, "Procedure and Accuracy Assessment Results of Livox MID-360 Scanners," *ISPRS Archives*, vol. XLVIII-1/W6-2025, pp. 147–153, 2025. DOI: 10.5194/isprs-archives-XLVIII-1-W6-2025-147-2025 *(prev. [21])*
`[LIDAR]` `[METROLOGY]`
> Metrological accuracy assessment of Livox MID-360 scanners under controlled conditions. Provides independent characterization of range accuracy and angular resolution. Use as reference for baseline sensor performance claims; cite alongside hardware specifications in HARDWARE_PAYLOAD §3.

---

**[31]** C. Periu, P. Leblanc, D. Boily, E. Toussaint, and É. Gingras, "Isolation of Vibrations Transmitted to a LIDAR Sensor Mounted on an Agricultural Vehicle," *Canadian Biosystems Engineering*, vol. 55, pp. 2.1–2.7, 2013.
`[LIDAR]`
> Measured positioning error increase in a LiDAR sensor on a tractor at operational speeds; reports ~27% error increase with vehicle speed due to transmitted vibrations. Old study on a different sensor type, but provides empirical support for the general claim that platform-induced vibration affects LiDAR noise statistics - the hypothesis this study tests for Mid-360 under controlled kinematic conditions.

---

**[32]** M. Pełka and J. Będkowski, "Calibration of Planar Reflectors Reshaping LiDAR's Field of View," *Sensors*, vol. 21, no. 19, 6501, 2021. DOI: 10.3390/s21196501. URL: [Publisher page](https://www.mdpi.com/1424-8220/21/19/6501)
`[LIDAR]`
> Addresses field-of-view reshaping for Livox-type sensors using planar reflectors, with extrinsic calibration. Relevant to understanding Mid-360 FoV characteristics and calibration when the sensor is mounted on the YuMi flange with potential occlusions from the arm structure.

---

## D. Camera Characterization

**[33]** A. Handa, T. Whelan, J. McDonald, and A. J. Davison, "A Benchmark for RGB-D Visual Odometry, 3D Reconstruction and SLAM," in *Proc. ICRA*, IEEE, 2014. *(prev. [4])*
`[CAMERA]` `[SIM2REAL]`
> Showed that SLAM algorithms fail in synthetic environments not only due to texture/illumination differences but critically due to incorrect noise modeling - a calibration mismatch between synthetic and real sensor statistics is sufficient to cause failure. Motivates the need for metrologically derived camera noise models for simulation; cite in RESEARCH_PLAN §3.3.

---

**[34]** O. Vila, I. Boada, D. Raba, and E. Farres, "A Method to Compensate for the Errors Caused by Temperature in Structured-Light 3D Cameras," *Sensors*, vol. 21, no. 6, 2073, 2021. DOI: 10.3390/s21062073. URL: [Publisher page](https://www.mdpi.com/1424-8220/21/6/2073)
`[CAMERA]`
> Demonstrates that temperature can strongly affect depth accuracy in structured-light 3D cameras and motivates thermal-control protocols. Use as related evidence for D455 thermal concerns, but explicitly mark the transfer as analogous rather than "same technology" direct evidence.

---

**[35]** C. Heindl, T. Pönitz, G. Reitmayr, and A. Steininger, "Spatio-temporal depth correction of RGB-D sensors based on Gaussian Process Regression in real-time," 2017. [[GitHub]](https://github.com/cheind/rgbd-correction)
`[CAMERA]`
> Confirms spatio-temporal drift in RealSense and Kinect depth sensors; proposes Gaussian Process-based real-time correction. Establishes that depth bias evolves both spatially (Fixed Pattern Noise) and temporally (thermal drift), supporting the dual static characterization of FPN and thermal intrinsic drift in Session D.

---

**[36]** E. Lachat, H. Macher, M.-A. Mittet, T. Landes, and P. Grussenmeyer, "Assessment and Calibration of a RGB-D Camera (Kinect v2 Sensor) Towards a Potential Use for Close-Range 3D Modeling," *Remote Sensing*, vol. 7, no. 10, pp. 13070–13097, 2015.
`[CAMERA]` `[METROLOGY]`
> Classic characterization study documenting 3D camera warm-up times and thermal stabilization requirements. Shows that metric accuracy improves significantly after 20–30 minutes of thermal stabilization. Supports the warm-up protocol for D455 in EXPERIMENTAL_DESIGN §2.4.

---

**[37]** L. Riou, C. Moran, D. Caballero, and C. Letort, "Infrared light field imaging system free of fixed-pattern noise," *Scientific Reports*, vol. 7, 2017. DOI: 10.1038/s41598-017-13595-7
`[CAMERA]`
> Characterizes fixed-pattern noise behavior and mitigation in an infrared imaging setup. Use as methodological/contextual support for FPN treatment; avoid claiming direct D455-specific thermal-depth behavior from this source alone.

---

**[38]** D. P. Haefner and S. D. Burks, "Measurements and metrics of fixed pattern noise: a new procedure for thermal camera noise measurement," in *Proc. SPIE Defense+Commercial Sensing*, vol. 12533, 2023.
`[CAMERA]`
> Defines rigorous FPN measurement and reporting methodology for thermal imaging cameras. Provides a standardized procedure analogous to what is applied to the D455 projector in the static characterization of Session D. Cite as methodological support for the camera FPN characterization protocol.

---

**[39]** Intel Corporation, "IMU Calibration Tool for Intel RealSense Depth Cameras (D435i, D455, L515)," Whitepaper rev. 1.4, July 2020. *(prev. [20])*
`[CAMERA]` `[IMU]`
> Official calibration tool documentation for the D455 BMI055 IMU. Describes the factory calibration procedure and its limitations. Cite as the reference for D455 IMU baseline factory parameters (M1) and the motivation for independent Allan Variance characterization (M2) to replace factory defaults.

---

## E. Ground Truth - Robot Manipulators

**[40]** J. Kuti, T. Piricz, and P. Galambos, "A Robust Method for Validating Orientation Sensors Using a Robot Arm as a High-Precision Reference," *Sensors*, vol. 24, no. 24, 8179, 2024. DOI: 10.3390/s24248179. URL: [Publisher page](https://www.mdpi.com/1424-8220/24/24/8179) *(prev. [10])*
`[ROBOT-GT]` `[IMU]`
> Uses an industrial robot arm as a high-precision orientation reference to validate IMU sensors under controlled dynamic trajectories. Validates the methodology of using a manipulator as ground truth for inertial sensor characterization. **Closest methodological sibling**: differs from this study in that it does not fit stochastic noise models, does not apply TOST, and targets deterministic orientation accuracy rather than variance characterization.

---

**[41]** C. Ferguson, T. Ertop, M. Herrell, and R. J. Webster III, "Unified Robot and Inertial Sensor Self-Calibration," *Robotica*, vol. 41, no. 5, pp. 1590–1616, 2023. DOI: 10.1017/S0263574723000012 *(prev. [42])*
`[ROBOT-GT]` `[IMU]`
> Uses a collaborative arm for joint robot/IMU self-calibration and supports residual-based calibration practice. Treat as related calibration precedent, but avoid framing it as direct robot-ground-truth validation for stochastic noise variance identification.

---

**[42]** J. Lin, G. Chen, and J. Mao, "An Accurate 6-DOF Dynamic Measurement System with Laser Tracker for Large-Scale Metrology," *Measurement*, vol. 204, 2022. DOI: 10.1016/j.measurement.2022.112052 *(prev. [7])*
`[ROBOT-GT]` `[METROLOGY]`
> Validated a 6-DOF dynamic measurement system using a laser tracker as ground truth, with IMU error analysis via Allan Variance and a fault-tolerant Kalman filter for sensor fusion. Published in *Measurement*, the study's primary target journal. Provides methodological precedent for using a high-precision external reference (laser tracker / robot arm) for dynamic sensor characterization with AVAR.

---

**[88]** National Institute of Standards and Technology (NIST), "3D Ground-Truth Systems for Object/Human Recognition and Tracking," NIST Report, 2019 (approx.). [PDF](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=913818) *(prev. [19])*
`[ROBOT-GT]` `[METROLOGY]`
> NIST overview of 3D ground-truth systems used for robotics and perception benchmarking, including traceability and uncertainty considerations. Supports the methodological framing for high-quality external references in sensor validation and complements the robot-arm and laser-tracker precedents used in this study.

---

**[43]** ABB, "IRB 14000 YuMi - Product specification," ABB Robotics. *(prev. [16])*
`[ROBOT-GT]` `[METROLOGY]`
> Official ABB product specification for the YuMi IRC5. Defines the key ground truth parameters: pose repeatability RP = ±0.02 mm (static points only), path repeatability RT = 0.10 mm (dynamic), path accuracy AT ≤ 1.36 mm (worst-case ISO 9283 conditions). **Primary source for ground truth uncertainty budget in METHODOLOGY §2.3.** The AT value is Type B uncertainty from this spec sheet; no traceable calibration certificate is available at IQS.

---

**[44]** L. Ricci, F. Taffoni, and D. Formica, "On the Orientation Error of IMU: Investigating Static and Dynamic Accuracy Targeting Human Motion," *PLOS ONE*, 2016. DOI: 10.1371/journal.pone.0161940. URL: [Publisher page](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0161940)
`[ROBOT-GT]` `[IMU]`
> Systematically separates static and dynamic IMU orientation accuracy using controlled motion as a reference. Shows that dynamic accuracy is consistently worse than static accuracy and that the gap depends on motion intensity. Supports the M1 < M2 < M3 < M4 hierarchy and the distinction between Stage 1 (static) and Stage 2 (dynamic) experimental characterization.

---

**[45]** Z. Wan, Y. Zhong, T. Li, X. Liu, D. Mao, and H. Zhao, "An Improved Design of the MultiCal On-Site Calibration Device for Industrial Robots," *Sensors*, vol. 23, no. 12, 5717, 2023. DOI: 10.3390/s23125717. URL: [Publisher page](https://www.mdpi.com/1424-8220/23/12/5717)
`[ROBOT-GT]` `[METROLOGY]`
> Reviews and improves on-site calibration methods for industrial robots, addressing accuracy of kinematic parameters and repeatability under operational conditions. Supports the framing of the YuMi as a metrological tool and the need for calibration verification (hand-eye, base frame) before each session.

---

**[46]** O. Limoyo et al., "Self-Calibration of Mobile Manipulator Kinematic and Sensor Extrinsic Parameters Through Contact-Based Interaction," arXiv:1803.06406, 2018.
`[ROBOT-GT]` `[HAND-EYE]`
> Uses contact-based self-calibration of kinematic and extrinsic parameters without an external metrology reference. Use as calibration-context evidence only; do not use it as direct support for treating RobotStudio flange poses as calibrated geometric ground truth.

---

## F. Hand-Eye Calibration

**[47]** I. Enebuse et al., "Accuracy Evaluation of Hand-Eye Calibration Techniques for Vision-Guided Robots," *PLoS ONE*, vol. 17, no. 10, e0273261, 2022. DOI: 10.1371/journal.pone.0273261. URL: [Publisher page](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0273261)
`[HAND-EYE]`
> Quantitative comparison of AX=XB hand-eye calibration techniques under varying noise and pose diversity. Use it to motivate pose-diversity and algorithm-sensitivity considerations, but avoid treating a single residual range as a fixed uncertainty-budget constant without task-specific validation.

---

**[48]** S. Liu, X. Shi, J. Cao, and Y. Zhu, "A Spatiotemporal Hand-Eye Calibration for Trajectory Alignment in Visual(-Inertial) Odometry Evaluation," arXiv:2404.14894, 2024.
`[HAND-EYE]` `[SLAM]`
> Spatiotemporal hand-eye calibration designed specifically for aligning sensor trajectories against motion capture or robot ground truth in VIO evaluation. **The only paper that directly parallels the study's use case**: sensors on a robot arm evaluated against kinematic reference via hand-eye transform. Establishes that spatiotemporal calibration uncertainty is a non-trivial systematic error in trajectory evaluation; cite in METHODOLOGY §2.3 uncertainty budget and EXPERIMENTAL_DESIGN §10.

---

**[49]** P. Furgale, J. Rehder, and R. Siegwart, "Unified Temporal and Spatial Calibration for Multi-Sensor Systems," in *Proc. IEEE/RSJ IROS*, 2013. *(prev. [37])*
`[HAND-EYE]` `[SLAM]`
> Introduced the Kalibr framework for calibrating extrinsic spatial and temporal parameters across multiple IMUs and cameras using a continuous-time B-spline representation. The standard tool for IMU-camera extrinsic calibration used in Sessions C and D. Assumes constant measurement noise covariance - M4 extends this by capturing how variance scales with kinematic state.

---

**[50]** J. Rehder, J. Nikolic, T. Schneider, T. Hinzmann, and R. Siegwart, "Extending kalibr: Calibrating the extrinsics of multiple IMUs and of individual axes," in *Proc. IEEE ICRA*, 2016. *(prev. [38])*
`[HAND-EYE]` `[SLAM]`
> Extends Kalibr to multi-IMU and per-axis calibration. Handles the calibration scenario in Sessions A–D where multiple sensors (IMU inside Mid-360, BMI055 inside D455) have factory extrinsics that require independent verification. Cite alongside [49] for the calibration methodology.

---

**[51]** L. Li, L. Wan, V. Krueger, and X. Zhang, "3D Hand-Eye Calibration for Collaborative Robot Arm: Look at Robot Base Once," arXiv:2504.21619, 2025.
`[HAND-EYE]`
> Proposes an efficient hand-eye calibration method for collaborative robot arms and reports representative residuals. Use as contemporary method context; do not use it as strict proof that a fixed pose count guarantees sub-mm residuals.

---

## G. Temporal Synchronization

**[52]** E. Olson, "A Passive Solution to the Sensor Synchronization Problem," in *Proc. IEEE/RSJ IROS*, pp. 1059–1064, 2010. DOI: 10.1109/IROS.2010.5650579
`[SYNC]`
> Derives the spatial error introduced by temporal offset between a moving sensor and a reference: at 90°/s angular velocity, a 10 ms synchronization error causes a 15.7 cm projection error. **The concrete numerical argument for preferring PTP IEEE 1588 (< 1 µs) over NTP (~ 0.2 ms) in the experimental protocol.** This number enters the uncertainty budget in METHODOLOGY §2.3; cite in EXPERIMENTAL_DESIGN §9.

---

**[53]** M. Faizullin, A. Kornilova, and G. Ferrer, "Open-Source LiDAR Time Synchronization System by Mimicking GNSS-Clock," arXiv:2107.02625, 2021.
`[SYNC]`
> Achieves 1 µs LiDAR-IMU synchronization precision using a low-cost MCU that mimics GNSS clock signals. Directly applicable to the Mid-360 synchronization protocol in Session C. Cite as a practical implementation reference for PTP-level synchronization without dedicated hardware.

---

**[54]** E. R. Jellum, T. H. Bryne, T. A. Johansen, and M. Orlandic, "The Syncline Model - Analyzing the Impact of Time Synchronization in Sensor Fusion," arXiv:2209.01136, 2022.
`[SYNC]`
> Analytical model of synchronization error impact in multi-sensor fusion. Shows that temporal offset accumulates into systematic trajectory error proportional to angular velocity and offset duration, with interaction effects between modalities. Complements [52] with a system-level analysis; cite when justifying why sync uncertainty is included in the METHODOLOGY §2.3 ground truth uncertainty budget.

---

## H. Sim-to-Real Gap (supplementary context)

**[55]** F. Muratore, M. Preuss, M. Killian, B. Alt, and J. Peters, "Robot Learning from Randomized Simulations: A Review," *Frontiers in Robotics and AI*, vol. 9, 799893, 2022. DOI: 10.3389/frobt.2022.799893
`[SIM2REAL]`
> Formalizes the *Simulation Optimization Bias*: a policy trained on a slightly incorrect simulator does not learn to be robust - it learns to exploit the model's errors because that maximizes simulation reward. Shows that domain randomization without validated sensor models teaches policies to adapt to simulator artifacts, not to solve the task. **Primary motivation for sensor model validation as a prerequisite to domain randomization**; cite in README §0 and RESEARCH_PLAN §1.

---

**[56]** E. Aljalbout, J. Elsner, D. Büchler, J. Peters, and C. Mastalli, "The Reality Gap in Robotics: Challenges, Solutions, and Best Practices," arXiv:2510.20808, 2025.
`[SIM2REAL]`
> Comprehensive survey of the sim-to-real gap across robotics sub-domains (manipulation, navigation, locomotion). Identifies unvalidated sensor models as a structurally distinct and unresolved source of the gap, separate from dynamics parameterization, contact models, and visual rendering. Quantifies gaps: 70% discrepancy in friction articulation, 83% in textile manipulation. **The authoritative recent survey supporting the study's core framing**; cite in README §0 and RESEARCH_PLAN §1.

---

**[57]** HAL record `hal-04673156`, "Evaluating the Sim-to-Real Gap for Contact-Rich Robotic Manipulation," 2024.
`[SIM2REAL]`
> Contextual sim-to-real manipulation comparison record. Keep only as supplementary context until full bibliographic metadata is normalized; avoid benchmark-superlative wording in active claims.

---

**[58]** A. Kadian, J. Truong, A. Gokaslan, A. Clegg, E. Wijmans, S. Lee, M. Savva, S. Chernova, and D. Batra, "Sim2Real Predictivity: Does Evaluation in Simulation Predict Real-World Performance?", *IEEE Robotics and Automation Letters*, vol. 5, no. 4, pp. 6670–6677, 2020. *(prev. [5])*
`[SIM2REAL]`
> Investigates whether simulation performance predicts real-world performance for navigation tasks. Finds significant discrepancy and identifies sensor fidelity as a contributing factor. Provides the formal "Sim2Real Predictivity" metric as a conceptual framework. Cite in RESEARCH_PLAN §3.4 as early formalization of the sim-to-real evaluation problem.

---

**[59]** W. Wang, D. Zhu, X. Wang, Y. Hu, Y. Qiu, C. Wang, Y. Hu, A. Kapoor, and S. Scherer, "TartanAir: A Dataset to Push SLAM to New Limits," in *Proc. IEEE/RSJ IROS*, 2020. *(prev. [6])*
`[SIM2REAL]` `[SLAM]`
> Synthetic dataset demonstrating significant reality gap for visual odometry algorithms dependent on fine-grained texture features. Shows that gap varies drastically with scene type and sensor configuration. Cite in RESEARCH_PLAN §3.4 to contextualize this study's focus on the sensor noise component of the gap vs. the visual/textural components.

---

**[60]** A. Carlson, K. Skinner, R. Vasudevan, and M. Johnson-Roberson, "Sensor Transfer: Learning Optimal Sensor Effect Image Augmentation for Sim-to-Real Domain Adaptation," arXiv:1803.07721, 2018. *(prev. [18])*
`[SIM2REAL]` `[CAMERA]`
> Presents learning-based augmentation to bridge visual sensor effects in sim-to-real pipelines. Use as camera-domain contextual support; avoid over-specific performance/generalization claims unless tied to a verified archival publication record.

---

**[61]** V.-L. Ngo, C. Fouard, and A. Brial, "A Multi-Layered Approach for Measuring the Simulation-to-Reality Gap of Radar Perception," arXiv:2106.08372, 2021.
`[SIM2REAL]`
> Proposes an implicit + explicit layered evaluation approach for radar sim-to-real gap, using descriptive distance metrics. **The closest prior work to our study**: evaluates sensor model fidelity at the signal level for a radar sensor, but uses descriptive metrics (KL divergence, percentile comparisons) rather than pre-specified equivalence bounds. Cite in RESEARCH_PLAN §3.7 as the most analogous prior work and the key differentiator: TOST with pre-specified δ vs. descriptive distance metrics.

---

**[62]** G. Mahajan, N. Okafor, and J. Moreno, "Quantifying the Sim2real Gap for GPS and IMU Sensors," arXiv:2403.11000, 2024.
`[SIM2REAL]` `[IMU]`
> Quantifies the sim-to-real gap specifically for GPS and IMU sensors using SLAM algorithms as evaluation proxies. Compares raw sim vs. real sensor data but without formal equivalence bounds - confirms the gap exists but cannot bound its practical significance. Cite as a recent example of sensor-level sim-to-real gap quantification without equivalence testing.

---

**[63]** M. Filipenko and I. Afanasyev, "Comparison of Various SLAM Systems for Mobile Robot in an Indoor Environment," in *Proc. IEEE Intelligent Systems (IS)*, 2018. *(prev. [3])*
`[SIM2REAL]` `[SLAM]`
> Found that SLAM systems that appear robust in simulation consistently underperform when deployed on real hardware, specifically attributing this to overly optimistic sensor models in simulation. Early empirical confirmation of the structural problem this study addresses; cite in RESEARCH_PLAN §1 as a motivating observation.

---

**[64]** B. Acosta et al., "Validating Robotics Simulators on Real-World Impacts," *IEEE Robotics and Automation Letters*, vol. 7, no. 3, pp. 6471–6478, 2022. DOI: 10.1109/LRA.2022.3174367.
`[SIM2REAL]`
> Compares Drake, MuJoCo, and PyBullet against real impact dynamics using descriptive validation. No equivalence testing; uses visual and RMSE-based metrics. Supports the argument that the robotics community lacks a standard statistical methodology for declaring equivalence - even when using multiple simulators on controlled tasks.

---

**[65]** A. Loquercio, E. Kaufmann, R. Ranftl, A. Dosovitskiy, V. Koltun, and D. Scaramuzza, "Deep Drone Racing: From Simulation to Reality with Domain Randomization," *IEEE Transactions on Robotics*, vol. 36, no. 1, pp. 1–14, 2020. *(prev. [39])*
`[SIM2REAL]`
> Demonstrates domain randomization for agile drone navigation. Randomizes textures, lighting, and camera parameters. Relevant as context for why this study deliberately does not use domain randomization: without validated sensor models, randomization cannot guarantee that real hardware falls within the training distribution ([55] Muratore). Cite in RESEARCH_PLAN §3.4 as an example of the domain randomization approach this study positions against.

---

## I. SLAM Backends and Trajectory Evaluation (archived context)

These references are retained for historical traceability and potential future extensions. They are not part of the active project objective.

**[66]** W. Xu, Y. Cai, D. He, J. Lin, and F. Zhang, "FAST-LIO2: Fast Direct LiDAR-Inertial Odometry," *IEEE Transactions on Robotics*, vol. 38, no. 4, pp. 2053–2073, 2022. DOI: 10.1109/TRO.2022.3141876 *(prev. [22])*
`[SLAM]`
> IESKF-based tight-coupled LiDAR-IMU odometry optimized for solid-state LiDARs including Livox. Primary 3D LiDAR backend for Session C. Frozen configuration tuned on real data is used identically across real hardware, standard simulation (M1), and metrological simulation (M4) - ensuring that differences in trajectory estimate reflect only the noise model, not estimator tuning.

---

**[67]** K. Koide, M. Yokozuka, S. Oishi, and A. Banno, "Globally Consistent and Tightly Coupled 3D LiDAR Inertial Mapping," in *Proc. IEEE ICRA*, 2022. *(prev. [23])*
`[SLAM]`
> Factor graph-based LiDAR-IMU SLAM with GPU acceleration and tight coupling; supports multiple sensor modalities. **Cross-modal baseline**: the only backend run in all four sessions (A–D), enabling consistent cross-modal comparison of noise model equivalence. Cite as the unifying baseline when discussing session-to-session TOST results.

---

**[68]** C. Campos, R. Elvira, J. J. Gómez Rodríguez, J. M. M. Montiel, and J. D. Tardós, "ORB-SLAM3: An Accurate Open-Source Library for Visual, Visual-Inertial and Multi-Map SLAM," *IEEE Transactions on Robotics*, vol. 37, no. 6, pp. 1874–1890, 2021. DOI: 10.1109/TRO.2021.3075644 *(prev. [24])*
`[SLAM]`
> Feature-based visual-inertial SLAM using ORB features and tight IMU-camera coupling. Primary visual-inertial backend for Session D. Provides a feature-based counterpart to GLIM's direct method, strengthening the argument that TOST conclusions are not artefacts of a single estimator architecture.

---

**[69]** R. Mur-Artal and J. D. Tardós, "ORB-SLAM2: An Open-Source SLAM System for Monocular, Stereo, and RGB-D Cameras," *IEEE Transactions on Robotics*, vol. 33, no. 5, pp. 1255–1262, 2017. *(prev. [8])*
`[SLAM]`
> Standard reference for the ORB-SLAM family that precedes [68]. Cite when describing ORB-SLAM3 lineage or when comparing against Session D results in the context of camera-only (no IMU) baselines.

---

**[70]** W. Hess, D. Kohler, H. Rapp, and D. Andor, "Real-Time Loop Closure in 2D LIDAR SLAM," in *Proc. IEEE ICRA*, 2016. DOI: 10.1109/ICRA.2016.7487258 *(prev. [25])*
`[SLAM]`
> Graph-based 2D SLAM with real-time loop closure (Cartographer). One of two 2D LiDAR backends for Session B. Provides loop-closure-enabled baseline against KISS-ICP's loop-closure-free odometry.

---

**[71]** I. Vizzo, T. Guadagnino, B. Mersch, L. Wiesmann, J. Behley, and C. Stachniss, "KISS-ICP: In Defense of Point-to-Point ICP - Simple, Accurate, and Robust Registration If Done the Right Way," *IEEE Robotics and Automation Letters*, vol. 8, no. 2, pp. 1029–1036, 2023. *(prev. [26])*
`[SLAM]`
> Point-to-point ICP-based LiDAR odometry without loop closure. Session B 2D LiDAR backend. Its simplicity makes it a clean baseline: performance differences between real and simulated noise models should be detectable with minimal estimator complexity obscuring them.

---

**[72]** D. He, W. Xu, N. Chen, F. Kong, C. Yuan, and F. Zhang, "Point-LIO: Robust High-Bandwidth LiDAR-Inertial Odometry," *Advanced Intelligent Systems*, vol. 5, no. 7, 2023. DOI: 10.1002/aisy.202200459. URL: [Publisher page](https://onlinelibrary.wiley.com/doi/10.1002/aisy.202200459) *(prev. [27])*
`[SLAM]`
> Dense point-based LiDAR-inertial odometry using stochastic process models rather than feature extraction. Third 3D LiDAR backend for Session C. Complements FAST-LIO2 and GLIM with a distinct noise modeling architecture.

---

**[73]** P. Geneva, K. Eckenhoff, W. Lee, Y. Yang, and G. Huang, "OpenVINS: A Research Platform for Visual-Inertial Estimation," in *Proc. IEEE ICRA*, 2020. *(prev. [28])*
`[SLAM]`
> EKF-based visual-inertial estimator using MSCKF-style feature marginalization. Session D backend. Provides an EKF-based counterpart to ORB-SLAM3's keyframe bundle adjustment and GLIM's factor graph, ensuring the TOST conclusions for Session D hold across different VIO architectures.

---

**[74]** M. Labbé and F. Michaud, "RTAB-Map as an Open-Source Lidar and Visual SLAM Library for Large-Scale and Long-Term Online Operation," *Journal of Field Robotics*, vol. 36, no. 3, pp. 416–446, 2018. DOI: 10.1002/rob.21831. URL: [Publisher page](https://onlinelibrary.wiley.com/doi/10.1002/rob.21831) *(prev. [30])*
`[SLAM]`
> Loose-coupled 2D/3D LiDAR, visual, and RGB-D SLAM. Optional baseline providing a loose-coupled comparison against tight-coupled backends across sessions, testing whether TOST conclusions differ by coupling strategy.

---

**[75]** M. Grupp, "evo: Python package for the evaluation of odometry and SLAM," GitHub, 2017. *(prev. [29])*
`[SLAM]`
> Standard trajectory evaluation tool computing ATE and RPE with Umeyama alignment. The unified evaluation tool for all SLAM backends across all three conditions (real, standard sim, metrological sim). Cite when describing trajectory metric computation in METHODOLOGY §2.2–2.3 (SLAM extension).

---

**[76]** S. Umeyama, "Least-squares estimation of transformation parameters between two point patterns," *IEEE Transactions on Pattern Analysis and Machine Intelligence*, vol. 13, no. 4, pp. 376–380, 1991. *(prev. [15])*
`[SLAM]`
> Defines the least-squares rigid-body alignment (Umeyama alignment) used by `evo` for trajectory comparison. Mandatory citation when reporting ATE/RPE in the SLAM extension (METHODOLOGY §7).

---

**[77]** M. Burri, J. Nikolic, P. Gohl, T. Schneider, J. Rehder, S. Omari, M. W. Achtelik, and R. Siegwart, "The EuRoC micro aerial vehicle datasets," *The International Journal of Robotics Research*, vol. 35, no. 10, pp. 1157–1163, 2016. *(prev. [33])*
`[SLAM]` `[METROLOGY]`
> Standard MAV dataset with high-quality synchronized sensing and external trajectory references. Use as benchmark context for VIO/SLAM evaluation quality; do not claim it as direct evidence of pre-session AVAR protocol requirements unless explicitly documented.

---

**[78]** D. Schubert, T. Goll, N. Demmel, V. Usenko, J. Stückler, and D. Cremers, "The TUM VI Benchmark for Evaluating Visual-Inertial Odometry," in *Proc. IEEE/RSJ IROS*, 2018. *(prev. [34])*
`[SLAM]`
> Visual-inertial benchmark with carefully engineered synchronization and external reference trajectories. Use as benchmark-design context; avoid asserting a specific pre-session AVAR workflow unless directly evidenced in the primary source.

---

**[79]** M. Helmberger, K. Morin, B. Berner, N. Kumar, G. Chli, and P. Furgale, "The Hilti SLAM Challenge Dataset," *IEEE Robotics and Automation Letters*, vol. 7, no. 3, pp. 7518–7525, 2022. *(prev. [35])*
`[SLAM]`
> Construction-site SLAM dataset using a laser tracker as the primary ground truth reference. Demonstrates industrial-grade ground truth methodology in a SLAM context. Cite as context for high-accuracy ground truth approaches when positioning the YuMi as a controlled-environment alternative.

---

**[80]** M. Ramezani, Y. Wang, M. Camurri, D. Wisth, M. Mattamala, and M. Fallon, "The Newer College Dataset: Handheld LiDAR, Inertial and Vision with Ground Truth," in *Proc. IEEE/RSJ IROS*, 2020. *(prev. [36])*
`[SLAM]`
> Large-scale outdoor LiDAR-visual dataset with survey-grade ICP map as ground truth. Provides context for the study's deliberate choice of controlled indoor conditions (YuMi) over uncontrolled outdoor environments - the latter are unsuitable for parametric noise model characterization due to uncontrolled variation in kinematic state and environment.

---

## J. Simulation Plugins

*(See also [25] Vultaggio et al. 2023, which built the primary existing Mid-360 plugin for Gazebo Classic.)*

**[81]** gazebosim/gz-sim, "Custom lidar sensor - Feature Request," GitHub issue #1958, opened April 2023. *(prev. [47])*
`[PLUGIN]`
> Community request for a custom non-repetitive LiDAR in Gazebo Fortress (gz-sim), open since April 2023 without resolution as of the study design date. **Direct documentation of the Fortress plugin gap**: confirms that even three years after ROS 2 Humble's release, no standard gz-sim solution for the Mid-360 Rosetta pattern exists. Cite in README §7 and RESEARCH_PLAN §6.3.

---

**[82]** RobotecAI, *RGLGazeboPlugin*, GitHub repository. [Repository](https://github.com/RobotecAI/RGLGazeboPlugin)
`[PLUGIN]`
> The only existing gz-sim plugin with Mid-360 Rosetta scan pattern support. Targets Gazebo Harmonic (not Fortress), requires NVIDIA OptiX/CUDA for ray tracing, and does not inject parametric state-dependent noise. Cite when explaining why the Fortress plugin gap remains open despite partial solutions: it is inaccessible to CPU-only hardware and incompatible with the ROS 2 Humble LTS stack.

---

## K. Standards and Textbooks

**[83]** IEEE Std 1293-2018, *IEEE Standard Specification Format Guide and Test Procedure for Linear Single-Axis, Nongyroscopic Accelerometers*, IEEE, 2018.
`[METROLOGY]` `[IMU]`
> Defines the Six-Position Test procedure for characterizing scale factor errors and gravitational bias per accelerometer axis. Referenced in EXPERIMENTAL_DESIGN §2.1 as the required static characterization procedure for all three IMUs before dynamic sessions begin.

---

**[84]** IEEE Std 952-2020, *IEEE Standard Specification Format Guide and Test Procedure for Single-Axis Interferometric Fiber Optic Gyros*, IEEE, 2020. (Applied to MEMS gyroscopes by extension; see El-Sheimy [9].)
`[METROLOGY]` `[IMU]`
> Provides a formal gyro test-specification reference in the interferometric-fiber-optic domain. For this project (MEMS IMUs), treat it as indirect standards context and pair it with MEMS-focused AVAR literature before deriving protocol-level requirements.

---

**[85]** ISO 9283:1998 (corrected 1999), *Manipulating industrial robots - Performance criteria and related test methods*, International Organization for Standardization.
`[METROLOGY]` `[ROBOT-GT]`
> Defines the pose repeatability (RP), path repeatability (RT), and path accuracy (AT) test procedures for industrial manipulators. The YuMi specifications (RP = ±0.02 mm, RT = 0.10 mm, AT ≤ 1.36 mm) are measured under ISO 9283 conditions. Cite when reporting ground truth uncertainty bounds in METHODOLOGY §2.3 alongside [43] (ABB product spec).

---

**[86]** JCGM 100:2008 (GUM), *Evaluation of measurement data - Guide to the expression of uncertainty in measurement*, Bureau International des Poids et Mesures, 2008.
`[METROLOGY]`
> The international standard for uncertainty expression in measurement. Defines Type A (statistical) and Type B (calibration/datasheet) uncertainty classifications and the combined expanded uncertainty U = k·u_c with k = 2 (~95% coverage). The METHODOLOGY §2.3 uncertainty budget must be restructured to comply with GUM before manuscript submission - a P1 item in `internal/PENDING_2026-03-27.md`.

---

**[87]** ISO 5725-3:2023, *Accuracy (trueness and precision) of measurement methods and results - Part 3: Intermediate precision of a standard measurement method*, International Organization for Standardization. (Supersedes ISO 5725-3:1994.)
`[METROLOGY]`
> Defines vocabulary for repeatability (within-session) and intermediate precision (cross-session, same laboratory). The study's experimental design measures within-session repeatability (each block) and cross-session intermediate precision (sessions A–D). Declaring this scope explicitly using ISO 5725-3 terminology in METHODOLOGY §3.1 and §3.5 is a P1 item in `internal/PENDING_2026-03-27.md`.

---

*End of consolidated references. Total: 88 entries.*

*Last updated: 2026-03-31. Supersedes: RESEARCH_PLAN.md §9, REFERENCES_POOL.md, EXTENDED_LITERATURE_REVIEW.md.*
