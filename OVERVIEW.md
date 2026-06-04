# OVERVIEW — Cross-Sensor Sense-Stage Latency Paper

**Status as of 2026-06-04:** Pre-data. Pre-registration DRAFT (v0.2, unfrozen). IMU characterized (prior work); ToF bring-up in progress; PCR not yet brought up. No confirmatory data collected.

This is the canonical one-page reference. Detail lives in `docs/pre-registration.md`, `DECISIONS.md`, `docs/literature-verification.md`, and `docs/related-work-draft.md`.

---

## What the paper is

A wire-level measurement study characterizing the **sense-stage latency term** — the interval from a physical event to a sensor's first output — across three sensing modalities, showing it is omitted by conventional verification and arises from mechanistically different causes per sensor. Framed against medical-device "correct-but-too-late" recall hazards. Capstone of a four-paper program on safety-critical edge-AI timing.

**Target venue:** IEEE Access.

## Novelty

The **cross-sensor mechanism distinction**: the sense-stage floor is not one phenomenon scaled by output rate but arises from distinct mechanisms —
- IMU MLC: cadence quantization from an internal update clock (706.5 ms, measured in prior work)
- PCR radar: multi-frame detector settling beyond one frame period (~83 ms frame + settling)
- ToF: ranging-period quantization coupled to multizone readout transfer

Confirmed unoccupied (no external wire-level cross-sensor sense-stage measurement exists; Capogrosso et al. is a processor inference benchmark, not sense-stage).

**NOT novel:** the wire-level method (our prior work), a new metric, single-sensor characterization, "latency varies across sensors" (adjacent literatures own variants).

**LOAD-BEARING CAVEAT:** novelty rides entirely on H3, which is unmeasured. If the three floors collapse to plain 1/rate quantization with no separable settling/readout terms, the distinction weakens and the paper falls toward crowded "latency varies" territory. **H3 must survive the data.**

## Methodology

External wire-level GPIO-edge timestamping (Saleae Logic Pro 8, independent of host clock — justified by measurement-perturbation theory). Two-axis servo rig generates ground-truth motion onset.

Per-trial decomposition:
- `L_sense  = t(INT) − t(servo onset)`  — onset → first sensor output (NOVEL TERM)
- `L_decide = t(decision) − t(INT)`      — sensor output → decision (reused LSENS method)
- `L_total  = t(decision) − t(servo onset)`

Pre-registered hypotheses; frozen + Zenodo-timestamped BEFORE confirmatory data; falsified hypotheses kept on record (LSENS discipline).

## Channel map (fixed for campaign)

D0 = servo PWM/command edge (onset) · D1 = decision GPIO Pin 11 (shared, one pipeline at a time) · D2 = IMU INT1 Pin 15 · D3 = ToF INT (pin 7 = gpiochip0 line 6, verified) · D4 = reserved PCR MCU_INT · D5–D7 spare.

## Parts

Jetson Orin Nano (host) · Saleae Logic Pro 8 (measurement) · 2-axis servo rig + PCA9685 (onset oracle) · LSM6DSOX IMU (mounted, anchor) · XM125/A121 PCR radar · VL53L5CX ToF (2 units: A experimental, B spare/optional H6). Sensors fixed, lightweight target on servo; only IMU rides the platform.

## Hypotheses (see pre-registration for full spec)

- **H1** sense-stage term exists, non-negligible (per sensor)
- **H2** cross-sensor variance ≥ 1 order of magnitude
- **H3** mechanism distinction — THE CONTRIBUTION, highest risk
- **H4** config-controllability (floor scales as 1/rate)
- **H5** decision-stage continuity with prior work
- **H6** (secondary, optional) ToF unit reproducibility — outside primary family

## Objectives

Deliver the three-way mechanism decomposition empirically; produce an accepted IEEE Access paper that strengthens the NIW program by extending the four-paper timing-safety arc.

## Action items (priority order)

0. **DEADLINE-CRITICAL, separate paper:** LSENS (SENSL-26-06-RL-0906) is in review — fast revision turnaround protects the mid-August RFE clock. Takes priority over this paper if revisions arrive.
1. ToF bring-up: userspace polling sanity check → kernel-module INT binding (pin 7) → wire-edge confirmed on Saleae.
2. PCR bring-up.
3. **Front-load H3 characterization** — confirm the three sense-stage terms decompose by distinct mechanism BEFORE full investment. This validates the load-bearing novelty early.
4. Freeze Section 8 open items (PCR settling-frames, ToF readout time, servo offset, H6 margin, block/seed/exclusion params) + Zenodo-timestamp pre-registration.
5. Run confirmatory campaign → analysis → writeup (related-work draft exists).

**Engineering note:** orchestrator MUST guarantee GPIO cleanup on any exit (try/finally + SIGTERM handler + known-state reset) — SSH-Ctrl-C does not deliver SIGINT reliably; a latched decision line injects phantom edges into the next run.

## The one thing that matters most

The highest-leverage next action is validating H3 early. The novelty, the related-work framing, and the paper's worth all ride on whether the three sense-stage terms actually decompose by distinct mechanism. Confirm that before committing the full campaign.
