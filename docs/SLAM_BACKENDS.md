# SLAM Backends and Noise Models

**Version:** 0.2  
**Date:** 2026-03-08  
**Status:** Design — subject to supervisor review and implementation details

---

## 1. Goals

This document defines the SLAM backends and sensor noise models that will be used in Phase I. The objective is to evaluate sensor model fidelity **independently of a particular SLAM implementation** by:

- Using **multiple tight-coupled SLAM systems** per sensing modality, with clearly different internal architectures.
- Keeping SLAM parameters **frozen across real and simulated data** once a baseline is established on real data.
- Comparing several **noise model configurations** for each sensor (manufacturer, static Allan, in-session static, and kinematic residuals).

GLIM is used as a **cross-modal baseline** wherever feasible, complemented by two well-established ROS/ROS 2 SLAM systems per modality. RTAB-Map is included as an optional loose-coupled baseline.

**Selection rationale (per modality):** Each backend is chosen so that conclusions about sensor model fidelity do not depend on a single architecture. We combine (i) one or two **established** backends (different paradigms: EKF vs factor graph vs dense, feature vs direct) with (ii) **GLIM** as a common cross-modal reference and (iii) **RTAB-Map** as the only loose-coupled baseline to contrast tight vs loose. Justification per backend is given in the tables below and in §7 (extension: repetitive LiDAR).
Using multiple backends also strengthens the TOST equivalence argument: if metrological simulation is declared equivalent to real hardware across architecturally distinct estimators simultaneously, the conclusion cannot be attributed to a single algorithm's tolerance to noise model mismatch.

---

## 2. Noise Model Configurations

Noise model definitions (M1–M4) and the kinematic regime taxonomy (S0–S3) are defined in **[METHODOLOGY.md §4 and §5](METHODOLOGY.md)**. M1–M3 are time-invariant; M4 is state-dependent on **measured temperature $T$** and kinematic state. This document only lists which SLAM backends are run per session; for formulas and coefficient pruning see METHODOLOGY.

---

## 3. Dataset Taxonomy

Phase I uses three dataset families:

- **D_static** — Long-duration static logs in laboratory conditions (IMUs, LiDARs, camera). Used for M2.
- **D_dynamic_yumi** — Dynamic sessions A/B/C/D on the ABB YuMi with ground truth from RobotStudio. Used for all dynamic evaluations and for M3/M4 extraction.
- **D_public (optional)** — Public datasets (e.g., KITTI, TartanAir) used only for sanity checks of the proposed noise models outside the YuMi setup.

---

## 4. SLAM Backends by Modality

### 4.1 IMU-Only (Session A)

| ID | Backend | Type | Justification |
|----|---------|------|---------------|
| A1 | IMU Integration (preintegration module) | Inertial-only | Same preintegration model as GLIM/FAST-LIO2 (no external observations). **Why:** Provides a lower bound on drift under M1–M4 and isolates IMU quality effect (H1) without LiDAR/camera; architecture already used in tight-coupled backends so parameters are comparable. |

The aim is not to benchmark INS algorithms but to quantify how IMU noise models (M1–M4) affect pure inertial drift under the T1/T2/T3 trajectories.

### 4.2 2D LiDAR (Session B, RPLiDAR A2M12)

| ID | Backend | Coupling | ROS2 | Justification |
|----|---------|----------|------|---------------|
| C1 | **Cartographer 2D** | Tight SLAM (scan + pose graph) | Yes (cartographer_ros2) | **Why:** De facto standard for 2D LiDAR; submap + pose graph gives loop closure and map consistency. Represents the **graph-based SLAM** paradigm — optimizes over submaps and loop closures. Highly sensitive to scan consistency; sensor model fidelity directly affects graph quality. |
| C2 | **KISS-ICP 2D** | Odom only (ICP) | Yes | **Why:** Pure **incremental odometry** — no loop closure, no graph. Frame-to-frame ICP only. Maximally sensitive to scan noise and motion blur; if M4 improves consistency here, the effect is attributable to better scan modelling rather than graph optimization. Complements Cartographer (graph) and GLIM (factor graph). |
| C3 | **GLIM (experimental)** | Tight (LiDAR) | Planned | **Why:** Cross-modal baseline; if 2D scans can be injected as planar 3D, enables **factor-graph smoothing** on the same pipeline as Session C. Optional; only if feasible without diverting effort from core sessions. |

