# 📈 VNINDEX Time Series Forecasting with ARIMA

> An end-to-end time series analysis and forecasting project for the Vietnam Stock Market Index (VNINDEX) — covering stationarity testing, automatic model selection, residual diagnostics, and 30-day forward forecasting using ARIMA.

---

## 📌 Project Overview

This project applies the **ARIMA (AutoRegressive Integrated Moving Average)** framework to forecast VNINDEX daily closing prices. Data is pulled directly from the Vietnamese stock market via the `vnstock` library, covering the period from January 2024 to March 2026 (~550 trading days).

The analysis follows a rigorous econometric pipeline — from stationarity testing to automatic order selection to residual validation — before generating a 30-day forward forecast with 95% confidence intervals.

| | |
|---|---|
| **Target** | VNINDEX daily closing price |
| **Data Source** | vnstock (VCI feed) |
| **Period** | Jan 1, 2024 – Mar 29, 2026 |
| **Frequency** | Business days (`asfreq("B")`, forward-filled) |
| **Forecast Horizon** | 30 trading days |
| **Train/Test Split** | 80% / 20% |

---

## 🗂️ Dataset

Data is fetched live using the `vnstock` library — no manual download required:

```python
from vnstock import Vnstock

stock = Vnstock().stock(symbol="VNINDEX", source="VCI")
df = stock.quote.history(start="2024-01-01", end="2026-03-29", interval="1D")
```

**Key fields used:**

| Column | Description |
|---|---|
| `time` | Trading date (set as DatetimeIndex) |
| `close` | Closing price — the target series |

**Preprocessing:**
- Resampled to business day frequency (`asfreq("B")`)
- Missing values forward-filled (`ffill()`) to handle non-trading days

---

## 🔄 Project Pipeline

```
Live VNINDEX Data (vnstock API)
        │
        ▼
1. Data Fetching & Preprocessing   ← Business day resampling, ffill
        │
        ▼
2. Stationarity Testing            ← ADF test on raw series + 1st difference
        │
        ▼
3. ACF / PACF Analysis             ← Identify AR and MA order candidates
        │
        ▼
4. Auto ARIMA (pmdarima)           ← Stepwise AIC-minimizing model selection
        │
        ▼
5. Model Training (SARIMAX)        ← Fit on 80% train split
        │
        ▼
6. Residual Diagnostics            ← Plot diagnostics + Ljung-Box test
        │
        ▼
7. Out-of-Sample Evaluation        ← MAE, RMSE, MAPE on 20% test set
        │
        ▼
8. 30-Day Forward Forecast         ← Refit on full series, forecast next 30 days
```

---

## 🔬 Methodology

### Step 1 — Stationarity Testing (ADF Test)

The Augmented Dickey-Fuller test is applied to both the raw series and the first-differenced series to confirm integration order `d`:

```
H0: Series has a unit root (non-stationary)
H1: Series is stationary

→ Raw series: p > 0.05 → Non-stationary → d = 1
→ First difference: p < 0.05 → Stationary → confirmed d = 1
```

### Step 2 — ACF / PACF Analysis

Autocorrelation and Partial Autocorrelation plots on the first-differenced series are used to visually identify candidate values for `p` (AR order) and `q` (MA order).

### Step 3 — Automatic Model Selection

`pmdarima.auto_arima` performs stepwise search over ARIMA(p, 1, q) combinations, selecting the model with the lowest **AIC** (Akaike Information Criterion):

```python
auto_model = pm.auto_arima(
    close, d=1,
    start_p=0, max_p=5,
    start_q=0, max_q=5,
    information_criterion="aic",
    stepwise=True
)
```

### Step 4 — Residual Diagnostics

After fitting on the training set, residuals are validated with:
- **Plot diagnostics** — standardized residuals, histogram, Q-Q plot, correlogram
- **Ljung-Box test** (lags 10 & 20) — tests for remaining autocorrelation in residuals

A well-fitted model should show white-noise residuals (no autocorrelation, p-value > 0.05 in Ljung-Box).

### Step 5 — Evaluation Metrics

Out-of-sample performance on the test set (20%):

| Metric | Description |
|---|---|
| **MAE** | Mean Absolute Error |
| **RMSE** | Root Mean Squared Error |
| **MAPE** | Mean Absolute Percentage Error |

### Step 6 — 30-Day Forward Forecast

The model is refit on the full series before generating a 30-day ahead forecast with 95% confidence intervals:

```python
final_model = SARIMAX(close, order=(p, d, q), trend="c").fit(disp=False)
future = final_model.get_forecast(steps=30)
```

---

## 📊 Visualizations

| Chart | Purpose |
|---|---|
| ACF / PACF plots | Identify AR and MA orders visually |
| Residual diagnostics (4-panel) | Validate model assumptions |
| Train / Test / Forecast overlay | Visual comparison of actual vs predicted |
| 30-day forward forecast with 95% CI band | Business-ready forecast output |

---

## 💡 Key Insights

1. **VNINDEX is non-stationary in levels** — the ADF test confirms a unit root on raw prices; first differencing achieves stationarity, consistent with financial theory (random walk component).
2. **Auto ARIMA selects a parsimonious model** — stepwise AIC minimization avoids overfitting while capturing the index's autocorrelation structure.
3. **Ljung-Box test validates residual quality** — white-noise residuals confirm the model has extracted the predictable signal from the series.
4. **MAPE provides an interpretable accuracy benchmark** — for financial indices, MAPE below 2% is considered strong short-term performance.
5. **Confidence intervals widen over the forecast horizon** — reflecting the inherent uncertainty of financial forecasting beyond the near term.

---

## 📁 File Structure

```
├── Stock_Time_Series.ipynb    ← Full analysis notebook
└── README.md                  ← Project documentation
```

> **Note:** No static dataset file is needed. Data is fetched live from the VCI feed via `vnstock` each time the notebook is run.

---

## ⚙️ Installation & Usage

```bash
pip install pandas numpy matplotlib statsmodels pmdarima vnstock scikit-learn
```

Open and run all cells in order:

```bash
jupyter notebook Stock_Time_Series.ipynb
```

> The notebook fetches live data on execution. If the `end` date in Cell 1 needs updating, change it to today's date for the most current forecast.

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Language | Python 3 |
| Data Fetching | `vnstock` (VCI feed) |
| Data Processing | `pandas`, `numpy` |
| Time Series Modeling | `statsmodels` (SARIMAX, ADF, ACF/PACF, Ljung-Box), `pmdarima` (auto_arima) |
| Evaluation | `scikit-learn` (MAE, RMSE, MAPE) |
| Visualization | `matplotlib` |

---

## 👤 Author

**Võ Quang Khải**
Data Analyst | Finance & Data Science Background
[LinkedIn](https://www.linkedin.com/in/voquangkhaikg2003/) · [GitHub](https://github.com/voquangkhai2003)
