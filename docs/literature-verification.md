# Literature Review — Verification Record

**Purpose:** Document the provenance and verification status of literature-review material received for this paper, so future use distinguishes (a) independently verified facts, (b) real-but-mischaracterized sources, and (c) unchecked entries. This record exists because at least one received review document fabricated a finding while citing a real paper — so citation existence and citation *characterization* must be tracked separately.

**Date compiled:** 2026-06-04
**Compiled during:** novelty re-evaluation prior to bench data collection (pre-registration still DRAFT/unfrozen).

---

## 1. Standing rule established from this exercise

A received literature review may cite **real papers** while **misstating what they found**. Verifying that a DOI resolves is necessary but NOT sufficient. Before any cited claim of the form "[X] found/measured/showed Y" enters the manuscript, the source itself must be read to confirm Y. One such claim was fabricated in received material (see §4, Capogrosso).

---

## 2. Independently VERIFIED anchors (source existence + correct citation confirmed)

These were checked against primary sources (publisher pages, DOI resolution, author pages) during the review. Existence, authors, venue, year, DOI confirmed.

| Ref | Citation | Verified facts | Role for our paper |
|---|---|---|---|
| Günzel et al. 2024 | "End-To-End Latency of Cause-Effect Chains: A Tutorial," ACM TECS 24(1), Art. 22, Dec 2024. DOI 10.1145/3703630 | Authors Günzel, Teper, von der Brüggen, Chen. Real. | Cluster 1 foundation: data-age vs reaction-time vocabulary; cause-effect decomposition. Cite-to-build-on. |
| Günzel et al. 2023 | "On the Equivalence of Maximum Reaction Time and Maximum Data Age for Cause-Effect Chains," ECRTS 2023, LIPIcs vol 262, 10:1–10:22. DOI 10.4230/LIPIcs.ECRTS.2023.10 | Authors Günzel, Teper, Chen, von der Brüggen, Chen. Real. | Cluster 1 theory. Cite-to-build-on. |
| Aymone & Pau 2024 | "Benchmarking In-Sensor Machine Learning Computing: An Extension to the MLCommons-Tiny Suite," Information 15(11):674, 2024. DOI 10.3390/info15110674 | Authors Aymone, Pau (STMicroelectronics). Real. In-sensor ML benchmark (QAT vs PTQ; HAR/PAM/sEMG). | Cluster 2. NOTE: software/accuracy benchmark, NOT sense-stage latency. Cite-to-distinguish. |
| Mytkowicz et al. 2009 | "Producing Wrong Data Without Doing Anything Obviously Wrong!" ASPLOS 2009, pp. 265–276. DOI 10.1145/1508244.1508275 | Authors Mytkowicz, Diwan, Hauswirth, Sweeney. Real. Measurement bias is systematic, commonplace, survives repetition. | Cluster 4 cornerstone: justifies wire-level (external) measurement. Cite-to-build-on. |
| Fan et al. 2025 (Zoox) | "Robust sensor fusion against on-vehicle sensor staleness," arXiv:2506.05780. Accepted CVPR 2025 Precognition Workshop. | Authors Fan, Zuo, Blaes, Montgomery, Das (Zoox). Real. Per-point timestamp-offset feature + augmentation for staleness in AV fusion. | Cluster 3 nearest adjacent. Software fusion-accuracy solution, NOT wire-level per-sensor verification. Cite-to-distinguish. |
| Capogrosso et al. 2026 | "Performance Analysis of Edge and In-Sensor AI Processors: A Comparative Review," I2MTC 2026. arXiv:2603.08725 | Authors Capogrosso, Bonazzi, Magno. Real. See §4 — characterization in received review was WRONG. | Cluster 2 motivation-for-gap. Cite-CORRECTLY (see §4). |

---

## 3. Our own prior work (method lineage — must be cited as foundation, not restated as novel)

- **LSENS** — Swami & Sonawane, "Wire-Level Interrupt-to-Decision Latency of On-Sensor MLC vs Host Inference on the Jetson Orin Nano: A Pre-Registered Measurement Study," IEEE Sensors Letters (submitted; arXiv:2606.00524). Establishes the wire-level GPIO-timestamp method, servo stimulus, pre-registration discipline, and the 706.5 ms MLC cadence finding. **The wire-level method is OUR PRIOR ART — not claimable as new in this paper.**
- **GPIO-overhead** — Swami & Chougule, arXiv:2605.02835. Establishes per-platform GPIO-overhead calibration (and that the wire-level method itself has platform-dependent bias — cite as honest limitation).
- **Observability** — Swami & Chougule, arXiv:2605.17701. Timing-observability can fail independently of execution.

