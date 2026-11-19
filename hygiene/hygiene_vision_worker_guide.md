# 🧼 Hygiene Vision Worker Guide

How to install, configure, and read the results of the **Hygiene Vision Worker**, the food-safety hygiene camera service, for the install team.

---

## 1. Overview

The **Hygiene Vision Worker** watches RTSP cameras in a food-production area. It finds every person in the frame and checks each one for the required gear and for forbidden behavior. Each violation is stored on the worker with a snapshot, and you review it on the worker's **Violations** page.

The worker is a separate product from the **ID Vision Worker**. It does not recognize faces or identify people. It only judges what each person is wearing or doing. It can run on its own, or send its violations to an **ID Vision Hub**.

### What it checks

| Code | Meaning | Reported when |
| ---- | ------- | ------------- |
| `no_hair_net` | Required hair net is missing | The head is visible, with some part of the face, and no hair net is found |
| `no_face_mask` | Required face mask is missing | The face is visible and no mask is found |
| `no_apron` | Required apron is missing | Both shoulders, both hips, and some part of the face are visible, and no apron is found |
| `bare_hands` | Hands without gloves | A bare hand is found at a visible wrist, and some part of the face is visible |
| `smoking` | Cigarette, vape, or e-cigarette | Found near the mouth, with confidence ≥ 0.25 |
| `person_down` | A person lying still on the floor (fainted, fallen) | The person lies still for 20 seconds |

The worker also shows codes ending in `_unverified` (for example `face_mask_unverified` or `hands_unverified`). These mean "the camera could not see that body part, so nobody can tell". They are **informational only**. They never make a person non-compliant and are never stored as violations.

---

## 2. Prerequisites

| Item | Requirement |
| ---- | ----------- |
| Operating system | Windows, 64-bit (x64) <!-- TODO: confirm the minimum Windows version; the installer does not set one --> |
| Account | Administrator rights to run the installer |
| Network | The worker must reach each camera's RTSP stream |
| Free port | TCP **8002** for the web UI (8001 is used by the **ID Vision Worker**) |
| Internet | Needed once, to fetch the setup files. Without internet, upload them by hand (see **Section 4**) |
| License | A signed license file (`license.json`) or a free trial license |

You do **not** need to install Python, FFmpeg, or NSSM yourself. The installer ships all of them.

**Note:** The worker runs on the CPU by default. The pose model alone takes about 0.8 s per frame on a 4-core CPU. It runs at most once every 2 seconds per camera. <!-- TODO: confirm a sizing table (cameras per CPU) for this worker -->

---

## 3. Install the Worker

1. Run `hygiene_vision_worker-setup-<VERSION>.exe` as Administrator.
2. Accept the default folder: `%LOCALAPPDATA%\HygieneVisionWorker`.
3. Wait for the step **"Setting up Python and Hygiene Vision Worker (first run only, no internet needed)..."** to finish.
4. Leave **"Open the Hygiene Vision Worker web UI"** checked and click **Finish**.

### Verify the installation

Open the web UI in a browser on the worker machine:

```
http://127.0.0.1:8002/
```

Success condition: the **Hygiene Vision Worker** home page loads. On a fresh install it opens the **License** page (maintenance mode) until a license and the setup files are in place.

Check the Windows service from a Command Prompt:

```bash
sc query hygiene_vision_worker
# Expected output includes: STATE : 4 RUNNING
```

To open the UI from another computer, allow inbound TCP 8002 in Windows Firewall, then browse to `http://<worker-ip>:8002/`.

**Important:** The web UI has no login. Expose it on the site network only, never to the internet.

### Service management

The installer registers the auto-start Windows service `hygiene_vision_worker` (display name **Hygiene Vision Worker**). It restarts automatically 5 seconds after any exit.

| Task | Command (Administrator Command Prompt) |
| ---- | -------------------------------------- |
| Status | `sc query hygiene_vision_worker` |
| Stop | `sc stop hygiene_vision_worker` |
| Start | `sc start hygiene_vision_worker` |
| Restart | `sc stop hygiene_vision_worker` then `sc start hygiene_vision_worker` |
| Uninstall | Start menu → **Hygiene Vision Worker** → **Uninstall Hygiene Vision Worker** |

