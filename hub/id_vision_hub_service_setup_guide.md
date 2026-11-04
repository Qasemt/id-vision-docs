# 📘 ID Vision Hub Service Installation & Management Guide

This document provides detailed instructions for installing, configuring, and managing the **ID Vision Hub** service on Windows using the `install_id_vision_hub_service.bat` script.
It ensures correct environment setup, offline/online package installation, and robust service management through **NSSM**.

---

## ⚙️ Prerequisites

Before running the installer, please ensure all dependencies are properly installed and configured.

### 🐍 Python

- **Required version:** Python **3.12.5** or newer
- **Download:** [Python 3.12.5 (x64)](https://www.python.org/ftp/python/3.12.5/python-3.12.5-amd64.exe)
- During setup, **check the box**:

  > ✅ “Add Python to PATH”

#### Verify installation

```bash
python --version
# Example: Python 3.12.5
```

```bash
pip --version
# Example: pip 23.0.1
```

---

### 🧱 Microsoft Visual C++ Redistributable

Some dependencies (e.g., numpy) require Visual C++ runtime libraries.
Install from Microsoft’s official source:

🔗 [VC++ Redistributable (x64)](https://aka.ms/vs/17/release/vc_redist.x64.exe)

---

### 🎞️ FFmpeg (Video & Media Processing)

**ID Vision Hub** depends on FFmpeg for handling video streams, snapshots, and encoding.

1. Download Windows build from:
   [FFmpeg Official Builds](https://www.gyan.dev/ffmpeg/builds/ffmpeg-release-essentials.7z)
2. Extract to:

   ```
   C:\work_space\tools\ffmpeg
   ```

3. Add the path to Windows environment variable:

   ```
   C:\work_space\tools\ffmpeg
   ```

> 🧩 The installer automatically validates and adds FFmpeg to both session and system PATH.

---

### 🔧 NSSM (Non-Sucking Service Manager)

Used to install Python applications as Windows services.

- **Download:** [https://nssm.cc/download](https://nssm.cc/download)
- **Place here:**

  ```
  C:\work_space\tools\nssm\nssm.exe
  ```

---

## 📂 Folder Structure

Prepare the following directory structure before installation:

```
C:\work_space\id_vision_hub\
│
├── install_id_vision_hub_service.bat
├── run_id_vision_hub.bat
├── id_vision_hub-0.0.xx-py3-none-any.whl
└── wheelhouse\
    └── numpy-1.26.0-cp312-cp312-win_amd64.whl
```

| File / Folder                       | Description                                 |
| ----------------------------------- | ------------------------------------------- |
| `install_id_vision_hub_service.bat` | Main service setup script                   |
| `run_id_vision_hub.bat`             | Entry script executed by NSSM service       |
| `id_vision_hub-*.whl`               | Offline wheel package                       |
| `wheelhouse\`                       | Contains offline dependencies (numpy, etc.) |

---

## 🚀 Installation Procedure

### 1️⃣ Run the Installer

- Right-click `install_id_vision_hub_service.bat`
- Choose **Run as Administrator**

The script will:

- Check administrative privileges
- Prompt for configuration paths
- Verify `nssm.exe` and `ffmpeg.exe`
- Automatically create or reuse a Python virtual environment
- Install the `id_vision_hub` package (offline or online)
- Provide a full **Service Management Menu**

---

### 2️⃣ Configuration Prompts

When executed, the script will ask for configuration values with defaults such as:

```
Enter the main project directory [C:\work_space\id_vision_hub]:
Enter the Service Name (no spaces) [id_vision_hub]:
Enter the Service Display Name [ID Vision Hub Service]:
Enter the Service Description [Service for running ID Vision Hub]:
Enter the path to nssm.exe [C:\work_space\tools\nssm\nssm.exe]:
Enter the FFMPEG directory [C:\work_space\tools\ffmpeg]:
```

Press **Enter** to accept defaults, or provide custom paths.

---

### 3️⃣ Selecting the Wheel File

The script automatically detects `.whl` files in your directory and displays a menu:

```
Found 1 wheel file(s) in C:\work_space\id_vision_hub:
[1] id_vision_hub-0.0.30-py3-none-any.whl
Select a wheel file (1-1) or press Enter for default [1]:
```

Choose the desired version and continue.

---

## 🧩 Installation Methods

You can choose between **offline** and **online** installation modes.

### 🔸 Option 1 — Offline Install (Recommended)

Uses local `.whl` and `wheelhouse` folder:

```
Please choose the installation method:
[1] Offline install (from project directory)
[2] Online install (from PyPI)
Enter your choice (1/2): 1
```

> Make sure your directory includes:
>
> - `id_vision_hub-*.whl`
> - `wheelhouse\numpy*.whl`

### 🔸 Option 2 — Online Install

If the system has internet access, select option **2** to install directly from PyPI:

```
pip install id_vision_hub
```

---

## 🧠 Virtual Environment Details

The script automatically creates and manages a virtual environment at:

```
C:\work_space\id_vision_hub\.venv
```

It then activates it for package installation and service operations.

To activate manually:

```bash
cd C:\work_space\id_vision_hub
.venv\Scripts\activate
```

---

## 🧰 Service Management Menu

After setup, the script shows this main control panel:

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
[7] Uninstall id_vision_hub Package
[8] Exit
---------------------------------------------------
Choose an option:
```

### 🔹 Quick Reference

| Option | Action                                                 |
| :----: | :----------------------------------------------------- |
|   1    | Install the service (creates Windows service via NSSM) |
|   2    | Remove the service                                     |
|   3    | Start the service                                      |
|   4    | Stop the service                                       |
|   5    | Restart the service                                    |
|   6    | View current service status                            |
|   7    | Uninstall the `id_vision_hub` package                  |
|   8    | Exit menu                                              |

---

## 🔐 License Activation & Maintenance Mode

**ID Vision Hub** requires a signed, hardware-bound license before it will leave Maintenance Mode.

### 🧩 How It Works

- On startup, the hub validates `files\license\license.json` against the packaged public key and this machine's hardware fingerprint.
- If the license is missing, invalid, or expired, the hub runs in **Maintenance Mode** until a valid license is uploaded.
- The license also carries the account's **worker** and **camera** limits (`max_workers`, `max_cameras`); once active, the hub enforces these limits when registering new workers/cameras.
- A license issued for **ID Vision Worker** is explicitly rejected by the hub, and vice versa — each product checks its own `product` field in the license file, so the hub and each worker need their own separate `license.json`, even though all are signed with the same key.

### 🚀 Activating a License

1. Open the hub's admin UI: `http://<hub-host>:<port>/maintenance/` (default port **5001** — <!-- TODO: confirm --> if changed during installation).
2. Copy the **hardware fingerprint** shown on the page.
3. Send the fingerprint to whoever issues licenses and obtain a `license.json` signed for **this** fingerprint with `"product": "id_vision_hub"`.
4. Upload the `.json` file on the `/maintenance/` page.
5. On success, the hub exits maintenance mode automatically.

### 📂 License File Locations

| File             | Default path (relative to working directory) | Override                    |
| ---------------- | ---------------------------------------------- | ----------------------------- |
| `license.json`   | `files\license\license.json`                  | `ID_VISION_LICENSE_PATH`     |
| `public_key.pem` | `files\license\public_key.pem`                | `ID_VISION_PUBLIC_KEY_PATH`  |

License levels are `TRIAL`, `BASIC`, `PRO`, `ENTERPRISE`.

### 🎁 Requesting a Free Trial License

While no valid license is active, the `/maintenance/` page offers **Request a Free Trial License**.

1. Make sure the hub machine has internet access.
2. Enter the customer's company or organization name.
3. Click **Request Trial License**.

The trial is issued for **this** machine's fingerprint and lasts **14 days**, with at most **5 cameras** and **1 worker**. Each machine gets **one** trial; a second request is refused. Trials are **not** available on a virtual machine or container — install on physical hardware, or obtain a full license.

### 🖥️ Licenses on a Virtual Machine

On a virtual machine or container, a license works for at most **2 years** from its issue date, even if its own expiry date is later. The license details on the `/maintenance/` page show the earlier of the two dates.

When the 2 years are over, the hub enters Maintenance Mode and the page says so. To keep using the hub, install it on a physical machine or use a hardware key (dongle) license. Physical machines and dongle licenses are not affected by this limit.

### 🤝 Partner-Issued Licenses

A license issued by a partner company from the **Issue a license (partner companies)** section of the `/maintenance/` page carries that partner's name. When the partner's branding is built into the hub, the **About** page and the page footer show the partner's name, logo, and copyright instead of **ID Vision Hub**'s. The software version shown is always the hub's real version.

---

## 📷 Managing the Camera Pool (`/workers/camera_pool`)

The hub's admin UI has a **Camera Pool** page, linked from the **Workers** page, where every camera in the deployment is defined once and then handed out to workers — instead of typing the same camera's RTSP link into more than one worker by hand, which risks the same physical camera ending up configured on two workers at once.

### 🧩 How It Works

- A camera is created in the pool with its **Camera Code**, **Title**, **RTSP Link**, and optional **Scale** — it starts **unassigned**.
- An unassigned camera is then **allocated** to one worker from the Camera Pool page. Only an allocated camera actually gets picked up and run by a worker.
- A camera can be **unallocated** at any time, returning it to the pool so it can be reassigned to a different worker without re-entering its details.
- Any worker whose **Camera Provider** setting (`Config → Commons` on the worker's own admin UI) is `hub` reads its camera list from this pool automatically — its own `/config/camera` page becomes **read-only** and simply displays what the hub has allocated to it. See the **Configuring Cameras** section of the [ID Vision Worker Service Setup Guide](../worker/id_vision_worker_service_setup_guide.md) for what that looks like from the worker side.

### 🧭 Creating and Allocating a Camera

1. On the **Camera Pool** page, click **Add Camera**.
2. Enter the **Camera Code**, **Title**, **RTSP Link**, and **Scale** — or click **Select from Padis Cloud...** to pick a camera from the Padis cloud catalogue and prefill these fields automatically.
3. Save. The camera appears in the pool as **unassigned**.
4. Click **Allocate** on that camera's row and choose the worker it should run on.
5. The worker is notified immediately; no manual restart of the worker service is required for the camera to start.

To move a camera to a different worker, click **Unallocate** first, then **Allocate** it to the new worker. Its tuning (see **Fine-Tuning and the Camera Pool** below) is not lost when you do this.

### 📥 Bulk Import / Export

- **Export CSV** downloads every pool camera (`camera_code`, `title`, `rtsp_link`, `scale`, `enabled`) as a backup file.
- **Import CSV** accepts the same file back in: a row whose `camera_code` matches an existing pool camera updates it, and an unmatched `camera_code` is added as a new, unassigned camera. Camera allocation (which worker owns it) is **not** included in the file and must be redone by hand after an import.

### 🚦 Lock Status and Conflicts

Once a camera is allocated and running, the Camera Pool page shows a **Lock Status** badge confirming which worker is actively streaming it, so you can tell "assigned to" (the admin's decision) apart from "actually running on" (what's really happening on site). If two workers both report running the same camera, a **Conflict** badge appears on that row, and a matching camera-conflict count badge appears on the Workers page — resolve it by unallocating the camera from whichever worker should not have it.

**Note:** this status only appears for a worker that has `report_camera_lock: true` set under `commons` in its own `config.yaml` (off by default — enable it per site once you've confirmed that site's workers should report their lock status). A worker with this left at its default reaffirms nothing, and its cameras always show as **Not connected** on this page even while running normally.

### 🎚️ Fine-Tuning and the Camera Pool

A worker's **Face Tuning** tool can save detection thresholds (yaw, pitch, eye ratio, and similar quality-gate values — see the [Face Tuning Tool Guide](../worker/face_tuning_guide.md) for what each one does) directly to a camera's Camera Pool record. Because this tuning lives on the pool record instead of on the worker, it survives that camera being unallocated and reallocated to a different worker later — you do not need to redo tuning after moving a camera between workers. The pool table's **Last Tuned** column shows when a camera was last tuned this way, and by which worker.

### 🔲 Detection Area (ROI)

Limit each camera to the part of the image where people should be detected — for example, the walkway in front of a door rather than the corridor behind it.

1. On the **Camera Pool** page, click **Edit** on the camera's row.
2. Click **Detection Area (ROI)**. The hub shows a snapshot from the camera. If the hub server cannot reach the camera network, a blank frame is shown instead; it stands for the whole camera image.
3. Click on the image to place the corners of an area. Click the first corner again (or **Close Shape**) to finish it. Add as many areas as needed.
4. Click **Apply**, then **Save Changes**. The camera's worker is notified and uses the new area without a restart.

To remove the limit, open the editor, click **Clear All**, then **Apply** and **Save Changes**. With no area, the whole image is used.

How each worker uses the area:

| Worker | A detection counts when |
| ------ | ----------------------- |
| **ID Vision Worker** | the centre of the face is inside an area |
| **Hygiene Vision Worker** | the person's feet (the bottom middle of their box) are inside an area — draw over the floor where people stand |

**Note:** the area is stored with the camera in the pool, so it moves with the camera when you allocate it to another worker.

---

## 🧾 Service Configuration Details

| Parameter                | Description                                     |
| ------------------------ | ----------------------------------------------- |
| **Service Name**         | `id_vision_hub` (default)                       |
| **Display Name**         | `ID Vision Hub Service`                         |
| **Restart Behavior**     | Automatically restarts even after exit code 0   |
| **Start Type**           | `SERVICE_AUTO_START`                            |
| **Environment Variable** | `RUNNING_AS_SERVICE=1` (used by Python process) |
| **Working Directory**    | `%WORKING_DIR%`                                 |
| **Runner Script**        | `run_id_vision_hub.bat`                         |

> Logs can optionally be redirected using `AppStdout` and `AppStderr` if desired.

---

## 🪪 Troubleshooting Tips

| Issue                            | Possible Cause                       | Solution                                      |
| -------------------------------- | ------------------------------------ | --------------------------------------------- |
| ❌ `NSSM not found`              | Wrong `nssm.exe` path                | Check `C:\work_space\tools\nssm\nssm.exe`     |
| ⚠️ `FFMPEG validation failed`    | Incorrect directory or missing files | Ensure `ffmpeg.exe` and `ffprobe.exe` exist   |
| 🐍 `Package installation failed` | Missing `.whl` or dependencies       | Verify `wheelhouse` and retry offline install |
| 🔁 `Service doesn't restart`     | NSSM config not updated              | Re-run option `[1] Install Service`           |
| 🔒 Redirected to `/maintenance/` on every page | License missing, invalid, expired, or issued for the wrong product | Upload a `license.json` signed for this hub's fingerprint with `"product": "id_vision_hub"` — see the **License Activation & Maintenance Mode** section above |
| 🖥️ Maintenance page says the virtual-machine period has ended | The license has run for 2 years on a virtual machine | Install the hub on a physical machine, or use a dongle license |
| 🎁 Trial request refused | A trial was already issued for this machine, the machine is a virtual machine, or there is no internet access | Obtain a full license, or check the internet connection and retry |

---

## 📄 Example Folder Snapshot

```
C:\work_space\id_vision_hub\
│
├── install_id_vision_hub_service.bat
├── run_id_vision_hub.bat
├── id_vision_hub-0.0.30-py3-none-any.whl
├── wheelhouse\
│   └── numpy-1.26.0-cp312-cp312-win_amd64.whl
└── .venv\
    ├── Scripts\
    └── Lib\
```

---

## ✅ Summary

| Step | Action                              |
| ---- | ----------------------------------- |
| 1    | Run script as Administrator         |
| 2    | Confirm or edit configuration paths |
| 3    | Select `.whl` file                  |
| 4    | Choose offline/online installation  |
| 5    | Install service (option 1)          |
| 6    | Start service and verify status     |

---

**Author:** Qasem Taheri
**Last Updated:** November 2026
