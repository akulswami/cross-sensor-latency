# Pre-Registration — Cross-Sensor Sense-Stage Latency Characterization

**Status:** DRAFT v0.1 — pre-data, pre-confirmatory. To be frozen and Zenodo-timestamped BEFORE any confirmatory data collection.

**Working title:** Onset-to-Decision Latency Decomposition Across Heterogeneous Edge Sensing Modalities: The Sense-Stage Term Conventional Verification Omits

**Target venue:** IEEE Access

**Relationship to prior work:** This study extends the wire-level GPIO timestamping methodology of Swami & Sonawane (IEEE Sensors Letters, arXiv:2605.02835 / LSENS) and Swami & Chougule (arXiv:2605.02835, 2605.17701). Those works measured *interrupt-to-decision* latency (sensor INT edge -> decision GPIO). This study adds physical motion onset as the measurement zero, decomposing total onset-to-decision latency into a **sense-stage term** (onset -> sensor first output) and a **decision-stage term** (sensor output -> decision GPIO), and characterizes the sense-stage term across three sensing modalities with mechanistically distinct cadence/settling behavior.

---

## 1. Motivation and core claim

Conventional functional/inference-timing verification measures from the sensor's data-ready interrupt forward. The interval between *physical stimulus onset* and the sensor's *first output* is not captured by such verification, yet it can dominate total latency and is the failure mode behind "the result was correct but arrived too late." In prior work the LSM6DSOX MLC exhibited a 706.5 ms decision cadence — a sense-stage floor invisible to interrupt-to-decision measurement.

**Core claim (to be tested, not assumed):** The sense-stage term exists in all three modalities, varies by orders of magnitude across them, and arises from *mechanistically distinct* causes (pure cadence quantization vs. multi-frame filter settling vs. quantization-plus-readout). A single wire-level method captures all three.