| Data | Location |
| ---- | -------- |
| Configuration | `%LOCALAPPDATA%\HygieneVisionWorker\files\configs\config.yaml` |
| Logs | `%LOCALAPPDATA%\HygieneVisionWorker\files\logs` (kept 10 days), also on **Monitoring → Logs** |
| Violation snapshots | `%LOCALAPPDATA%\HygieneVisionWorker\files\images` |
| License | `%LOCALAPPDATA%\HygieneVisionWorker\files\license` |

---

## 4. Activate the License and Fetch the Setup Files

Open **🔑 License** (`/maintenance/`). The worker stays on this page, and no camera runs, until both items below are done.

### 4.1 License

Choose one:

- **Upload a license**: click **📋 Copy Fingerprint** and send the fingerprint to your administrator. When you receive `license.json`, drop it on **Click or Drag & Drop License File Here** and click **⬆️ Upload and Validate License**.
- **Free trial**: fill in the customer name under **Request a Free Trial License**. The trial is valid for 14 days. You get one trial per machine. A trial is refused inside a virtual machine or container.

**Note:** Inside a virtual machine or container, a license without a USB dongle needs the license server every 12 hours and stops working 2 years after it was issued. Use a physical machine, or a dongle-bound license, for permanent installs.

Success condition: **License Status** shows the customer, level, and expiry date.

### 4.2 Setup files

The detection models are not part of the installer. Choose one:

- **With internet**: click **Fetch Setup Files Automatically**.
- **Without internet**: copy the setup zip to the worker machine and upload it with **Upload Setup File**. <!-- TODO: confirm how the install team gets the setup zip for offline sites -->

Success condition: **Setup Status** reports the setup files as installed.

When both are done, the page shows **This worker is licensed and ready**. Restart the service to start the cameras:

```bash
sc stop hygiene_vision_worker
sc start hygiene_vision_worker
# Expected output of `sc query hygiene_vision_worker`: STATE : 4 RUNNING
```

---

## 5. Add Cameras

Open **🧰 Tools → 🎥 Cameras** (`/config/camera`).

1. Click **➕ Add Camera**.
2. Enter a **Camera Code**, a **Title**, and the **RTSP / Video Source**, for example `rtsp://user:password@192.168.1.10:554/stream1`.
3. Click **📸 Preview** to confirm the stream opens.
4. Click **💾 Save Camera List**.

Saving the list applies it at once. New cameras start, removed or disabled ones stop, and cameras you did not change keep running. No restart is needed.

**Note:** If the worker takes its cameras from the hub (see **Section 9**), add and enable cameras on the hub's **Camera Pool** page instead. The list fields are read-only here.

### 5.1 Per-camera settings

Under **🎛️ Camera Settings**, each camera can override:

| Setting | What it does | Default |
| ------- | ------------ | ------- |
| **Violations** | The violation codes this camera reports. Unchecked codes are not reported for this camera | All checked |
| **✏️ Detection Area** | Polygons drawn on the camera snapshot. A person counts only when the bottom middle of their box (their feet) is inside | Whole frame |
| **Lying Angle (°)** | How far the torso must tilt from vertical to count as lying, for `person_down` | 60° |
| **Frame Min Diff** | How much the image must change before the worker looks at a frame. Lower catches slower movement but uses more CPU | 5 |
| **Max Captures** / **Window (s)** | At most this many violation events per camera in this rolling window | 30 / 60 s |

Click **💾 Save Camera Settings**. The camera restarts with the new values.

### 5.2 Test a camera before going live

Open **🧰 Tools → 🧪 PPE Test** (`/tools/ppe_test`). Upload a photo, pick a sample, or pick a camera to grab one frame. The page shows the annotated frame and a report per person. Nothing is stored unless **Save violations** is ticked.

