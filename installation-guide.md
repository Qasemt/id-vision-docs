# Installation Guide

This document describes how to set up and install the **ID Vision** facial detection system.

---

## 1. Overview

The **ID Vision System** consists of two main components:

- **ID Vision Worker** → Handles video streams and performs facial detection tasks.
- **ID Vision Hub** → Manages communication, coordination, and aggregation of data between multiple workers.

Each component must be installed and configured properly to ensure a stable and scalable system.

---

## 2. Installation Components

Below are detailed installation guides for both components:

### 🧩 ID Vision Worker Service

For instructions on how to install and manage the **Worker** service, see:

➡️ [ID Vision Worker Service Setup Guide](worker/id_vision_worker_service_setup_guide.md)

---

### 🌐 ID Vision Hub Service

For instructions on how to install and manage the **Hub** service, see:

➡️ [ID Vision Hub Service Setup Guide](hub/id_vision_hub_service_setup_guide.md)

---

## 3. System Requirements

| Component    | Minimum Requirement      |
| ------------ | ------------------------ |
| OS           | Windows 10 / 11 (64-bit) |
| Python       | 3.10 or newer            |
| RAM          | 8 GB or higher           |
| Disk Space   | 2 GB minimum             |
| Dependencies | NSSM, FFmpeg, virtualenv |

---

## 4. Recommended Installation Order

1. Install **FFmpeg** and **NSSM** tools.
2. Install **ID Vision Hub** (central manager).
3. Install one or more **ID Vision Worker** services on the same or remote machines.
4. Verify service connectivity through the **Hub dashboard** or logs.

---

## 5. Notes

- Ensure that the **Hub** service is installed before connecting any **Workers**.
- Both services can run on separate systems if network access is properly configured.
- Run installation scripts as **Administrator** to allow service registration.

---

**Author:** Qasem taheri
**Last Updated:** November 2025
