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

| Number of Cameras | Recommended Number of Workers | CPU (Cores / Estimated Load)                                                                  | RAM (GB)                      | GPU (Optional)                                                       | Storage (SSD) |
| ----------------- | ----------------------------- | --------------------------------------------------------------------------------------------- | ----------------------------- | -------------------------------------------------------------------- | ------------- |
| 1–4 cameras       | 1 worker                      | Min: 4–8 cores (20–50% load)<br>Recommended: 8 cores (Intel i7 or AMD Ryzen 7)                | Min: 4–8<br>Recommended: 8    | Optional: NVIDIA GTX 1650+ (reduces CPU usage by 10–30%)             | ≥ 33 GB       |
| 5–10 cameras      | 2 workers (3–5 cameras each)  | Min: 8–16 cores (50–80% load)<br>Recommended: 16 cores (Intel Xeon or AMD EPYC)               | Min: 8–16<br>Recommended: 16  | Optional: NVIDIA RTX 3060+ (6 GB VRAM+, reduces CPU usage by 30–50%) | ≥ 83 GB       |
| 10+ cameras       | 3+ workers (4–6 cameras each) | Min: 16–32 cores (80–120% load)<br>Recommended: 32 cores (server-grade, e.g., AWS c5.9xlarge) | Min: 16–32<br>Recommended: 32 | Optional: NVIDIA RTX 4080+ (12 GB VRAM+) or cloud equivalent         | ≥ 167 GB      |

---

### For the Central Server

| Component          | CPU (Cores / Estimated Load)                                                 | RAM (GB)                   | GPU (Optional)            | Storage (SSD)                      | Network                    |
| ------------------ | ---------------------------------------------------------------------------- | -------------------------- | ------------------------- | ---------------------------------- | -------------------------- |
| Central Hub Server | Min: 8 cores (20–40% load)<br>Recommended: 16 cores (Intel Xeon or AMD EPYC) | Min: 16<br>Recommended: 32 | Optional: NVIDIA RTX 3060 | Min: 167 GB<br>Recommended: 333 GB | Gigabit Ethernet or faster |

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
