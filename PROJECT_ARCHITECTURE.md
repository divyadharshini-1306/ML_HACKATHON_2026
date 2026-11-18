# PathSnap – Project Architecture & System Design

## 1. System Overview

PathSnap is an intelligent GPS map-matching and sequence-based route reconstruction platform.
It reconstructs the most probable actual road network path traversed by a vehicle from noisy, sparse, or jump-prone GPS measurements.

The system combines:
1. **Spatial & Network Indexing:** Rapid candidate road segment retrieval using spatial bounding and projection.
2. **Supervised Machine Learning:** Probabilistic scoring of candidate road segments using geometric and kinematic features.
3. **Hidden Markov Model (HMM) & Viterbi Decoding:** Sequence optimization balancing emission likelihoods with topological road network transition constraints.
4. **Topological Route Decoder:** Continuous path completion bridging missing intermediate arcs via shortest path graph routing.
5. **Interactive Full-Stack Web Application:** FastAPI/Starlette REST backend and React/Leaflet interactive mapping UI.

---

## 2. End-to-End Pipeline Architecture

```
        Raw GPS Data (.track)
                  ↓
       [Data Preprocessing]
  (Cleaning, Outliers, Kinematics)
                  ↓
       [Road Network Processing]
       (.nodes + .arcs Graph)
                  ↓
     [Candidate Road Generation]
  (Spatial Search Radius R ~ 50m)
                  ↓
     [ML Feature Extraction]
  (Perp dist, Heading diff, Speed)
                  ↓
        [5 ML Classifiers]
  (Logistic Regression, RF, GBDT, MLP, SVM)
                  ↓
      [Emission Probabilities]
  (ML calibrated scores + Gaussian prior)
                  ↓
     [Transition Probabilities]
  (Topological network distance vs GPS displacement)
                  ↓
      [HMM + Viterbi Decoding]
  (Optimal state sequence across time)
                  ↓
     [Continuous Route Decoder]
  (Shortest-path gap filling for continuity)
                  ↓
  [Interactive UI & Performance Metrics]
  (Dual map overlay, statistics, confusion metrics)
```

---

## 3. Core Modules & Responsibilities

### A. Road Network (`backend/src/network/`)
- `graph.py`: Road network graph class wrapping directed arcs and vertices. Provides node lookup, arc geometry computation, segment length, and directed Dijkstra / A* shortest path calculation.
- `spatial_index.py`: Spatial indexing (KDTree / Grid) for sub-millisecond candidate arc queries around any GPS coordinate.

### B. Machine Learning Pipeline (`backend/src/ml/`)
- `features.py`: Computes 9 geometric and kinematic features for candidate arc-GPS pairs:
  1. `perp_dist`: Orthogonal distance to candidate segment (meters).
  2. `heading_diff`: Absolute angular difference between vehicle heading and arc direction ($[0^\circ, 180^\circ]$).
  3. `projection_ratio`: Relative projection position along segment ($[0, 1]$).
  4. `dist_to_start`: Haversine distance to source node $u$.
  5. `dist_to_end`: Haversine distance to target node $v$.
  6. `arc_length`: Segment length in meters.
  7. `spatial_rank`: Rank among candidates sorted by distance (0 = nearest).
  8. `instantaneous_speed`: Vehicle speed $v_t$ (m/s).
  9. `speed_consistency`: Ratio between trajectory speed and implied arc traversal.
- `models.py`: Implementations of the 5 machine learning models:
  1. Logistic Regression
  2. Random Forest
  3. HistGradientBoosting / GBDT
  4. Multi-Layer Perceptron (MLP)
  5. Support Vector Machine (SVM)
- `train.py`: Training script with cross-validation on 64 training trajectories, evaluating on 16 validation trajectories.
- `evaluate.py`: Generates precision, recall, F1, accuracy, ROC-AUC, and confusion matrices.

### C. Sequence Reconstruction & HMM (`backend/src/hmm/`)
- `candidate_generator.py`: Generates top-K candidate arcs within a spatial search radius (e.g. 50 meters).
- `emission.py`: Computes $P(z_t | c_{t,i})$ using ML prediction scores calibrated with perpendicular distance Gaussian kernel.
- `transition.py`: Computes $P(c_{t,j} | c_{t-1,i})$ using exponential decay of the difference between road network shortest path distance and great-circle GPS displacement:
  $$P(c_{t,j} | c_{t-1,i}) = \frac{1}{\beta} \exp\left(-\frac{|d_{network}(c_{t-1,i}, c_{t,j}) - d_{great\_circle}(z_{t-1}, z_t)|}{\beta}\right)$$
- `viterbi.py`: Log-space dynamic programming sequence decoder finding the maximum a posteriori (MAP) sequence of arcs.
- `route_decoder.py`: Compresses consecutive repeated arcs, bridges graph discontinuities using shortest-path interpolation, and outputs the final continuous road route.

### D. Backend Service (`backend/`)
- `app.py`: FastAPI / ASGI service.
- Endpoints:
  - `POST /api/upload`: Upload and validate `.track`, `.nodes`, `.arcs`, `.route` files.
  - `GET /api/trajectories`: List available trajectories and metadata.
  - `POST /api/reconstruct`: Run map-matching and route reconstruction (selecting ML model or pure HMM).
  - `GET /api/metrics`: Retrieve evaluation metrics, confusion matrices, and model comparison benchmarks.
  - `GET /api/health`: Health status.

### E. Frontend Application (`frontend/`)
- Modern React + Vite application.
- `Leaflet` / OpenStreetMap map component:
  - Layer 1: Noisy GPS trajectory (red dotted line + points).
  - Layer 2: Reconstructed road route (bold blue line snapped to actual roads).
  - Layer 3: Candidate road segments (faint grey/green lines on click/hover).
  - Layer 4: Start (green pin) and End (checkerboard pin) markers.
- Dashboard with trajectory overview, distance, duration, and reconstruction speed.
- Model Performance tab with interactive comparison charts (Precision, Recall, F1, Reconstruction Error, Inference Time).
- Upload tab supporting drag-and-drop file upload with live validation.
