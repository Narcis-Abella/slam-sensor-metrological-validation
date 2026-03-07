# Methodology — Phase I

**Version:** 0.3  
**Date:** 2026-03-06  
**Status:** Pending supervisor review

---

## Table of Contents

1. [Allan Variance Analysis](#1-allan-variance-analysis)
2. [Trajectory Evaluation Metrics](#2-trajectory-evaluation-metrics) — includes [Ground Truth Uncertainty Budget](#24-ground-truth-uncertainty-budget)
3. [Statistical Test Design](#3-statistical-test-design)
4. [Sensor Noise Models (M1–M4)](#4-sensor-noise-models-m1m4)
5. [Kinematic Residual Extraction](#5-kinematic-residual-extraction)
6. [Three-Condition Comparison Pipeline](#6-three-condition-comparison-pipeline)
7. [CW/CCW Asymmetry Analysis](#7-cwccw-asymmetry-analysis)
8. [Frequency-Sweep Trajectory (Optional)](#8-frequency-sweep-trajectory-optional)
9. [Analysis Toolchain Summary](#9-analysis-toolchain-summary)

---

## 1. Allan Variance Analysis

### 1.1 Background

Allan Variance (AVAR) and its square root, Allan Deviation (ADEV), are the standard IEEE tools for identifying stochastic noise processes in inertial sensors. The log-log ADEV curve reveals different noise types at different averaging times ($\tau$):

| Noise process | ADEV slope | Parameter extracted |
|---------------|-----------|---------------------|
| Angle Random Walk (ARW) | $-1/2$ | Read at $\tau = 1$ s on log-log plot [°/√h] |
| Bias Instability (BI) | 0 (minimum) | Read at minimum of curve [°/h] |
| Rate Random Walk (RRW) | $+1/2$ | Read at $\tau = 3$ h on log-log plot [°/h$^{3/2}$] |
| Rate Ramp | +1 | Long-term deterministic drift; not always present |

RRW identification requires averaging times of several hours. This is why 10–12 h static logs are necessary — shorter logs do not reach the $\tau$ range where the $+1/2$ slope is observable.

### 1.2 Procedure

1. Record static IMU log at ≥ 200 Hz for 10–12 h (see [EXPERIMENTAL_DESIGN.md §2.1](EXPERIMENTAL_DESIGN.md))
2. Compute ADEV using overlapping Allan Variance estimator (maximizes data use)
3. Fit slopes to identify noise process boundaries
4. Extract numerical coefficients at the specified averaging times
5. Compute confidence intervals (typically $\pm 1\sigma$, propagated from Chi-squared distribution on the ADEV estimator)

Tooling: `imu_utils` (ROS package) or equivalent Python implementation (e.g., `allan_variance_ros`, `allantools`)

### 1.3 Reporting Standard

All coefficients must be reported as:

```
ARW:  X.XX ± Y.YY °/√h    (read at τ = 1 s)
BI:   X.XX ± Y.YY °/h     (read at ADEV minimum)
RRW:  X.XX ± Y.YY °/h^(3/2)  (if identifiable; state "not identifiable" otherwise)
```

Units and confidence intervals are mandatory. Curves alone are insufficient for reproducibility.

### 1.4 Six-Position Test Reporting

Scale factor and gravitational bias per axis, reported as:

```
Scale factor error (X/Y/Z): X.XX ppm
Gravitational bias (X/Y/Z): X.XX mg
```

---

## 2. Trajectory Evaluation Metrics

### 2.1 Absolute Trajectory Error (ATE)

ATE measures the global consistency of an estimated trajectory against ground truth. It quantifies drift accumulated over the full trajectory.

Computation:
1. Align estimated trajectory with ground truth using Umeyama alignment (least-squares rigid body alignment; `evo_ape` with `--align` flag)
2. Compute Euclidean distance between aligned estimated poses and ground truth poses at corresponding timestamps
3. Report: RMSE, mean, median, max, and standard deviation of ATE [mm or m]

Tool: `evo_ape` from the `evo` package

```bash
evo_ape tum ground_truth.txt estimated.txt --align --plot --save_results results.zip
```

**Reference:** Umeyama (1991) — least-squares rigid body alignment. See [RESEARCH_PLAN.md](RESEARCH_PLAN.md) References [15].

### 2.2 Relative Pose Error (RPE)

RPE measures local consistency — the drift accumulated over a fixed time or distance window. It is less sensitive to global alignment issues and better captures frame-to-frame odometry quality.

Computation:
1. Define window size (e.g., Δt = 1 s or Δd = 0.1 m)
2. For each pair of poses separated by the window, compute the relative transform error
3. Report: RMSE, mean, median, max, std [mm/m and °/m or °/s]

**Tool:** `evo_rpe` from the `evo` package

```bash
evo_rpe tum ground_truth.txt estimated.txt --align --delta 1 --delta_unit s --plot
```

### 2.3 Ground Truth Format

YuMi RobotStudio trajectory export → convert to TUM format (timestamp tx ty tz qx qy qz qw) for `evo` compatibility.

All timestamps must be synchronized (see [EXPERIMENTAL_DESIGN.md §9](EXPERIMENTAL_DESIGN.md)) before metric computation.

### 2.4 Ground Truth Uncertainty Budget

The ground truth trajectory against which ATE/RPE are computed is not perfect. Its uncertainty must be quantified so that (i) reviewers can assess the validity of the comparison and (ii) we do not over-interpret differences smaller than the ground truth floor. All contributions below must be documented at the time of experimentation; the values given here are design targets or typical orders of magnitude.

| Source | Type | Magnitude (design / typical) | Notes |
|--------|------|------------------------------|--------|
| **YuMi repeatability** | Random (per pose) | ±0.02 mm | ABB specification; repeatability, not volumetric accuracy. Dominant mechanical term. See [HARDWARE_PAYLOAD.md §1](HARDWARE_PAYLOAD.md) and RESEARCH_PLAN References [16]. |
| **Hand-eye calibration** | Systematic (constant offset per session) | &lt; 0.5 mm translation, &lt; 0.5° rotation | AX=XB (Tsai-Lenz or equivalent); re-done per session. See [EXPERIMENTAL_DESIGN.md §10](EXPERIMENTAL_DESIGN.md). Translation error propagates as systematic bias in position comparison. |
| **Temporal synchronization** | Random / systematic (time offset) | \( \Delta x \approx v_{\max} \times \Delta t \) | At ~100 mm/s, 10 ms offset → ~1 mm spatial error. PTP (IEEE 1588) preferred; NTP fallback with jitter documented. The **measured** offset and jitter must be reported; \( \Delta t \) is the measured or conservative bound. See [EXPERIMENTAL_DESIGN.md §9](EXPERIMENTAL_DESIGN.md). |
| **Forward kinematics / timestamp interpolation** | Negligible if documented | Sub-mm for typical interpolation | RobotStudio export + FK; interpolation of poses to sensor timestamps. Assume negligible if timestamps are aligned and interpolation is linear or spline-based over short intervals. |

Combined uncertainty (position): Under the assumption of independent contributions, a conservative **combined standard uncertainty** for ground truth position can be taken as the root-sum-square of the relevant terms, or as a worst-case bound (e.g. 0.02 + 0.5 + \(v_{\max}\,\Delta t\) in mm if all are treated as half-widths). The exact combination (RSS vs worst-case) and the numerical values shall be fixed in the experimental report once sync and hand-eye are measured.

Interpretation of ATE/RPE: Metrics ATE and RPE are interpreted as being above the ground truth uncertainty floor. Differences between conditions (Real vs Sim M1/M4) that are of the same order as or smaller than this floor are not claimed as significant; the uncertainty budget is reported alongside ATE/RPE in any publication so that readers can judge. This budget does not replace the statistical tests (e.g. paired t-test, Wilcoxon) but clarifies the physical limit below which the comparison is not meaningful.

---

## 3. Statistical Test Design

### 3.1 Test Selection

The primary statistical comparison is between ATE distributions across the three conditions:
- Condition R: Real hardware
- **Condition S:** Standard Gazebo simulation
- Condition M: Metrological Gazebo simulation (with empirically derived parameters)

For each trajectory type (T1, T2, T3) and each session (A, B, C, D):

**Step 1 — Normality test:** Apply Shapiro-Wilk test on each ATE distribution (n = 9 per trajectory × 3 repetitions × 3 blocks).

Step 2 — Select test:
- If normal (p > 0.05 in Shapiro-Wilk for all three conditions): Paired t-test
- If non-normal (any condition fails normality): **Wilcoxon signed-rank test** (non-parametric equivalent)

Step 3 — Apply test:
- Compare Condition M vs. Condition R → **TOST (Two One-Sided Tests)** for equivalence — see §3.4.
- Compare Condition S vs. Condition R → **Standard NHST** (paired t-test or Wilcoxon): expect p < 0.05, confirming standard simulation is significantly inferior.

### 3.2 Significance Level

$\alpha =$ **0.05** (two-tailed for NHST; each one-sided test of TOST at $\alpha = 0.05$)

### 3.3 Effect Size

Report effect size alongside p-value:
- For parametric (t-test): Cohen's d
- For non-parametric (Wilcoxon): **rank-biserial correlation r**

### 3.4 Equivalence Testing for H0 (TOST)

**Why not p > 0.05?** A non-significant result (p > 0.05) does **not** demonstrate equivalence. "Absence of evidence is not evidence of absence." With limited sample size (n = 9), failure to reject the null may simply indicate insufficient statistical power to detect a real difference. Metrology-oriented journals (e.g. IEEE TIM, *Measurement*) expect equivalence to be demonstrated explicitly, not inferred from non-significance [Schuirmann, 1987; Lakens, 2017].

**TOST procedure:** The Two One-Sided Tests procedure [Schuirmann, 1987] tests whether the mean difference $\mu_M - \mu_R$ lies within a pre-specified equivalence margin $[-\delta, +\delta]$. Equivalence is declared if **both** one-sided tests reject:
- $H_{01}$: $\mu_M - \mu_R \leq -\delta$ (test that difference is not below $-\delta$)
- $H_{02}$: $\mu_M - \mu_R \geq +\delta$ (test that difference is not above $+\delta$)

Equivalently: construct a 90% confidence interval for $\mu_M - \mu_R$; if it lies entirely within $[-\delta, +\delta]$, equivalence is declared.

**Equivalence margin $\delta$:** Pre-specified before data collection. Recommended options (to be fixed in the experimental protocol):
- **Option A:** $\delta = 5$ mm ATE RMSE (absolute margin; conservative for sub-centimetre trajectories).
- **Option B:** $\delta = 10\%$ of the reference ATE from Real hardware on the same (session, trajectory) — e.g. $\delta = 0.1 \times \mathrm{ATE}_{\mathrm{Real}}^{\mathrm{T3}}$ for the most demanding trajectory.
- **Option C:** $\delta = \max(5\,\mathrm{mm},\, 0.1 \times \mathrm{ATE}_{\mathrm{Real}})$ — whichever is larger.

The chosen margin and rationale must be documented in the pre-registration or methods section.

**Success criterion (H0):**

| Condition | Test | Expected result | Interpretation |
|-----------|------|-----------------|----------------|
| Metrological sim vs. Real | TOST with margin $\delta$ | 90% CI for $\mu_M - \mu_R$ within $[-\delta, +\delta]$ | Metrological model **equivalent** to hardware within margin ✓ |
| Standard sim vs. Real | NHST (t-test / Wilcoxon) | p < 0.05, medium/large effect | Standard model significantly inferior — validates need for metrological approach ✓ |

**If TOST fails (equivalence not declared):** This is a publishable result — it indicates that even empirically derived noise parameters are insufficient to match real hardware within the chosen margin, which has important implications for the field and motivates Phase II.

**References:** Schuirmann (1987) [RESEARCH_PLAN Ref. 31]; Lakens (2017) [RESEARCH_PLAN Ref. 32].

### 3.5 Multiple Comparisons

With 4 sessions × 3 trajectories = 12 primary comparisons, apply Bonferroni correction to the family-wise error rate:

$\alpha_{\mathrm{corrected}} = 0.05 / 12 \approx 0.004$

- **For the control test (S vs. R):** Report both uncorrected and Bonferroni-corrected p-values. The primary conclusion should be based on corrected values.
- **For TOST (M vs. R):** Each equivalence test uses a 90% CI. For family-wise control across 12 TOST comparisons, use a $(1 - 0.10/12) \approx 99.2\%$ CI per comparison, or apply Holm–Bonferroni to the two one-sided p-values. The chosen procedure must be pre-specified in the analysis plan.
- Uncorrected values are reported for reference and comparison with other works.

---

## 4. Sensor Noise Models (M1–M4)

The metrological simulation uses four families of sensor noise models, as described in detail in `SLAM_BACKENDS.md`:

| ID | Name | Source | Description |
|----|------|--------|-------------|
| M1 | Manufacturer | Datasheet / typical values | Default noise parameters taken from vendor specifications or typical values used in the community. Represents \"standard\" simulation. |
| M2 | Static-Allan | Allan Variance on long static logs | Coefficients ARW, BI, RRW extracted from 10–12 h static logs (D_static). Time-invariant model. |
| M3 | In-Session-Static | Allan Variance on concatenated static windows within dynamic sessions | Same sensor, mount and temperature as dynamic runs, but using only 60 s static segments before/after trajectories (S0). Captures thermal equilibrium and mounting effects. |
| M4 | Kinematic-Residual | Residual-based dynamic model | State-dependent model obtained from residuals during motion, with noise variance expressed as a function of velocity, acceleration and jerk. |

M1–M3 feed directly into the covariance parameters of the Gazebo IMU/LiDAR/camera plugins. M4 adds an explicit dependency on the kinematic state of the sensor, learned from YuMi-based experiments.

---

## 5. Kinematic Residual Extraction

Static Allan analysis (Section 1) characterizes noise in the absence of motion. To model motion-induced noise (e.g., g-sensitivity in MEMS IMUs, motion blur in LiDAR/camera), residuals are computed using the YuMi kinematic ground truth.

### 5.1 Residual Definition

For IMUs, the true specific force $a_{\mathrm{true}}(t)$ and angular rate $\omega_{\mathrm{true}}(t)$ at the sensor frame are obtained by:

1. Interpolating the YuMi flange poses with a smooth spline (e.g., cubic B-splines) in SE(3).
2. Differentiating the spline analytically to obtain linear and angular velocities and accelerations.
3. Applying the hand-eye transform to express $a_{\mathrm{true}}(t), \omega_{\mathrm{true}}(t)$ in the sensor frame.

The residual signal is then:

$$
R_a(t) = a_{\mathrm{measured}}(t) - a_{\mathrm{true}}(t), \quad
R_\omega(t) = \omega_{\mathrm{measured}}(t) - \omega_{\mathrm{true}}(t).
$$

For LiDAR and camera, analogous residuals are obtained by projecting measurements into a static reference map (for LiDAR) or reference pose (for AprilTags) using ground truth poses and measuring deviations.

### 5.2 Kinematic Segmentation

Residuals are analysed in segments corresponding to different kinematic regimes (see `SLAM_BACKENDS.md`):

- S0: static-in-session ($|\omega| < \epsilon_\omega$, $|a| < \epsilon_a$ for ≥ 60 s),
- S1: low-velocity, low-acceleration,
- S2: sustained acceleration with moderate angular velocity,
- S3: high jerk (step changes in acceleration/velocity, e.g., T3 waypoints).

This segmentation makes it possible to quantify how noise statistics change with motion intensity.

### 5.3 Dynamic Noise Model (M4) with Thermal Component

Let $t_{\mathrm{on}}(t)$ denote the time elapsed since sensor power-on (or since the beginning of the log). The static Allan-based variance is extended to depend on $t_{\mathrm{on}}$ to capture thermal warm-up effects:

$$
\sigma^2_{\mathrm{static}}(t_{\mathrm{on}}) =
\sigma^2_{\infty} +
\bigl(\sigma^2_{\mathrm{cold}} - \sigma^2_{\infty}\bigr) e^{-t_{\mathrm{on}}/\tau_T},
$$

where $\sigma^2_{\mathrm{cold}}$ is the variance at cold start (estimated from early D_static windows), $\sigma^2_{\infty}$ is the variance in thermal steady state (estimated from S0 windows), and $\tau_T$ is an effective thermal time constant fitted from the data. This form is applied separately to IMU coefficients (e.g., ARW, BI) and to LiDAR/camera range or pose noise as appropriate.

The full state-dependent variance used for M4 then becomes:

$$
\sigma^2(t) =
\sigma^2_{\mathrm{static}}\bigl(t_{\mathrm{on}}(t)\bigr) +
 c_v \|\omega_{\mathrm{true}}(t)\| +
 c_a \|a_{\mathrm{true}}(t)\| +
 c_j \|\dot{a}_{\mathrm{true}}(t)\|,
$$

where $c_v, c_a, c_j$ are fitted from the residuals $R_a(t), R_\omega(t)$ over S1–S3.

This model is then used in the Gazebo plugins so that the injected sensor noise scales with both the current thermal state and the current kinematic state of the sensor, rather than remaining constant in time.

### 5.4 Sensor-Specific Instantiation

- **IMU:** $\sigma^2_{\mathrm{static}}(t_{\mathrm{on}})$ models the evolution of ARW/BI with warm-up; $c_v, c_a, c_j$ capture increased noise under rotation, acceleration and jerk (g-sensitivity and structural vibration).
- **LiDAR:** $\sigma^2_{\mathrm{static}}(t_{\mathrm{on}})$ and a time-varying range bias model drift in Time-of-Flight; coefficients $c_v, c_a$ capture increased spread of point-to-plane residuals under high rotational speed and acceleration.
- **Camera (RGB-D):** $\sigma^2_{\mathrm{static}}(t_{\mathrm{on}})$ approximates thermal drift of intrinsics and depth noise; $c_v$ captures effective degradation due to motion blur, modeled as increased pose/depth variance with angular speed.

### 5.5 M4 Generalization Check (held-out trajectory)

To demonstrate that M4 does not overfit to the specific trajectories used for fitting, coefficients $c_v, c_a, c_j$ (and thermal parameters) are fitted using only T1 (smooth) and T2 (moderate) data. The model is then evaluated on T3 (aggressive) as a held-out set. If M4 improves ATE/RPE on T3 relative to M1–M3, the model generalizes beyond the fitting regime. This protocol is reported explicitly; if generalization fails, the limitation is stated in the manuscript.

---

## 6. Three-Condition Comparison Pipeline

The full comparison pipeline for each session:

```
Stage 1: Real Hardware Data Collection
  ↓ YuMi dynamic session (T1/T2/T3 × MIX/CW/CCW blocks)
  ↓ Raw sensor logs + YuMi ground truth (RobotStudio export)
  ↓ Hand-eye transform applied → sensor trajectory in world frame
  ↓ FAST-LIO2 / GLIM estimation → estimated trajectory R

Stage 2: Standard Simulation
  ↓ Import YuMi trajectory into Gazebo (identical joint commands)
  ↓ Standard Gazebo IMU/LiDAR plugins (default Gaussian noise)
  ↓ Same SLAM backend → estimated trajectory S

Stage 3: Metrological Simulation
  ↓ Same Gazebo trajectory
  ↓ Custom plugin: ARW + BI + RRW noise injection, Rosetta topology
  ↓ Same SLAM backend → estimated trajectory M

Stage 4: Evaluation
  ↓ evo_ape + evo_rpe (ATE, RPE) for R, S, M against YuMi ground truth
  ↓ Statistical tests (§3)
  ↓ CW/CCW asymmetry analysis (§5)
```

**Critical constraint:** The SLAM backend and its parameters must be identical across all three conditions. Any tuning of SLAM parameters is frozen after the real-hardware baseline is established — no per-condition tuning allowed, as it would confound the comparison.

---

## 7. CW/CCW Asymmetry Analysis

Secondary hypothesis H2: gyroscope bias drift differs between CW and CCW rotation.

Method:
1. For each sensor, extract the end-of-trajectory orientation error (vs. ground truth) for all CW repetitions and all CCW repetitions separately
2. Compute mean and standard deviation of yaw error for CW group vs. CCW group
3. Test for significant difference: paired t-test or Wilcoxon (same selection criterion as §3.1)

Expected result: A non-negligible CW/CCW asymmetry is expected in MEMS gyroscopes due to mechanical asymmetry in the Coriolis structure. Its magnitude relative to the noise floor determines whether it is relevant to practical SLAM performance.

Reporting: ATE and yaw error broken down by rotation direction, with statistical comparison.

---

## 8. Frequency-Sweep Trajectory (Optional)

**Status:** Optional — add only if core protocol is completed and time permits.

Motivation: Identify the empirical cut-off angular velocity above which the SLAM estimator collapses or shows unacceptable drift accumulation. This is rarely characterized in the literature and could be a differentiating result.

Design: Sinusoidal yaw rotation with linearly increasing frequency over time (chirp signal):
- Angular rate $\omega(t) = A \cdot \sin(2\pi \cdot f(t) \cdot t)$, where $f(t)$ increases from 0.05 Hz to 2 Hz over 10 min
- Amplitude A selected to stay within YuMi dynamic limits

Metric: ATE and RPE computed in sliding windows. Plot ATE vs. instantaneous angular frequency. The cut-off frequency is defined as the frequency at which ATE exceeds 3× the T1 (smooth) ATE baseline (practical threshold for "unacceptable" drift; adjustable in analysis).

This experiment applies to all four sessions (A, B, C, D) and provides a sensor-comparative view of dynamic limit.

---

## 9. Analysis Toolchain Summary

| Task | Tool | Notes |
|------|------|-------|
| Allan Variance | `imu_utils` (ROS) / `allantools` (Python) | Overlapping ADEV estimator |
| Six-Position Test | Custom Python script | Manual per-axis extraction |
| LiDAR plane fitting | `Open3D` or `PCL` in Python | RANSAC plane fit at t=0, then orthogonal residuals |
| Camera AprilTag pose | `apriltag_ros` | 6-DOF pose at 1 Hz from static board |
| Trajectory format conversion | `evo` utilities | TUM / KITTI / EuRoC → TUM for comparison |
| ATE / RPE evaluation | `evo_ape`, `evo_rpe` | Umeyama alignment, per-trajectory |
| Statistical tests | `scipy.stats` (Python) | Shapiro-Wilk, t-test, Wilcoxon, Cohen's d |
| Visualization | `matplotlib`, `evo` plot tools | ADEV curves, trajectory overlays, box plots |
| Simulation (standard) | Gazebo Fortress default plugins | Baseline condition S |
| Simulation (metrological) | Custom Gazebo plugin (to be developed) | Condition M; open-source output |
