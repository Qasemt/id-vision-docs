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
