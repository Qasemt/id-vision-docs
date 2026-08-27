# Proposal for a Facial Detection and Recognition System

## Introduction

We are pleased to present this concise proposal for implementing a facial detection and recognition system. The system is designed in a modular fashion, allowing it to efficiently and reliably scale from a single camera to dozens of cameras, depending on project requirements.

The architecture is based on a **worker model**, where each worker independently processes a subset of cameras. This distributed approach enables optimal resource allocation, increased fault tolerance, and improved scalability.

Each worker manages camera streams, performs initial processing (such as face detection), and sends the processed data to a **central hub server**, which is responsible for aggregating and analyzing the results.

The system is optimized for handling up to **1,000 personnel**. We recommend starting with a **Proof of Concept (PoC)** phase to evaluate system performance in real-world conditions.

---

## Hardware Requirements

The following specifications represent the **minimum and recommended** configurations for real-time operation.

**Note:** GPU acceleration is **optional** — it is currently under testing and will be added in the near future. GPU usage can significantly reduce CPU load.

**Important:** When calculating total system requirements, always account for **operating system overhead** separately. For example:

- The OS (e.g., debian 12,13.1 or Windows) typically consumes 2–4 GB RAM and 10–20% CPU when idle.

---

### For Camera Processing Nodes
"These hardware specifications listed in the table below are Recommended for achieving ideal and optimal system performance. Using the Minimum configurations may reduce performance or cause issues under heavier loads. For the best results, please use the Recommended specifications."
| Number of Cameras | Recommended Number of Workers | CPU (Cores / Estimated Load)                                                                  | RAM (GB)                      | GPU (Optional)                                                       | Storage (SSD) |
| ----------------- | ----------------------------- | --------------------------------------------------------------------------------------------- | ----------------------------- | -------------------------------------------------------------------- | ------------- |
| 1–4 cameras       | 1 worker                      | Min: 5–8 cores (20–50% load)<br>Recommended: 8 cores (Intel i7 or AMD Ryzen 7)                | Min: 8–12<br>Recommended: 12    | Optional: NVIDIA GTX 1650+ (reduces CPU usage by 10–30%)             | ≥ 33 GB       |


---

### For the Central Server

| Component          | CPU (Cores / Estimated Load)                                                 | RAM (GB)                   | GPU (Optional)            | Storage (SSD)                      | Network                    |
| ------------------ | ---------------------------------------------------------------------------- | -------------------------- | ------------------------- | ---------------------------------- | -------------------------- |
| Central Hub Server | Min: 8 cores (20–40% load)<br>Recommended: 16 cores (Intel Xeon or AMD EPYC) | Min: 16<br>Recommended: 32 | Optional: NVIDIA RTX 3060 | Min: 167 GB<br>Recommended: 333 GB | Gigabit Ethernet or faster |

---

### How Per-Camera Requirements Are Measured

Rather than relying only on a generic estimate, the camera processing node application ships
with an automated capacity-benchmark tool (`capacity_benchmark.py`, with an optional local web
UI) that runs the real face-detection pipeline against simulated camera streams directly on a
candidate machine. It measures, per camera, on top of the machine's own idle baseline:

- CPU cost, as both logical cores used and an approximate GHz-equivalent (cores used × the
  machine's clock speed — a rough shopping-conversation figure only; it does not account for
  CPU architecture/generation differences, so treat it as an order-of-magnitude guide)
- RAM cost, in megabytes
- End-to-end per-frame processing latency and internal queue backlog, to confirm the machine
  is keeping up in real time — not just technically staying online

This lets an integrator get a defensible, hardware-specific number for a specific deployment by
running the benchmark on the actual candidate machine with the same detection settings
(resolution, GPU on/off, optional features) intended for production, instead of relying only on
the general table above.

**Example measurement** (illustrative only — this run was on a constrained 4-core test machine
and is shown to demonstrate the report format, not as a recommended spec; note the result is a
FAIL):

| Cameras | CPU avg% | CPU cores used | CPU cores Δ/camera | CPU GHz Δ/camera | RAM Δ vs baseline (MB) | RAM Δ/camera (MB) | Queue avg | Sat. ratio | Latency (ms) | Alive | Result |
| ------- | -------- | --------------- | ------------------- | ----------------- | ----------------------- | ------------------- | --------- | ---------- | ------------- | ----- | ------ |
| 1       | 99.95%   | 4.0              | 3.8                  | 7.96               | 1754.64                 | 1754.64              | 243.53    | 0.37       | 29375.92      | 1     | fail   |

Here, a single camera already saturated all 4 CPU cores of the test machine and per-frame
latency degraded past 29 seconds — a sign that this particular test machine lacked enough CPU
headroom for even one camera at these settings, not a statement about the product's real-world
cost. A camera-processing node sized per the table above (with real CPU headroom) is expected to
produce a passing result instead.

<!-- TODO: replace/supplement this example with a measurement taken on adequately-sized reference
     hardware once available, so the illustrative row also shows a PASS scenario. -->

<!-- TODO: this measured per-camera cost (~3.8 CPU cores for one camera) and the "For Camera
     Processing Nodes" table above (~8 cores recommended for 1–4 cameras) come from two different
     estimation methods and are not yet formally reconciled — flagging for the docs/engineering
     owner rather than silently editing the table above. -->

---

## Implementation Plan

1. **Initial Setup:** Deploy PoC with 1–2 cameras.
2. **Testing:** Validate performance and recognition accuracy.
3. **Full Deployment:** Scale up to all target cameras.
4. **Support:** Provide ongoing maintenance and updates.

---

## Conclusion

This system provides an efficient and scalable solution for facial detection and recognition needs.
If you are interested in a live demo or customized implementation, please let us know.
