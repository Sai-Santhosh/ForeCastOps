# Counterfactual GDP Forecaster
**Interactive macro forecasting + policy “what-if” simulator** built in Streamlit.

This repo contains a deployable app that forecasts **log(GDP)** from key macroeconomic indicators and lets you **simulate counterfactual interest-rate shocks** (e.g., “What happens to predicted GDP if the Fed Funds Rate increases by +1.0%?”). It also includes an offline workflow to compute and visualize an **estimated treatment-effect time series** for Fed policy regimes.

> ⚠️ Note: This is a practical forecasting + counterfactual simulation tool. The “treatment effect” plots are **model-based estimates**, not definitive causal claims. Use for exploration and learning—not real policy decisions.

---

## What you can do
- **Refresh macro data** from FRED (GDP, CPI, unemployment, Fed Funds Rate, employment, M2).
- **Train & compare multiple models** for forecasting log(GDP):
  - Ridge (linear baseline)
  - XGBoost, LightGBM, CatBoost (tree ensembles)
- **Run counterfactual scenarios** by perturbing the Fed Funds Rate and comparing baseline vs counterfactual predictions.
- **View an estimated policy impact curve** (precomputed offline) that summarizes how “high-rate vs low-rate” regimes shift predicted log(GDP).

---

## App (Streamlit)
The app provides:
- A **model selector** (Ridge / XGBoost / LightGBM / CatBoost)
- A **Fed-rate shock slider** (e.g., −2.0pp to +2.0pp, step 0.1pp)
- Plots for:
  - Baseline vs counterfactual **log(GDP) forecasts**
  - An **estimated treatment-effect time series** (Fed Rate → log(GDP))

---

## Project layout
```
counterfactual-gdp-forecaster/
├── data/
│   ├── raw_macro.csv              # raw pull from FRED
│   ├── clean_macro.csv            # cleaned / aligned time series
│   └── final_macro.pkl            # modeling-ready DataFrame (used by app)
│
├── models/
│   ├── ridge_forecast.pkl
│   ├── xgb_forecast.pkl
│   ├── lgbm_forecast.pkl
│   ├── catboost_forecast.pkl
│   ├── treatment_effect.npy       # offline policy-effect series used by app
│   └── avg_treatment_effect.txt   # quick summary metric
│
├── notebooks/
│   ├── models.ipynb               # training pipeline + causal estimation
│   └── model_eval.ipynb           # model comparison (MAE/MSE/R²)
│
├── scripts/
│   └── fetch_fred_data.py         # automated FRED ingestion
│
├── app.py                         # Streamlit app entry point
├── requirements.txt
└── README.md
```

---

## Data sources (FRED)
This project uses the `fredapi` client to pull common macro indicators:
- GDP
- CPI
- Unemployment rate
- Federal Funds Rate
- Payroll employment
- Money supply (M2)

You can refresh these series locally any time.

---

## Method overview

### 1) Forecasting (supervised regression)
We model **log(GDP)** as a function of macro covariates. Multiple model families are trained so you can compare:
- Linear baseline (Ridge)
- Nonlinear models (XGBoost / LightGBM / CatBoost)

Evaluation uses a **time-ordered split** (no shuffling) so results reflect forward-looking performance.

### 2) Counterfactual simulation (policy “what-if”)
Counterfactuals are computed by:
1. Predicting log(GDP) under observed inputs (baseline)
2. Perturbing the **Fed Funds Rate** by a user-defined shock (e.g., +1.0pp)
3. Predicting again under the shocked inputs
4. Plotting baseline vs counterfactual trajectories and their deltas

This is a **model-based counterfactual**: it answers “What would the model predict if fed_rate changed and everything else stayed the same?”

### 3) Offline treatment-effect series (T-learner style)
To provide an interpretable “policy regime effect” curve, we compute a treatment-effect time series offline:
- Create a binary treatment label based on whether fed_rate is above a threshold (e.g., median).
- Fit separate regressors for treated vs control.
- Compute effect = predicted_y(treated) − predicted_y(control) over time.

The resulting series is saved to `models/treatment_effect.npy` and visualized in the app.

---

## Quickstart

### 1) Create environment
```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# Mac/Linux:
source .venv/bin/activate

pip install -r requirements.txt
```

### 2) Set your FRED API key
Create a file at:
```bash
echo "YOUR_FRED_API_KEY" > ~/.fred_api_key
```

### 3) Fetch macro data
```bash
python scripts/fetch_fred_data.py
```

This writes the raw macro dataset (CSV) under `data/`.

### 4) Train models + generate artifacts
Open and run:
- `notebooks/models.ipynb` (training + treatment effect generation)

This should produce:
- `models/*_forecast.pkl`
- `models/treatment_effect.npy`
- `data/final_macro.pkl`

### 5) Launch the Streamlit app
```bash
streamlit run app.py
```

---

## Model evaluation
Run:
- `notebooks/model_eval.ipynb`

It loads the saved models and produces a simple comparison table with:
- **MSE**
- **MAE**
- **R²**

(Use this to pick a “champion” model for demos.)

---

## Deployment (Streamlit Community Cloud)
1. Push this repository to GitHub.
2. In Streamlit Cloud, create a new app.
3. Set entry point to `app.py`.
4. Add a secret for your FRED key if you want data refresh in the cloud (optional).

If you don’t want to expose API keys, deploy with pre-generated `data/final_macro.pkl` and skip live refresh.

---

## Reproducibility notes
- Forecasting uses a time-ordered split to avoid leakage.
- If you update FRED pulls, re-run training so models and artifacts stay consistent.
- The treatment-effect series is **precomputed** offline and loaded by the app.

---

## Known limitations
- This is not a structural econometric model.
- Counterfactuals hold other variables fixed, which may not reflect real-world macro dynamics.
- The “treatment effect” series depends on the chosen threshold and model assumptions.

---

## Roadmap (nice next steps)
- Multi-variable shocks (CPI, unemployment, M2) via additional sliders
- Rolling-origin backtesting + horizon-specific error metrics (wMAPE/MASE)
- Scenario logging (“save run” + export to CSV)
- Event-window causal estimators (DiD / synthetic control) for discrete macro shocks

---

