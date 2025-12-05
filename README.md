# 🚀 InterBLINK — Ongoing Development Log

*A high-performance CLI video upscaler powered by NVIDIA NIS & RTX Video Super Resolution.*

InterBLINK is in **active development**.
This README tracks the full plan, design decisions, developer tasks, runtime instructions, and release checklist — all updated to reflect our conversations.

---

# 📌 Project Overview

**InterBLINK** is a **C++ command-line application** that upscales video using GPU acceleration:

* **NVIDIA Image Scaling (NIS)** — fast, MIT-licensed, redistributable (bundled with InterBLINK).
* **NVIDIA RTX Video Super Resolution (VSR)** — advanced proprietary AI upscaling (uses NVIDIA driver/runtime; SDK not bundled).
* **FFmpeg** — decoding & encoding frames (either via `ffmpeg.exe` or libav*).

**Primary goal:** provide a fast, lightweight, terminal-based upscaler that supports **2× and 4×** scaling and works out-of-the-box for most users (NIS), while enabling RTX acceleration automatically when the user’s NVIDIA driver includes the RTX runtime.

---

# 🧭 Two Executables (User-facing)

InterBLINK ships as **two standalone C++ console tools** (no installers required):

1. `interblink.enginecheck` — checks RTX/CUDA runtime availability on the machine
   Example output:

   ```
   -> interblink.enginecheck
    ✔ nvngx_dlss.dll
    ✔ nvngx_vsr.dll (RTX VSR runtime)
    ✔ nvngx.dll (NV core)
    ✔ CUDA / Tensor support

   RTX upscaling enabled.
   ```

2. `interblink` — the main interactive CLI upscaler
   Interactive demo flow:

   ```text
   Enter Video Path: C:\Users\Downloads\video.mp4
   Choose Engine [nis/rtx]: rtx
   Choose Scale [2x/4x]: 4x
   Enter output file name: upscale.mp4
   Enter output location: C:\Users\Desktop\

   [##########--------] 55%  Processing...
   Done! Saved to C:\Users\Desktop\upscale.mp4
   ```

`interblink` also supports flags-mode:

```
interblink --input "C:\video.mp4" --engine rtx --scale 4 --output "C:\out\upscale.mp4"
```

---

# 🔧 Full Plan — What We Will Build (step-by-step)

## Architecture (high level)

* `cli/` — CLI input parsing (interactive + flags)
* `ffmpeg/` — FFmpeg wrapper (decode → frames, encode → MP4)
* `engines/`

  * `nis/` — NIS engine (bundled shaders / integration)
  * `rtx/` — RTX engine wrapper (dynamic runtime detection; no SDK files in repo)
  * `common/` — GPUInfo, shared helpers
* `utils/` — Logger (spdlog), ProgressBar, FileUtils
* `bin/` — built executables: `interblink.exe`, `interblink.enginecheck.exe`
* `docs/`, `tests/`, `assets/` — supporting files

## Milestones

