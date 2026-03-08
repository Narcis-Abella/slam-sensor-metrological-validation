# Revisión de coherencia científica — Cambios post-auditoría

**Fecha:** 2026-03-08  
**Alcance:** Verificación exhaustiva de los cambios implementados (n, ground truth 0.02→0.10/1.36, modelo térmico T, nomenclatura A–D, referencias circulares) en todos los documentos del repositorio.

---

## 1. Objetivo del estudio (recordatorio)

- **H0:** Simulación metrológica (M) equivalente a hardware real (R) dentro del margen δ (TOST).
- **H1:** La calidad del IMU afecta sistemáticamente al rendimiento SLAM (ATE/RPE).
- **H2:** Asimetría CW/CCW observable en bias del giroscopio.
- **Condiciones:** R (real), S (sim estándar), M (sim metrológica M1–M4). Comparación por sesión y tipo de trayectoria (T1/T2/T3).

---

## 2. Nomenclatura de sesiones (A, B, C, D)

| Sesión | Sensor | Modalidad | Coherencia comprobada |
|--------|--------|-----------|------------------------|
| **A** | WitMotion WT901C | IMU-only (3D) | EXPERIMENTAL_DESIGN §4, METHODOLOGY §3.5 (no en endpoints primarios), SLAM_BACKENDS §4.1, RESEARCH_PLAN tabla S1→A, README, HARDWARE_PAYLOAD, PROGRESS_LOG, AUDIT. |
| **B** | RPLiDAR A2M12 | 2D LiDAR (yaw only) | EXPERIMENTAL_DESIGN §5, SLAM_BACKENDS §4.2, RESEARCH_PLAN S2→B, README, HARDWARE_PAYLOAD (Session B 2D-only), AUDIT (Session B 2D LiDAR). |
| **C** | Livox Mid-360 | 3D LiDAR + IMU | EXPERIMENTAL_DESIGN §6, METHODOLOGY endpoints primarios (Session C Mid-360), SLAM_BACKENDS §4.3, RESEARCH_PLAN S3→C, README, HARDWARE_PAYLOAD (payload 70%), PROGRESS_LOG, AUDIT, REFERENCES_POOL (plugin Rosetta → Session C). |
| **D** | RealSense D455 | RGB-D + IMU (3D) | EXPERIMENTAL_DESIGN §7, METHODOLOGY endpoints primarios (Session D D455), SLAM_BACKENDS §4.4, RESEARCH_PLAN S4→D, README, HARDWARE_PAYLOAD (illumination Session D), PROGRESS_LOG (Session D backends), REFERENCES_POOL (Session D visual/depth). |

**Corrección aplicada durante la revisión:** `docs/REFERENCES_POOL.md` citaba "Session B (visual/depth)" para RealSense; actualizado a **Session D**.

---

## 3. Tamaño muestral (n) y estructura de bloques

- **Definición única:** Cada bloque (MIX, CW, CCW) dura ~2 h. Cada repetición: [60 s estático] → [trayectoria 1–2 min] → [60 s estático]. Eso da **n ≈ 40–60 por bloque** y **n ≈ 120–180 por tipo de trayectoria** (T1, T2 o T3). CW y CCW cuentan como repeticiones del mismo tipo de trayectoria para el TOST (M vs R).
- **Dónde está:** METHODOLOGY §3.1 (Step 1) y §3.4 (Power analysis); EXPERIMENTAL_DESIGN §3 (Block structure). Sin restos de n=9 en la metodología activa.
- **Nota:** En `internal/AUDIT_CONSOLIDATED_REVIEW.md` las menciones a n=9 se mantienen como **histórico** del problema detectado; se añadió una nota de actualización (n≈120–180 y σ²_static(T)) en la sección de fórmula M4 para no contradecir METHODOLOGY.

---

## 4. Ground truth: pose vs path

- **Pose repeatability (RP) ±0.02 mm:** Solo puntos estáticos (inicio/fin de trayectoria, waypoints T3). No se usa como cota dominante en tramos dinámicos.
- **Path repeatability (RT) 0.10 mm** y **path accuracy (AT) hasta 1.36 mm:** Cotas relevantes para ATE/RPE en trayectorias dinámicas. Presupuesto combinado en METHODOLOGY §2.4 usa 0.10 + hand-eye + v_max Δt (ejemplo).
- **Comprobado en:** METHODOLOGY §2.4 (tabla y párrafo combinado), EXPERIMENTAL_DESIGN §9 (sync) y §10 (hand-eye), README (§1, §3, §9), RESEARCH_PLAN §5.1, HARDWARE_PAYLOAD §1, PROGRESS_LOG, internal/YUMI_DATASHEET_REFERENCE (RP vs RT/AT), AUDIT (Dimensión 3 y 4). Referencia [16] en RESEARCH_PLAN mantiene "±0.02 mm per datasheet" como dato de especificación ABB (correcto).

