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
