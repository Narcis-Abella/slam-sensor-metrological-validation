# ABB YuMi — Datasheet Reference & Model Verification

**Purpose:** Consolidated reference of all YuMi specifications found in ABB documentation. Includes clarification on model identification source.

**Last updated:** 2026-03-07

---

## 1. Source of Model Identification

### ⚠️ Critical: We are NOT certain the IQS robot is exactly IRB 14000

| Question | Answer |
|----------|--------|
| **Where does "IRB 14000 YuMi" come from?** | From the **project's own documentation** — RESEARCH_PLAN.md, HARDWARE_PAYLOAD.md, and supervisor feedback (Salazar, Feb 2026) which specified "ABB YuMi as kinematic ground truth." |
| **Did the AI infer or guess the model?** | **No.** The model IRB 14000 was already declared in the project. The AI looked up specifications for that declared model. |
| **Can we confirm IQS has exactly IRB 14000?** | **No.** This requires verification with the IQS lab, supervisor, or facility manager. |

### ABB YuMi variants (both exist; specs may differ)

| Model | Configuration | Controller | Notes |
|-------|----------------|------------|-------|
| **IRB 14000** | Dual-arm (2× 7-DOF) | IRC5 Single Cabinet | Original YuMi; table-mounted. Project assumes this. |
| **IRB 14050** | Single-arm (1× 7-DOF) | OmniCore C30 | Newer; lighter (9.5 kg); flexible mounting. Same repeatability ±0.02 mm, payload 0.5 kg. |

**Action required:** Confirm with IQS which exact model is available (IRB 14000, IRB 14050, or other). If it is IRB 14050, the path accuracy (AT/RT) values below may differ — the datasheet consulted was for IRB 14000 only.

---

## 2. ABB Product Specification — IRB 14000 (YuMi)

**Document:** ABB Product specification - IRB 14000 (YuMi)  
**Document ID:** 3HAC052982-010  
**Revision:** U (checked in 2025-11-11)  
**Source:** [ABB Library](https://library.e.abb.com/public/2155da52da5740108beebce7488b0cb8/3HAC052982%20PS%20IRB%2014000-zh-cn.pdf) (Chinese version; English available via [search.abb.com](https://search.abb.com/library/Download.aspx?Action=Launch&DocumentID=3HAC052982-001&LanguageCode=en))

### 2.1 ISO 9283 Performance (Section 1.8.2)

Test conditions (from datasheet): *"At rated maximum load, maximum offset, and 1.5 m/s speed on inclined ISO test surface, all 6 axes in motion. Values are average measurements from a small number of robots. Results may vary with robot position in workspace, speed, arm structure, approach direction, and load direction. Gear backlash also affects results."*

| Parameter | ISO 9283 symbol | Value | Description |
|-----------|-----------------|-------|-------------|
| Pose repeatability | RP | **0.02 mm** | Repeatability when returning to the same programmed point. |
| Pose accuracy | AP | **0.02 mm** | Accuracy of reaching programmed positions. |
| Linear path repeatability | RT | **0.10 mm** | Repeatability when following the same path repeatedly. |
| Linear path accuracy | AT | **1.36 mm** | Maximum deviation of actual path from programmed path. |
| Pose stabilization time | Pst | 0.37 s (within 0.1 mm) | Time to settle at target. |

**Interpretation for ground truth:**
- **RP (0.02 mm):** Relevant for point-to-point tasks. Our docs have used this as the dominant mechanical term.
- **AT (1.36 mm):** Relevant for **trajectory comparison** (ATE/RPE). The actual path can deviate up to 1.36 mm from the programmed path under worst-case conditions (max load, 1.5 m/s).
- **RT (0.10 mm):** Path repeatability — consistency when repeating the same path. Lower than AT.

Our trajectories (T1 ~15–20 mm/s, T2 ~40–50 mm/s, T3 ~80–100 mm/s) are slower than the 1.5 m/s test condition, so actual path accuracy may be better than 1.36 mm. Conservative approach: use 1.36 mm in uncertainty budget unless measured.

### 2.2 General Specifications

| Parameter | Value |
|-----------|-------|
| Type | Dual-arm collaborative robot |
| DOF per arm | 7 |
| Payload per arm | 0.5 kg |
| Reach | 0.559 m |
| Repeatability (position) | ±0.02 mm |
| Max TCP speed | 1.5 m/s |
| Max TCP acceleration | 11 m/s² |
| Controller | IRC5 |
| Programming | RobotStudio (offline trajectory + playback) |
| Mounting | Table or flat surface only |
| Protection | IP30 |
| Clean room | Class 5 (optional) |

### 2.3 Calibration Options

- **Standard calibration** (default)
- **Absolute Accuracy** (optional): Improves Cartesian positioning accuracy. Typical production data: mean global accuracy ~0.5 mm, max ~1 mm, high % within 1 mm. Requires recalibration after structural maintenance.
- **Wrist Optimization:** Improves reorientation accuracy for welding, dispensing, etc.

### 2.4 Standards

- ISO 9283:1998 — Manipulating industrial robots, performance criteria and test methods
- ISO 10218-1 — Safety requirements for industrial robots

---

## 3. Implications for METHODOLOGY.md §2.4

The ground truth uncertainty budget should include:

| Source | Value | Notes |
|--------|-------|-------|
| YuMi pose repeatability (RP) | ±0.02 mm | Point-to-point. |
| YuMi path accuracy (AT) | 1.36 mm | Path-following; worst-case per datasheet. Use in trajectory (ATE/RPE) budget. |
| YuMi path repeatability (RT) | 0.10 mm | Alternative if path repeatability is the relevant metric. |
| Hand-eye calibration | < 0.5 mm, < 0.5° | Per session. |
| Temporal sync | v_max × Δt | e.g. 1 mm at 100 mm/s, 10 ms offset. |

---

## 4. Verification Checklist

- [ ] Confirm with IQS lab/supervisor: exact model (IRB 14000 vs. IRB 14050 vs. other).
- [ ] If IRB 14050: obtain its product specification (3HAC064627) and check AT/RT values.
- [ ] Verify controller type (IRC5 vs. OmniCore) for PTP/NTP sync capability.
- [ ] Document whether Absolute Accuracy calibration is installed on the IQS unit.

---

*This document is for internal reference. Update when IQS model is confirmed.*