These three make the cross-sensor paper a coherent program (good for the record), but mean the *contribution* of THIS paper cannot be "wire-level sense-stage measurement" (already published by us). The contribution must be the cross-sensor **mechanism distinction** (see §6).

---

## 4. MISCHARACTERIZED source — correction on record

**Capogrosso et al. (arXiv:2603.08725), I2MTC 2026.**

**What the received "unified five-cluster review" claimed (FALSE):**
- That Capogrosso "used external instruments to reveal sense-stage dominance."
- That it "measured sense-stage latency for common TinyML sensors using external instruments, revealing... sensor acquisition (5–20 ms) often dominates total system latency."

**What the paper ACTUALLY does (verified by reading full text):**
- It is a **processor inference-latency benchmark**, not a sense-stage study.
- Benchmarks one model (PicoSAM2, 336M MAC) on three **compute platforms**: GAP9, STM32N6, Sony IMX500.
- "Latency" is explicitly defined as the **end-to-end time to execute a single forward pass** (inference compute), measured by **cycle-accurate profiling** — NOT external/wire-level instrumentation, NOT sensor acquisition time.
- Latency results (13.7 / 14.3 / 42.1 ms) are forward-pass compute times attributed to clock and memory hierarchy.
- Sense-stage / AFE+ADC is mentioned ONLY qualitatively in an energy-budget discussion (§II.B: "negligible for low-bandwidth modalities, scales for vision"). No measurement of acquisition latency. No servo, no logic analyzer, no onset reference, no cross-sensor sense-stage comparison.

**Correct use of Capogrosso for our paper:** as MOTIVATION FOR THE GAP — even a recent, careful in-sensor processor benchmark reports only forward-pass compute latency and omits the physical-onset-to-first-output sense-stage term entirely. It does NOT occupy our gap; it illustrates that the field measures the wrong (or only partial) thing. Cite-to-distinguish / motivation.

---

## 5. UNVERIFIED entries (use with caution — verify before citing)

The received review listed 90+ scholarly references plus claimed ~45–66 patents. Only the anchors in §2 were independently verified. **All other entries — and ALL characterizations of what any cited work found — remain UNVERIFIED.** Notably unverified and flagged:
- The "Clemson outside-observer high-speed-camera thesis" — referenced in review prose as a key methodological ancestor but **absent from the reference list**; no resolvable citation. Do not cite until located and verified.
- All DOI-less entries in the received list.
- Standards (IEC 62304, ISO 14971, FDA-2024-D-4488): real, but **verify CURRENT status/version** — FDA-2024-D-4488 is draft guidance and may have changed since Jan-2026 cutoff. Reuse the verified, venue-formatted entries from the LSENS reference list rather than re-deriving.
- Duplicate detected: received Cluster-1 [6] and [18] are the same ECRTS 2023 paper listed twice.

---

## 6. Novelty conclusion (as of 2026-06-04, pre-data)

Searched four collision zones + safety-verification angle, and read Capogrosso in full. **Finding: no direct collision. The cross-sensor mechanism-distinction contribution is unoccupied.**

- Nobody (incl. Capogrosso) has done external wire-level cross-sensor sense-stage latency measurement.
- The mechanism-distinction (onset→first-output floor arising from mechanistically DIFFERENT causes per modality: IMU cadence-quantization vs PCR multi-frame settling vs ToF ranging-quantization+readout-coupling) is the load-bearing novelty and is not present in any surveyed work.
- Adjacent literatures to cite-and-distinguish: AV sensor-staleness fusion (Fan/Zoox — software, accuracy-oriented); edge-AI inference benchmarking (Capogrosso, Aymone — compute latency, software-timed, sense-stage omitted); cause-effect-chain theory (Günzel — analytical, assumes zero-latency sensor at chain head).
- Method lineage to cite-as-foundation (NOT as new): our LSENS/GPIO/observability papers.

**Caution carried forward:** the mechanism-distinction (pre-reg H3) is also the contribution most at risk empirically. If bench data shows the three floors collapse to plain 1/rate quantization with no separable settling/readout terms, the distinction weakens and the paper falls back toward the crowded "latency varies" territory. H3 must survive the data.