**Falsification of the core claim:** If the sense-stage terms across sensors reduce to a single common mechanism (cadence quantization = 1/output-rate) with no separable settling or readout component, the "mechanistically distinct" claim is FALSIFIED and the paper instead reports cadence quantization as a *unified* sense-stage mechanism. (This fallback is itself a publishable, honest finding — see LSENS H7' precedent.)

---

## 2. Measurement architecture

Three Saleae Logic Pro 8 channels (>=250 MS/s), all wire edges, independent of host clock:

- **D0 = servo command edge** — physical motion onset. THE GROUND-TRUTH ZERO. (Departs from LSENS, where D0 was the sensor INT.)
- **D1 = sensor first-output edge** — sensor INT / data-ready (LSM6DSOX INT1; A121/XM125 MCU_INT or GPIO; VL53L5CX INT).
- **D2 = decision GPIO** — pipeline output edge (decision-tree / detector threshold crossing -> GPIO toggle).

**Latency decomposition, per trial:**
- `L_sense  = t(D1) - t(D0)`  (onset -> sensor first output)  ← novel cross-sensor term
- `L_decide = t(D2) - t(D1)`  (sensor output -> decision)      ← LSENS-method term, reused
- `L_total  = t(D2) - t(D0)`  (onset -> decision)

**Servo actuation offset:** The servo command edge precedes physical motion by a fixed mechanical latency `L_servo`. `L_servo` is characterized ONCE per servo (accelerometer-tap or high-speed reference) and reported; `L_sense` is reported as command-referenced with `L_servo` stated, OR `L_servo`-subtracted. Decision pre-registered before confirmatory data: **report command-referenced, state L_servo separately** (subtraction deferred to a sensitivity check, not the primary measure).

---

## 3. Sensors and fixed configurations

Configurations MUST be frozen here before data. Floors are functions of config; an unfrozen config voids the prediction.

### Sensor A — LSM6DSOX (IMU). Reference anchor (partly measured in LSENS).
- Accel: CTRL1_XL = 0x50 (208 Hz, +/-2 g, LPF2 off) [matches LSENS for continuity]
- MLC: 26 Hz, 2-class motion/still, 75-sample window (mlc_motion_w75) [matches LSENS]
- I2C bus 7 @ 400 kHz, addr 0x6A
- Pipelines: (a) host decision-tree, (b) MLC bank-switch read

### Sensor B — A121 / XM125 (PCR radar). [config to FREEZE on bench bring-up]
- Detector: Presence detector
- Frame rate: **12 Hz default** (inter-frame period ~83.3 ms) — PRIMARY config
- Sweeps per frame: 16 (default); HWAAS: 32 (default)
- Inter-frame filter / presence timeout: DEFAULTS — record exact values from firmware at bring-up
- Interface: I2C or UART (decide at bring-up); MCU_INT as D1
- Secondary frame-rate sweep (5 Hz, 20 Hz) as a config-sensitivity arm — see H4

### Sensor C — VL53L5CX (ToF). [config to FREEZE on bench bring-up]
- Resolution: 4x4 (PRIMARY) — ranging freq up to 60 Hz; and 8x8 (SECONDARY) — up to 15 Hz
- Ranging frequency: set explicitly; record value
- Ranging mode: continuous; INT (data-ready) as D1
- I2C @ 400 kHz (or 1 MHz if validated); record bus speed
- Zone-count is the readout-coupling variable: 16 zones (4x4) vs 64 zones (8x8)

---

## 4. Predicted sense-stage floors (L_sense)

Numbers marked **[derivable]** are computed from frozen config and committed here.
Numbers marked **[TBC]** are predicted to EXIST with a stated rough scale, to be characterized empirically — NOT guessed to a false precision.

| Sensor / pipeline | Quantization term | Settling/readout term | Mechanism |
|---|---|---|---|
| IMU host | ~window-fill (75 smpl @208 Hz ~360 ms) **[TBC — re-measure]** | none | window assembly |
| IMU MLC | **706.5 ms cadence [measured, LSENS]** mean wait ~353 ms | none (pure clock) | internal window-cadence clock |
| PCR 12 Hz | **~83.3 ms frame period [derivable]**, mean onset wait ~41.7 ms | multi-frame filter settling **[TBC — predict >0, scale 1–several frames]** | frame quantization + inter-frame filter |
| ToF 4x4@60Hz | **~16.7 ms [derivable]**, mean wait ~8.3 ms | per-frame multizone I2C readout **[TBC — predict >0, scales with zone count]** | ranging quantization + readout transfer |
| ToF 8x8@15Hz | **~66.7 ms [derivable]**, mean wait ~33.3 ms | larger readout (64 zones) **[TBC]** | ranging quantization + readout transfer |

---

## 5. Hypotheses

**H1 (sense-stage term exists and is non-negligible).** For each sensor, median `L_sense` is significantly greater than median `L_decide`. Test: one-sided Mann-Whitney U per sensor; Hodges-Lehmann shift + 95% bootstrap CI (10,000 resamples). Falsified for a sensor if `L_sense <= L_decide`.

**H2 (cross-sensor variance).** Median `L_sense` differs across the three sensors by at least one order of magnitude (max/min >= 10). Test: pairwise Mann-Whitney with Holm-Bonferroni; report HL shifts. Falsified if all three medians lie within one order of magnitude.

**H3 (mechanism distinction — the core, highest-risk hypothesis).** The sense-stage term decomposes differently by sensor: IMU-MLC is pure quantization (settling ~ 0); PCR has a settling component strictly beyond one frame period; ToF has a readout component that scales with zone count (8x8 readout term > 4x4 readout term). Test: (a) PCR — fraction of `L_sense` exceeding one frame period is > 0 with bootstrap CI excluding 0; (b) ToF — readout term (L_sense minus quantization) for 8x8 > 4x4, bootstrap contrast. **Falsified if** PCR settling ~ 0 (collapses to pure quantization) AND ToF readout term is negligible/zone-independent — in which case the unified-quantization fallback is reported.

**H4 (config-controllability).** PCR `L_sense` quantization scales as 1/frame-rate across {5, 12, 20} Hz; ToF `L_sense` quantization scales as 1/ranging-freq across resolutions. Test: regression of mean wait on 1/rate; report slope CI. (Supports the "designer-tunable vs structural" discussion.)

**H5 (decision-stage continuity with prior work).** `L_decide` for register-read pipelines (IMU MLC bank-switch, ToF/PCR register reads) is dominated by I2C transaction overhead, consistent with LSENS (I2C read protocol as dominant decision-stage contributor). Test: compare against a no-read binary-toggle floor per sensor (LSENS mlc-binary precedent). Equivalence/ordering via TOST or Mann-Whitney as appropriate.

**H6 (SECONDARY / robustness — unit reproducibility; NOT part of the primary confirmatory family).** Two nominally-identical VL53L5CX units (both shipped in the VL53L5CX-SATEL package) produce statistically equivalent sense-stage floors `L_sense` under identical configuration and condition. Purpose: establish whether the ToF sense-stage floor is a property of the *modality* (transferable) or of the *specific board* (unit-bound), directly addressing the single-unit limitation of prior work (LSENS characterized one LSM6DSOX). Test: TOST equivalence on median `L_sense` between Unit A and Unit B at the primary config (4x4 @ chosen rate, idle), pre-registered margin set in §8. **Outcome interpretation:** equivalence SUPPORTED -> floor is modality-level, strengthens generalizability; equivalence NOT supported -> floor has a unit-specific component, reported as a measured limitation. Either outcome is reportable; neither is a failure.

- **Scope guard:** H6 is explicitly a *unit-variation robustness check*, NOT a fourth sensor and NOT a second data point in the cross-sensor variance claim (H2). The two ToF units do not enter H1–H5. The primary design remains three sensors spanning three cadence regimes. The second unit's other role is failure-insurance spare (see DECISIONS D5).
- **Anti-fishing commitment:** H6 is registered here, pre-data, with its margin and single comparison fixed in §8 before collection. No additional unit comparisons, configs, or conditions will be added to H6 post-hoc; if exploratory unit observations arise they will be labelled exploratory and excluded from confirmatory claims.

**Pre-registered falsifiable predictions left on record regardless of outcome.** Per LSENS practice, all hypotheses — including any falsified — remain in the published pre-registration chain. H3 is explicitly the one most at risk; its falsification is anticipated as a valid scientific outcome, not a failure. H6 is secondary and reported separately from the primary confirmatory family.

---

## 6. Measurement protocol

- Per (sensor, pipeline, condition) cell: blocks of fixed duration; N candidate onsets per block; deterministic pre-registered shuffle seed (record seed).
- Conditions: **idle**, **CPU stress** (stress-ng), **I2C contention** (hammer) — matching LSENS condition set for cross-paper comparability.
- Inclusion criterion: exactly one D1 edge and one D2 edge per onset window; violations categorized and reported (not silently dropped), with a pre-registered exclusion ceiling (e.g. 10%).
- Jetson pinned: MAXN_SUPER_JC nvpmodel, governor=performance, jetson_clocks engaged; record effectiveness.
- Warmup structurally enforced (N unmeasured inferences before first measured) — LSENS lesson.

## 7. Statistical treatment

- Effect estimates: Hodges-Lehmann shift + percentile bootstrap CI (10,000 resamples).
- Ordering tests: one-sided Mann-Whitney U.
- Equivalence: TOST with pre-registered margin (set per measure).
- Multiplicity: Holm-Bonferroni across the PRIMARY confirmatory family {H1 (x3), H2, H3, H5}; family and alpha frozen here. **H6 is secondary/robustness and is NOT included in this family** — it is reported separately with its own pre-registered TOST margin, so it neither inflates nor is corrected against the primary family.
- Run-level reporting (per-run mean, run-mean SD, P95, P99, max); ECDFs and run-level boxplots for multimodal cells (LSENS lesson: multimodality is the story).

## 8. Open items to FREEZE before confirmatory data (do not collect until resolved)
1. PCR interface (I2C vs UART) and exact inter-frame filter / timeout values read from firmware.
2. ToF I2C bus speed and per-zone readout time (measure on bench; this becomes the H3 ToF readout term).
3. PCR detector settling-in-frames for the servo motion signature (measure; becomes H3 PCR settling term).
4. `L_servo` mechanical actuation offset.
5. Final block count, N per block, shuffle seed, exclusion ceiling, TOST margins.
6. H6 unit-reproducibility: TOST equivalence margin for ToF `L_sense` (Unit A vs Unit B), and which physical board is designated Unit A (experimental) vs Unit B (comparison); the remaining role of the second unit is failure-insurance spare. Freeze before collecting any H6 data.

## 9. Amendment log
- v0.1 (DRAFT, pre-data): initial structure. NOT yet frozen. No confirmatory data collected.
- v0.2 (DRAFT, pre-data): added H6 secondary/robustness unit-reproducibility hypothesis (two VL53L5CX units), explicitly outside the primary confirmatory family and outside the cross-sensor variance claim. Added open item 6. Still NOT frozen; no confirmatory data collected.