**Note:** The PPE Test page judges one frame. The live pipeline is stricter: a violation must appear in 2 of a person's last 3 checks, and `person_down` needs 20 seconds without movement.

---

## 6. Reading the Results

Each person gets a box and a number. A **green** box labelled `OK` means compliant. A **red** box lists the violations. All images below are real output from the worker, made with the default settings on sample photos.

### Compliant worker

Hair net, mask, gloves, and the person is facing the camera. Nothing to report.

<img src="../assets/hygiene_compliant_worker.jpg" alt="A worker in hair net, mask, and black gloves inside a green OK box" width="70%">

### Bare hands

Both workers wear a hair net, mask, and apron, but neither wears gloves. Each one is reported as `bare_hands`.

<img src="../assets/hygiene_bare_hands.jpg" alt="Two workers with hair nets, masks, and aprons, each boxed in red with bare_hands" width="70%">

### Several people in one frame

The worker judges each person on their own:

- **#1** (back to the camera, hair net and apron visible) is `OK`.
- **#2** has no mask and bare hands.
- **#3** has bare hands. In this frame, a mask pulled down under the chin was **not** reported as `no_face_mask`.

<img src="../assets/hygiene_mixed_team.jpg" alt="Three workers: one OK, one with no_face_mask and bare_hands, one with bare_hands" width="70%">

### A cap is not a hair net

A baseball cap does not count as a hair net, so this worker is reported as `no_hair_net`. The person cut off at the bottom-left wears a hair net and is `OK`.

<img src="../assets/hygiene_cap_instead_of_hair_net.jpg" alt="A worker in a yellow cap and apron reported as no_hair_net and bare_hands" width="70%">

### A headscarf alone is reported as no hair net

The detector looks for a hair net. A headscarf without a hair net is reported as `no_hair_net`.

<img src="../assets/hygiene_headscarf_without_hair_net.jpg" alt="A worker in a headscarf, mask, and gloves reported as no_hair_net" width="70%">

If the site's hygiene policy accepts a headscarf, uncheck `no_hair_net` in that camera's **Violations** list. That turns the hair-net check off for **everyone** on that camera. Agree on this with the site's food-safety officer first.

### When the camera cannot see a body part

Person **#2** stands with their back to the camera. The worker cannot see the face or hands, so it records `face_mask_unverified` and `hands_unverified`. These are informational, so the box stays green. The supervisor on the left (**#1**) is clearly visible, and the missing mask is reported.

<img src="../assets/hygiene_back_view_unverified.jpg" alt="A supervisor reported as no_face_mask, and a worker seen from behind marked OK" width="70%">

---

## 7. Person Down

The worker reports `person_down` when a person lies on the floor without moving for **20 seconds**. It checks each camera at most once every 2 seconds, including when nothing moves. Each fallen person creates one event. There is no live alarm yet. Review `person_down` events on the **Violations** page.

### Detected

Four people lying across the frame are detected. The two nearest the camera lie along the camera's line of sight. Their torsos look upright in the image, so they are **not** detected.

<img src="../assets/hygiene_person_down_group.jpg" alt="Six people lying on the ground; four boxed as person_down, two nearest the camera not detected" width="60%">

From directly overhead, a person whose head points to the **bottom** of the image is detected:

<img src="../assets/hygiene_person_down_head_down.jpg" alt="Top-down view of a person lying with head toward the bottom of the image, boxed as person_down" width="70%">

### Not detected

From directly overhead, a person whose head points to the **top** of the image looks the same as a standing person, so it is **not** detected:

<img src="../assets/hygiene_person_down_missed_head_up.jpg" alt="Top-down view of a person lying with head toward the top of the image, not detected" width="45%">

A person half-lying, for example on stairs, with the torso at about 35–60°, is **not** detected at the default **Lying Angle** of 60°:

<img src="../assets/hygiene_person_down_missed_stairs.jpg" alt="A person fallen half-lying on stairs, not detected" width="32%">

