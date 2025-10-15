# 🇮🇹 Energy Load Forecasting — Prophet vs SARIMAX

An end-to-end, reproducible Jupyter Notebook that compares Facebook Prophet and SARIMAX for hourly electricity load forecasting (Italy, 2016). The notebook includes data cleaning (DST/anomaly repair), EDA, model training with an exogenous solar regressor, and side-by-side model evaluation and visualizations.

---

## 🎯 Project Objective

Build accurate hourly (1-hour resolution) load forecasts and compare two complementary forecasting approaches:

- **Facebook Prophet** — automatic handling of multiple seasonalities, holidays, and external regressors.
- **SARIMAX** — a classical statistical model with explicit seasonal structure and exogenous regressors.

Goal: produce reliable 30-day (720-hour) ahead forecasts and evaluate models using MAE, RMSE, and MAPE.

---

## 📁 Repository Structure

```
📦 energy-load-forecasting
│
├── Energy Demand Forecasting.ipynb    # Main end-to-end notebook (EDA → models → evaluation)
├── plots/                             # Saved static images produced by the notebook
└── README.md                          # This file
```

Notes:

- The primary artifact is `Energy Demand Forecasting.ipynb`. Run it in Jupyter or open in VS Code.
- Plots produced by the notebook may be saved into the `plots/` folder.

---

## 📊 Dataset Overview

- Expected dataset: hourly electricity load for Italy (2016) with solar generation as an exogenous regressor.
- Required columns (or equivalent names):
  - `utc_timestamp` (datetime)
  - `IT_load_new` (load) or a column that will be renamed to `Load_MW`
  - `IT_solar_generation` (solar) or a column that will be renamed to `Solar_MW`

Behavior when dataset is missing:

- The notebook includes a synthetic data fallback so the analysis and modeling pipeline remain runnable when the CSV is not available.

Default in-notebook placeholder path:

- Edit the `file_path` variable in the notebook (search the notebook for `file_path = r"YOUR_FILE_PATH"`) and set it to your CSV location.

---

## 🧭 Data Preprocessing (what the notebook performs)

1. Read CSV with `parse_dates=['utc_timestamp']` (or generate synthetic data if file not found).
2. Set `utc_timestamp` as the DataFrame index.
3. Inspect shape, dtypes, and missing values.
4. Handle missing values: forward-fill then backward-fill.
5. Rename columns to `Load_MW` and `Solar_MW` for clarity.
6. Detect and repair DST-related anomalies (a visual inspection window is used; linear interpolation is applied across the anomaly window).
7. Create derived features for EDA (hour, day-of-week, day name).
8. Split data into train / test (last 30 days = 720 hours used as the test set).

---

## 🧠 Models Implemented

- Prophet (with `solar` as external regressor and Italian national holidays enabled).
- SARIMAX (with `Solar_MW` as exogenous regressor). The notebook uses a sensible hard-coded configuration: `order=(2,1,1)` and `seasonal_order=(1,0,1,24)` after an optional `auto_arima` step.

Modeling highlights:

- Prophet requires a DataFrame with `ds` (datetime) and `y` (target) columns; `solar` is added via `add_regressor('solar')`.
- SARIMAX is fit using `train_df['Load_MW']` with `exog=train_df[['Solar_MW']]`.

---

## 📈 Evaluation Metrics

Metrics calculated on the test set (last 30 days / 720 hours):

- MAE — Mean Absolute Error
- RMSE — Root Mean Squared Error
- MAPE — Mean Absolute Percentage Error

The notebook reports these metrics for each model and uses them to select a winner based on the lower MAPE. It also shows error distributions and comparison plots.

---

## 🧾 Visual Outputs

Key visualizations created by the notebook (and saved into `plots/` when cells save them):

- Italy 2016 — Hourly Energy Data Overview (Load & Solar)
- Time Series Decomposition (Trend / Seasonal / Residual)
- Solar Generation — Anomaly Detection Window (DST repair visualization)
- Average Hourly Load Heatmap by day-of-week
- Prophet forecast vs actual (with 95% confidence interval)
- SARIMAX forecast vs actual
- Model comparison plot (Prophet vs SARIMAX vs Actual)
- Error distribution histograms

