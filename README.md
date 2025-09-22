# Practice Effects in Digital Health Assesment Tools
This repo contains analysis done on practice effects.

---

## Quick Start

```bash
git clone <your-repo>
cd <your-repo>
python -m venv .venv
source .venv/bin/activate     # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter lab                   # or: jupyter notebook
```

Open `analysis.ipynb` and run cells top-to-bottom.

---

## Repo Layout

```
.
├─ README.md
├─ requirements.txt
├─ analysis.ipynb              # Notebook with all analysis & plots
└─ data/
   ├─ test_data_moca.csv       # Crossroads paper per-sample trajectory + MoCA
   ├─ final_slip_features.csv  #  per-slip episodic features
   └─ details (4).csv          # extra slip-level details
```

---

## Data Dictionary

### `data/test_data_moca.csv` — per-sample trajectory (main file)

- **Identifiers**
  - `pid`: participant ID
  - `blockno`: block index (main task typically has 8 blocks)

- **Time & Kinematics**
  - `coordt`: timestamp (ms or s; inferred in the notebook)
  - `coordx`, `coordy`: touch coordinates (screen units)
  - `speed`: instantaneous/per-sample speed (precomputed; units depend on pipeline)
  - `pausevalue`: pause indicator/value (positive ⇒ paused;)


### (Optional) `data/final_slip_features.csv`
Per-slip summaries: slip onset/detection/correction, slip duration, confidence, error flags.

---

## Study Design (condensed)

- **One lab session** per participant, paired/counter-balanced with MoCA.
- **Main task:** typically **8 blocks**; fixed sequence of targets/distractors.
- Data are a **continuous per-sample stream**: timestamps, x/y, speed, pause flag/value.
- We compute **block-level aggregates** from this stream.

---

## What the Notebook Does

The notebook is organized into clearly labeled cells:

1. **Imports & Config**  
   - Sets plotting defaults and paths  
   - Targets MoCA scores **[26, 28, 30]**

2. **Load & Sanity Checks**  
   - Confirms required columns: `pid, blockno, coordt, speed, pausevalue, moca`  
   - Coerces types; drops critical nulls

3. **Time Normalization & Δt**  
   - Sorts by `(pid, blockno, coordt)`  
   - **Infers** whether `coordt` is in **milliseconds** or **seconds** by median inter-sample Δt  
   - Computes `time_s` and per-sample `dt` (with basic cleaning for negatives/NaNs)

4. **Selecting Three Participants (MoCA 26/28/30)**  
   - For each target MoCA score, automatically **selects the PID** with the most complete data (most blocks/rows)  
   - You can override with explicit PIDs

5. **Block-level Metrics**  
   For each `(pid, blockno)`, the notebook computes:
   - **Total pause time (s):** sum of `dt` where `pausevalue > 0`  
   - **Completion time (s):** `max(time_s) − min(time_s)` within the block  
   - **Median speed:** median of `speed` within the block

6. **Group Reference Lines**  
   - For each block, computes **group mean** and **group median** across **all participants** in `test_data_moca.csv`

7. **Plotting**  
   - Three figures (pause time, completion time, median speed)  
   - Each plot shows: the **three selected participants** (MoCA 26/28/30) + **Group Mean** + **Group Median**  
   - X-axis = **block number**

---

## Configuration & Customization

Open **Cell 1** in `analysis.ipynb`:

- **Data path**
  ```python
  from pathlib import Path
  DATA_DIR = Path("data")
  TRAJ_FILE = DATA_DIR / "test_data_moca.csv"
  ```

- **MoCA targets**
  ```python
  TARGET_MOCA = [26, 28, 30]
  ```

- **Pick exact PIDs (optional)**
  ```python
  picked = {26: "P001", 28: "P014", 30: "P022"}  # example IDs
  ```

- **Restrict blocks to 1..8 (optional)**
  ```python
  df = df[df["blockno"].isin(range(1, 9))]
  ```

---

## Assumptions

1. **Pause definition**  
   `pausevalue > 0` ⇒ paused sample; total pause time is the **sum of Δt** over those samples.  
   If `pausevalue` already stores per-sample **duration**, replace the summand with `pausevalue` (convert units if needed).

2. **Time units (`coordt`)**  
   We infer units. If you **know** `coordt` is in ms, force:
   ```python
   time_scale = 1_000.0
   ```
   If in s:
   ```python
   time_scale = 1.0
   ```

3. **Group lines**  
   Group mean/median use **all participants** in `test_data_moca.csv`. Filter first if you want a subset.

---

## Expected Outputs

Running the notebook produces three line charts:

1. **Total Pause Time per Block (s)**  
2. **Completion Time per Block (s)**  
3. **Median Speed per Block (units of `speed`)**

Each chart overlays **MoCA 26**, **MoCA 28**, **MoCA 30**, **Group Mean**, **Group Median**.

---

## Requirements

Minimal Python packages:
```
pandas
numpy
matplotlib
```

Optional:
```
jupyterlab
```

---

## Troubleshooting

- **Missing columns:** Ensure `data/test_data_moca.csv` contains at least  
  `pid, blockno, coordt, speed, pausevalue, moca`.
- **Participants not found for a MoCA score:** The notebook prints a warning.  
  Set `picked = {...}` with explicit PIDs.
- **Weird time scaling:** Override `time_scale` directly if inference seems off.
- **Uneven blocks across PIDs:** Consider restricting to a known block set (e.g., `1..8`) for aligned x-axes.

---

## Attribution

Dataset and task design by the Crossroads + MoCA team (Sujit et al.).  
