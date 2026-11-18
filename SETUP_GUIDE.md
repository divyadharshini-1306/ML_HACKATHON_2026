# PathSnap – Local Setup & Execution Guide

This guide describes how to install dependencies, run tests, train models, and start the PathSnap application locally.

---

## 1. Prerequisites

- **Operating System:** Windows, Linux, or macOS
- **Python:** 3.10+ (tested with Python 3.13)
- **Node.js (Optional):** v18+ (tested with Node v22)

---

## 2. Environment Setup

### A. Clone & Workspace Directory
```bash
cd c:\Users\bhmyt\PathSnap
```

### B. Python Virtual Environment (Recommended)
```bash
python -m venv venv
# Windows:
.\venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate
```

### C. Install Python Dependencies
```bash
pip install -r backend/requirements.txt
```
*Installed core libraries:* `Flask`, `scikit-learn`, `numpy`, `scipy`, `pandas`, `joblib`, `pytest`, `starlette`, `uvicorn`.

---

## 3. Running Automated Tests

Run the full automated test suite covering parsers, cleaning, road network topology, ML models, HMM decoding, and REST APIs:

```bash
python -m pytest -v tests
```

Expected output:
```
============================= 19 passed in 6.84s =============================
```

---

## 4. Training the Machine Learning Models

To re-train all 5 models (Logistic Regression, Random Forest, HistGradientBoosting, MLP, and SVM) on the training dataset and evaluate on the validation split:

```bash
python -m backend.src.ml.train
```

Trained models and evaluation metrics will be serialized into `models/saved/`:
- `logistic_regression.joblib`
- `random_forest.joblib`
- `gradient_boosting.joblib`
- `mlp.joblib`
- `svm.joblib`
- `metrics_comparison.json`

---

## 5. Starting the Local Application

Launch the PathSnap full-stack server:

```bash
python backend/run_server.py
```

Output:
```
=================================================================
  PathSnap – Intelligent GPS Route Reconstruction Server
  Access Web UI & REST APIs at: http://127.0.0.1:8000/
=================================================================
```

Open your browser and navigate to:
**`http://127.0.0.1:8000`**

---

## 6. Using the Web Interface

1. **Dashboard Tab:** View overall project metrics, dataset statistics, architecture components, and model health.
2. **Route Reconstruction Tab:**
   - Select any of the **90 trajectories** (`train` or `student_test`).
   - Choose from the **5 trained ML models** or Pure HMM.
   - Adjust candidate search radius ($50 - 100\text{ m}$).
   - Click **Reconstruct Route** to visualize the raw GPS points (red dashed) vs. the continuous reconstructed road route (vibrant blue) on Leaflet OpenStreetMap.
   - Inspect route stats, distance, duration, gaps bridged, and precision/recall comparison against ground truth.
3. **Model Performance Tab:**
   - Inspect the offline validation benchmark table for all 5 models.
   - View interactive comparison charts of F1-Score, Accuracy, and the **+24% Recall gain** achieved by HMM sequence decoding.
4. **Upload Dataset Tab:**
   - Drag and drop or upload custom `.track`, `.nodes`, `.arcs`, and `.route` files.
   - Validate structural integrity in real time and reconstruct routes immediately.
