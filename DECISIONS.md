# Decisions Log

Chronological record of experimental design decisions. Append-only; do not edit prior entries.

## D1 — Measurement zero is the servo command edge (D0), not the sensor INT
**Decision:** Total latency is measured from physical motion onset (servo command edge), decomposed into L_sense (onset->INT) and L_decide (INT->decision).
**Rationale:** The sense-stage term — the paper's contribution — lives in the onset->INT gap and is invisible to INT-referenced measurement (as in prior work). Measuring from INT would reduce this to a restatement of prior work. INT is retained as D1 to preserve continuity and enable the two-stage decomposition.
**Status:** Locked.

## D2 — Three sensors, selected to span sense-stage cadence regimes
**Decision:** LSM6DSOX (anchor), A121/XM125 (PCR), VL53L5CX (ToF). Not 4–5.
**Rationale:** The cross-sensor claim is proven by mechanism contrast, not sample size. Three regimes (pure-quantization, settling-dominated, readout-coupled) prove the variance; a 4th in a covered regime adds integration risk without strengthening the argument. Fallback: a defensible 2-sensor (IMU + one) paper if a 3rd integration slips.
**Status:** Locked.

## D3 — Sensors fixed, target on servo
**Decision:** PCR and ToF mounted fixed, aimed at servo arc; lightweight target on servo platform. Only IMU rides the platform.
**Rationale:** Radar/ToF detect external motion; mounting them on the moving servo corrupts the measurement. Also keeps the 9g servo unloaded for crisp, repeatable onset timing.
**Status:** Locked.

## D4 — Servo actuation offset reported, not subtracted (primary measure)
**Decision:** L_sense reported command-referenced; L_servo characterized once and stated separately; subtraction is a sensitivity check only.
**Rationale:** Avoids baking an estimated offset into the primary measure.
**Status:** Locked (revisit if L_servo proves large relative to fastest L_sense).

## D5 — Two VL53L5CX units: one experimental, one spare; optional secondary reproducibility arm
**Date:** 2026-06-03
**Decision:** The VL53L5CX-SATEL package ships two ToF breakout boards. Unit A is the experimental sensor in the three-regime design (IMU / PCR / ToF). Unit B is held as a static-safe failure-insurance spare. Optionally, after the primary three-sensor campaign is collected and working, Unit B may serve a pre-registered SECONDARY unit-reproducibility check (H6) — TOST equivalence of L_sense between units at the primary config.
**Rationale:** Redundancy exists only for the ToF (and its reorder lead time was long), so spare-first protects the build. A second identical-modality unit does NOT strengthen the cross-sensor variance claim (would repeat the "more sensors = stronger" error); it only characterizes unit-to-unit variation, which is a distinct, narrower question. Framed as a robustness arm it cleanly addresses the single-unit limitation that prior work (LSENS, one LSM6DSOX) had to list.
**Guardrails:** H6 is pre-registered (pre-data) as secondary, kept out of the primary confirmatory multiplicity family and out of H1–H5, with margin and single comparison fixed before collection to prevent post-hoc fishing.
**Status:** Locked (H6 execution optional, contingent on time after primary campaign).