| Situation | What to do |
| --------- | ---------- |
| Camera looks straight down | Expect missed falls. Uncheck `person_down` for that camera, or mount it at an angle |
| Stairs or ramps | Lower **Lying Angle (°)** for that camera only. Lower it only where nobody works bent over |
| Someone lies under a machine to repair it | Expect a false `person_down`. Exclude that spot with the **Detection Area** |

---

## 8. Camera Placement

The examples above lead to these rules:

- **Mount cameras at an angle that shows faces, not straight down.** From directly overhead the apron is foreshortened. In the frame below, both workers wear aprons, yet both are reported as `no_apron`. Mask, hair net, and apron checks need the front of the body.

<img src="../assets/hygiene_overhead_camera_false_apron.jpg" alt="Overhead view of two workers in aprons, both wrongly reported as no_apron" width="70%">

- **Frame the whole body.** `no_apron` needs both shoulders, both hips, and part of the face in view. A person cut off at the waist is never reported for the apron.
- **Face the direction people work in.** Workers seen only from behind are never reported for mask or hands.
- **Use the Detection Area** to ignore corridors, doorways, and areas outside the hygiene zone.

For general mounting height, lighting, and bitrate advice, see [Camera Setup](../camera-setup.md).

---

## 9. Send Violations to the ID Vision Hub

The worker can send its violations to an **ID Vision Hub** and take its camera list from the hub. Configure this on **🧰 Tools → ⚙️ Config** (`/config`):

1. In the **🛰️ Hub** section, turn on **Send Events**, and fill in **Hub URL** and **API Key**. The API key must belong to a hub service account with the **Operator** role.
2. Optional: turn on **Cameras from Hub** and click **Register with Hub**. The worker registers itself as a hygiene worker, and the hub admin allocates cameras to it on the **Camera Pool** page.
3. Restart the service.

Success condition: the **Violations** page shows a **Hub** column, and new events move from **pending** to **sent**.

Events are kept on the worker for 30 days, whether or not they were sent to the hub. The hub setup itself is covered in the [ID Vision Hub Service Setup Guide](../hub/id_vision_hub_service_setup_guide.md).

---

## Troubleshooting

| Symptom | Likely Cause | Resolution |
| ------- | ------------ | ---------- |
| Browser cannot open `http://127.0.0.1:8002/` | Service not running, or port 8002 already in use | Run `sc query hygiene_vision_worker`. Check `files\logs` for a port error, and free port 8002 |
| UI opens on the worker but not from other PCs | Windows Firewall blocks TCP 8002 | Allow inbound TCP 8002 |
| Every page redirects to **License** | No valid license, or setup files missing | Complete **Section 4**, then restart the service |
| **Fetch Setup Files Automatically** fails | No internet access on the worker | Use **Upload Setup File** with the setup zip |
| Trial request refused | Machine is a virtual machine or container, or a trial was already issued for this machine | Use a physical machine, or request a paid license |
| License lost after it worked inside a VM | License server unreachable for more than 12 hours, or 2 years passed since issue | Restore access to the license server, or move to a physical machine or dongle license |
| Camera shows no image in **📸 Preview** | Wrong RTSP URL, credentials, or network path | Test the URL in VLC from the worker machine, and fix the link |
| Many `no_apron` reports for people wearing aprons | Camera looks straight down | Re-mount the camera at an angle (see **Section 8**) |
| Only `_unverified` codes, never real violations | Camera sees people from behind, or cuts off their heads | Re-aim the camera to see faces and full bodies |
| Every worker in a headscarf reported `no_hair_net` | A headscarf is not a hair net to the detector | Agree on the policy with the site, then uncheck `no_hair_net` for that camera if needed |
| No `person_down` events for a real fall | Camera looks straight down, person is half-lying, or the person moved before 20 s | See the table in **Section 7** |
| False `person_down` near a machine | Staff lying down to repair it | Exclude that spot with the **Detection Area** |
| Violations stop after a burst | Per-camera rate limit reached (30 events per 60 s) | Raise **Max Captures** or **Window (s)** for that camera |

---

**Author:** Qasem Taheri
**Last Updated:** November 2026
