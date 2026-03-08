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
| **YuMi pose repeatability (RP)** | Random (per pose) | $\pm0.02$ mm | ABB ISO 9283 specification. Applies only to static points (start/end of trajectories, waypoints in T3). |
| **YuMi linear path accuracy (AT)** | Systematic (trajectory) | $1.36$ mm (worst-case ISO condition) | ABB ISO 9283 specification for actual vs programmed path. Tested at max speed (1.5 m/s) and max load. Defines the absolute bounds of volumetric accuracy. Since our trajectories max out at 0.1 m/s, the practical AT will be significantly lower, but this bounds the worst-case. |
| **YuMi linear path repeatability (RT)** | Random (trajectory) | $0.10$ mm | ABB ISO 9283 specification. Consistency when repeating the same path. More relevant for block repetition comparisons. |
| **Hand-eye calibration** | Systematic (constant offset per session) | &lt; 0.5 mm translation, &lt; 0.5° rotation | AX=XB (Tsai-Lenz or equivalent); re-done per session. See [EXPERIMENTAL_DESIGN.md §10](EXPERIMENTAL_DESIGN.md). Translation error propagates as systematic bias in position comparison. |
| **Temporal synchronization** | Random / systematic (time offset) | \( \Delta x \approx v_{\max} \times \Delta t \) | At ~100 mm/s, 10 ms offset → ~1 mm spatial error. PTP (IEEE 1588) preferred; NTP fallback with jitter documented. The **measured** offset and jitter must be reported; \( \Delta t \) is the measured or conservative bound. See [EXPERIMENTAL_DESIGN.md §9](EXPERIMENTAL_DESIGN.md). |
| **Forward kinematics / timestamp interpolation** | Negligible if documented | Sub-mm for typical interpolation | RobotStudio export + FK; interpolation of poses to sensor timestamps. Assume negligible if timestamps are aligned and interpolation is linear or spline-based over short intervals. |

Combined uncertainty (position): Under the assumption of independent contributions, a conservative **combined standard uncertainty** for ground truth position during dynamic trajectory comparison can be taken as the root-sum-square of the relevant terms, or as a worst-case bound. For *dynamic* segments the dominant mechanical terms are path repeatability (0.10 mm) and path accuracy (≤1.36 mm), not pose repeatability (0.02 mm). Example bound: 0.10 + 0.5 + \(v_{\max}\,\Delta t\) in mm. The exact combination (RSS vs worst-case) and the numerical values shall be fixed in the experimental report once sync and hand-eye are measured.

Interpretation of ATE/RPE: Metrics ATE and RPE are interpreted as being above the ground truth uncertainty floor. Differences between conditions (Real vs Sim M1/M4) that are of the same order as or smaller than this floor are not claimed as significant; the uncertainty budget is reported alongside ATE/RPE in any publication so that readers can judge. This budget does not replace the statistical tests (e.g. paired t-test, Wilcoxon) but clarifies the physical limit below which the comparison is not meaningful.

---

## 3. Statistical Test Design

### 3.1 Test Selection

The primary statistical comparison is between ATE distributions across the three conditions:
- Condition R: Real hardware
- **Condition S:** Standard Gazebo simulation
- Condition M: Metrological Gazebo simulation (with empirically derived parameters)

For each trajectory type (T1, T2, T3) and each session (A, B, C, D):

**Step 1 — Normality test:** Apply Shapiro-Wilk test on each ATE distribution. Sample size per trajectory type: each block (MIX, CW, CCW) runs ~2 h with repetitions of [60 s static → 1–2 min trajectory → 60 s static], yielding **$n \approx 40$–60 repetitions per block**; the three blocks together give **$n \approx 120$–180** per trajectory type (T1, T2, or T3). CW and CCW executions count as repetitions of the same trajectory type for the purpose of testing equivalence (M vs R). See [EXPERIMENTAL_DESIGN.md §3](EXPERIMENTAL_DESIGN.md) for the exact block structure.

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

A non-significant result (p > 0.05) in a standard hypothesis test does not demonstrate equivalence. "Absence of evidence is not evidence of absence." With limited sample size, failure to reject the null may simply indicate insufficient statistical power to detect a real difference. Metrology-oriented journals expect equivalence to be demonstrated explicitly [Schuirmann, 1987; Lakens, 2017].

