# Hardware & Payload Analysis — Phase I

**Version:** 0.3  
**Date:** 2026-03-06  
**Status:** Weight figures are estimates — **must be verified with physical scale before scheduling YuMi arm time**

---

## Table of Contents

1. [Ground Truth System: ABB YuMi IRC5](#1-ground-truth-system-abb-yumi-irc5)
2. [YuMi Payload Constraint Analysis](#2-yumi-payload-constraint-analysis)
3. [Sensor Specifications](#3-sensor-specifications)
4. [Session Payload Breakdown](#4-session-payload-breakdown)
5. [LiDAR Selection: Mid-360 vs. Mid-70](#5-lidar-selection-mid-360-vs-mid-70)
6. [Mitsubishi Robot — Backup Option](#6-mitsubishi-robot--backup-option)
7. [Support Hardware and Cables](#7-support-hardware-and-cables)
8. [Weight Verification Checklist](#8-weight-verification-checklist)

---

## 1. Ground Truth System: ABB YuMi IRC5

| Parameter | Value |
|-----------|-------|
| Type | Dual-arm collaborative robot (7-DOF per arm) |
| Pose repeatability (RP) | ±0.02 mm (static points only) |
| Path repeatability (RT) | 0.10 mm (dynamic trajectory comparison) |
| Path accuracy (AT) | up to 1.36 mm (worst-case; lower at our speeds) |
| Payload capacity | ~500 g per arm |
| Programming | RobotStudio (offline trajectory + playback) |
| Controller | IRC5 |
| ROS interface | `abb_robot_driver` (ROS 2) |

**Important:** For *dynamic* trajectory comparison (ATE/RPE), the mechanical floor is path repeatability (0.10 mm) and path accuracy (up to 1.36 mm). Pose repeatability (±0.02 mm) applies only to static points. See RESEARCH_PLAN References [16] and [METHODOLOGY.md §2.4](METHODOLOGY.md#24-ground-truth-uncertainty-budget).

**Trajectory reference source:** RobotStudio exports the executed joint trajectory. Forward kinematics applied to extract end-effector pose at each timestamp. This is the ground truth trajectory against which all sensor estimates are compared.

**Uncertainty budget:** The combined uncertainty of the ground truth (YuMi repeatability, hand-eye calibration, temporal synchronization) is documented in [METHODOLOGY.md §2.4 Ground Truth Uncertainty Budget](METHODOLOGY.md#24-ground-truth-uncertainty-budget). ATE/RPE are interpreted above that floor.

---

## 2. YuMi Payload Constraint Analysis

The YuMi's ~500 g payload capacity per arm is the primary physical constraint on experimental design. This limits which sensors can be mounted simultaneously and in which configurations.

**Design rule:** All Phase I sessions are designed as **single-sensor sessions** (one primary sensor per session). No multi-sensor simultaneous mounting. This maximizes payload margin and eliminates mechanical coupling between sensors.

**Safety margin target:** Keep each session payload at **≤ 70% of capacity** (≤ 350 g) to allow for support structure uncertainty and to avoid introducing structural vibrations that would contaminate the ground truth at dynamic trajectories.

Sessions A–D follow modality order: IMU-only → 2D LiDAR → 3D LiDAR → visual-inertial.

| Session | Estimated payload | % of capacity | Margin |
|---------|-----------------|---------------|--------|
| A (WitMotion IMU) | ~100 g | 20% | Comfortable |
| B (RPLiDAR A2M12) | ~250 g | 50% | Safe |
| C (Livox Mid-360) | ~350 g | 70% | At margin — verify exact weight |
| D (RealSense D455) | ~275 g | 55% | Safe |

> **Action required:** Weigh all sensors + mounts on a physical scale before finalizing Session C scheduling. If the Mid-360 assembly exceeds 400 g, redesign mount to reduce weight.

---

## 3. Sensor Specifications

### 3.1 WitMotion WT901C-TTI

| Parameter | Value |
|-----------|-------|
| IMU chip | InvenSense MPU9250 |
| Axes | 9-DOF (3-axis gyro, accel, magnetometer) |
| Gyroscope rate | 200 Hz |
| Gyroscope range | ±2000°/s |
| Estimated weight (sensor only) | ~50 g |
| Estimated mount weight | ~50 g |
| **Total estimated** | **~100 g** |
| Quality grade | Mid-high (literature-characterized chip) |
| Role | **Reference IMU of the study** |
| Extrinsic calibration | Manual (no factory IMU-camera or IMU-LiDAR calibration) |

**Notes:** The MPU9250 is one of the most widely characterized MEMS IMUs in the literature (e.g. Allan Variance studies in RotorS/Furrer et al., 2016). Its coefficients from independent studies will serve as a sanity check for our characterization. This is the IMU we expect to produce the most accurate tight-coupling results.

---

### 3.2 Intel RealSense D455

| Parameter | Value |
|-----------|-------|
| Type | RGB-D camera (stereo IR + RGB) |
| IMU chip | Bosch BMI055 (6-DOF: 3-axis gyro + accel) |
| IMU rate | Up to 400 Hz |
| Depth range | 0.6–6 m |
| RGB resolution | Up to 1280×800 @ 30 fps |
| Estimated weight (sensor only) | ~195 g |
| Estimated mount weight | ~80 g |
| **Total estimated** | **~275 g** |
| Quality grade | Low-mid (BMI055 exhibits higher ARW/bias instability than MPU9250 or ICM40609 in typical characterizations; see manufacturer datasheets and independent studies) |
| Extrinsic calibration | Factory-calibrated camera-IMU (D4XX series) |

**Notes:** The BMI055 is not suitable for standalone inertial navigation at the quality level of the MPU9250 or ICM40609. However, its factory-calibrated camera-IMU extrinsics make it valuable for **visual-inertial tight coupling** without additional calibration. The RealSense session (D) is expected to show the largest Reality Gap due to visual SLAM's dependence on texture and illumination, which standard Gazebo cannot replicate faithfully.

---

### 3.3 Livox Mid-360

| Parameter | Value |
|-----------|-------|
| Type | Solid-state 3D LiDAR |
| Scanning pattern | Non-repetitive "Rosetta" (360° horizontal, ~59° vertical FOV) |
| Range | Up to 40 m (typical indoor: 0.5–20 m) |
| Point rate | ~200,000 pts/s |
| IMU chip | TDK InvenSense ICM40609 |
| IMU rate | 200 Hz |
| LiDAR-IMU extrinsics | **Factory-calibrated by Livox** |
| Estimated weight (sensor only) | ~250 g |
| Estimated mount weight | ~100 g |
| **Total estimated** | **~350 g** |
| Quality grade | Mid (ICM40609 better than BMI055, below MPU9250) |

**Notes:** The Mid-360 is the primary 3D LiDAR for this study. Its factory-calibrated LiDAR-IMU extrinsics eliminate the need for additional extrinsic calibration when using tight-coupled backends (FAST-LIO2, GLIM). GLIM natively supports the Mid-360. The Rosetta scanning pattern provides full spherical coverage over time — a main advantage over the Mid-70's limited FOV and the A2M12's planar-only coverage.

---

### 3.4 RPLiDAR A2M12

| Parameter | Value |
|-----------|-------|
| Type | Mechanical 2D LiDAR (360° planar) |
| Scan rate | 10 Hz |
| Range | Up to 12 m |
| Angular resolution | ~0.9° per sample |
| IMU | None |
| Estimated weight (sensor only) | ~170 g |
| Estimated mount weight | ~80 g |
| **Total estimated** | **~250 g** |
| Quality grade | Consumer-grade 2D LiDAR |

**Notes:** Session B is 2D-only (Z fixed). The A2M12 provides a planar SLAM baseline established in the literature and commonly used in 2D benchmarks, with no geometric degeneracy risk in properly structured environments. It serves as the minimal-complexity comparison baseline before evaluating the 3D sensors.

---

## 4. Session Payload Breakdown

**Weight verification status:** Not yet verified — estimates only

| Session | Sensor | Mount | Total est. | Verified | Notes |
|---------|--------|-------|------------|----------|-------|
| A | 50 g | 50 g | **100 g** | No | Simplest mount; low risk |
| B | 170 g | 80 g | **250 g** | No | 2D only; Z constraint critical |
| C | 250 g | 100 g | **350 g** | No | At 70% capacity; priority to verify |
| D | 195 g | 80 g | **275 g** | No | Camera mount needs cable management |

**Cable weight:** Not included in estimates above. USB 3.0 + power cables for each sensor add ~20–50 g. Include in final verified weight.

---

## 5. LiDAR Selection: Mid-360 vs. Mid-70

The original proposal (Abella, Feb 2026) used the Livox Mid-70. The recommendation was changed to the **Mid-360** for the following reasons:

| Criterion | Mid-70 | Mid-360 | Winner |
|-----------|--------|---------|--------|
| Weight | ~590 g | ~250 g | Mid-360 |
| YuMi compatibility | Exceeds capacity even without mount | Within capacity with margin | Mid-360 |
| FOV | 70°×77° (limited) | 360° horizontal × 59° vertical | Mid-360 |
| Geometric degeneracy risk | High (limited FOV; Liu & Zhang, 2021) | Significantly lower | Mid-360 |
| Rosetta pattern | Yes | Yes (improved coverage) | Tie |
| Factory IMU-LiDAR extrinsics | No | Yes (ICM40609 calibrated) | Mid-360 |
| GLIM native support | Partial | Full | Mid-360 |
| Cost (indicative) | Similar | Similar | Tie |
| Community adoption (mobile robotics) | Lower | High (LIO-SAM, FAST-LIO2, GLIM) | Mid-360 |

**Conclusion:** The Mid-360 is unambiguously the correct choice for this study. The Mid-70 would have been physically incompatible with the YuMi and significantly weaker scientifically. The switch from Mid-70 to Mid-360 should be explicitly documented if mentioned in any publication.

---

## 6. Mitsubishi Robot — Backup Option

The IQS flexible manufacturing cell includes a Mitsubishi robot that could serve as an alternative ground truth if the YuMi is not viable for a specific session.

**Current status:** Reserved as fallback only. Obstacles to use:
- Requires PLC programming
- Fixed installation in the flexible cell — may not be relocatable
- Requires formal permission from the lab responsible
- Unknown payload and repeatability specifications

**When to invoke the Mitsubishi option:** If the YuMi payload is definitively exceeded for a specific session (most likely Session C if Mid-360 + mount > 500 g), request access to the Mitsubishi as fallback. Document the different ground truth source in the experimental record.

---

## 7. Support Hardware and Cables

| Item | Purpose | Notes |
|------|---------|-------|
| Custom 3D-printed mounts | Attach sensors to YuMi flange | Require CAD design + printing per sensor |
| M6 hex bolts + locknuts | Secure mount to flange | Use thread-locking compound for dynamic sessions |
| USB 3.0 cables (×2) | RealSense + WitMotion | Flexible, low-weight; route along arm |
| RJ45 Ethernet cable | Livox Mid-360 (Ethernet-based) | Requires adapter if robot arm cable routing is USB-only |
| USB-C → USB-A adapter | RPLiDAR (USB-C sensor) | Verify compatibility with recording PC |
| Calibrated thermometer | Temperature logging | Digital, ±0.1°C resolution |
| LED panel + constant-current driver | Stable illumination for Session D (camera) | 30 min warm-up before session start |
| Anti-vibration mat | Static characterization surface | Decouples sensor from building vibration |

**Mount design note:** Mounts must be designed and printed **before** scheduling YuMi time. Include cable management to avoid cable tension affecting sensor pose relative to flange.

---

## 8. Weight Verification Checklist

Before scheduling any dynamic session, complete the following:

- [ ] Weigh WitMotion WT901C sensor body (without cables)
- [ ] Weigh WitMotion mount (printed + bolts)
- [ ] Weigh RealSense D455 sensor body (without cables)
- [ ] Weigh RealSense mount (printed + bolts)
- [ ] Weigh RPLiDAR A2M12 sensor body (without cables)
- [ ] Weigh RPLiDAR mount (printed + bolts)
- [ ] Weigh Livox Mid-360 sensor body (without cables)
- [ ] Weigh Mid-360 mount (printed + bolts)
- [ ] Weigh all relevant cables (USB 3.0, Ethernet) per session length
- [ ] Compute total per session with cable weight included
- [ ] Confirm all sessions ≤ 500 g (hard limit) and ≤ 350 g (safety target)
- [ ] Update `HARDWARE_PAYLOAD.md` Session table with verified weights
- [ ] Update `PROGRESS_LOG.md` with verification date and results
