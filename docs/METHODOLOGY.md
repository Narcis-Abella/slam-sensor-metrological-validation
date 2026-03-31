# Methodology

* **Version:** 0.6
* **Date:** 2026-03-28
* **Status:** Pending supervisor review

---

## Table of Contents

1. [Allan Variance Analysis](#1-allan-variance-analysis)
2. [Residual Metrics and Ground Truth](#2-residual-metrics-and-ground-truth) (includes [Ground Truth Uncertainty Budget](#23-ground-truth-uncertainty-budget))
3. [Statistical Test Design](#3-statistical-test-design)
4. [Sensor Noise Models (M1-M4)](#4-sensor-noise-models-m1-m4)
5. [Kinematic Residual Extraction](#5-kinematic-residual-extraction)
6. [Residual Comparison Pipeline (Primary)](#6-residual-comparison-pipeline-primary)
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

RRW identification requires averaging times of several hours. This is why 10 to 12 h static logs are necessary: shorter logs do not reach the $\tau$ range where the $+1/2$ slope is observable.

### 1.2 Procedure

1. Record static IMU log at ≥ 200 Hz for 10 to 12 h (see [EXPERIMENTAL_DESIGN.md §2.1](EXPERIMENTAL_DESIGN.md))
2. Compute ADEV using overlapping Allan Variance estimator (maximizes data use)
3. Fit slopes to identify noise process boundaries
4. Extract numerical coefficients at the specified averaging times
5. Compute confidence intervals from the chi-squared distribution of the overlapping ADEV estimator; report k = 2 coverage

Tooling: `imu_utils` (ROS package) or equivalent Python implementation (e.g., `allan_variance_ros`, `allantools`)

### 1.3 Reporting Standard

All coefficients must be reported as:

```txt
ARW:  X.XX ± Y.YY °/√h    (read at τ = 1 s)
BI:   X.XX ± Y.YY °/h     (read at ADEV minimum)
RRW:  X.XX ± Y.YY °/h^(3/2)  (if identifiable; state "not identifiable" otherwise)
```

Units and confidence intervals are mandatory. Curves alone are insufficient for reproducibility.

### 1.4 Six-Position Test Reporting

Scale factor and gravitational bias per axis, reported as:

```txt
Scale factor error (X/Y/Z): X.XX ppm
Gravitational bias (X/Y/Z): X.XX mg
```

---

## 2. Residual Metrics and Ground Truth

### 2.1 Sensor Residual Metrics (primary outcomes)

The primary measurands of this study are sensor residuals: the difference between measured sensor output and the true value derived from YuMi kinematics. These are the quantities on which TOST is applied.

| Sensor | Residual definition | Physical quantity | Units |

|--------|---------------------|-------------------|-------|
| IMU (S1, S3-IMU, S4-IMU) | $R_\omega(t) = \omega_\text{measured}(t) - \omega_\text{true}(t)$; $R_a(t) = a_\text{measured}(t) - a_\text{true}(t)$ | Angular rate error; linear acceleration error | deg/s; m/s² |
| LiDAR (S2, S3) | Point-to-plane distance from scan points to a reference plane established at $t = 0$ | Range noise; thermal ToF drift | mm |
| Camera (S4) | AprilTag board pose drift: $\Delta p(t) = p_\text{board}(t) - p_\text{board}(0)$; $\Delta R(t)$ | Thermal intrinsic drift; FPN | mm (translation), deg (rotation) |

Residuals are computed for both real hardware (Stage 2 sessions A-D) and simulated data (Gazebo Fortress plugin). The TOST equivalence test compares the distribution of real hardware residuals against the distribution of simulated residuals under each noise model M1 through M4. The residual computation procedure is defined in §5.

### 2.2 Ground Truth Format

YuMi RobotStudio trajectory export is transformed to the analysis format required by the residual extraction pipeline (time-synchronized pose and sensor streams with explicit frame metadata).

All timestamps must be synchronized (see [EXPERIMENTAL_DESIGN.md §9](EXPERIMENTAL_DESIGN.md)) before metric computation.

### 2.3 Ground Truth Uncertainty Budget

The ground truth trajectory is not perfect. Its uncertainty must be quantified so that (i) reviewers can assess the validity of comparisons and (ii) differences smaller than the ground truth floor are not over-interpreted. All contributions must be documented at the time of experimentation; values below are design targets or typical orders of magnitude.

| Source | Type | Magnitude (design / typical) | Notes |

|--------|------|------------------------------|-------|
| **YuMi pose repeatability (RP)** | Random (per pose) | $\pm0.02$ mm | ABB ISO 9283 specification. Applies only to static points (start/end of trajectories, waypoints in T3). |
| **YuMi linear path accuracy (AT)** | Systematic (trajectory) | $1.36$ mm (worst-case ISO condition) | ABB ISO 9283 specification for actual vs programmed path. Tested at max speed (1.5 m/s) and max load. Since sessions max at 0.1 to 0.5 m/s, the practical AT is lower, but this defines the worst-case bound. Declared as Type B uncertainty; no calibration certificate exists for the IQS unit. |
| **YuMi linear path repeatability (RT)** | Random (trajectory) | $0.10$ mm | ABB ISO 9283 specification. Consistency when repeating the same path. Relevant for inter-block residual comparisons. |
| **CAD fixture nominal tolerance** | Systematic (constant per sensor) | TBD per fixture (SolidWorks design) | Type B. 3D-printed fixture designed from sensor published CAD + YuMi flange spec. Tolerance declared per fixture in experimental report. |
| **Fixture dimensional measurement** | Systematic (per piece) | ±? mm TBD (caliper) | Type B. Per-piece caliper measurement of printed fixture after fabrication. Value TBD. |
| **Contact probing (within session)** | Random | TBD (≥10 repeated touches) | Type A. Standard deviation of conical-tip probing procedure for target pose characterization. |
| **Contact probing (cross-session)** | Random (session-to-session) | TBD (per-session) | Type A. Quantifies cross-session remounting uncertainty. Acceptance criterion documented in experimental report. |
| **Operator contact tip placement** | Random | TBD | Type A. Human contribution to probing repeatability; not eliminated; quantified via the probing protocol. |
| **Laser rangefinder 1D uncertainty** | Systematic | TBD (instrument spec) | Type B. Only applicable if variable-distance targets are used for LiDAR (decision pending). |
| **Temporal synchronization** | Random / systematic (time offset) | $\Delta x \approx v_{\max} \times \Delta t$ | At 100 mm/s, a 10 ms offset introduces 1 mm spatial error [[52]](REFERENCES_CONSOLIDATED.md). PTP preferred; NTP fallback with jitter documented. The measured offset must be reported. |
| **Cable routing effects** | Systematic (variable per trajectory) | < 0.5 mm at T1; higher at T2/T3 | USB 3.0 and Ethernet cables along the YuMi arm introduce variable external forces during dynamic trajectories. Mitigated by cable chain routing; residual effect included as a declared budget contributor. |
| **Forward kinematics / timestamp interpolation** | Negligible if documented | Sub-mm for typical intervals | RobotStudio FK + spline interpolation to sensor timestamps. Negligible if timestamps are aligned and interpolation intervals are short. |

Combined uncertainty (position): Under independent contributions, a conservative combined standard uncertainty for dynamic trajectory comparison is the root-sum-square of the dominant terms, or a worst-case bound. For dynamic segments the dominant terms are path repeatability (0.10 mm) and path accuracy (≤1.36 mm). The exact combination and numerical values are fixed in the experimental report once sync offset, fixture tolerances, and probing repeatability are measured.

---

### 2.4 Sensor-to-Flange Transform (CAD-based)

Sensor-to-flange transforms $T_{\text{sensor→flange}}$ are derived from CAD-designed 3D-printed fixtures. Each fixture is designed in SolidWorks from the sensor's published mechanical CAD model and the YuMi flange specification; the nominal transform is therefore known by design.

Per-sensor frame corrections applied as fixed offsets to $T_{\text{sensor→flange}}$:

- **IMU (WT901C, ICM-40609, BMI055):** Inertial frame coincides with housing to within published tolerances. No correction applied.
- **Livox Mid-360:** Point cloud frame origin is the optical center of the Risley prism system. The offset from the housing is taken from Livox technical documentation and applied as a fixed correction.
- **RealSense D455:** Optical frame origin and intrinsic parameters from Intel factory calibration, accessible via the RealSense SDK. Factory intrinsics used directly; nominal accuracy declared in §2.3.

Before each session, target poses in the laboratory frame are characterized via contact probing: a conical tip fixture (CAD-known geometry) is mounted on the YuMi flange, ≥3 non-collinear reference points on each target are touched, and YuMi FK (RT = 0.10 mm) provides their 3D coordinates in the world frame. Six-DOF target pose is reconstructed analytically from ≥3 points. For targets beyond YuMi reach (e.g. far walls for LiDAR characterization), a laser rangefinder in a CAD-known fixture provides 1D distance at known YuMi poses; ≥3 non-collinear measurements give the target pose. Whether variable-distance targets are required is pending decision and is marked TBD in §2.3.

Fixture nominal tolerances and per-piece caliper measurements enter the budget as Type B contributors; probing repeatability (within-session and cross-session) enters as Type A contributors (see §2.3).

Interpretation: Residual differences between conditions (real vs. simulated under M1/M4) that are of the same order as or smaller than the ground truth uncertainty floor are not claimed as significant. This budget does not replace the TOST but defines the physical limit below which comparisons are not meaningful.

---

## 3. Statistical Test Design

### 3.1 Test Selection

The primary statistical comparison is between sensor residual distributions under two conditions:

- **Condition R:** real hardware sensor residuals ($\text{measured} - \text{true}_\text{YuMi}$, per §2.1 and §5.1)
- **Condition M:** simulated sensor residuals under the metrological simulation ($\text{sim output} - \text{sim true}$, per §6)

For each sensor type, each session (A-D), and each trajectory profile (T1, T2, T3):

**Step 1: Normality test.** Apply Shapiro-Wilk to the distribution of per-repetition mean residuals. Each block (MIX, CW, CCW) yields approximately 40 to 60 independent repetitions (60 s static + 1 to 2 min trajectory + 60 s static per repetition); the three blocks together give approximately 120 to 180 repetitions per trajectory profile. See [EXPERIMENTAL_DESIGN.md §3](EXPERIMENTAL_DESIGN.md) for the block structure.

**Step 2: Select test.**

- If normal (p > 0.05 Shapiro-Wilk for both conditions): paired t-test
- If non-normal (either condition fails normality): Wilcoxon signed-rank test

**Step 3: Apply test.**

- Compare Condition M vs. Condition R: TOST for equivalence (see §3.4)
- Compare Condition S (standard sim, M1) vs. Condition R: standard NHST (paired t-test or Wilcoxon); expected result p < 0.05, confirming standard simulation residuals differ from real hardware

### 3.2 Significance Level

$\alpha = 0.05$ (two-tailed for NHST; each one-sided test of TOST at $\alpha = 0.05$)

### 3.3 Effect Size

Report effect size alongside p-value:

- For parametric (t-test): Cohen's d
- For non-parametric (Wilcoxon): rank-biserial correlation r

### 3.4 Equivalence Testing (TOST)

Standard hypothesis testing asks whether two distributions differ significantly. For simulation validation this is the wrong question: a non-significant result (p > 0.05) does not mean the simulation is good enough: it may simply mean the sample was too small to detect a real difference. The correct question is whether the difference between metrological simulation and real hardware is small enough to be practically irrelevant for sensor design decisions. TOST answers this by testing whether the mean residual difference falls within a pre-specified equivalence margin $\delta$, providing positive evidence of sufficiency rather than merely absence of detected difference. This methodology has been standard in pharmaceutical bioequivalence since Schuirmann (1987) [[1]](REFERENCES_CONSOLIDATED.md) and is applied here, to the best of our knowledge, at the sensor residual level in robotics. Prior analogues in adjacent domains include Robinson (2004) [[4]](REFERENCES_CONSOLIDATED.md) for ecological model validation, Ramert (2020) [[5]](REFERENCES_CONSOLIDATED.md) for defense M&S validation, and practical-equivalence framing in synthetic ADAS validation [[6]](REFERENCES_CONSOLIDATED.md).

**TOST procedure:** Tests whether the mean residual difference $\mu_M - \mu_R$ lies within $[-\delta, +\delta]$. Equivalence is declared if **both** one-sided tests reject:

- $H_{01}$: $\mu_M - \mu_R \leq -\delta$ (test that difference is not below $-\delta$)
- $H_{02}$: $\mu_M - \mu_R \geq +\delta$ (test that difference is not above $+\delta$)

Equivalently: construct a 90% confidence interval for $\mu_M - \mu_R$; if it lies entirely within $[-\delta, +\delta]$, equivalence is declared. Reference: Lakens (2017) [[2]](REFERENCES_CONSOLIDATED.md); Wellek (2010) [[3]](REFERENCES_CONSOLIDATED.md).

**Equivalence margin $\delta$:** Defined per sensor type in physical signal units. Anchored to external system requirements (task-driven navigation performance thresholds from published literature) and agreed with the supervisor before any data collection. It must not be defined as a percentage of the measured residuals, and must not be changed after data collection begins.

| Sensor | Measurand | $\delta$ units |

|--------|-----------|----------------|
| LiDAR (S2, S3) | Point-to-plane residuals | mm |
| IMU gyroscope (S1, S3-IMU, S4-IMU) | Angular rate residuals | deg/s |
| IMU accelerometer | Linear acceleration residuals | m/s² |
| Camera (S4) | AprilTag pose drift: translation | mm |
| Camera (S4) | AprilTag pose drift: rotation | deg |

Numerical values of $\delta$ are locked in this section before session execution. Sensitivity to alternative margin choices ($\delta/2$ and $2\delta$) is reported in the Discussion.

**Power analysis and sample size:** Exact TOST power functions [[7]](REFERENCES_CONSOLIDATED.md) are used to compute the required $n$ before data collection. Target: $1-\beta \geq 0.80$ at $\alpha = 0.05$, with $\sigma$ estimated from Stage 1 static logs and literature values. This power analysis is a P0 blocker: sessions cannot begin until the planned $n$ achieves the required power given the chosen $\delta$.

**Success criteria:**

| Condition | Test | Success criterion | Interpretation |

|-----------|------|-------------------|----------------|
| Metrological sim (M) vs. real hardware (R) | TOST with $\delta$ per sensor type | 90% CI for mean residual difference within $[-\delta, +\delta]$ | Simulated residuals equivalent to real hardware within margin |
| Standard sim (S) vs. real hardware (R) | NHST (t-test / Wilcoxon) | p < 0.05, medium/large effect | Standard noise parameters insufficient; metrological approach necessary |

**If TOST fails (equivalence not declared):** This is a publishable result. It indicates that even empirically derived noise parameters are insufficient to match real hardware residuals within the chosen margin, which motivates further investigation. Negative results are reported explicitly.

### 3.5 Primary and Exploratory Endpoints

**Primary endpoints (subject to TOST and Holm-Bonferroni correction across sessions):**

- IMU residuals ($R_\omega$, $R_a$): Session A (WT901C), Trajectory T2
- LiDAR 2D point-to-plane residuals: Session B (RPLiDAR A2M12), Trajectory T2
- LiDAR 3D point-to-plane residuals: Session C (Livox Mid-360), Trajectory T2
- Camera AprilTag pose drift: Session D (RealSense D455), Trajectory T2

T2 (moderate dynamics) is the primary trajectory because it represents the operational range of interest for indoor mobile robots: below the maximum dynamics that cause system failure, but above the static floor where noise model differences are imperceptible. Canonical nominal velocity and angular-rate values per session are defined in [EXPERIMENTAL_DESIGN.md](EXPERIMENTAL_DESIGN.md).

**Exploratory endpoints (reported without correction, flagged as exploratory):**

- All T1 and T3 residual comparisons (T3 also serves as the M4 held-out generalization check)
- CW/CCW asymmetry per session (§7)
- Secondary noise model rungs M1 and M3 at T2

---

## 4. Sensor Noise Models (M1-M4)

The metrological simulation uses four families of sensor noise models. Definitions below are the single source of truth for all noise model semantics used in the study.

### 4.1 Variables In Scope

Characterized variables are those intrinsic to the sensor's physical measurement principle and controllable or observable under 6DOF kinematic ground truth in a laboratory setting. External environmental variables are explicitly out of scope and deferred to future work.

#### IMU (WT901C, ICM-40609, BMI055)

| Variable | Status | Justification |
| -------- | ------ | ------------- |
| Thermal drift (bias vs temperature) | IN | Intrinsic; measurable via static logs with thermal monitoring |
| Kinematic state-dependent noise (g-sensitivity, acceleration, jerk, angular rate) | IN | Captured in M4 via YuMi dynamic residuals |
| Cross-axis sensitivity | IN | Intrinsic; extracted from Six-Position Test at negligible additional cost |
| Quantization noise | IN | Deterministic from bit resolution and measurement range specs |
| Vibration from external mechanical sources | OUT (future work) | External, platform-dependent; M4 kinematic terms provide a lower bound |
| Magnetic interference | OUT (future work) | External, environment-dependent |
| Power supply noise | OUT (future work) | External, platform-dependent |
| Aging / long-term drift | OUT (future work) | Requires longitudinal study over months to years |

#### LiDAR 2D Mechanical (RPLiDAR A2M12)

| Variable | Status | Justification |
| -------- | ------ | ------------- |
| Thermal range drift | IN | Intrinsic; characterizable via static wall sessions with thermal monitoring |
| Range-dependent noise | IN | Intrinsic; measurable by varying sensor-to-wall distance |
| Incidence angle dependence | IN | Intrinsic to beam geometry; controllable by rotating sensor relative to reference wall |
| Kinematic-induced point scatter (M4) | IN | Captured via YuMi dynamic sessions |
| Quantization noise | IN | Deterministic from range resolution specs |
| Material reflectivity | OUT (future work) | External, surface-dependent |
| Ambient light interference | OUT (future work) | External, environment-dependent |
| Multi-return / crosstalk | OUT (future work) | Requires complex multi-surface environments |

#### LiDAR 3D Solid-State (Livox Mid-360)

| Variable | Status | Justification |
| -------- | ------ | ------------- |
| Thermal range drift | IN | Intrinsic; same protocol as RPLiDAR |
| Range-dependent noise | IN | Intrinsic; measurable at multiple wall distances |
| Scan pattern geometry (Rosetta non-repetitive) | IN | Core software contribution: Gazebo Fortress plugin reproduces Risley prism scan topology |
| Kinematic-induced point scatter (M4) | IN | Captured via YuMi dynamic sessions |
| Quantization noise | IN | Deterministic from range resolution specs |
| Incidence angle as independent variable | OUT (declared) | Mid-360 covers all angles simultaneously by design. Angle effects are mixed into range residuals but cannot be isolated without a goniometric setup. Declared as contributor to residual variance. |
| Material reflectivity | OUT (future work) | External, surface-dependent |
| Ambient light interference | OUT (future work) | External, environment-dependent |
| Multi-return / crosstalk | OUT (future work) | Requires complex multi-surface environments |

#### RGB-D Camera (Intel RealSense D455)

| Variable | Status | Justification |
| -------- | ------ | ------------- |
| Thermal intrinsic drift | IN | Intrinsic; characterized via AprilTag pose drift during 3–4h static observation |
| IR projector fixed-pattern noise | IN | Intrinsic; observable in static sessions as spatially structured depth residuals |
| Depth accuracy vs range | IN | Intrinsic; measurable at multiple distances from reference plane |
| Motion blur at high velocity | IN | Intrinsic to exposure time; captured in T3 dynamic sessions |
| Kinematic-induced noise (M4) | IN | Captured via YuMi dynamic sessions |
| Quantization noise | IN | Deterministic from depth resolution specs |
| Ambient light / sun blinding | OUT (future work) | External, environment-dependent |
| Surface reflectivity / transparency | OUT (future work) | External, surface-dependent |
| Rolling shutter | OUT (future work) | High modeling complexity |

### 4.2 Noise Model Definitions (M1–M4)

| ID | Name | Source | Description |

|----|------|--------|-------------|
| M1 | Manufacturer | Datasheet / typical values | Default noise parameters taken from vendor specifications or typical values used in the community. Represents "standard" simulation. |
| M2 | Static-Allan | Allan Variance on long static logs | Coefficients ARW, BI, RRW extracted from 10 to 12 h static logs. Time-invariant model. |
| M3 | In-Session-Static | Allan Variance on concatenated static windows within dynamic sessions | Same sensor, mount and temperature as dynamic runs, using 60 s static segments before/after trajectories (S0). Captures thermal equilibrium and mounting effects. |
| M4 | Kinematic-Residual | Residual-based dynamic model | State-dependent model: variance as a function of measured temperature $T$ and kinematic state (velocity, acceleration, jerk). See §5.3-5.4 for formulas and candidate formulations. |

> **Note on M4 positioning:** State-dependent noise covariance is not a new concept: learning-based approaches (VIO-DualProNet [[18]](REFERENCES_CONSOLIDATED.md), AirIMU [[19]](REFERENCES_CONSOLIDATED.md)) already demonstrate its value in VIO systems. M4 contributes an explicit parametric formulation with physically interpretable coefficients, simultaneous combination of measured temperature and full kinematic state vector, and coefficients fitted against a sub-millimetre robot arm reference rather than learned from uncontrolled data. Its primary role in this study is to provide sufficient simulation fidelity for the TOST equivalence test to have statistical power within task-relevant margins. See [RESEARCH_PLAN.md §3.6](RESEARCH_PLAN.md) for full differentiation from prior work.

**Kinematic regime taxonomy (used for M4 segmentation):**

| ID | Name | Definition (illustrative) | Purpose |

|----|------|---------------------------|---------|
| S0 | Static-in-session | $|\omega| < \epsilon_\omega$, $|a| < \epsilon_a$ for ≥ 60 s | Source for M3 (in-session static Allan). |
| S1 | Low-velocity | small $\|\omega\|$, low $\|a\|$ (e.g., T1) | Baseline dynamic regime. |
| S2 | Moderate-acceleration | sustained $\|a\|$ with moderate $\|\omega\|$ (e.g., T2) | Tests acceleration-induced noise. |
| S3 | High-jerk | large $\|\dot{a}\|$ / step changes (e.g., T3 waypoints) | Excites vibration, g-sensitivity and motion blur. |

M1-M3 feed directly into the covariance parameters of the Gazebo IMU/LiDAR/camera plugins. M4 adds an explicit dependency on kinematic state and measured temperature, fitted from YuMi-based experiments.

---

## 5. Kinematic Residual Extraction

Static Allan analysis (§1) characterizes noise in the absence of motion. To model motion-induced noise (e.g., g-sensitivity in MEMS IMUs, motion blur in LiDAR/camera), residuals are computed using the YuMi kinematic ground truth.

### 5.1 Residual Definition

For IMUs, the true specific force $a_{\mathrm{true}}(t)$ and angular rate $\omega_{\mathrm{true}}(t)$ at the sensor frame are obtained by:

1. Interpolating the YuMi flange poses with a smooth spline (e.g., cubic B-splines) in SE(3).
2. Differentiating the spline analytically to obtain linear and angular velocities and accelerations.
3. Applying the CAD-derived $T_{\text{sensor→flange}}$ (with sensor frame offset correction per sensor type; see §2.6) to express $a_{\mathrm{true}}(t), \omega_{\mathrm{true}}(t)$ in the sensor frame.

The residual signal is then:

$$
R_a(t) = a_{\mathrm{measured}}(t) - a_{\mathrm{true}}(t), \quad
R_\omega(t) = \omega_{\mathrm{measured}}(t) - \omega_{\mathrm{true}}(t).
$$

For LiDAR and camera, analogous residuals are obtained by projecting measurements into a static reference map (for LiDAR) or reference pose (for AprilTags) using ground truth poses and measuring deviations. For LiDAR specifically, points belonging to the YuMi structure itself are removed before residual computation using a self-filter based on the robot's geometric model (URDF + meshes); this prevents the arm's motion from contaminating statistics that are meant to characterise the external environment.

### 5.2 Kinematic Segmentation

Residuals are analysed in segments corresponding to different kinematic regimes (S0-S3 defined in §4 above).

This segmentation makes it possible to quantify how noise statistics change with motion intensity.

### 5.3 Dynamic Noise Model (M4): Goal and Thermal Component

Goal: Obtain an explicit variance formula $\sigma^2(t) = f\bigl(T(t), \|v\|, \|\omega\|, \|a\|, \|\dot{\omega}\|, \|\dot{a}\|\bigr)$ with identifiable coefficients, fitted from residuals against YuMi ground truth. The aim is to demonstrate empirically which formulation fits the data, not to deduce the correct form a priori. Several candidate formulas are proposed below; the experimental study will validate or discard each.

Thermal component: Let $T(t)$ denote the measured temperature of the sensor (or its immediate environment). The static variance is modelled as a function of $T$ to capture thermal drift:

$$
\sigma^2_{\mathrm{static}}(T) =
\sigma^2_{\infty} +
\bigl(\sigma^2_{\mathrm{cold}} - \sigma^2_{\infty}\bigr) \, g(T; T_{\mathrm{ref}}, \tau_T),
$$

where $\sigma^2_{\mathrm{cold}}$ is variance at cold start (early static characterization windows), $\sigma^2_{\infty}$ is variance in thermal steady state (S0 windows), and $g$ is a suitable function of temperature (e.g. exponential settling toward $T_{\mathrm{ref}}$). Temperature measurement: $T$ is logged throughout static characterization and during YuMi dynamic sessions via a calibrated thermometer in thermal contact with the sensor housing, or via hardware temperature topics where available (Livox Mid-360 and RealSense D455 expose internal temperature). The exact sensor and placement are documented in the experimental report.

### 5.4 Candidate Formulas for the Kinematic Term (to validate empirically)

The full M4 variance combines $\sigma^2_{\mathrm{static}}(T)$ with a kinematic term. The kinematic state includes: linear velocity $\|v\|$, angular velocity $\|\omega\|$, linear acceleration $\|a\|$, angular acceleration $\|\dot{\omega}\|$, and jerk $\|\dot{a}\|$. The following candidates are proposed for experimental validation. Coefficients are fitted from residuals $R_a(t), R_\omega(t)$ over S1-S3; the study determines which formulations generalise to T3 (held-out) and improve metrics vs. M1-M3.

| # | Formulation | Formula | Notes |

|---|-------------|---------|-------|
| 1 | **Additive linear** | $\sigma^2 = \sigma^2_{\mathrm{static}} + c_v \|v\| + c_\omega \|\omega\| + c_a \|a\| + c_{\dot{\omega}} \|\dot{\omega}\| + c_j \|\dot{a}\|$ | Requires explicit units per coefficient; risk of $\sigma^2 < 0$ if coefficients are negative. |
| 2 | **Multiplicative** | $\sigma^2 = \sigma^2_{\mathrm{static}} \cdot \max\bigl(1 + c_v \|v\| + c_\omega \|\omega\| + c_a \|a\| + c_{\dot{\omega}} \|\dot{\omega}\| + c_j \|\dot{a}\|,\; \epsilon\bigr)$ | Guarantees $\sigma^2 > 0$; all $c \geq 0$; dimensionless scaling. |
| 3 | **Single term (g-sensitivity)** | $\sigma^2 = \sigma^2_{\mathrm{static}} \cdot \bigl(1 + c_\omega \|\omega\|\bigr)$ | One parameter; better identifiability; ignores $v$, $a$, $\dot{\omega}$, $\dot{a}$. |
| 4 | **Conservative scaling** | $\sigma^2 = \sigma^2_{\mathrm{static}} \cdot k_{\mathrm{dyn}}$, $k_{\mathrm{dyn}} \geq 1$ | Single factor during dynamic phases; supported by practice (e.g. Kalibr: static params underestimate low-cost IMU uncertainty). |

The chosen formulation(s) are reported in the manuscript; candidates that fail to generalise or improve metrics are discarded and the limitation stated.

#### 5.4.1 Sensor-specific coefficient pruning

Not all kinematic terms are physically relevant for every sensor. Coefficients are fixed to zero when the underlying mechanism does not apply, reducing parameters and improving identifiability. The following pruning is applied by default:

| Sensor | Coefficient fixed to 0 | Rationale |

|--------|-------------------------|-----------|
| **IMU** | $c_v$ (linear velocity) | The IMU measures acceleration and angular rate, not velocity. Velocity is an integrated quantity; it does not directly affect IMU noise. G-sensitivity and vibration depend on $a$, $\omega$, $\dot{a}$, $\dot{\omega}$, not on $v$. |
| **IMU** | $c_{\dot{\omega}}$ (optional) | Angular acceleration may be collinear with jerk in many trajectories; if identifiability is poor, $c_{\dot{\omega}} = 0$ can be applied. |
| **LiDAR** | (none by default) | All terms potentially relevant: $\|v\|$ and $\|\omega\|$ for motion blur during scan; $\|a\|$, $\|\dot{\omega}\|$, $\|\dot{a}\|$ for structural vibration. Prior simulation-based work [[27]](REFERENCES_CONSOLIDATED.md) suggests range may be robust to vibration while angular accuracy degrades, but this has not been validated experimentally. Pruning only if data show non-identifiability. |
| **Camera (RGB-D)** | $c_{\dot{\omega}}$, $c_j$ (optional) | Motion blur is dominated by $\|v\|$ and $\|\omega\|$. Acceleration and jerk have weaker direct effect; can be pruned if not identifiable. |

IMU default: $c_v = 0$ always. The remaining terms $c_\omega, c_a, c_{\dot{\omega}}, c_j$ are fitted; $c_{\dot{\omega}}$ may be set to 0 if VIF or condition number indicate collinearity with $c_j$.

LiDAR / Camera: Start with the full set; prune only after checking identifiability (VIF threshold < 10, condition number) or if a coefficient is not significant in the residual fit.

Model selection across candidate formulas and pruning choices: AIC/BIC. Fallback if collinearity persists after pruning: ridge regression with leave-one-out cross-validation.

### 5.5 Sensor-Specific Instantiation

- **IMU:** $\sigma^2_{\mathrm{static}}(T)$ models the evolution of ARW/BI with temperature; the kinematic term (per chosen candidate) captures increased noise under rotation, acceleration and jerk (g-sensitivity and structural vibration).
- **LiDAR:** $\sigma^2_{\mathrm{static}}(T)$ and a time-varying range bias model capture ToF drift; kinematic coefficients capture increased spread of point-to-plane residuals under high rotational speed and acceleration. Whether kinematic terms are identifiable and significant for the Mid-360 is an open empirical question addressed by this study.
- **Camera (RGB-D):** $\sigma^2_{\mathrm{static}}(T)$ approximates thermal drift of intrinsics and depth noise; angular-velocity term captures effective degradation due to motion blur.

### 5.6 M4 Generalization Check (held-out trajectory)

To demonstrate that M4 does not overfit to the specific trajectories used for fitting, kinematic coefficients and thermal parameters are fitted using only T1 (smooth) and T2 (moderate) data. The model is then evaluated on T3 (aggressive) as a held-out set. If M4 residual distributions on T3 are closer to real hardware than M1-M3, the model generalizes beyond the fitting regime. This protocol is reported explicitly; if generalization fails, the limitation is stated in the manuscript.

The YuMi geometric model used in the LiDAR self-filter is taken from publicly available ROS description packages such as `abb_irb14000_description` (KTH), which provide URDF and mesh files for the IRB 14000. This ensures the same approximate link geometry is available in both the real-data self-filter and any Gazebo Fortress configuration where the robot is visualised.

---

## 6. Residual Comparison Pipeline (Primary)

The primary comparison pipeline for each session operates entirely at the sensor residual level. No SLAM estimation step is required.

```txt
Pipeline Step 1: Real hardware data collection
  → YuMi dynamic session (T1/T2/T3 × MIX/CW/CCW blocks) for each sensor (Sessions A-D)
  → Raw sensor logs + YuMi ground truth (RobotStudio flange pose export)
  → CAD-derived T_sensor→flange: sensor true kinematics expressed in sensor frame (§5.1)
  → Residual computation: R_real(t) = measured(t) - true_YuMi(t), per §2.1
  → Kinematic segmentation into S0-S3 regimes (§5.2)
  → M4 coefficient fitting on T1 + T2 residuals (§5.3-5.4)

Pipeline Step 2: Simulated data
  → Import YuMi trajectory into Gazebo Fortress (identical joint commands)
  → Run simulation under each noise model M1 / M2 / M3 / M4
  → Custom plugin: M4 state-dependent covariance injection; Rosetta topology for Session C
  → Residual computation: R_sim(t) = simulated_output(t) - simulator_true(t)

Pipeline Step 3: TOST equivalence testing
  → Compare residual distributions R_real vs R_sim per noise model
  → TOST per sensor type, per session (A-D), T2 as primary trajectory (§3.4)
  → δ per sensor type as defined and locked in §3.4
  → Holm-Bonferroni correction across sessions (A-D)
  → T3 residuals evaluated as held-out generalization check for M4 (§5.6)
```

**Critical constraint:** Simulation uses the identical YuMi trajectory command as the real session. The noise model parameters (M1-M4) are the only varying factor between conditions.

---

## 7. CW/CCW Asymmetry Analysis

Secondary hypothesis H2: gyroscope bias drift differs between CW and CCW rotation.

Method:

1. For each sensor, extract the end-of-trajectory orientation error (vs. ground truth) for all CW repetitions and all CCW repetitions separately
2. Compute mean and standard deviation of yaw error for CW group vs. CCW group
3. Test for significant difference: paired t-test or Wilcoxon (same selection criterion as §3.1)

Expected result: A non-negligible CW/CCW asymmetry is expected in MEMS gyroscopes due to mechanical asymmetry in the Coriolis structure. Its magnitude relative to the noise floor determines whether it is relevant to practical navigation performance.

Reporting: IMU angular rate residuals and yaw error broken down by rotation direction, with statistical comparison.

---

## 8. Frequency-Sweep Trajectory (Optional)

**Status:** Optional. Add only if core protocol is completed and time permits.

Motivation: Identify the empirical cut-off angular velocity above which sensor residuals exceed acceptable bounds for the intended application. This is rarely characterized in the literature and could be a differentiating result.

Design: Sinusoidal yaw rotation with linearly increasing frequency over time (chirp signal):

- Angular rate $\omega(t) = A \cdot \sin(2\pi \cdot f(t) \cdot t)$, where $f(t)$ increases from 0.05 Hz to 2 Hz over 10 min
- Amplitude A selected to stay within YuMi dynamic limits

Metric: sensor residuals computed in sliding windows. Plot mean sensor residuals vs. instantaneous angular frequency. The cut-off frequency is defined as the frequency at which mean sensor residuals exceed 3× the T1 static residual baseline (practical threshold for "unacceptable" drift; adjustable in analysis).

This experiment applies to all four sessions (A, B, C, D) and provides a sensor-comparative view of dynamic limits.

---

## 9. Analysis Toolchain Summary

| Task | Tool | Notes |

|------|------|-------|
| Allan Variance | `imu_utils` (ROS) / `allantools` (Python) | Overlapping ADEV estimator; chi-squared CI, k = 2 |
| Six-Position Test | Custom Python script | Manual per-axis scale factor and bias extraction |
| IMU residual computation | Custom Python (spline differentiation + CAD T_sensor→flange) | Cubic B-spline on SE(3) flange poses; residuals per §5.1 |
| LiDAR point-to-plane residuals | `Open3D` or `PCL` in Python | RANSAC plane fit at t = 0; orthogonal residuals per frame |
| Camera AprilTag pose drift | `apriltag_ros` | 6-DOF pose at 1 Hz from static board; drift relative to t = 0 |
| Trajectory format conversion | `evo` utilities | TUM / KITTI / EuRoC to TUM for evo compatibility |
| Stationarity test (AVAR pre-condition) | `statsmodels` (ADF, KPSS) | Applied to each static log before AVAR fitting |
| Normality test | `scipy.stats.shapiro` | Applied to per-repetition mean residuals before TOST |
| Equivalence testing (TOST) | `scipy.stats` + manual 90% CI | Or `pingouin` (`tost` function); power analysis via [[7]](REFERENCES_CONSOLIDATED.md) |
| Standard NHST | `scipy.stats` | t-test or Wilcoxon; effect size (Cohen's d or rank-biserial) |
| Visualization | `matplotlib`, `evo` plot tools | ADEV curves, residual time series, box plots, trajectory overlays |
| Simulation (standard, M1) | Gazebo Fortress default plugins | Baseline condition S |
| Simulation (metrological, M2-M4) | Custom Gazebo Fortress plugin (to be developed) | Condition M; open-source output; ROS 2 Humble compatible |
