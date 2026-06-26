# 🐭 Mouse Video Processor

A Jupyter Notebook pipeline for trimming, cropping, and merging behavioral videos from mouse arena recordings. Designed for videos with multiple arenas captured from side and bottom cameras.

**What it does:**
1. Loads a folder of videos
2. Lets you draw custom crop regions (rectangles or trapezoids) directly on a video frame
3. Trims each crop to a specified start time and duration
4. Merges the side and bottom camera views side-by-side into a final output video

GPU-accelerated when an NVIDIA GPU is available; automatically falls back to CPU with a warning.

---

## Requirements

Before opening the notebook, you need the following installed on your computer:

### 1. Python (version 3.9 or higher)
Download from [python.org](https://www.python.org/downloads/). During installation on Windows, **check the box that says "Add Python to PATH"** — this is easy to miss and will cause errors later if skipped.

### 2. ffmpeg
ffmpeg is the video processing engine the notebook uses under the hood.

**Windows:**
- Download from [ffmpeg.org/download.html](https://ffmpeg.org/download.html) (choose the "Windows builds" option from gyan.dev or BtbN)
- Unzip the folder, find the `bin` folder inside it, and copy the full path (e.g. `C:\ffmpeg\bin`)
- Search for "Edit the system environment variables" in the Windows Start menu → click "Environment Variables" → find "Path" under System variables → click Edit → click New → paste the path → click OK on everything
- Open a new Command Prompt and type `ffmpeg -version` — if you see version info, it worked ✅

**Mac:**
```bash
brew install ffmpeg
```

**Linux / UCSD Server:**
```bash
sudo apt install ffmpeg
```

### 3. Jupyter Notebook or JupyterLab
If you do not already have Jupyter installed, open a terminal (Mac/Linux) or Command Prompt (Windows) and run:
```bash
pip install jupyterlab
```

### 4. Python packages
The notebook will attempt to install most packages automatically when you run the first cell. However, one package (`ipympl`) needs to be installed **before** opening the notebook, otherwise the interactive drawing canvas will not work:

```bash
pip install ipympl
```

---

## How to Run

### Step 1: Download the notebook
- On this GitHub page, click on `MouseVideoProcessor.ipynb`
- Click the **Download raw file** button (the down-arrow icon in the top right of the file view)
- Save it somewhere easy to find, like your Desktop or a project folder

### Step 2: Open Jupyter
Open a terminal or Command Prompt and type:
```bash
jupyter lab
```
This will open a browser tab with the Jupyter interface. Navigate to where you saved the notebook and click on it to open it.

### Step 3: Run the notebook cell by cell
**Important: you must run cells in order, top to bottom.**

Each cell has a **▶ Run** button (or press `Shift + Enter` to run and move to the next cell).

**How to know a cell is done:**  
Look at the bracket to the left of the cell:
- `[ ]` = not yet run
- `[*]` = currently running — **wait for this to finish before clicking anything else**
- `[1]` (or any number) = finished ✅

> ⚠️ **Do not click buttons inside the notebook or run the next cell while you see `[*]`.** Wait for the number to appear first.

---

## Walkthrough

### Part 1: Setup & Configuration
- Run each cell in order
- When you reach the folder input, type the full path to your video folder and click **📂 Scan Folder**
- You will see a list of all video files found
- An output folder called `processed` will be created automatically inside your video folder (or you can specify a different location)

### Part 2: Draw Crop Regions
- Run the cell — it will grab a frame from ~10 minutes into the first video (chosen late to avoid camera setup disturbances)
- Click **➕ Add Crop Region**, then click on the image to place the corners of your crop area
  - Click as many corners as you need (works for rectangles, tilted boxes, trapezoids, or any polygon shape)
  - Press **Enter** or click back on the first point to close the shape
- A form will appear below the image — type the **MousePair ID** (e.g. `M1_2`, `PairA`, `mouse03`) and select whether this is the **side** or **bottom** camera view
- Click **💾 Save This Crop**, then repeat for each arena (up to 8 per video)
- When all arenas are labeled, click **✅ Confirm & Next Video**
- **Re-run the cell** to load the next video and repeat

> 💡 If you draw a shape wrong, click **↩ Remove Last** to undo it and try again.

### Part 3: Set Start/Stop Times & Process
- A row will appear for every crop you defined — enter the **start time** and **duration** for each in `MM:SS` or `HH:MM:SS` format
- Run the estimation cell to see how long processing will take before committing
- Click **🚀 Start Processing** — do not close the notebook or let your computer sleep during this step
- Output files will be named `{MousePairID}_side_trimmed.mp4` and `{MousePairID}_bottom_trimmed.mp4`
- A session config `.txt` file is automatically saved to your output folder with all crop coordinates and timing for your records

### Part 4: Merge Side-by-Side
- Click **🎬 Merge All Pairs**
- For each MousePair ID, the side and bottom trimmed videos are placed side-by-side into a single file named `{MousePairID}_merged.mp4`
- If one camera view is missing for a pair, the notebook will skip that pair and tell you which one is missing rather than crashing

---

## Output File Naming

| File | Description |
|------|-------------|
| `{PairID}_side_trimmed.mp4` | Cropped and trimmed side-camera video |
| `{PairID}_bottom_trimmed.mp4` | Cropped and trimmed bottom-camera video |
| `{PairID}_merged.mp4` | Final side-by-side merged video |
| `session_config_YYYYMMDD_HHMMSS.txt` | Record of all crop regions, timings, and settings used |

---

## GPU vs. CPU

The notebook automatically detects whether an NVIDIA GPU is available:
- **GPU detected** ✅ — uses `h264_nvenc` for fast hardware-accelerated encoding
- **No GPU** ⚠️ — falls back to `libx264` (CPU encoding); a warning is printed and processing will take longer

On the UCSD computing cluster, GPU availability depends on your job allocation. CPU fallback will still produce correct output, just more slowly.

---

## Troubleshooting

**The drawing canvas is blank or not interactive**
Run `pip install ipympl` in your terminal, restart the notebook kernel (Kernel → Restart Kernel), and re-run all cells from the top.

**`ffmpeg not found` error**
ffmpeg is not on your PATH. See the installation instructions above. On Windows, make sure you opened a *new* Command Prompt after editing the environment variables.

**`[*]` is stuck for a very long time**
Large video files take time — check the time estimate printed before processing. If it appears truly frozen (no output after much longer than expected), you can interrupt with Kernel → Interrupt Kernel, check the error message, and try again.

**A crop region looks off in the output video**
The frame used for drawing is from ~10 minutes in. If the camera shifted after that point, re-draw with a frame from a different time (this option can be added on request).

**Merge step says a camera view is missing**
Make sure both `_side` and `_bottom` crops were defined and processed for that MousePair ID. Check that the IDs were entered identically (they are case-sensitive).

---

## Contact

For questions or issues, contact the Smith Lab at UC San Diego.  
Notebook developed by Krysten O'Hara (k1ohara@ucsd.edu).
