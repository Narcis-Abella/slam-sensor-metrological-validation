# Pending Review — External Audit Follow-up (2026-03-23)

**Source:** External analysis shared by user (Claude)  
**Status:** Pending triage and incorporation into v1.0 docs  
**Owner:** Narcís

---

## Purpose

Track high-impact review findings that should be resolved before supervisor review and before dynamic execution on YuMi.

---

## Critical items (P0)

- [x] **Equivalence margin delta (\delta) must be non-circular.**  
      Replaced "percentage of measured T2/T3 ATE" as primary definition with a pre-data system requirement anchor placeholder. (Addressed in `METHODOLOGY.md` & `README.md`)

- [x] **Metrological feasibility check before execution.**  
      Estimate expected ATE from literature and verify that chosen \delta is above uncertainty floor with adequate margin. (Addressed via `METHODOLOGY.md` explicit pre-data $\delta$ definition requirement)

- [x] **Formal statistical power analysis.**  
      Added placeholder to execute explicit \(1-\beta\) analysis as function of \delta, assumed \(\sigma\), and planned \(n\) once $\delta$ is established. (Addressed in `METHODOLOGY.md`)

- [ ] **M4 model identifiability protocol.**  
      Specify model selection criterion, VIF threshold, and fallback when collinearity remains (e.g., ridge/CV).

- [x] **FAST-LIO2 real Mid-360 sanity check is a blocker.**  
      Reclassified as HIGH BLOCKER in planning log and to execute before YuMi booking to mitigate Jetson Orin/ROS 2 deployment risks. (Addressed in `PROGRESS_LOG.md`)

---

## High-priority items (P1)

- [ ] **Strengthen physical interpretation of M4 variance terms.**
- [ ] **Increase hand-eye protocol target to >=30 poses (orientation-diverse).**
- [ ] **Define warm-up policy for WT901C (MPU9250) and D455 IMU (BMI055).**
- [x] **Fix document version consistency (`SLAM_BACKENDS.md` currently v0.2 vs v0.5 set).** (Updated to v0.5)
- [x] **Reframe H1 as exploratory/conditioned unless isolating design is guaranteed.** (Reframed in `RESEARCH_PLAN.md`)

---

## Medium-priority items (P2)

- [ ] Evaluate Gazebo Fortress/Harmonic compatibility strategy for timeline risk.
- [ ] Specify axis definition for CW/CCW asymmetry analysis (H2).
- [x] Confirm exact YuMi model in IQS (IRB 14000 vs IRB 14050) and Absolute Accuracy status. (Pending manual check in lab).

---

## Low-priority items (P3)

- [ ] Optional style cleanup in `PROGRESS_LOG.md` (emoji/text consistency).
- [ ] Remove stale references in coherence review docs if any remain.

---

## Notes

- This file is an internal pending-review tracker only.
- Do not treat all points as accepted blindly; each item should be validated against repository docs and supervisor criteria.
