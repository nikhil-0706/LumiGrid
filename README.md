# 🚗 Adaptive LiDAR Mapping (SIH26053)

> **Adaptive Variable-Resolution 2.5D LiDAR Mapping & Real-Time Semantic Perception System for Autonomous Driving**

[![Python 3.10+](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-009688.svg)](https://fastapi.tiangolo.com/)
[![Next.js 16](https://img.shields.io/badge/Next.js-16.3-black.svg)](https://nextjs.org/)
[![React 19](https://img.shields.io/badge/React-19.0-61DAFB.svg)](https://react.dev/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-EE4C2C.svg)](https://pytorch.org/)
[![Deck.gl](https://img.shields.io/badge/Deck.gl-9.3-green.svg)](https://deck.gl/)
[![SemanticKITTI](https://img.shields.io/badge/Dataset-SemanticKITTI-orange.svg)](http://www.semantic-kitti.org/)

---

## 📌 Executive Overview

**Adaptive LiDAR Mapping** is a high-performance end-to-end perception and spatial mapping framework designed for autonomous vehicles navigating complex and unstructured environments. 

The system implements **foveated (variable-resolution) 2.5D spatial grid partitioning** around the ego-vehicle, dramatically reducing memory bandwidth and computational footprint while retaining high spatial fidelity in critical near-field regions. Powered by state-of-the-art 3D point cloud segmentation deep learning models (**Cylinder3D**, **MinkUNet**, **PointNet++**), the platform provides real-time semantic understanding, elevation mapping, obstacle detection, and distance-aware accuracy evaluation.

---

## ✨ Key Features

### 🎯 1. Foveated 2.5D Spatial Grid Mapping
Organizes LiDAR point clouds into concentric spatial resolution rings relative to vehicle distance, achieving massive memory savings and high processing speeds:
* **0 – 10 m (Near Field):** **5 cm** cell size — ultra-fine detail for precise obstacle detection & path planning.
* **10 – 30 m (Mid Range):** **10 cm** cell size — balanced resolution for drivable area identification.
* **30 – 60 m (Far Range):** **25 cm** cell size — coarse grid for surrounding traffic and structural mapping.
* **60 – 100 m (Extended Range):** **50 cm** cell size — global contextual layout.

### 🧠 2. Multi-Model 3D Perception Suite
* **Cylinder3D:** Cylindrical voxel partition + 3D sparse convolution for scalable 3D point cloud segmentation.
* **MinkUNet / SparseUNet:** Sparse tensor 3D UNet architecture optimized for high-density point clouds.
* **PointNet++:** Hierarchical feature extractor baseline for 3D point cloud point-wise classification.
* **Precomputed Engine:** High-speed offline prediction reader for rapid playback and benchmark comparisons.

### 📊 3. Distance-Dependent Evaluation Framework
* Dynamic mIoU and per-class precision metrics calculated across distance bands (`0–10m`, `10–30m`, `30–60m`, `60–100m`).
* Real-time point density, classification confidence, and accuracy decay profiling over distance.

### 🏔️ 4. Real-Time 2.5D Elevation & Terrain Surface Profiling
* Computes min/max elevation, height variance, ground vs non-ground surface classification, and ground slope per grid cell.
* Interactive 3D surface mesh and cross-sectional profile slice visualization.

### ⚡ 5. Binary Streaming Protocol & Interactive Web Dashboard
* Custom binary frame serialization over WebSockets (`/ws/stream`) delivering 60 FPS visualization with minimal network latency.
* **Next.js 16 Web Application:** Featuring Deck.gl 3D point cloud renderer, Three.js 2.5D elevation viewer, interactive top-down 2D semantic grid, dynamic accuracy analytics with ECharts, and real-time performance HUDs.

---

## 🏗️ System Architecture

```mermaid
flowchart TD
    subgraph Data Layer
        A[SemanticKITTI Velodyne Scans .bin] --> D[Data Loader]
        B[SemanticKITTI Labels .label] --> D
        C[Ego Poses poses.txt] --> D
    end

    subgraph Backend Engine (FastAPI + PyTorch)
        D --> E[Frame Processor]
        E --> F[3D Model Inference Engine\nCylinder3D / MinkUNet / PointNet++]
        F --> G[Foveated 2.5D Grid Engine\n0-10m: 5cm | 10-30m: 10cm | 30-60m: 25cm | 60-100m: 50cm]
        G --> H[Elevation & Terrain Mapper]
        G --> I[Distance Evaluator\nmIoU / Accuracy per Ring]
        G --> J[DBSCAN Object Clustering]
    end

    subgraph Binary Stream Protocol
        H & I & J --> K[Binary Serialization Engine]
        K -->|WebSocket Streaming /ws/stream| L[React / Next.js Dashboard]
    end

    subgraph Web Frontend (Next.js 16 + Deck.gl + Three.js)
        L --> M[3D LiDAR Point Cloud Scene]
        L --> N[2D Foveated Semantic Grid Map]
        L --> O[3D Terrain & Elevation Slice View]
        L --> P[Per-Distance Accuracy & Metrics HUD]
    end
```

---

## 📁 Repository Structure

```
Adaptive_LiDAR_Mapping/
├── backend/                        # ML & Backend Processing Engine
│   ├── config/                     # Dataset yaml & model configurations
│   ├── config.py                   # Master configuration dataclasses
│   ├── core/                       # Core algorithms
│   │   ├── adaptive_grid.py        # Foveated spatial grid implementation
│   │   ├── data_loader.py          # SemanticKITTI loader & sequence iterators
│   │   ├── distance_evaluator.py   # Distance-band mIoU & accuracy evaluator
│   │   ├── elevation.py            # 2.5D terrain & elevation map computation
│   │   └── frame_processor.py      # Master frame processing orchestrator
│   ├── models/                     # Deep learning model wrappers
│   │   ├── base.py                 # Abstract base segmentation interface
│   │   ├── cylinder3d_wrapper.py   # Cylinder3D network integration
│   │   ├── minkuNet_wrapper.py     # MinkUNet sparse conv model wrapper
│   │   ├── pointnet2_wrapper.py    # PointNet++ model implementation
│   │   └── precomputed_wrapper.py  # Offline predictions loader
│   ├── serialization/              # Binary & JSON serialization protocol
│   │   └── binary_frame.py         # Compact WebSocket frame pack/unpack
│   ├── scripts/                    # Utility, benchmark & training scripts
│   │   ├── benchmark.py            # Latency, memory & throughput benchmarker
│   │   ├── evaluate_models.py      # Model comparison script
│   │   └── train_pointnet2.py      # PointNet++ training script
│   ├── utils/                      # Class definitions & color maps
│   │   └── class_mapping.py        # SemanticKITTI 20-class ontology
│   └── server.py                   # FastAPI REST API & WebSocket server
│
├── Frontend/                       # Web Dashboard (Next.js + TypeScript)
│   ├── app/                        # Next.js App Router pages & layouts
│   │   ├── components/             # Visualization UI components
│   │   │   ├── AccuracyChart.tsx   # Per-distance accuracy line/bar chart
│   │   │   ├── ElevationMap3D.tsx  # 3D terrain mesh viewer
│   │   │   ├── ElevationSlice.tsx  # Cross-sectional terrain slice viewer
│   │   │   ├── LidarScene.tsx      # Deck.gl 3D Point Cloud renderer
│   │   │   ├── LidarViewer.tsx     # Main dashboard controller & controls
│   │   │   ├── MemorySavingsHUD.tsx# Memory compression statistics panel
│   │   │   ├── MetricsHUD.tsx      # Real-time FPS & processing latency panel
│   │   │   └── SemanticMap2D.tsx   # Top-down 2D foveated grid map
│   │   └── lidar-viewer/           # Main application view route
│   └── package.json                # Frontend dependencies & scripts
│
├── patch_frame_processor.py       # Helper patch script for pose/object tracking
├── patch_lidarviewer.py            # UI component patch script
├── patch_lidarviewer_sequences.py  # Sequence selector patch script
├── requirements.txt                # Master Python dependencies
└── README.md                       # Project Documentation
```

---

## 🚀 Getting Started

### Prerequisites
* **Python**: `3.10` or higher
* **Node.js**: `v18.0.0` or higher (`npm` / `pnpm` / `bun`)
* **PyTorch**: `2.0+` (CUDA highly recommended for real-time inference)

---

### 1️⃣ Backend Setup

1. **Create and activate a virtual environment:**
   ```bash
   python -m venv .venv
   # Windows (PowerShell):
   .venv\Scripts\Activate.ps1
   # Linux / macOS:
   source .venv/bin/activate
   ```

2. **Install Python dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

3. **Start the FastAPI Backend Server:**
   ```bash
   python -m backend.server
   ```
   *The backend server will run on `http://localhost:8000`.*
   *API documentation is available at `http://localhost:8000/docs`.*

---

### 2️⃣ Frontend Setup

1. **Navigate to the `Frontend` directory:**
   ```bash
   cd Frontend
   ```

2. **Install Node.js dependencies:**
   ```bash
   npm install
   ```

3. **Run the Next.js development server:**
   ```bash
   npm run dev
   ```

4. **Access the Dashboard:**
   Open [http://localhost:3000](http://localhost:3000) in your web browser.

---

## 📡 API & WebSocket Specification

### REST API Endpoints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/api/status` | Server status, active model, loaded models, device, and resolution configuration. |
| `GET` | `/api/models` | List all available segmentation models and checkpoint status. |
| `POST` | `/api/model/select?name=<model_name>` | Dynamically switch active inference model (`cylinder3d`, `pointnet2`, `minkuNet`, `precomputed`). |
| `GET` | `/api/classes` | Fetch SemanticKITTI class IDs, names, and RGB/HEX color maps. |
| `GET` | `/api/sequences` | List available dataset sequences and frame counts. |
| `GET` | `/api/frame/{sequence}/{frame_id}` | Fetch processed single frame results in JSON or binary format. |

### WebSocket Protocol (`ws://localhost:8000/ws/stream`)

Send JSON control payloads to manipulate the live frame stream:
```json
{ "action": "start", "sequence": "00", "model": "cylinder3d", "fps": 10 }
{ "action": "pause" }
{ "action": "resume" }
{ "action": "seek", "frame": 120 }
{ "action": "set_model", "model": "pointnet2" }
{ "action": "stop" }
```

---

## 🔬 Benchmarks & Performance Evaluation

Run the automated benchmarking script to measure frame latency, compression ratios, and distance-band accuracies:

```bash
python backend/scripts/benchmark.py --sequence 00 --num-frames 50
```

---

## 🛠️ Technology Stack

* **Machine Learning & Core:** Python, PyTorch, NumPy, SciPy, scikit-learn, DBSCAN.
* **Backend Services:** FastAPI, Uvicorn, WebSockets.
* **Frontend Framework:** Next.js 16 (React 19), TypeScript.
* **Visualization & Rendering:** Deck.gl, Three.js / React Three Fiber, MapLibre GL, ECharts.
* **Styling & Icons:** Tailwind CSS, Lucide React.
* **Dataset:** SemanticKITTI (KITTI Vision Benchmark Suite).

---

## 📜 License

Distributed under the MIT License. See `LICENSE` for more details.