**TOST procedure:** The Two One-Sided Tests procedure tests whether the mean difference $\mu_M - \mu_R$ lies within a pre-specified equivalence margin $[-\delta, +\delta]$. Equivalence is declared if **both** one-sided tests reject:
- $H_{01}$: $\mu_M - \mu_R \leq -\delta$ (test that difference is not below $-\delta$)
- $H_{02}$: $\mu_M - \mu_R \geq +\delta$ (test that difference is not above $+\delta$)

Equivalently: construct a 90% confidence interval for $\mu_M - \mu_R$; if it lies entirely within $[-\delta, +\delta]$, equivalence is declared.

**Equivalence margin $\delta$:** The margin is defined in units of Absolute Trajectory Error (ATE). To ensure clinical/practical relevance, the margin is set to 10% of the reference ATE from Real hardware on the most demanding trajectory (T3). For example, if the real hardware accumulates 50 mm ATE RMSE on T3, the equivalence margin is $\pm 5$ mm. This anchors the margin to the specific sensor and trajectory context rather than an arbitrary absolute value.

**Power analysis and sample size:** Each 2 h block yields **$n \approx 40$–60** independent repetitions (60 s static → 1–2 min trajectory → 60 s static per repetition). With three blocks per trajectory type, **$n \approx 120$–180** per trajectory type. This provides high statistical power for TOST within a 10% equivalence margin. The final count will be confirmed from the actual trajectory duration and block length during experimentation.

**Success criterion (H0):**

| Condition | Test | Expected result | Interpretation |
|-----------|------|-----------------|----------------|
| Metrological sim vs. Real | TOST with margin $\delta$ | 90% CI for $\mu_M - \mu_R$ within $[-\delta, +\delta]$ | Metrological model **equivalent** to hardware within margin ✓ |
| Standard sim vs. Real | NHST (t-test / Wilcoxon) | p < 0.05, medium/large effect | Standard model significantly inferior — validates need for metrological approach ✓ |

**If TOST fails (equivalence not declared):** This is a publishable result — it indicates that even empirically derived noise parameters are insufficient to match real hardware within the chosen margin, which has important implications for the field and motivates Phase II. Negative results will be reported explicitly.

**References:** Schuirmann (1987) [RESEARCH_PLAN Ref. 31]; Lakens (2017) [RESEARCH_PLAN Ref. 32].

### 3.5 Primary and Exploratory Endpoints

To mitigate the multiple comparisons problem (with over 200 possible comparisons across trajectories, sensors, and backends), the study defines a strict hierarchy of endpoints:

**Primary endpoints (subject to TOST and Holm-Bonferroni correction):**
- ATE RMSE under FAST-LIO2 for Session C (Mid-360) on Trajectory T2
- ATE RMSE under ORB-SLAM3 for Session D (D455) on Trajectory T2
- ATE RMSE under GLIM for Session C (Mid-360) on Trajectory T2

**Exploratory endpoints (reported without correction, flagged as exploratory):**
- All T1 and T3 comparisons
- RPE metrics
- Secondary SLAM backends (Point-LIO, Cartographer, etc.)

This focuses statistical power on the most representative moderate dynamic conditions (T2) using the most established backends, while still exploring limits (T3) and baselines (T1).

---

## 4. Sensor Noise Models (M1–M4)

The metrological simulation uses four families of sensor noise models. Definitions below are the single source of truth; [SLAM_BACKENDS.md](SLAM_BACKENDS.md) lists which SLAM backends are run per session and references this section for noise model semantics.

| ID | Name | Source | Description |
|----|------|--------|-------------|
| M1 | Manufacturer | Datasheet / typical values | Default noise parameters taken from vendor specifications or typical values used in the community. Represents \"standard\" simulation. |
| M2 | Static-Allan | Allan Variance on long static logs | Coefficients ARW, BI, RRW extracted from 10–12 h static logs (D_static). Time-invariant model. |
| M3 | In-Session-Static | Allan Variance on concatenated static windows within dynamic sessions | Same sensor, mount and temperature as dynamic runs, but using only 60 s static segments before/after trajectories (S0). Captures thermal equilibrium and mounting effects. |
| M4 | Kinematic-Residual | Residual-based dynamic model | State-dependent model: variance as a function of **measured temperature $T$** and kinematic state (velocity, acceleration, jerk). See §5.3–5.4 for formulas and candidate formulations. |

