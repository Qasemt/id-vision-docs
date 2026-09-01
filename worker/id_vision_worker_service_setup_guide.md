# 📘 ID Vision Worker Service Installation & Management Guide

This document provides step-by-step instructions for installing, configuring, and managing the **ID Vision Worker** service on Windows systems using the `install_id_vision_worker_service.bat` script.

---

## ⚙️ Prerequisites

Before running the installation script, make sure all the following components are installed properly.

### 🐍 Python

- **Version:** Python **3.12.5** or later
- **Download:** [Python 3.12.5 (Windows x64)](https://www.python.org/ftp/python/3.12.5/python-3.12.5-amd64.exe)
- During installation, **check the box**:

  > ✅ “Add Python to PATH”

#### Verify installation

```bash
python --version
# Expected output: Python 3.12.5

pip --version
# Example output: pip 23.0.1
```

---

### 🧱 Microsoft Visual C++ Redistributable

Required to avoid DLL errors when running compiled modules (e.g., ffmpeg or numpy).

- **Download link:** [Microsoft Visual C++ Redistributable (x64)](https://aka.ms/vs/17/release/vc_redist.x64.exe)
- Install and reboot your system if prompted.

---

### 🎞️ FFmpeg (for video processing)

1. Download **FFmpeg** Windows build from:
   [FFmpeg Official Builds](https://www.gyan.dev/ffmpeg/builds/ffmpeg-release-essentials.7z)

2. Extract the archive to a folder, e.g.:

   ```
   C:\work_space\tools\ffmpeg
   ```

3. Add the FFmpeg `bin` folder to your **Windows PATH**:

   - Right-click **This PC → Properties**
   - Go to **Advanced system settings**
   - Click **Environment Variables**
   - Under “System variables”, select `Path` → **Edit**
   - Add:

     ```
     C:\work_space\tools\ffmpeg
     ```

---

### 🔧 NSSM (Non-Sucking Service Manager)

NSSM is used to install Python scripts as Windows services.

- **Download:** [NSSM Official Site](https://nssm.cc/download)
- **Recommended path:**

  ```
  C:\work_space\tools\nssm\nssm.exe
  ```

---

## 📂 Project Folder Setup

Create a workspace folder and copy the necessary files:

```bash
C:\work_space\id_vision_worker\
```

Place the following files in this folder:

| File / Folder                              | Description                                                    |
| ------------------------------------------ | -------------------------------------------------------------- |
| `install_id_vision_worker_service.bat`     | Main installation and service management script                |
| `id_vision_worker-0.0.26-py3-none-any.whl` | Offline package wheel file                                     |
| `wheelhouse\`                              | Folder containing dependency wheels (e.g., `numpy` and others) |
| `run_id_vision_worker.bat`                 | The service runner script                                      |

> ⚠️ **Note:** The `.whl` file and `wheelhouse` folder **must** be in the same directory as the script.

Example structure:

```
C:\work_space\id_vision_worker\
│
├── install_id_vision_worker_service.bat
├── run_id_vision_worker.bat
├── id_vision_worker-0.0.26-py3-none-any.whl
└── wheelhouse\
     └── numpy-1.26.0-cp312-cp312-win_amd64.whl
```

---

## 🚀 Running the Installer Script

1. **Right-click** `install_id_vision_worker_service.bat`
2. Choose **Run as Administrator**

The script will:

- Check admin privileges
- Ask for paths (working directory, NSSM, FFmpeg, etc.)
- Validate the environment
- Create a Python virtual environment
- Install dependencies (offline or online)
- Offer a service management menu

---

## 🧩 Offline Installation (Recommended)

When prompted:

```
Please choose the installation method:
[1] Offline install (from project directory)
[2] Online install (from PyPI)
```

Choose option **1** for offline installation.

Make sure your folder includes:

- `id_vision_worker-0.0.26-py3-none-any.whl`
- `wheelhouse\numpy*.whl`

---

## ⚙️ Service Management Menu

After setup, you’ll see the following menu:

```
===================================================
Service Management Menu
===================================================
[1] Install Service
[2] Remove Service
[3] Start Service
[4] Stop Service
[5] Restart Service
[6] Check Service Status
[7] Uninstall id_vision_worker Package
[8] Exit
---------------------------------------------------
Choose an option:
```

### 🔹 Common Tasks

| Option | Description                              |
| -----: | ---------------------------------------- |
|  **1** | Install service using NSSM               |
|  **3** | Start service                            |
|  **4** | Stop service                             |
|  **5** | Restart service                          |
|  **6** | Check current service status             |
|  **7** | Uninstall the `id_vision_worker` package |
|  **2** | Completely remove the service            |

---

## 🔐 License Activation & Maintenance Mode

**ID Vision Worker** requires a signed, hardware-bound license before it will run its camera capture / face-detection pipeline. The license is tied to the specific machine it is issued for.

### 🧩 How It Works

- On startup, the worker validates `files\license\license.json` against the packaged public key and this machine's hardware fingerprint.
- If the license is **missing, invalid, or expired**, the worker starts in **Maintenance Mode**:
  - The camera capture / face-detection pipeline does **not** start.
  - The admin web UI and hub connectivity (`/api/*`) stay available, so the worker can be diagnosed and licensed without shell access to the device.
  - Every other page is redirected to `/maintenance/`.
- A license issued for **ID Vision Hub** will **not** activate a worker (and vice versa) — each product checks a `product` field in the license file, even though both are signed with the same key.

### 🚀 Activating a License

1. Open the worker's admin UI: `http://<worker-host>:<port>/maintenance/` (default port **8001** — <!-- TODO: confirm --> if changed during installation).
2. Copy the **hardware fingerprint** shown on the page.
3. Send the fingerprint to whoever issues licenses and obtain a `license.json` signed for **this** fingerprint with `"product": "id_vision_worker"`.
4. On the `/maintenance/` page, upload the `.json` file (10 MB max).
5. On success, the worker exits maintenance mode. **Restart the `id_vision_worker` service** (see the **Service Management Menu** section above) so the capture/detection pipeline starts.

### 📂 License File Locations

| File             | Default path (relative to working directory) | Override                          |
| ---------------- | ---------------------------------------------- | ---------------------------------- |
| `license.json`   | `files\license\license.json`                  | `ID_VISION_WORKER_LICENSE_PATH`   |
| `public_key.pem` | `files\license\public_key.pem`                | `ID_VISION_WORKER_PUBLIC_KEY_PATH` |

License levels are `TRIAL`, `BASIC`, `PRO`, `ENTERPRISE`; the worker also enforces the license's `expiration_date`.

### 🪪 Troubleshooting

| Symptom                                              | Likely Cause                                          | Resolution                                                                                      |
| ----------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------ |
| Redirected to `/maintenance/` on every page           | License missing, invalid, or expired                  | Follow **Activating a License** above                                                            |
| `License validation failed` on upload                 | Wrong fingerprint, wrong `product` field, bad signature | Re-issue the license for the fingerprint shown on `/maintenance/` with `"product": "id_vision_worker"` |
| License uploaded but capture pipeline still not running | Worker left maintenance mode but wasn't restarted    | Restart the `id_vision_worker` service                                                           |

---

## ⚙️ Configuring Face Recognition (`/config/face_recognition`)

Face-recognition tuning has its own dedicated page in the worker's admin UI — `/config/face_recognition` — separate from the general **Config** page. It edits the `face_recognition:` section of the worker's `config.yaml` through form fields instead of hand-editing YAML, and includes a live **📊 Visual Guide** (a collapsible panel above the form) that redraws a pipeline diagram and threshold dials as you type, before you save, so you can see the effect of a value before committing to it.

> **Important for installers:** most fields here directly control detection accuracy versus false-positive/false-negative rate on site. Read each field's description below (or hover the ℹ️ icon next to it in the UI, which shows the same text) before changing it, and change one group at a time so you can attribute any behavior change to the right setting. Most changes require an **application restart** to take effect — the page shows this warning at the top.

### Detection pipeline (what order things are checked)

`Frame → Motion Gate → Face Found (detector) → Angle / Eye Check → Duplicate Check → Saved & Sent to hub`

The field groups below are named after the pipeline stage they gate.

### General

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| Consumer Threads | `num_consumer_threads` | `3` | Number of worker threads consuming frames from the detection queue after the motion gate. |
| Frame Min Diff | `frame_min_diff_threshold` | `5` | Motion-diff sensitivity for the frame gate ahead of face detection. Values below `1` are raised to `1` automatically at startup. |

### ⏱️ Check-Interval Guard (adaptive load-shedding)

Widens the gap between face-detection checks when the processing queue is under load, then relaxes it back down once the queue drains — so a burst of traffic doesn't overwhelm detection.

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| Check Frame Interval | `check_frame_interval` | `15` | How many frames pass between face-detection checks under normal load. |
| Max Interval Multiplier | `check_interval_max_multiplier` | `4` | Under high queue load, **Check Frame Interval** is widened up to `Check Frame Interval × this multiplier`, so fewer motion-passing frames get queued. It relaxes back down once the queue drains below **Low Load Ratio**. |
| High Load Ratio | `check_interval_high_load_ratio` | `0.8` | Fraction of queue capacity (`0`–`1`) that counts as "high load" — once the queue fills past this, the check interval starts widening toward the max multiplier. |
| Low Load Ratio | `check_interval_low_load_ratio` | `0.4` | Fraction of queue capacity (`0`–`1`) that counts as "low load" — once the queue drains below this, the check interval relaxes back to its base value. |
| Max Queue Size | `check_interval_max_queue_size` | `400` | Capacity of the frame queue between the camera reader and the face-detection threads. High/Low Load Ratio are fractions of this number (e.g. `0.8 × 400 = 320` queued frames counts as high load). |

### 🚦 Capture Rate Limit

Caps how many new (non-duplicate) faces a camera may log per rolling time window — protects against a flood of genuinely distinct new faces at a crowded event, which neither the motion-diff fields above nor Duplicate Detection below can filter, since each such face really is a new person.

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| Enable Capture Rate Limit | `capture_rate_limit_enabled` | `true` | Master switch for this group. |
| Max Captures | `capture_rate_limit_max_captures` | `30` | Maximum new faces logged per camera within the rolling window below. |
| Window (s) | `capture_rate_limit_window_seconds` | `60` | Length, in seconds, of the rolling window **Max Captures** applies to. |

These are the **defaults for all cameras**; per-camera overrides are set on the Camera config page (`/config/camera`).

### 🧬 Duplicate Detection

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| Similarity Threshold | `similarity_threshold` | `0.62` | How close two faces need to be to count as the same person. Lower = stricter (harder to match, more faces logged as new/different people); higher = looser (easier to match, more faces treated as a repeat of someone already seen). |
| Similarity Timeout (s) | `similarity_timeout_seconds` | `30` | How long, in seconds, the worker remembers a face after seeing it. Seeing the same person again within this window is treated as a repeat and not re-logged. Larger = fewer duplicate entries for someone staying in view; smaller = more frequent re-logging of the same person. |

### 📐 Pose & Eye Rejection

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| **Reject Low-Quality Faces** | `reject_low_quality_faces` | `true` | **Master switch for this whole group.** ON: enables the head-turn, tilt, and eye-position checks below, so turned or tilted faces get rejected. OFF: every detected face is accepted regardless of angle, and the three checks below don't run. |
| Rejection Yaw (°) | `rejection_threshold_yaw` | `32.0` | How far a face can turn left/right and still be accepted. Turned further than this, the face is rejected. |
| Min Eye Ratio | `min_eye_ratio` | `0.32` | Front-facing check based on how far apart the eyes appear relative to face width — **not** about blinking or closed eyes. A face turned too far to the side (eyes appear closer together) is rejected below this value. |
| Rejection Pitch (°) | `rejection_threshold_pitch` | `15` | How far a face can tilt up/down and still be accepted — enforced only while **Enable Pitch Check** (below) is ON. |
| Enable Pitch Check | `enable_pitch_check` | `true` | ON: the tilt check above is enforced. OFF: tilt is never checked, regardless of how tilted the face is. **Renamed** from `ignore_pitch_deviation` (the old name read as the inverse of what it did — `true` already meant "pitch is checked"). Old configs are **not** automatically migrated: if upgrading from a version that shipped `ignore_pitch_deviation` in a saved `config.yaml`, re-check this switch on `/config/face_recognition` after upgrading. |

### 🔍 Detection Quality Rejection

Rejects low-quality detections (blurry, tiny, or low-confidence) before they reach the hub — independent of, and in addition to, the pose/eye checks above.

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| Min Detection Score | `min_det_score` | `0.5` | The face detector's own confidence (`0`–`1`) that a detected region is really a face. Detections scoring below this are rejected — catches things like hair, ears, or partial occlusions being mistaken for a face. |
| Min Face Area Ratio | `min_face_area_ratio` | `0.01` | Minimum size of the detected face box relative to the full frame area (e.g. `0.01` = 1% of the frame). Rejects faces too small/far away to be reliably identified. |
| Min Blur Variance | `min_blur_variance` | `15.0` | Sharpness of the face crop, measured as Laplacian variance (higher = sharper). Faces blurrier than this are rejected. Tune per camera — low-resolution or heavily compressed streams may need a lower value. |

### Other toggles

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| Use GPU (CUDA) | `use_gpu` | `false` | Runs face detection/embedding on a CUDA GPU instead of CPU. |
| Age Estimation | `age_estimation_enabled` | `false` | Estimates and logs an approximate age for each detected face. |
| Enable Face-Recognition Logging | `loggin_enable_face_recognition` | `false` | Extra diagnostic logging for the face-recognition pipeline. |

### Where these are stored in `config.yaml`

The **Check-Interval Guard** and **Capture Rate Limit** groups above are stored nested under `face_recognition.guards` (`guards.check_interval.*`, `guards.capture_rate_limit.*`); **Frame Min Diff** is stored under `guards.frame_diff.frame_min_diff_threshold`. Everything else in this section is a flat key directly under `face_recognition:`. Per-camera overrides sent from the hub's camera metadata still use flat keys (`frame_min_diff_threshold`, `capture_rate_limit_enabled`, `capture_rate_limit_max_captures`, `capture_rate_limit_window_seconds`) and are merged into the nested `guards` structure automatically — no change needed on the hub side.

---

## 🎯 Face Tuning Tool (`/tools/face_tuning`)

Per-camera detection tuning, with a live pass/fail report and plain-language explanations for the install team, now has its own page: see the **[Face Tuning Tool Guide](face_tuning_guide.md)**.

---

## 📷 Configuring Cameras (`/config/camera`)

Camera setup has its own dedicated page in the worker's admin UI — `/config/camera` — with two sections reachable from a sticky sidebar: **📷 Camera List** and **🎛️ Camera Settings**. As on the Face Recognition page, most changes here require an **application restart** to take effect (the page shows this warning in the sidebar).

### 📷 Camera List

This table is only **editable** when **Camera Provider** (`Config → Commons`) is set to `local`. With any other provider, the hub owns the camera list, the fields here are shown **read-only**, and the page tells you where to switch it back if you need to edit locally.

> **Camera Provider = `hub`:** cameras are no longer typed in on this page at all — they are created once on the hub's **Camera Pool** page (`/workers/camera_pool`) and allocated to this worker from there. This page then simply mirrors what the hub has allocated, live, and updates without a restart when the allocation changes. See the **Managing the Camera Pool** section of the [ID Vision Hub Service Setup Guide](../hub/id_vision_hub_service_setup_guide.md) for how to add and allocate a camera.
>
> Two optional keys under `commons` in this worker's `config.yaml` control a diagnostic feature for this mode: `report_camera_lock` (`true`/`false`, default `false`) makes the worker tell the hub which camera(s) it is actively streaming, so the hub's Camera Pool page can show a lock status and warn about two workers claiming the same camera; `lock_reaffirm_interval_sec` (default `60`) is how often, in seconds, it repeats that report. Leave `report_camera_lock` off unless you specifically want this status visible on the hub.

| Column | Config key | Description |
| ------ | ---------- | ------------ |
| Id | `id` | Local identifier for the camera row. |
| Camera Code | `camera_code` | The camera's code as known to the rest of the system (numeric if it parses as a number, otherwise kept as text). |
| Title | `title` | Display name for the camera. |
| RTSP / Video Source | `rtsp_link` | The stream URL or path the worker reads frames from — an `rtsp://` URL, a Windows webcam device (`video=...`), or a local video file. |

Other controls on this table:
- **➕ Add Camera** / **✖** — add or remove a row. Rows left completely blank are dropped silently when you save.
- **📸 Preview** — fetches a live snapshot from that row's RTSP/Video Source right from the browser (via `/api/video_snapshot`), so you can confirm a stream URL works **before** saving the camera list.
- **⚙️ (per-camera overrides)** — expands an advanced row with per-camera settings that override the site-wide defaults from the **Configuring Face Recognition** section above, for that one camera only:

  | Field | Overrides | Behavior when left blank / "Inherit" |
  | ----- | --------- | -------------------------------------- |
  | Frame Min Diff | `frame_min_diff_threshold` | Uses the site-wide **Frame Min Diff** value |
  | Rate Limit | `capture_rate_limit_enabled` | Uses the site-wide **Enable Capture Rate Limit** switch |
  | Max Captures | `capture_rate_limit_max_captures` | Uses the site-wide **Max Captures** value |
  | Window (s) | `capture_rate_limit_window_seconds` | Uses the site-wide **Window (s)** value |

  Use this when one camera needs different motion sensitivity or capture pacing than the rest — for example, a camera pointed at a busy entrance versus one covering a quiet corridor.

**Setting these from Face Tuning instead:** the worker's **Face Tuning** tool (`/tools/face_tuning`) can capture a short burst of frames from a chosen camera, suggest values for these same per-camera fields based on what it sees, and save them with one click via its **Save Thresholds to Camera** button — instead of computing and typing them in here by hand. With **Camera Provider** set to `local`, this writes into the camera's row in `config.yaml`, same as editing it on this page. With **Camera Provider** set to `hub`, it instead saves to that camera's record on the hub's **Camera Pool** page and applies immediately on this worker, without an application restart — and because it's stored on the hub, the tuning is not lost if the camera is later reassigned to a different worker. See the **[Face Tuning Tool Guide](face_tuning_guide.md)** for a step-by-step walkthrough, plain-language explanations of each value with its min/max range, and guidance for profile/angled capture.

### 🎛️ Camera Settings

These apply to every camera (there is no per-camera override for this section).

| Field | Config key | Default | Description |
| ----- | ---------- | ------: | ------------ |
| Detection Size (w × h) | `det_size` | `640 × 640` | Frame size the face detector is prepared with. Larger sizes improve accuracy at range but cost more CPU/GPU per frame. |
| Best Resolution (w × h) | `best_Resolution` | `1280 × 720` | Resolution ceiling for the incoming stream. If the camera's actual resolution is larger, frames are scaled down to this size; it is also the resolution used when **Fetch Resolution From Source** is off, or when detecting the source resolution fails. |
| Image Retention (min) | `image_retention_minutes` | `120` | How long stored face-log images are kept before being deleted. |
| Connect Timeout (s) | `camera_connect_timeout_seconds` | `6` | How long to wait when connecting to a camera stream before treating the attempt as failed and retrying. |
| Display Frame Interval | `display_frame_interval` | `10` | Only relevant while **OpenCV Preview Window** is on — how many frames pass between window redraws. |
| Fetch Resolution From Source | `fetch_resolution_from_source` | `false` | ON: probes the camera to read its actual resolution at connect time (capped by **Best Resolution**), falling back to **Best Resolution** if the probe fails. OFF: skips the probe and uses **Best Resolution** directly, which connects faster. |
| Set Scale | `is_set_scale` | `true` | ON: the incoming stream is rescaled to the resolved width/height before processing, so every frame is a consistent size. OFF: the stream is processed at its native size. |
| OpenCV Preview Window | `enable_opencv_win` | `false` | Opens a local preview window on the machine running the worker (not in the browser) for debugging camera feeds on-site. Leave off in a normal unattended/headless deployment. |

---

## 🧠 Notes & Tips

- Always **run as Administrator**.
- The service runs Python inside the virtual environment located at:

  ```
  .venv\Scripts\python.exe
  ```

- The environment variable `RUNNING_AS_SERVICE=1` is automatically set.
- Logs (if enabled in the script) will appear as:

  ```
  service.log
  service_error.log
  ```

---

## 🧾 Example Workflow Summary

1. Create folder:

   ```
   C:\work_space\id_vision_worker
   ```

2. Copy:

   - `install_id_vision_worker_service.bat`
   - `id_vision_worker-0.0.26-py3-none-any.whl`
   - `wheelhouse\` folder
   - `run_id_vision_worker.bat`

3. Run script as **Administrator**
4. Choose:

   - Offline install (`1`)
   - Install service (`1`)

5. Start service

---

**Author:** Qasem Taheri
**Last Updated:** September 2026
