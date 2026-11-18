# PathSnap – Intelligent GPS Route Reconstruction

**PathSnap** is an intelligent GPS map-matching and route-reconstruction platform. It reconstructs the most likely continuous road route travelled by a vehicle using noisy, inaccurate, or sparse GPS observations.

---

## Key Features

- **Robust GPS Preprocessing:** 2D constant-velocity kinematic Kalman filtering, speed anomaly detection ($> 150\text{ km/h}$), and duplicate coordinate handling.
- **Efficient Road Network Modeling:** Directed graph representation with Haversine segment lengths, compass bearing calculations, and Dijkstra shortest path routing.
- **Fast Spatial Candidate Indexing:** Uniform spatial grid partitioning enabling sub-millisecond candidate arc lookups around any GPS coordinate.
- **5 Trained Machine Learning Models:**
  1. *Logistic Regression* (Linear probabilistic baseline)
  2. *Random Forest Classifier* (Bagging ensemble)
  3. *Histogram Gradient Boosting* (Boosted trees)
  4. *Multi-Layer Perceptron* (Deep feedforward neural network)
  5. *Support Vector Machine* (Maximal margin with Platt probability scaling)
- **HMM & Viterbi Sequence Optimization:** Dynamic programming combining ML candidate match probabilities with spatial Gaussian priors and topological network transition penalties.
- **Continuous Route Bridging:** Shortest-path graph interpolation resolving network discontinuities for 100% continuous road paths.
- **Full-Stack Web Interface:** Single-page application with interactive Leaflet/OpenStreetMap mapping, dual layer visualization (raw GPS vs. reconstructed route), model comparison charts, and dataset upload.
- **High-Performance REST API:** Endpoints for trajectory listing, file upload, validation, inference, and benchmarking.

---

## Benchmark Results

### 1. ML Model Comparison (Held-out Validation Split: 16 Trajectories, 36,901 Samples)

| Model Name | Accuracy | Precision | Recall | F1-Score | ROC-AUC | Inference Time |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Logistic Regression** | $76.14\%$ | $63.70\%$ | $76.69\%$ | $0.6960$ | $0.8274$ | $1.4\text{ ms}$ |
| **Random Forest** | $77.86\%$ | $67.47\%$ | $73.05\%$ | $0.7015$ | $0.8348$ | $4.2\text{ ms}$ |
| **HistGradientBoosting** | $77.69\%$ | $66.61\%$ | $74.87\%$ | $0.7050$ | $0.8388$ | $2.1\text{ ms}$ |
| **Multi-Layer Perceptron (MLP)** | **81.08%** | **77.87%** | $65.47\%$ | **0.7114** | **0.8500** | $2.8\text{ ms}$ |
| **Support Vector Machine (SVM)** | $78.60\%$ | $71.11\%$ | $67.20\%$ | $0.6910$ | $0.8259$ | $1.8\text{ ms}$ |

### 2. Sequence Reconstruction Improvement (ML-Only vs. HMM/Viterbi)

| Metric | ML-Classifier Only | HMM + Viterbi Reconstructed | Improvement |
| :--- | :---: | :---: | :---: |
| **Route Recall** | $73.52\%$ | **97.52%** | **+24.00%** |
| **Route F1-Score** | $0.7122$ | **0.8591** | **+0.1469** |
| **Jaccard Similarity** | $55.30\%$ | **75.29%** | **+19.99%** |
| **Route Continuity** | Broken / Disjoint | **100% Continuous Path** | Gaps Bridged via Dijkstra |

---

## Quickstart

### 1. Run Automated Tests
```bash
python -m pytest -v tests
```
*Result: 19 passed.*

### 2. Launch Local Web Application
```bash
python backend/run_server.py
```
Open your browser at **`http://127.0.0.1:8000`**.

---

## Documentation Index

- [PROJECT_ARCHITECTURE.md](file:///c:/Users/bhmyt/PathSnap/PROJECT_ARCHITECTURE.md): System design, dataflow, and component interactions.
- [DATASET_ANALYSIS.md](file:///c:/Users/bhmyt/PathSnap/DATASET_ANALYSIS.md): Comprehensive inventory and audit of 90 trajectories.
- [DATA_CLEANING_REPORT.md](file:///c:/Users/bhmyt/PathSnap/DATA_CLEANING_REPORT.md): Quality audit, outlier handling, and Kalman smoothing.
- [MODEL_DOCUMENTATION.md](file:///c:/Users/bhmyt/PathSnap/MODEL_DOCUMENTATION.md): Formulations, features, and benchmarks of the 5 ML models and HMM.
- [API_DOCUMENTATION.md](file:///c:/Users/bhmyt/PathSnap/API_DOCUMENTATION.md): Complete REST API specifications.
- [SETUP_GUIDE.md](file:///c:/Users/bhmyt/PathSnap/SETUP_GUIDE.md): Step-by-step local execution and verification instructions.