**Kinematic regime taxonomy (used for M4 segmentation):**

| ID | Name | Definition (illustrative) | Purpose |
|----|------|---------------------------|---------|
| S0 | Static-in-session | $|\omega| < \epsilon_\omega$, $|a| < \epsilon_a$ for ≥ 60 s | Source for M3 (in-session static Allan). |
| S1 | Low-velocity | small $\|\omega\|$, low $\|a\|$ (e.g., T1) | Baseline dynamic regime. |
| S2 | Moderate-acceleration | sustained $\|a\|$ with moderate $\|\omega\|$ (e.g., T2) | Tests acceleration-induced noise. |
| S3 | High-jerk | large $\|\dot{a}\|$ / step changes (e.g., T3 waypoints) | Excites vibration, g-sensitivity and motion blur. |

M1–M3 feed directly into the covariance parameters of the Gazebo IMU/LiDAR/camera plugins. M4 adds an explicit dependency on the kinematic state and on measured temperature, learned from YuMi-based experiments.

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

Residuals are analysed in segments corresponding to different kinematic regimes (S0–S3 defined in §4 above).

This segmentation makes it possible to quantify how noise statistics change with motion intensity.

### 5.3 Dynamic Noise Model (M4) — Goal and Thermal Component

Goal: Obtain an explicit variance formula $\sigma^2(t) = f\bigl(T(t), \|v\|, \|\omega\|, \|a\|, \|\dot{\omega}\|, \|\dot{a}\|\bigr)$ with identifiable coefficients, fitted from residuals against YuMi ground truth. The aim is to demonstrate empirically which formulation fits the data — not to deduce the correct form a priori. Several candidate formulas are proposed below; the experimental study will validate or discard each.

Thermal component: Let $T(t)$ denote the **measured temperature** of the sensor (or its immediate environment). The static variance is modelled as a function of $T$ to capture thermal drift:

$$
\sigma^2_{\mathrm{static}}(T) =
\sigma^2_{\infty} +
\bigl(\sigma^2_{\mathrm{cold}} - \sigma^2_{\infty}\bigr) \, g(T; T_{\mathrm{ref}}, \tau_T),
$$

where $\sigma^2_{\mathrm{cold}}$ is variance at cold start (early D_static windows), $\sigma^2_{\infty}$ is variance in thermal steady state (S0 windows), and $g$ is a suitable function of temperature (e.g. exponential settling toward $T_{\mathrm{ref}}$). **Temperature measurement:** Ideally $T$ is logged throughout static characterization and during YuMi dynamic sessions — via a calibrated thermometer in thermal contact with the sensor housing, or via hardware temperature topics where available (e.g. Livox Mid-360 and RealSense D455 expose internal temperature). The exact sensor and placement will be documented in the experimental report.

<a id="54-candidate-formulas-for-the-kinematic-term-to-validate-empirically"></a>

### 5.4 Candidate Formulas for the Kinematic Term (to validate empirically)

The full M4 variance combines $\sigma^2_{\mathrm{static}}(T)$ with a kinematic term. The kinematic state includes: linear velocity $\|v\|$, angular velocity $\|\omega\|$, linear acceleration $\|a\|$, angular acceleration $\|\dot{\omega}\|$, and jerk $\|\dot{a}\|$. The following candidates are proposed for experimental validation. Coefficients are fitted from residuals $R_a(t), R_\omega(t)$ over S1–S3; the study will determine which formulations generalise to T3 (held-out) and improve ATE/RPE vs. M1–M3.