1. Project scaffolding, CMake & VS solution, README (this file).
2. CLI core: interactive mode + flags mode + validation.
3. FFmpeg wrapper: decode frames, re-encode to MP4/H.264.
4. NIS integration: implement 2× and 4×, include shaders/assets.
5. GPU detection & `interblink.enginecheck` (scan runtime + dynamic load).
6. RTX wrapper: attempt to dynamically load driver runtime (enable if available) — *no SDK files included*.
7. Progress UI (terminal bar + ETA), performance metrics.
8. Testing, packaging, release (ZIP with `InterBlink.exe`, README, runtime_instructions`).
9. Optional: CI, regression tests, future GUI wrapper (separate project).

---

# 🧩 Runtime behaviour & UX decisions

## Default UX (for normal users)

* **NIS first**: InterBLINK bundles NIS shaders so the normal user can download one ZIP and run `interblink.exe` immediately.
* **RTX automatic**: If the user’s machine has the RTX runtime (installed as part of NVIDIA drivers), `interblink` will detect and enable RTX without further downloads. That makes it “install-and-go” in practice for many users (same approach used by big apps).

## RTX vs NIS — exact policy

* If user picks `--engine rtx` and RTX runtime is present → use RTX mode.
* If user picks `--engine rtx` and RTX runtime is **not** present → print a clear message and **automatically fallback to NIS**, unless user forces an exit.
* If user picks `--engine nis` → always use NIS.

## Why this is legal & safe

* RTX runtime is supplied by NVIDIA drivers installed on the user’s machine (not by your repo). You **must not** include or redistribute proprietary SDK DLLs or model files. InterBLINK must only dynamically load existing system DLLs at runtime.

---

# ✅ interblink.enginecheck — What it does

* Scans common system locations for NVIDIA runtime DLLs (e.g. `C:\Windows\System32\`), such as:

  * `nvngx.dll`
  * `nvngx_vsr.dll` (or similar RTX VSR runtime DLL)
  * `nvngx_dlss.dll` (if present)
* Optionally checks basic CUDA driver availability (via a small runtime query or checking `nvcuda.dll` presence).
* Reports a simple pass/fail summary and returns non-zero if RTX is not fully available.

**Output must be user-friendly and copy-pasteable to support guides / help desks.**

---

# ⚙️ Detection & Fallback Implementation Notes (developer)

* Use **filesystem checks** for DLL presence (System32 + Program Files paths).
* Use **LoadLibrary/GetProcAddress** to attempt dynamic linking; do not fail build if DLLs missing.
* Provide `GPUInfo` helper class with:

  * `bool hasNvidiaDriverRuntime()` — existence of `nvngx` or `nvngx_vsr`.
  * `bool hasCUDA()` — check `nvcuda.dll` or query CUDA driver version (optional).
* Fallback logic:

  ```text
  if (engine == "rtx" && !hasNvidiaDriverRuntime()) {
      print("RTX unavailable. Falling back to NIS.");
      engine = "nis";
  }
  ```

---

# 🗂 Recommended Repo Layout (final)

```
InterBLINK/
├── .gitignore
├── LICENSE.txt
├── README.md
├── CMakeLists.txt
├── Project_InterBLINK.sln
├── src/
│   ├── main.cpp                # launches interblink or interblink.enginecheck depending build target
│   ├── enginecheck.cpp
│   ├── cli/
│   ├── ffmpeg/
│   ├── engines/
│   │   ├── nis/
│   │   └── rtx/
│   ├── utils/
│   └── version/
├── assets/
│   ├── shaders/                # NIS shaders (bundled)
│   └── samples/
├── deps/
│   └── ffmpeg/                 # optional redistributable ffmpeg binary (check license)
├── docs/
│   ├── runtime_requirements.md
│   └── INSTALL.md
├── scripts/
│   └── build_windows.bat
├── tests/
└── bin/                        # produced executables after build
```

---

# 🛠 Build & Development Requirements

* **Windows 10/11** (primary target)
* **Visual Studio 2022** (Desktop C++ workload) OR MSVC toolchain
* **CMake 3.26+**
* **FFmpeg**: either provide `ffmpeg.exe` on PATH or link against libav* (libavformat/libavcodec) — build-tested option
* **Optional**: CUDA Toolkit & NVIDIA drivers (for local RTX testing)
* **Recommended libs**:

  * `spdlog` (logging)
  * `cxxopts` (CLI parsing)
  * `fmt` (formatting)

---

# 📦 Packaging & Release (what to ship)

**InterBlink-v1.0-alpha.zip** should include:

* `interblink.exe` (main)
* `interblink.enginecheck.exe`
* bundled NIS shaders / assets
* optional `ffmpeg.exe` (if you choose to bundle; check FFmpeg redistributable terms)
* `README.md`, `runtime_instructions.txt`, `LICENSE.txt`

**Do NOT include**:

* RTX SDK files, proprietary NVIDIA DLLs, model files, or any other proprietary redistributables.

Include clear `runtime_instructions.txt` telling users:

* Update NVIDIA drivers (link to official site)
* Use `interblink.enginecheck` to verify RTX
* Example commands

---

# 🔬 Testing Plan (detailed)

## Functional tests

* 360p → 720p (2×)
* 720p → 1440p (2×)
* 1080p → 4K (4×)

## Edge-case tests

* Missing input file
* Corrupted / partial video files
* Unsupported codecs (provide clear failure and suggestion)
* Low memory / VRAM situations (warn and abort gracefully)
* No NVIDIA GPU, or driver too old

## Performance tests

* Measure FPS/time for NIS vs RTX on available GPUs
* VRAM & CPU usage logs under `--stats` mode

## Automated tests

* Small regression test scripts in `tests/` that run small clip conversions and compare expected output (structure/checksum/metadata).

---

# 🎯 Goals for v1.0-alpha

* Working CLI (interactive + flags)
* NIS upscaling: 2× and 4×
* FFmpeg input/output pipeline (decode → process → encode)
* Terminal progress bar & basic ETA
* `interblink.enginecheck` runtime detector
* Basic GPU detection & automatic fallback to NIS
* Packaged Windows build ZIP + runtime instructions

---

# 🚧 Next Tasks — what I’ll help you with (choose any)

1. **Finish CLI argument mode** (flags parsing & interactive)
2. **Implement FFmpeg wrapper skeleton** (decode/encode loop)
3. **Create NIS engine skeleton** (apply scaling shader / CPU fallback)
4. **Implement `interblink.enginecheck` C++ code** (DLL scan + dynamic load)
5. **Add progress bar & `--stats`** (spdlog + ProgressBar)
6. **Prepare release packaging script** and `runtime_instructions.txt`

Tell me which to generate now and I’ll produce the exact source files.

---

# ⚠️ NVIDIA Legal Notice (must appear in README & releases)

This project **uses NVIDIA technologies**, but does **not** include or redistribute any proprietary NVIDIA SDK files, model files, or runtime DLLs.
If you want to use RTX VSR features, please ensure your system has the latest NVIDIA drivers installed. The RTX runtime is installed by official NVIDIA drivers — InterBLINK will detect this runtime at startup and enable RTX upscaling only if the runtime is present.

---

# ✍️ Example quickstart (for `README.md` users)

1. Download `InterBlink-v1.0-alpha.zip` and extract.
2. (Optional) Put `ffmpeg.exe` on PATH or keep it in the same folder.
3. Run runtime check:

   ```
   interblink.enginecheck
   ```
4. Upscale a video:

   ```
   interblink --input "C:\Users\Downloads\video.mp4" --engine rtx --scale 4 --output "C:\Users\Desktop\upscale.mp4"
   ```
5. If RTX not available you will see a message and processing will continue with NIS.

---

# 🙋 Final notes & next-step suggestion

You have a solid, legally-safe plan that gives **normal users** an “install-and-go” experience (via bundled NIS and automatic driver-based RTX detection), while remaining safe for GitHub distribution (no proprietary files).

Tell me which file you want me to generate next (I can produce ready-to-build C++ templates):

* `interblink.enginecheck` C++ source (full, tested skeleton)
* `interblink` main CLI skeleton + flags parsing (cxxopts example)
* FFmpeg decode → process → encode skeleton (frame loop)
* NIS engine wrapper (shader or CPU fallback skeleton)
* CMakeLists.txt for Visual Studio + release ZIP script