**Paradigm coverage (2D LiDAR):** With Cartographer 2D + KISS-ICP 2D + GLIM we cover three distinct paradigms: **graph-based SLAM** (submap + pose graph), **incremental odometry** (frame-to-frame ICP), and **factor-graph smoothing** (GPU-accelerated), so that conclusions are not backend-specific.

RTAB-Map 2D (loose-coupled) may be added as an optional baseline if time permits; **rationale:** contrast tight vs loose on the same 2D data.

### 4.3 3D LiDAR Solid-State (Session C, Livox Mid-360)

| ID | Backend | Coupling | ROS2 | Justification |
|----|---------|----------|------|---------------|
| D1 | **FAST-LIO2** | Tight (IESKF, LiDAR+IMU) | Yes | **Why:** Established LiDAR-inertial baseline (commonly used in LIO benchmarks). **IESKF (iterated extended Kalman filter)** — the classical **filter-based** paradigm for LIO. Native Livox (incl. Mid-360) support. Ensures conclusions hold for the dominant family of LIO algorithms in industry and research. |
| D2 | **Point-LIO** | Tight (dense points, no features) | Yes | **Why:** **Dense point-based** — no explicit feature extraction (edges, planes). Evaluates how the Mid-360 non-repetitive (Rosetta) pattern affects registration when the algorithm relies on raw point clouds. Architecturally distinct from filter (FAST-LIO2) and factor graph (GLIM); complements both. |
| D3 | **GLIM** | Tight (factor graph, LiDAR+IMU) | Yes | **Why:** **Factor-graph smoothing** with GPU-accelerated scan-matching factors. Native Mid-360 support. Represents the modern non-filter family; same pipeline used in Session D (visual-inertial), so sensor model conclusions are comparable across modalities. |

**Paradigm coverage (3D LiDAR):** With FAST-LIO2 + Point-LIO + GLIM we cover three distinct paradigms: **filter-based LIO** (IESKF), **dense point-based** (no features), and **factor-graph smoothing** (GPU), so that conclusions are not backend-specific.

LIO-SAM is **not** included in Phase I because it is optimized for **repetitive-pattern** (spinning) LiDARs; the Mid-360 has a non-repetitive Rosetta pattern and LIO-SAM has limited ROS 2 support for Livox. See **§7** for backends to consider if a 360° spinning LiDAR becomes available.

RTAB-Map 3D (loose-coupled) can be used in a reduced subset of experiments to contrast tight vs. loose coupling.

### 4.4 Visual / Visual-Inertial (Session D, RealSense D455)

| ID | Backend | Coupling | ROS2 | Justification |
|----|---------|----------|------|---------------|
| B1 | **ORB-SLAM3** | Tight visual / visual-inertial | Community ports | **Why:** De facto standard feature-based RGB-D/VIO. **Keyframe bundle adjustment** — optimizes over a pose graph with keyframes and IMU preintegration. Native RGB-D support. Sensitive to motion blur and texture; good stress test for M4. Ensures results are comparable to the most cited visual SLAM. |
| B2 | **GLIM** | Tight visual-inertial | Yes | **Why:** **Factor-graph smoothing** with GPU-accelerated visual factors. Same cross-modal baseline as Sessions C/D; allows comparing sensor model impact across visual vs LiDAR with one consistent backend. |
| B3 | **OpenVINS** | Tight visual-inertial (EKF) | ROS/ROS2 | **Why:** **EKF-based VIO** (MSCKF-style) — recursive filter, not keyframe BA. Architecturally distinct from ORB-SLAM3 (graph) and GLIM (factor graph). Widely cited research platform; tight-coupled; ROS 2 compatible. |
| — | **R3LIVE** | Tight visual-LiDAR-IMU | ROS/ROS2 forks | **Excluded from this study.** Ideally one would run a camera+LiDAR fusion test; however, Session D uses RGB-D+IMU only (no LiDAR on arm). R3LIVE expects simultaneous LiDAR+camera and has limited ROS 2 support for this configuration. Reserved for future extension if a multi-sensor session is added. |

**Paradigm coverage (Visual / VIO):** With ORB-SLAM3 + OpenVINS + GLIM we cover three distinct paradigms: **keyframe bundle adjustment** (ORB-SLAM3), **EKF VIO** (OpenVINS), and **factor-graph smoothing** (GLIM), so that conclusions are not backend-specific.

RTAB-Map RGB-D (loose-coupled) may be run on selected trajectories; **rationale:** only loose-coupled baseline for Session D, contrasts with B1/B2/B3.

