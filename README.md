# Wind Turbine Drivetrain Fault Detection

**Predicting drivetrain failures on utility-scale wind turbines from SCADA sensor data — so a degrading gearbox or bearing is caught before it fails, not after.**

### 📊 [**View the Project Report →**](https://fomenky.github.io/wind-turbine-fault-detection/)

Published via GitHub Pages — no download or setup required. The landing page is a clean **executive summary**; it links through to the **full detailed report** (every chart, the model-comparison tables, and the complete analysis). Direct links:

- **Executive summary (Pages landing page):** https://fomenky.github.io/wind-turbine-fault-detection/
- **Full detailed report:** https://fomenky.github.io/wind-turbine-fault-detection/reports/wind_turbine_fault_detection_full_report.html
- Source copies: [`reports/executive_summary.html`](reports/executive_summary.html) · [`reports/wind_turbine_fault_detection_full_report.html`](reports/wind_turbine_fault_detection_full_report.html)

---

## Problem

A wind operator running a fleet of 1,150 utility-scale (2.5 MW) turbines sells its energy under power purchase agreements, so every hour a turbine is offline is revenue that cannot be recovered. The costliest failures are in the **drivetrain** — the gearbox and main bearing — which are expensive, slow to source, and need a mobile crane to replace. An unplanned drivetrain failure takes a turbine offline for an estimated **7–21 days**, and a degrading main bearing left running can destroy the gearbox it feeds, turning a contained repair into a major one.

The core difficulty is **recognition**. By the time a fault is severe enough to trip a fixed-threshold alarm, the turbine is often already offline. Before that, a genuinely degrading drivetrain can run for days looking "normal" to simple thresholds, buried in the noise of a machine reacting to gusty wind across dozens of sensor channels.

## Objective

Build a **binary classifier** that reads each turbine's multi-channel SCADA signature and labels the drivetrain as **fault** or **normal**, so control-room analysts can focus on the handful of machines genuinely degrading rather than scanning the whole fleet — catching faults in the window where the turbine can still be stopped before damage cascades.

Because a **missed fault** (false negative) costs 7–21 days of lost revenue plus emergency repair, while a **false alarm** (false positive) costs only a routine inspection, the model is deliberately tuned to favour **recall** (catching true failures).

## Data

- **Source:** course-provided SCADA dataset (`data/wind_turbine_detection.csv`), synthetic/representative — **no personal or confidential data**.
- **Scale:** ~130,000 records of **10-minute averaged SCADA intervals** across 15 turbines.
- **Target:** `failure` (0 = normal, 1 = drivetrain fault), a realistic **~5–8% positive rate** (class imbalance handled explicitly).
- **Features (30+):** grouped into
  - **Environmental** — wind speed/direction, turbulence, air density, ambient temperature, humidity
  - **Operational** — power output, rotor/generator speed, blade pitch, yaw misalignment
  - **Thermal** — gearbox oil & bearing temps, generator bearing/winding temps, main bearing & nacelle temps
  - **Vibration / FFT** — drivetrain & tower RMS, bearing outer/inner-race defect frequencies (BPFO/BPFI), gearmesh and sideband amplitudes
  - **Lubrication** — oil particle count, oil pressure
  - **Lifecycle** — operating hours, cumulative energy, load cycles, hours since maintenance, prior fault count, component age

## Methodology

1. **EDA** — distributions, class imbalance, correlation of each sensor group with failure, missing-value assessment.
2. **Preprocessing** — median imputation (fit on training data only) for the few features with gaps, train/test split preserving class balance.
3. **Modeling** — a **Decision Tree** baseline, a **neural network** (TensorFlow/Keras) comparison, and a **tuned XGBoost** classifier as the primary model.
4. **Imbalance & threshold handling** — recall-oriented tuning so the model favours catching true faults over avoiding inspections.

## Evaluation & key findings

- **Primary metric:** **recall** (fault-detection rate), chosen because the business cost of a missed fault dwarfs that of a false alarm. Precision, F1, and accuracy are reported alongside for context.
- **Strongest fault indicators** surfaced by the analysis:
  - **Vibration / FFT signatures** — drivetrain RMS, bearing BPFO/BPFI, gearmesh, and sideband energy are the primary fault fingerprint.
  - **Thermal escalation** — gearbox bearing/oil and main bearing temperatures rise as degradation progresses.
  - **Oil contamination** — oil particle count is markedly higher in fault states (metallic wear debris).
  - **Power-output drop** at equivalent wind speed, consistent with mechanical drag.
  - **Maintenance history** — higher prior fault counts and longer gaps since service compound risk.

See the [full report](https://fomenky.github.io/wind-turbine-fault-detection/reports/wind_turbine_fault_detection_full_report.html) for the complete model-comparison table, metrics, and visualizations.

## Interpretation

The results support a shift from reactive, fixed-interval maintenance toward **condition-based intervention**: the model learns the multi-sensor signature of early drivetrain degradation that single-threshold alarms miss, giving analysts a prioritized, evidence-backed fault signal.

## Limitations & potential improvements

- Trained on a **single synthetic/representative dataset** (15 turbines) — a production system would need validation on real fleet data across turbine types and seasons.
- Labels are point-in-time fault/normal, not remaining-useful-life — extending to **time-to-failure / RUL regression** would add lead-time value.
- No temporal modeling of each turbine's trajectory; sequence models (LSTM/Temporal CNN) on the 10-minute series could capture degradation trends.
- Threshold choice trades precision for recall; the operating point should be set with real inspection-cost data.

## Technologies

Python · pandas · NumPy · scikit-learn · **XGBoost** · TensorFlow/Keras · matplotlib · seaborn · Jupyter

## Reproducing the analysis

```bash
# 1. Clone
git clone https://github.com/fomenky/wind-turbine-fault-detection.git
cd wind-turbine-fault-detection

# 2. (Recommended) create a virtual environment
python3 -m venv .venv && source .venv/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch the notebook
jupyter notebook notebooks/wind_turbine_fault_detection.ipynb
```

The notebook loads `data/wind_turbine_detection.csv` with a relative path, so it runs from the repo root as-is.

## Repository layout

```text
wind-turbine-fault-detection/
├── README.md
├── index.html                 # GitHub Pages landing page (clean executive summary → links to full report)
├── notebooks/
│   └── wind_turbine_fault_detection.ipynb              # Full Python analysis & modeling
├── reports/
│   ├── wind_turbine_fault_detection_full_report.html   # Full rendered analysis + all charts
│   └── executive_summary.html                          # Condensed business-facing summary
├── data/
│   └── wind_turbine_detection.csv
├── requirements.txt
├── .gitattributes             # Language-bar: count notebook, exclude generated HTML
└── .gitignore
```

---

### About this project

This work was completed as a capstone project in the **Post Graduate Program in Artificial Intelligence & Machine Learning (UT Austin / Great Learning)**. It is shared as a demonstration of applied, end-to-end machine-learning problem solving. The business scenario, dataset, and problem framing are course-provided academic material; results reflect a proof-of-concept on that dataset and are **not** claims of production deployment or commercial outcomes.