| # | Formulation | Formula | Notes |
|---|-------------|---------|-------|
| 1 | **Additive linear** | $\sigma^2 = \sigma^2_{\mathrm{static}} + c_v \|v\| + c_\omega \|\omega\| + c_a \|a\| + c_{\dot{\omega}} \|\dot{\omega}\| + c_j \|\dot{a}\|$ | Requires explicit units per coefficient; risk of $\sigma^2 < 0$ if coefficients are negative. |
| 2 | **Multiplicative** | $\sigma^2 = \sigma^2_{\mathrm{static}} \cdot \max\bigl(1 + c_v \|v\| + c_\omega \|\omega\| + c_a \|a\| + c_{\dot{\omega}} \|\dot{\omega}\| + c_j \|\dot{a}\|,\; \epsilon\bigr)$ | Guarantees $\sigma^2 > 0$; all $c \geq 0$; dimensionless scaling. |
| 3 | **Single term (g-sensitivity)** | $\sigma^2 = \sigma^2_{\mathrm{static}} \cdot \bigl(1 + c_\omega \|\omega\|\bigr)$ | One parameter; better identifiability; ignores $v$, $a$, $\dot{\omega}$, $\dot{a}$. |
| 4 | **Conservative scaling** | $\sigma^2 = \sigma^2_{\mathrm{static}} \cdot k_{\mathrm{dyn}}$, $k_{\mathrm{dyn}} \geq 1$ | Single factor during dynamic phases; supported by practice (e.g. Kalibr: static params underestimate low-cost IMU uncertainty). |

The chosen formulation(s) will be reported in the manuscript; candidates that fail to generalise or improve metrics will be discarded and the limitation stated.

<a id="541-sensor-specific-coefficient-pruning"></a>

#### 5.4.1 Sensor-specific coefficient pruning

Not all kinematic terms are physically relevant for every sensor. Coefficients are fixed to zero when the underlying mechanism does not apply, reducing parameters and improving identifiability. The following pruning is applied by default:

| Sensor | Coefficient fixed to 0 | Rationale |
|--------|-------------------------|-----------|
| **IMU** | $c_v$ (linear velocity) | The IMU measures acceleration and angular rate, not velocity. Velocity is an integrated quantity; it does not directly affect IMU noise. G-sensitivity and vibration depend on $a$, $\omega$, $\dot{a}$, $\dot{\omega}$ — not on $v$. |
| **IMU** | $c_{\dot{\omega}}$ (optional) | Angular acceleration may be correlated with jerk in many trajectories; if identifiability is poor, $c_{\dot{\omega}} = 0$ can be tried. |
| **LiDAR** | — | All terms potentially relevant: $\|v\|$ and $\|\omega\|$ for motion blur during scan; $\|a\|$, $\|\dot{\omega}\|$, $\|\dot{a}\|$ for structural vibration. Pruning only if data show non-identifiability. |
| **Camera (RGB-D)** | $c_{\dot{\omega}}$, $c_j$ (optional) | Motion blur is dominated by $\|v\|$ and $\|\omega\|$. Acceleration and jerk have weaker direct effect; can be pruned if not identifiable. |

IMU default: $c_v = 0$ always. The remaining terms $c_\omega, c_a, c_{\dot{\omega}}, c_j$ are fitted; $c_{\dot{\omega}}$ may be set to 0 if VIF or condition number indicate collinearity with $c_j$.

LiDAR / Camera: Start with the full set; prune only after checking identifiability (VIF, condition number) or if a coefficient is not significant in the residual fit.

### 5.5 Sensor-Specific Instantiation

- **IMU:** $\sigma^2_{\mathrm{static}}(T)$ models the evolution of ARW/BI with temperature; the kinematic term (per chosen candidate) captures increased noise under rotation, acceleration and jerk (g-sensitivity and structural vibration).
- **LiDAR:** $\sigma^2_{\mathrm{static}}(T)$ and a time-varying range bias model capture ToF drift; kinematic coefficients capture increased spread of point-to-plane residuals under high rotational speed and acceleration.
- **Camera (RGB-D):** $\sigma^2_{\mathrm{static}}(T)$ approximates thermal drift of intrinsics and depth noise; angular-velocity term captures effective degradation due to motion blur.

### 5.6 M4 Generalization Check (held-out trajectory)

To demonstrate that M4 does not overfit to the specific trajectories used for fitting, kinematic coefficients (and thermal parameters) are fitted using only T1 (smooth) and T2 (moderate) data. The model is then evaluated on T3 (aggressive) as a held-out set. If M4 improves ATE/RPE on T3 relative to M1–M3, the model generalizes beyond the fitting regime. This protocol is reported explicitly; if generalization fails, the limitation is stated in the manuscript.

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
  ↓ CW/CCW asymmetry analysis (§7)
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
