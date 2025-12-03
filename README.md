

# 🚀 InterBLINK — Ongoing Development Log

*A high-performance CLI video upscaler powered by NVIDIA NIS & RTX Video Super Resolution.*

InterBLINK is currently in **active development**.  
This README tracks ongoing progress, decisions, tasks, and upcoming milestones.

---

# 📌 Project Overview

**InterBLINK** is a C++ command-line application that upscales videos using:

- **NVIDIA Image Scaling (NIS)** — fast, MIT-licensed, redistributable.

- **NVIDIA RTX Video Super Resolution SDK** — optional proprietary AI upscaling (SDK not included).

- **FFmpeg** — for decoding/encoding video frames.

The goal is to provide a **fast**, **clean**, and **GPU-accelerated** upscaler with **2× and 4× enhancements** directly from the terminal.

---


🧪 2. **User Flow (Finalized CLI Interaction)**

```text
Enter Video Path: C:\Users\Downloads\video.mp4
Choose Engine [nis/rtx]: rtx
Choose Scale [2x/4x]: 4x
Enter output file name: upscale.mp4
Enter output location: C:\Users\Desktop\

[##########--------] 55%  Processing...
Done! Saved to C:\Users\Desktop\upscale.mp4

```


---

# 🔧 Development Roadmap (Ongoing)

### **1. Core CLI + Video Handling**

- Interactive mode (`Enter video path:` prompts)

- Flags mode (`--input`, `--engine`, `--scale`, `--output`)

- Check for valid paths, codecs, sizes

### **2. FFmpeg Wrapper**

Implement a clean abstraction for:

- Input decoding (frame-by-frame)

- Output encoding (MP4/H.264 by default)

- Error handling & logging

### **3. NIS Upscaler (First Working Engine)**

- Integrate NVIDIA Image Scaling shader pipeline

- Support **2×** and **4×**

- GPU fallback detection

- Benchmark FPS and stability

- Produce consistent output resolution

### **4. RTX VSR Engine (SDK Required)**

- Create an RTX stub with optional SDK linking

- Users must download SDK externally

- Auto-switch to NIS if SDK unavailable

### **5. Error Handling & User Experience**

- Detect missing GPU

- Detect unsupported input resolution

- Warnings for low VRAM

- Cleanly fail with explanations

### **6. UI / Terminal Improvements**

- Progress bar with ETA

- Colored logs via `spdlog`

- `--stats` mode (FPS, time, GPU usage)



# 🖥️ Development Requirements

- Windows 10/11

- Visual Studio 2022 (or MSVC Build Tools)

- CMake 3.26+

- FFmpeg (compiled libs or `ffmpeg.exe` in PATH)

- NVIDIA GPU + updated drivers

- (Optional) NVIDIA RTX Video SDK installed locally

---

# ⚠️ NVIDIA Legal Notice

This project **uses NVIDIA technologies**, but does **not** include:

- RTX VSR SDK files

- NVIDIA DLLs

- Model files or proprietary binaries

Users must download required SDKs from NVIDIA directly.

---

# 🧪 Testing Plan (Ongoing)

### Functional tests:

- 360p → 720p (2×)

- 720p → 1440p (2×)

- 1080p → 4K (4×)

### Edge-case tests:

- Missing input file

- Corrupt video

- Zero-size resolution

- No NVIDIA GPU

- RTX SDK missing (fallback to NIS)

### Performance:

- Measure FPS across different resolutions

- Compare NIS vs RTX VSR results

---

# 🎯 Goals for First Public Release (v1.0-alpha)

- Working CLI

- NIS 2× & 4× upscaling

- FFmpeg input/output pipeline

- Progress bar

- Basic GPU detection

- Clean README + setup guide

- Windows build executable

After alpha release → start RTX engine integration.

---

# 🚧 What’s Actively Being Worked on Right Now?

**Next tasks (in order):**

1. Finish CLI argument mode

2. Finalize FFmpeg wrapper (decode → GPU → encode loop)

3. Implement NIS upscaling pipeline

4. Begin internal testing with sample videos

5. Integrate progress UI

6. Prepare alpha release packaging

This list updates continuously as development progresses.

---

# 💬 Contributing (Future)

Right now InterBLINK is in **solo-developer mode**.  
Contribution guidelines will be added once v1.0-alpha is released.

---

# ⭐ Vision

InterBLINK aims to be:

- **Fast**

- **Clean**

- **Lightweight**

- **GPU-powered**

- **Easy to use**

- **Free and open source**

Your terminal should be able to upscale a video **in seconds**, not minutes.