---

## ⚙️ Quick Start (Windows, Git Bash / WSL - bash.exe)

1. Create and activate a virtual environment (recommended):

```bash
python -m venv .venv
# Activate in Git Bash / WSL (bash.exe)
source .venv/Scripts/activate
# (PowerShell) .\.venv\Scripts\Activate.ps1  OR  (CMD) .\.venv\Scripts\activate.bat
```

2. Install required packages (minimal list used in the notebook):

```bash
pip install --upgrade pip
pip install pandas numpy matplotlib plotly statsmodels prophet pmdarima scikit-learn
```

Notes about Prophet on Windows:

- Installing `prophet` from PyPI on Windows may require a C++ build toolchain. If installation fails, use conda:

```bash
conda install -c conda-forge prophet
```

3. Open the notebook and run all cells (or run step-by-step):

```bash
jupyter notebook "Energy Demand Forecasting.ipynb"
# or open the notebook in VS Code
```

4. Before running: update the `file_path` variable in the notebook to point to your CSV, or allow the synthetic fallback.

---

## ✅ Contract (small and precise)

- Inputs: CSV with hourly timestamps and columns for load and solar (or no CSV → synthetic data). Notebook parameter: `file_path`.
- Outputs: Forecast arrays/plots (Prophet and SARIMAX), evaluation metrics (MAE/RMSE/MAPE), and optional saved images in `plots/`.
- Error modes: If CSV missing → synthetic dataset generated; if Prophet install fails → guidance to use conda; DST anomalies → repaired via linear interpolation in the notebook.

---

## ⚠️ Edge Cases & Handling

- Missing CSV: safe fallback synthetic dataset is created so the pipeline remains runnable.
- DST-duplicated hours: visually inspected and repaired using linear interpolation across a defined anomaly window.
- Timezone-aware timestamps: notebook removes timezone information from `ds` prior to fitting Prophet.
- Negative or unphysical solar values after repair: linear interpolation is chosen to reduce overshoot risk.

---

## 🧪 Quality Gates (smoke checklist)

- Build: Not applicable — notebook-based analysis.
- Lint / Typecheck: Not included (not essential for exploratory notebook).
- Tests: None included; adding unit tests for preprocessing is recommended as a next step.
- Smoke test: Notebook can run end-to-end using the synthetic fallback dataset when the CSV is missing.

Requirements coverage: This README documents dataset expectations, how to run, preprocessing steps, models used, and evaluation — Done.

---

## 🔬 Key Learnings

1. Seasonal patterns (daily/weekly/yearly) and calendar effects dominate hourly electricity demand.
2. Solar generation is a strong exogenous predictor and meaningfully improves forecasts.
3. DST transitions can introduce duplicated/missing hours — detect and repair these before modeling.
4. Prophet is fast to configure and models holidays easily; SARIMAX provides transparent seasonal parameterization.

---

## ♻️ Future Improvements

- Add `requirements.txt` or `environment.yml` for exact dependency pinning.
- Factor preprocessing and modeling code into reusable Python modules (for testing and CI).
- Add unit tests for preprocessing, a small sample dataset, and CI workflow.
- Add a small CLI wrapper (e.g., `run_forecast.py`) accepting `--data-path`, `--plots-path`, and `--save-forecasts`.
- Experiment with ensembles (Prophet + SARIMAX) or machine-learning regressors for hybrid models.

---

## 📌 Reproducibility Tips

- Freeze your environment after a successful run: `pip freeze > requirements.txt`.
- If `prophet` is troublesome on Windows, prefer conda-forge: `conda install -c conda-forge prophet`.
- Save model outputs (forecasts, metrics, plots) after a run for reproducibility.

---

## 👨‍💻 Author

**Alexander Olomokoro**  
Full-Stack Developer | Machine Learning Engineer  
📍 Nigeria  
🔗 [LinkedIn](https://www.linkedin.com/in/alexander-olomukoro-699255199) • [Twitter/X](https://x.com/xandersavage7) • [GitHub](https://github.com/xandersavage)

---

## 📝 License (suggestion)

MIT
