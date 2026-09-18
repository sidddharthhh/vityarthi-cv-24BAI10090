<div align="center">

# 👁️‍🗨️ Smart Attendance — Face-Recognition Register

**Show your face, get marked present.** A desktop app that swaps the roll-call sheet for a webcam and a bit of computer vision.

![Python](https://img.shields.io/badge/Python-3.x-blue)
![OpenCV](https://img.shields.io/badge/OpenCV-LBPH-green)
![GUI](https://img.shields.io/badge/GUI-Tkinter-orange)
![License](https://img.shields.io/badge/License-MIT-lightgrey)

</div>

---

## Table of Contents

1. [What this is](#what-this-is)
2. [Quickstart](#quickstart)
3. [Everyday workflow](#everyday-workflow)
4. [Under the hood](#under-the-hood)
5. [Repo map](#repo-map)
6. [Data files it creates](#data-files-it-creates)
7. [Troubleshooting](#troubleshooting)
8. [Known limitations](#known-limitations)
9. [Roadmap ideas](#roadmap-ideas)
10. [Credits & license](#credits--license)

---

## What this is

Instead of a physical register, this app watches a webcam feed, recognizes registered faces, and logs "who was here and when" straight into a CSV. It's built around three ideas:

- **Enroll once** — capture ~100 face samples per person.
- **Train once** — turn those samples into a recognition model.
- **Recognize repeatedly** — every future session just checks faces against that model.

Everything runs locally in a single Tkinter window — no server, no cloud, no accounts beyond a local admin password.

## Quickstart

**You'll need:** Python 3, a working webcam, and a few seconds of patience for OpenCV's C++ build to install.

```bash
# 1. get the code
git clone <YOUR-GITHUB-REPOSITORY-URL>
cd <repo-folder>

# 2. install what it needs
pip install opencv-contrib-python numpy pillow pandas

# 3. launch it
python main.py
```

> ⚠️ Use `opencv-contrib-python`, not plain `opencv-python` — the recognizer (`cv2.face.LBPHFaceRecognizer_create`) only ships in the contrib build.

Not sure your webcam even works with OpenCV? Run the included sanity check first:

```bash
python camera_test.py
```

A window should pop up showing your live feed. Press `q` to close it.

## Everyday workflow

| Step | What you do | What happens behind the scenes |
|------|--------------|---------------------------------|
| **Register** | Type an ID + name, click **Take Images** | Webcam captures up to 100 grayscale face crops via a Haar cascade detector and saves them to `TrainingImage/` |
| **Train** | Click **Save Profile**, enter the admin password | All saved face images are fed to an LBPH recognizer, producing `TrainingImageLabel/Trainner.yml` |
| **Take attendance** | Click **Take Attendance** | Live feed is scanned frame-by-frame; recognized faces are matched to an ID and logged with date + time |
| **Review** | Look at the right-hand panel | Today's log populates a live table (ID / Name / Time) pulled from the day's CSV |

First time running the app? It'll ask you to set an admin password on the fly — that password gates both training and future changes.

## Under the hood

The recognition pipeline is intentionally simple and classic (no deep learning, no GPU needed):

```
Webcam frame
   → Haar Cascade face detection (haarcascade_frontalface_default.xml)
   → Grayscale crop of detected face
   → LBPH (Local Binary Patterns Histograms) feature comparison
   → Confidence score < threshold? → match found → log attendance
```

LBPH was chosen over embedding-based methods because it trains fast on a CPU with a handful of sample images per person — a reasonable trade-off for a small classroom or office roster rather than a large-scale deployment.

## Repo map

```
.
├── main.py                              # the whole app: GUI + registration + training + recognition
├── camera_test.py                       # standalone webcam sanity check
├── haarcascade_frontalface_default.xml  # pretrained OpenCV face detector
├── TrainingImage/                       # raw face captures, one set per enrolled person
├── TrainingImageLabel/
│   ├── Trainner.yml                     # the trained LBPH model
│   └── psd.txt                          # stored admin password (plain text — see Limitations)
├── StudentDetails/
│   └── StudentDetails.csv               # serial no. ↔ ID ↔ name lookup
├── Attendance/
│   └── Attendance_<DD-MM-YYYY>.csv      # one file per day, appended to as people are recognized
└── screenshots/                         # GUI captures
```

## Data files it creates

- **`StudentDetails/StudentDetails.csv`** — the roster: serial number, ID, and name for everyone enrolled.
- **`Attendance/Attendance_<date>.csv`** — a fresh file per day with columns `ID, Name, Date, Time`, one row per recognized visit.
- **`TrainingImageLabel/Trainner.yml`** — the serialized LBPH model; regenerated every time you re-train.

All three are plain CSV/YAML, so they're easy to inspect, back up, or import into a spreadsheet.

## Troubleshooting

<details>
<summary><strong>App closes immediately with a "file missing" popup</strong></summary>

`haarcascade_frontalface_default.xml` isn't sitting next to `main.py`. Make sure you cloned the full repo rather than downloading just the script.
</details>

<details>
<summary><strong>Webcam window never opens</strong></summary>

Run `camera_test.py` in isolation. If that also fails, another app is likely holding the camera, or `cv2.VideoCapture(0)` isn't the right device index on your machine — try `1` or `2`.
</details>

<details>
<summary><strong>Recognized as "Unknown" even though I registered</strong></summary>

The match confidence threshold (currently 50) may be too strict for your lighting or sample quality. Re-enroll with better, more varied lighting, or retrain after capturing more samples.
</details>

<details>
<summary><strong>`pip install` fails on `opencv-contrib-python`</strong></summary>

Make sure you're on a 64-bit Python build and that `pip` itself is up to date (`python -m pip install --upgrade pip`) before retrying.
</details>

## Known limitations

- The admin password is stored **in plain text** (`TrainingImageLabel/psd.txt`) — fine for a coursework project, not for production.
- LBPH recognition accuracy drops in poor or uneven lighting, and it doesn't scale gracefully to hundreds of enrolled faces.
- Only one webcam / one face-recognition session at a time.
- No database — everything lives in flat CSV files, so concurrent multi-user access isn't supported.

## Roadmap ideas

- [ ] Swap plain-text password storage for a hashed credential
- [ ] Move records into SQLite (or another lightweight DB) instead of CSV
- [ ] Add a simple analytics/reporting view (attendance % per person, per date range)
- [ ] Try a deep-learning face embedding model for better accuracy at scale
- [ ] Package as a standalone executable so it doesn't need a Python setup step

## Credits & license

Built as a computer vision coursework project (Vityarthi CV submission, roll no. `24BAI10090`).

Released under the **MIT License** — see [`LICENSE`](./LICENSE) for the full text.