---

## 5. Modelo térmico M4 (temperatura T)

- **Cambio:** $\sigma^2_{\mathrm{static}}(t_{\mathrm{on}})$ sustituido por $\sigma^2_{\mathrm{static}}(T)$ con **temperatura medida** $T(t)$. Idealmente T se registra con termómetro calibrado o topics de temperatura del sensor (Mid-360, D455) durante caracterización estática y sesiones YuMi.
- **Dónde está:** METHODOLOGY §5.3 (objetivo, fórmula, medición de T), §5.4 (combinación con término cinemático), §5.5 (instanciación por sensor); README §5.1 (fórmula y texto); RESEARCH_PLAN (§3.6 novedad, §6.4 plugin); EXPERIMENTAL_DESIGN §3 (cooling: registrar T para M4); SLAM_BACKENDS §2 (M4 dependiente de T); internal/AUDIT (fórmula M4 actualizada a T y nota post-auditoría).
- **Sin restos de** "time since power-on" o "t_on" en la metodología activa (docs/ y README). Solo queda en el historial del AUDIT, con la aclaración de actualización.

---

## 6. Referencias circulares (M1–M4, S0–S3)

- **Fuente única:** METHODOLOGY.md §4 (tabla M1–M4 y tabla S0–S3). §5.2 remite a "§4 above" para regímenes cinemáticos.
- **SLAM_BACKENDS.md §2:** No duplica tablas; remite a METHODOLOGY §4 y §5 para definiciones y fórmulas. §3 (Dataset taxonomy) ya no incluye la tabla S0–S3.
- Flujo de lectura: METHODOLOGY → definiciones físicas; SLAM_BACKENDS → listado de backends por sesión y referencia a METHODOLOGY.

---

## 7. Endpoints primarios y coherencia con sesiones

- **Endpoints primarios (TOST + Holm-Bonferroni):**
  - ATE RMSE FAST-LIO2, **Session C** (Mid-360), T2
  - ATE RMSE ORB-SLAM3, **Session D** (D455), T2
  - ATE RMSE GLIM, **Session C** (Mid-360), T2
- Session C = Livox Mid-360 (3D LiDAR) → FAST-LIO2 y GLIM aplicables. Session D = RealSense D455 (visual-inertial) → ORB-SLAM3 aplicable. Consistente con SLAM_BACKENDS y EXPERIMENTAL_DESIGN (T2 3D igual para C y D).

---

## 8. Caracterización estática vs sesiones dinámicas

- **IMU (Allan, Six-Position):** WitMotion, RealSense D455 (IMU), Livox Mid-360 (IMU) — es decir sensores de sesiones A, D y C. No depende de la letra de sesión; la lista por sensor es correcta.
- **LiDAR (plano ortogonal):** Livox Mid-360, RPLiDAR A2M12 — sesiones C y B.
- **Cámara (AprilTag):** RealSense D455 — sesión D.
- No hay conflicto entre orden A–D (dinámico) y listados por tipo de sensor en estático.

---

## 9. Resumen de correcciones aplicadas en esta revisión

| Archivo | Cambio |
|---------|--------|
| `docs/REFERENCES_POOL.md` | "Session B (visual/depth)" → "Session D (visual/depth)" para RealSense. |
| `internal/AUDIT_CONSOLIDATED_REVIEW.md` | Fórmula M4 actualizada a σ²_static(T); nota post-auditoría (n≈120–180, T); hallazgo Audit 1 ajustado (n aumentado, identificabilidad). |

---

## 10. Conclusión

- **Nomenclatura A–D** y mapeo S1–S4 son coherentes en todos los documentos revisados.
- **n ≈ 120–180** por tipo de trayectoria está fijado en METHODOLOGY y EXPERIMENTAL_DESIGN; el AUDIT queda alineado con una nota de actualización.
- **Ground truth:** 0.02 mm solo para estático; 0.10 mm y 1.36 mm para dinámico; sin usos de 0.02 como cota dominante en trayectorias.
- **Modelo térmico:** T (temperatura medida) es la única formulación activa en docs y README.
- **Referencias:** Una sola fuente de verdad para M1–M4 y S0–S3 (METHODOLOGY); SLAM_BACKENDS solo enlaza.
- **Endpoints primarios** coinciden con Session C (Mid-360) y Session D (D455) y con los backends definidos por modalidad.

La coherencia científica entre documentos y el objetivo del estudio (H0/H1/H2, TOST, comparación R/S/M por sesión y trayectoria) queda verificada tras esta revisión.
