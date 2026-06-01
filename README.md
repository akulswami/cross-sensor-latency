# cross-sensor-latency

Wire-level onset-to-decision latency decomposition across three heterogeneous edge sensing modalities (IMU, pulsed-coherent radar, time-of-flight), isolating the **sense-stage term** — the interval between physical stimulus onset and the sensor's first output — that conventional interrupt-to-decision verification omits.

**Target venue:** IEEE Access

**Status:** Pre-registration draft; hardware on order (XM125/XE125 PCR, VL53L5CX-SATEL ToF). LSM6DSOX IMU anchor characterized in prior work. No confirmatory data collected.

## Relationship to prior work

Extends the wire-level GPIO timestamping methodology of:
- Swami & Sonawane, *Wire-Level Interrupt-to-Decision Latency of On-Sensor MLC vs Host Inference* (IEEE Sensors Letters / arXiv).
- Swami & Chougule, *Per-Platform GPIO Overhead* (arXiv:2605.02835) and *Architecture-Dependent Temporal Observability* (arXiv:2605.17701).

Those works measured INT-to-decision latency. **This work adds physical motion onset (servo command edge) as the measurement zero**, decomposing total latency into a sense-stage term (onset -> first sensor output) and a decision-stage term (sensor output -> decision GPIO). The sense-stage term is the novel cross-sensor contribution.

## Sensors (selected to span sense-stage cadence regimes)

| Sensor | Modality | Cadence regime | Role |
|---|---|---|---|
| LSM6DSOX | 6-axis IMU | sub-ms host path; 706.5 ms MLC cadence | reference anchor (prior work) |
| A121 / XM125 | pulsed-coherent radar | ~83 ms frame + multi-frame settling | mid-cadence, settling-dominated |
| VL53L5CX | time-of-flight | ~17–67 ms ranging + multizone readout | mid-cadence, readout-coupled |

## Hardware

- NVIDIA Jetson Orin Nano Developer Kit (MAXN_SUPER_JC, jetson_clocks)
- Saleae Logic Pro 8 (3 channels: D0 servo edge, D1 sensor INT, D2 decision GPIO)
- 2-axis servo rig (PCA9685 PWM controller) — motion ground-truth oracle; sensors fixed, target on rig
- LSM6DSOX (mounted), XM125/XE125, VL53L5CX-SATEL

## Repository structure

- `paper/` — manuscript source and figures
- `code/` — measurement harness, per-sensor pipelines, analysis
- `data/` — captured measurements and processed outputs
- `docs/` — pre-registration, hardware setup, measurement protocol, lab notebook
- `env/` — environment versions for reproducibility

## Reproducibility

Experimental decisions logged in `DECISIONS.md`. Pre-registration in `docs/pre-registration.md`, frozen and Zenodo-timestamped before confirmatory data collection. Per prior-work practice, falsified hypotheses remain on the public record.
