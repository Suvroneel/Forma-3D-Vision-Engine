# Forma - 3D Vision Engine

> Convert any 2D photo into a fully reconstructed 3D mesh using monocular depth estimation and point cloud surface reconstruction

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0+-red.svg)](https://pytorch.org/)
[![Open3D](https://img.shields.io/badge/Open3D-0.17+-green.svg)](http://www.open3d.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## Demo

### Input Image
<!-- Replace with your input image -->
![Image](DATA/car.jpg)

### Depth Map + 3D Mesh Output
<!-- Replace with your output video or gif -->
![Output Demo](RESULT/car_output.gif)

---

## Overview

Forma is a computer vision pipeline that takes a standard 2D photograph and produces a dense, colored 3D mesh - no stereo camera, no LiDAR, no special hardware required.

It combines a transformer-based monocular depth estimation model (GLPN) with Open3D's Poisson surface reconstruction to convert a flat image into an interactive 3D object you can rotate, inspect, and export.

**Core Use Cases:**
- E-commerce product visualization (2D product photos → 3D previews)
- Jewellery and retail AR/VR content pipelines
- Robotics and autonomous system depth perception
- Medical imaging preprocessing
- Architectural and heritage site digitization

---

## How It Works

```
Input Image (JPG/PNG)
        │
        ▼
┌─────────────────────┐
│   Image Preprocessing│  → Resize to model-compatible dimensions (multiples of 32)
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  GLPN Depth Model   │  → Transformer-based monocular depth estimation
│  (vinvino02/glpn-nyu)│    Produces per-pixel depth values
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  RGBD Construction  │  → Fuses RGB image with depth map
│  + Camera Intrinsics│    Pinhole camera model (f=500, principal point = center)
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  Point Cloud (Open3D)│ → Projects RGBD pixels into 3D space
│  + Outlier Removal  │    Statistical outlier removal (nb=20, std=6.0)
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  Normal Estimation  │  → Surface normal estimation + orientation alignment
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  Poisson Surface    │  → Watertight mesh reconstruction (depth=10)
│  Reconstruction     │
└─────────────────────┘
        │
        ▼
┌─────────────────────┐
│  Mesh Export        │  → .obj / .ply output + interactive Open3D viewer
└─────────────────────┘
```

---

## Performance

| Metric | Value |
|--------|-------|
| Processing Latency | ~1.5 - 2 seconds per image (CPU) |
| Depth Estimation | Sub-second inference (GLPN transformer) |
| Outlier Removal | Statistical filter at nb=20, std_ratio=6.0 |
| Mesh Reconstruction | Poisson depth=10 (high fidelity) |
| Output Formats | .obj, .ply |
| Input Formats | JPG, PNG |

---

## Tech Stack

**Depth Estimation:**
- GLPN (Global-Local Path Networks) - `vinvino02/glpn-nyu`, transformer-based monocular depth model trained on NYU Depth V2
- Hugging Face Transformers - model loading and feature extraction
- PyTorch - inference backend

**3D Reconstruction:**
- Open3D - point cloud generation, outlier removal, normal estimation, Poisson surface reconstruction, interactive mesh visualization
- NumPy - depth map processing and array operations

**Visualization:**
- Matplotlib (TkAgg backend) - side-by-side 2D image and depth map visualization
- Open3D Visualizer - interactive 3D point cloud and mesh viewer

---

## Project Structure

```
Forma/
├── CODE/
│   └── main.py              # Full pipeline: depth estimation → mesh reconstruction
├── DATA/
│   └── toy.jpg              # Input image (replace with your own)
├── RESULT/
│   └── Toy3D.obj            # Exported 3D mesh output
├── requirements.txt
└── README.md
```

---

## Setup

### Prerequisites
- Python 3.10+
- Conda (recommended)

### Installation

```bash
# Create isolated environment
conda create -n forma python=3.10
conda activate forma

# Install dependencies
pip install -r requirements.txt
```

---

## Usage

### Run the Pipeline

```bash
cd CODE
python main.py
```

### Change Input Image

In `main.py`, update line 3:

```python
image = Image.open("../DATA/your_image.jpg")
```

### Output

The pipeline produces three interactive windows in sequence:

1. **Depth Visualization** - Original image alongside the plasma-colored depth map (5 second display)
2. **Point Cloud Viewer** - Raw 3D point cloud in Open3D interactive viewer
3. **3D Mesh Viewer** - Final Poisson-reconstructed mesh with back-face rendering

Exported mesh saved to `RESULT/Toy3D.obj`.

---

## Pipeline Stages in Detail

### Stage 1 - Image Preprocessing
Resizes input to GLPN-compatible dimensions (height capped at 480px, both dimensions forced to multiples of 32). Preserves aspect ratio.

### Stage 2 - Monocular Depth Estimation
GLPN (Global-Local Path Networks) predicts a dense per-pixel depth map from a single RGB image using a hierarchical transformer encoder. No stereo setup or special hardware required. Depth values scaled to millimeter range.

### Stage 3 - RGBD Image Construction
Combines RGB image and depth map into an Open3D RGBDImage. A synthetic pinhole camera (focal length 500px, principal point at image center) is used as the projection model.

### Stage 4 - Point Cloud Generation and Cleaning
Projects RGBD pixels into 3D space using camera intrinsics. Statistical outlier removal (20 neighbors, std ratio 6.0) eliminates noise from the raw cloud.

### Stage 5 - Normal Estimation
Estimates surface normals for each point and orients them consistently toward a reference direction - required for accurate surface reconstruction.

### Stage 6 - Poisson Surface Reconstruction
Converts the oriented point cloud into a watertight triangle mesh using Screened Poisson Reconstruction at depth 10. Mesh rotated 180° around X-axis for correct upright orientation.

### Stage 7 - Export
Mesh exported as `.obj` for compatibility with Blender, Unity, Unreal Engine, and other 3D tools.

---

## Sample Results

| Input | Depth Map | 3D Mesh |
|-------|-----------|---------|
| <!-- add image --> | <!-- add depth map --> | <!-- add mesh screenshot --> |

---

## Roadmap

- [ ] FastAPI REST endpoint (upload image → download .obj)
- [ ] GPU acceleration via CUDA (target <500ms latency)
- [ ] Swap GLPN for MiDaS or Depth Anything V2 for improved accuracy
- [ ] Multi-image batch processing
- [ ] Streamlit web UI for browser-based demo
- [ ] Docker containerization for cloud deployment
- [ ] .stl export for 3D printing workflows

---

## License

MIT License - See LICENSE file for details.

---

## Author

**Suvroneel Nathak**
Computer Vision | AI/ML Engineering

[GitHub](https://github.com/Suvroneel) | [LinkedIn](https://www.linkedin.com/in/suvroneel-nathak/)
