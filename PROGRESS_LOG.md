# Progress Log — Phase I: Metrological Validation of Visual-LiDAR SLAM Sensors

> This is the **living chronological tracker** of Phase I. Every decision, milestone, completed task, open item, and status change is recorded here in chronological order. Nothing is deleted; entries are only appended.

**Format:** `### YYYY-MM-DD — [Event type]: Title`  
**Event types:** `DECISION` | `MILESTONE` | `TASK_DONE` | `OPEN_ITEM` | `SUPERVISOR` | `BLOCKER` | `NOTE`

---

## Current Status Summary

> Updated: 2026-03-07

| Area | Status | Next action |
|------|--------|-------------|
| Research Plan | ✅ v0.3 drafted | Supervisor review → v1.0 |
| Experimental Design | ✅ v0.3 drafted | Supervisor review |
| Methodology | ✅ v0.3 drafted | Supervisor review |
| Hardware & Payload | ✅ v0.3 drafted | Physical weight verification |
| Static characterization | ❌ Not started | Pending hardware availability |
| Dynamic sessions | ❌ Not started | Pending: weight verification + YuMi booking |
| Simulation plugin | ❌ Not started | Pending: static characterization results |
| FAST-LIO2 baseline | ❌ Not started | Pending: real sensor availability (Mid-360 confirmed) |
| Statistical analysis | ❌ Not started | Pending: data collection |
| Publication draft | ❌ Not started | Pending: results |
| GitHub repository | ✅ Pushed | [github.com/Narcis-Abella/slam-sensor-metrological-validation](https://github.com/Narcis-Abella/slam-sensor-metrological-validation) |

---

## Documentation Milestones

> Documentation tasks from the Phase I audit (2026-03). v1.0 = post–supervisor review.

| Milestone | Status | Notes |
|-----------|--------|-------|
| README public-ready (motivation, plain language, links to refs) | ✅ Done | Section 0 + §1 restructured; citations linked to RESEARCH_PLAN §9. |
| Audit of references (all citations have entry in §9) | ✅ Done | Cross-checked README + docs; no orphan citations. |
| Ground truth uncertainty budget (METHODOLOGY §2.4) | ✅ Done | YuMi + hand-eye + sync; ATE/RPE interpreted above floor. |
| Sharpen language (HARDWARE_PAYLOAD, EXPERIMENTAL_DESIGN, SLAM_BACKENDS) | ✅ Done | Replaced colloquial/vague terms with neutral formulations. |
| v1.0 RESEARCH_PLAN / EXPERIMENTAL_DESIGN / METHODOLOGY / HARDWARE_PAYLOAD | ⏳ Pending | After supervisor review; implement feedback. |
| v1.0 README "public-ready" (final pass post–supervisor) | ⏳ Pending | Minor tweaks if supervisor requests changes to framing. |

---

## Open Items

> Sorted by priority. Check off when resolved and add entry to log.
>
> **Hard prereqs for YuMi booking:** weight verification, sync method defined, mounts designed, supervisor sign-off on protocol. **Can run in parallel:** static characterization (no YuMi), documentation review, FAST-LIO2 sanity check. **Planned:** YuMi dynamic sessions in June (after exams); simulation and plugin development from July onwards. Session order is at the author’s discretion.

- [x] **[BLOCKER — HIGH]** Confirm Livox Mid-360 availability. *(Done — Mid-360 confirmed for Session C.)*
- [ ] **[BLOCKER — HIGH]** Verify actual sensor + mount weights with physical scale. Session C (Mid-360) is at 70% payload margin based on estimates — must confirm before booking YuMi.
- [ ] **[MEDIUM — optional extension]** Secure access to an **industry-standard repetitive (spinning) 360° LiDAR** (e.g. Velodyne VLP-16/32, Ouster OS0/OS1, Hesai) — widely cited in SLAM, navigation and mapping literature. Would enable extension of the study with LIO-SAM and similar backends; not a blocker for the current scope.
- [ ] **[HIGH]** Supervisor review of v0.3 documents (RESEARCH_PLAN, EXPERIMENTAL_DESIGN, METHODOLOGY, HARDWARE_PAYLOAD). Implement feedback → v1.0.
- [ ] **[HIGH]** Define temporal synchronization method: PTP IEEE 1588 vs. NTP. Assess IRC5 network capabilities.
- [ ] **[MEDIUM]** Design and 3D-print sensor mounts for each session. Mount design must be done before any dynamic session.
- [ ] **[MEDIUM]** Identify and book a suitable YuMi session slot (pending weight verification and supervisor sign-off on protocol).
- [ ] **[MEDIUM]** Verify FAST-LIO2 operation with real Livox Mid-360 data (sensor confirmed). Must establish real-hardware baseline before simulation comparison.
- [ ] **[MEDIUM]** Assess availability of metrology or optics laboratory at IQS for static characterization sessions (better temperature control).
- [ ] **[LOW]** Decide waypoint strategy for T3 3D aggressive trajectory: fixed set vs. pseudo-random with fixed seed. Confirm with supervisor.
- [x] **[LOW]** Push repository to GitHub remote (create public repo under github.com/Narcis-Abella). *(Done — repo at github.com/Narcis-Abella/slam-sensor-metrological-validation)*
- [ ] **[LOW]** Confirm Mitsubishi robot access conditions (permission, PLC programming, payload/repeatability specs).

---

## Changelog / Chronological Log

---

### 2026-02-16 — MILESTONE: Original three-phase proposal submitted

- Submitted "Comprehensive Validation and AI-Driven Optimization of Visual-LiDAR SLAM for Mobile Robotics — A Three-Stage Research Framework" to Dr. Salazar.
- Proposal covered: (I) Metrological sensor validation, (II) Reality Gap benchmark (mobile platform), (III) Bayesian SLAM optimization.
- Hardware at time of proposal: Livox Mid-70, RPLiDAR A2M12, RealSense D455, WitMotion WT901C, AgileX Tracer 2.0.

---

### 2026-02 (approx.) — SUPERVISOR: Salazar feedback — strategic reorientation

**Source:** Email from Dr. A.G. Salazar Martín (exact date TBC — add when available)

**Key decisions from supervisor feedback:**
1. **Scope:** Focus exclusively on Phase I (metrological validation). Do not include Reality Gap or optimization in this paper.
2. **Ground truth:** Use ABB YuMi as kinematic ground truth (path repeatability 0.10 mm, path accuracy up to 1.36 mm; pose repeatability ±0.02 mm at static points only).
3. **RobotStudio:** Use as independent trajectory predictor.
4. **Payload concern:** YuMi arm capacity ~500 g. Original Livox Mid-70 (~590 g) exceeds capacity without mount. Requires review.
5. **Static IMU logs:** 10–12 h, temperature documented. Extract ARW, BI, RRW numerically with confidence intervals.
6. **Dynamic trajectories:** Three types (smooth / moderate / aggressive), defined in RobotStudio, repeated for variability analysis.
7. **FAST-LIO2 baseline:** Verify with real hardware before any simulation work.
8. **Statistical test:** Explicit test (t-test / Wilcoxon), $\alpha = 0.05$. Define success criterion explicitly.

---

### 2026-02/03 (approx.) — NOTE: Response and plan from Narcís — v0.1 experimental plan

**Key additions from Narcís's response:**
1. **LiDAR change proposed:** Mid-360 (~250 g) vs. Mid-70 (~590 g). Resolves payload problem. Mid-360 additionally: Rosetta pattern, factory LiDAR-IMU extrinsics, native GLIM support, higher community adoption.
2. **Three IMUs:** WT901C (MPU9250), D455 (BMI055), Mid-360 (ICM40609) — three quality grades for H1 (IMU quality effect on tight coupling).
3. **Four sessions defined:** A (IMU only), B (camera+IMU), C (2D LiDAR), D (3D LiDAR).
4. **Block structure defined:** MIX / CW / CCW blocks × 3 repetitions, 30 min cooling between blocks.
5. **Trajectory structure:** T1/T2/T3 for 2D and 3D, with velocity and angular rate parameters.
6. **Controlled environment:** Foam panels + LED stable lighting + temperature logging.

---

### 2026-02/03 (approx.) — NOTE: Methodological addenda from Narcís — v0.2 plan

**Key additions:**
1. **Temporal synchronization:** PTP IEEE 1588 preferred. NTP fallback with jitter measurement. Mandatory to document — any reviewer will flag missing synchronization protocol.
2. **Hand-eye calibration:** AX=XB required to map flange pose to sensor center. Without it, ground truth has a systematic offset that invalidates all metrics. Must be redone per session.
3. **Six-Position Test (IEEE Std 1293):** Before long IMU logs. Isolates scale factor and gravitational bias — not obtainable from horizontal logs alone. Critical for tight-coupling initialization.
4. **LiDAR static characterization:** Planar orthogonal residual method (not absolute distance). Avoids ICP errors, isolates thermal ToF drift.
5. **Camera static characterization:** AprilTag board for 3–4 h. Quantifies thermal intrinsic drift and FPN in RealSense D455. Documents Reality Gap origin.
6. **Settling time verification for T3:** Measure vibration PSD during inter-waypoint pauses. Criterion: amplitude < 2× noise floor before accepting pause as valid.
7. **evo for trajectory evaluation:** Standard tool in the community. Umeyama alignment for ATE and RPE. Facilitates comparison with other published results.
8. **Optional — frequency sweep trajectory:** Empirical cut-off frequency identification. Not in core protocol; add if time permits.

---

### 2026-03-06 — MILESTONE: Repository initialized and documentation drafted (v0.3)

**Tasks completed this session:**
- Created repository structure locally: `C:\Users\narci\Documents\Cursor\slam-phase1\`
- Drafted all core documentation files:
  - `README.md` — public entry point and overview
  - `docs/RESEARCH_PLAN.md` — full research plan (motivation, hypotheses, platform, contributions)
  - `docs/EXPERIMENTAL_DESIGN.md` — session-by-session protocol, trajectory parameters, synchronization, hand-eye calibration
  - `docs/METHODOLOGY.md` — Allan Variance, ATE/RPE, statistical test design, three-condition pipeline
  - `docs/HARDWARE_PAYLOAD.md` — sensor specs, weight breakdown, Mid-360 vs. Mid-70 rationale, verification checklist
  - `PROGRESS_LOG.md` — this file

**Document version:** All files at v0.3 (design phase, pending supervisor review)

**Open items identified (added to tracker above):**
- Physical weight verification (priority before YuMi booking)
- Mid-360 purchase confirmation
- Supervisor review of v0.3 documents
- Synchronization method decision (PTP vs. NTP)
- Mount design and printing

---

### 2026-03-07 — MILESTONE: Documentation audit completed (plan tasks 1–5)

**Documentation improvements applied:**
- README: Section 0 (motivation, prior work, gap, core contribution) restructured; citations in §0 and §5.1 linked to RESEARCH_PLAN §9.
- References: Full audit; all cited studies have entries in RESEARCH_PLAN §9.
- METHODOLOGY §2.4: Ground truth uncertainty budget (YuMi repeatability, hand-eye, sync); ATE/RPE interpreted above floor.
- SLAM_BACKENDS: Justification column per backend; §7 extension table for repetitive 360° LiDAR.
- HARDWARE_PAYLOAD, EXPERIMENTAL_DESIGN, SLAM_BACKENDS: Colloquial/vague language replaced with neutral formulations.
- PROGRESS_LOG: Documentation milestones table added; hard prereqs vs parallel tasks clarified in Open Items.

---

### 2026-03-07 — DECISION: Mid-360 confirmed; R3LIVE excluded; M4 hold-out; timeline; plugin contribution

**Hardware & scope:**
- Livox Mid-360 confirmed available for Session C. Open item "Confirm Mid-360 availability" marked done.
- Repetitive (spinning) 360° LiDAR added as optional extension: target industry-standard, widely cited (Velodyne, Ouster, Hesai) for possible future extension; not a blocker.

**Methodology:**
- METHODOLOGY §5.5: M4 generalization check — fit on T1+T2, evaluate on T3 (held-out) to demonstrate no overfitting.

**Session D backends (visual-inertial):**
- R3LIVE excluded from this study: ideally would test camera+LiDAR fusion, but Session D is RGB-D+IMU only; R3LIVE expects LiDAR+camera and has limited ROS 2 fit. Noted as future extension.
- B3 replacement: **OpenVINS** (EKF-based VIO, tight-coupled, ROS 2, distinct from ORB-SLAM3 and GLIM). SLAM_BACKENDS §4.4 and RESEARCH_PLAN §7 updated; README §6 updated.

**Timeline:**
- YuMi dynamic sessions planned for June 2026 (after exams). Simulation and plugin work from July 2026. Session order at author’s discretion. README §8 and Open Items updated.

**Contributions:**
- RESEARCH_PLAN §6.4: Explicit contribution added — Gazebo plugin for **non-repetitive scan pattern** (Mid-360 Rosetta, optionally Mid-70) as part of expected open-source output.

---

## How to Use This Log

**When to add an entry:**
- Any supervisor meeting or feedback received
- Any decision made that affects protocol design
- Any task completed (static log, calibration, session, analysis step)
- Any blocker identified or resolved
- Any change to the experimental design or methodology
- Any publication milestone (submission, revision, acceptance)

**Entry format:**

```markdown
### YYYY-MM-DD — [TYPE]: Title

- Bullet point with key information
- Include: who was involved, what was decided, what action follows
- Link to relevant document if applicable
```

**What NOT to put here:**
- Raw data (goes in `data/`)
- Analysis results (goes in `results/`)
- Code (goes in `analysis/`)
- This file is for process documentation only.