---

## 5. SLAM Configuration and Sensitivity Analysis

For each SLAM backend, three configuration regimes will be considered:

| ID | Description |
|----|-------------|
| CFG_LOW | Conservative noise parameters (e.g., 0.5× nominal covariances). |
| CFG_NOM | Nominal configuration, tuned once on real data with noise model M2 or M3 and then **frozen**. |
| CFG_HIGH | Aggressive noise parameters (e.g., 2× nominal covariances). |

The SLAM backends are first tuned on real YuMi data using M2 or M3 to obtain CFG_NOM. CFG_LOW and CFG_HIGH are derived by scaling key covariance terms. All three configurations are then reused unchanged for all simulation conditions (M1–M4) to assess whether conclusions about sensor model fidelity are robust to reasonable parameter variation.

---

## 6. Evaluation Matrix (Conceptual)

For each session (A–D), trajectory type (T1–T3) and SLAM backend, the following combinations will be evaluated:

- Real data (D_dynamic_yumi) with CFG_LOW / CFG_NOM / CFG_HIGH.
- Simulated data with noise models M1–M4, each with CFG_LOW / CFG_NOM / CFG_HIGH.

This yields, for each sensor/SLAM pair, families of ATE/RPE distributions that can be compared:

- Real vs. Sim(M1) — standard vs. reality.
- Real vs. Sim(M2/M3) — impact of static Allan-based models.
- Real vs. Sim(M4) — impact of kinematic-residual modelling.

The expectation is that M4 will produce ATE/RPE distributions closest to real hardware across multiple backends and configurations. The formal validation is not a comparison of means but a TOST equivalence test: metrological simulation is declared equivalent to real hardware only if the 90% confidence interval for the mean ATE difference lies entirely within the pre-specified margin δ — applied independently per backend and per trajectory type. Standard simulation (M1) is expected to fail this test, confirming that metrological modeling is necessary. See METHODOLOGY.md §3.4 for the full TOST protocol.

---

## 7. Extension: Repetitive (Spinning) 360° LiDAR

Phase I uses a **solid-state** 3D LiDAR (Livox Mid-360, non-repetitive Rosetta pattern). If access to a **mechanical spinning 360° LiDAR** (e.g. Velodyne, Ouster, Hesai) becomes available in a follow-up or extended study, the following backends are relevant. They are optimized for repetitive scan patterns and are **not** in the current Phase I scope.

| Backend | Type | Tight-coupled method | ROS / ROS 2 | Why consider (if 360° spinning available) |
|---------|------|------------------------|-------------|------------------------------------------|
| **LIO-SAM** | Tight (LiDAR+IMU) | Factor graph (keyframe + scan-to-map, IMU preintegration) | ROS 1; ROS 2 ports community | Commonly used for spinning LiDAR; scan-to-submap + loop closure. Established baseline for repetitive-pattern LIO. |
| **FAST-LIO2** | Tight (LiDAR+IMU) | IESKF, same as current Session C | Yes | Already in Phase I for Mid-360; works equally well with spinning LiDAR. No change of backend — only sensor. |
| **LIO-SAM2** | Tight (LiDAR+IMU) | Evolution of LIO-SAM (e.g. ikd-tree, better efficiency) | ROS 1 / ROS 2 | Improved robustness and efficiency; good candidate if LIO-SAM is used. |
| **Point-LIO** | Tight (dense, no features) | Dense point-based, same as Session C | Yes | Same as Phase I; applicable to spinning LiDAR. Complements feature/graph-based backends. |
| **LeGO-LOAM** | Tight (LiDAR only or + IMU in forks) | Feature-based (edge/plane) + pose graph | ROS 1 | Lightweight; feature-based rather than dense. Contrast with FAST-LIO2/Point-LIO. |
| **GLIM** | Tight (LiDAR+IMU) | Factor graph, cross-modal | Yes | Native support for several spinning LiDARs; keeps cross-modal baseline consistent. |
| **RTAB-Map 3D** | Loose | ICP / feature-based odom + loop closure | Yes | Same loose-coupled baseline as in Phase I; contrast tight vs loose for spinning data. |

**Note:** Exact sensor support (Velodyne VLP-16/32, Ouster OS0/OS1, Hesai, etc.) and ROS 2 maturity should be checked at the time of the extension. This table is a planning reference only; Phase I does not depend on it.

