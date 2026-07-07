# Time Series Analysis of U.S. Wheat, Corn, and Soybean Futures Prices

MATH 422 (Time Series Analysis) final project analyzing daily CBOT futures prices for wheat, corn, and soybeans from January 2000 to December 2024 (~6,300 trading days). The project models trends, volatility, and cross-commodity dependence using ARIMA/ARIMAX, univariate and multivariate GARCH, and spectral analysis, then cross-validates forecasts against actual October–December 2024 prices.

## Data

- Daily closing prices and trading volume for wheat, corn, and soybean futures contracts (CBOT), sourced from Investing.com / Yahoo Finance: `data/wheat_futures.csv`, `data/corn_futures.csv`, `data/soybean_futures.csv`.
- Supplementary volume series used to fill gaps in the primary files (`data/wheat_volume.csv`, `data/soybean_volume.csv`), merged in via linear interpolation (`na.approx`) for missing values.
- Each contract represents 5,000 bushels; prices are quoted in U.S. cents per bushel in the raw data (converted to dollars for analysis/plots).

## Method

1. Visualize prices/volumes and test for seasonality via STL decomposition, non-parametric smoothing (`ksmooth`), and spectral analysis — none of the three series showed significant seasonal periodicity, consistent with futures prices reflecting market expectations rather than production cycles.
2. Fit ARIMA models per commodity on price differences; validate stationarity (ADF/PP/KPSS/ERS) and residual independence/normality.
3. Fit univariate GARCH(1,1) models (via `rugarch`'s ARFIMA+sGARCH with skewed Student-t errors) on daily returns to capture volatility clustering.
4. Fit a multivariate (DCC) GARCH model across all three return series to estimate time-varying cross-commodity correlation.
5. Fit ARIMAX models using each commodity's most-correlated peer as an exogenous regressor.
6. Cross-validate every model class against actual Oct 1–Dec 5, 2024 prices before considering any further forecasting.

## Key Findings

- **No seasonality:** unlike many economic time series, none of the three futures prices show significant seasonal structure — spectral analysis and STL both came back empty, consistent with prices being driven by macro conditions rather than fixed cycles.
- **ARIMA:** best fits were ARIMA(1,1,3) for wheat, ARIMA(0,1,3) for corn, and ARIMA(0,1,0) (a pure random walk) for soybeans. All three had stationary but non-independent, non-normal residuals — not reliable for forecasting.
- **GARCH:** ARFIMA(0,0,0)+GARCH(1,1) with skewed Student-t errors fit all three return series, with high volatility persistence (α₁+β₁≈1). Out-of-sample cross-validation against Oct–Dec 2024 produced **negative R² for all three** (-0.012, -0.0009, -0.033) — worse than predicting the mean, so no further forecasts were pursued.
- **Multivariate GARCH:** all three commodities are highly correlated, wheat–corn most strongly, with a highly persistent DCC structure (β≈0.963).
- **ARIMAX:** adding the most-correlated commodity as an exogenous regressor produced statistically significant coefficients, but residuals still failed independence/normality checks.
- Overall conclusion: no model class here reached forecasting-grade reliability; the value of the project was in demonstrating the process (xts/quantmod workflow, ARFIMA/eGARCH/mGARCH model-building) rather than producing a deployable price forecast.

## Repository Structure

```
analysis.qmd     # Quarto source covering data wrangling, ARIMA, and GARCH for wheat & corn
data/            # Daily price/volume series for wheat, corn, and soybeans
report/          # Final paper (paper.docx) and initial project proposal (proposal.docx)
```

**Note:** `analysis.qmd` reflects an earlier working stage of the analysis (wheat and corn only). The full scope described above — soybeans, ARIMAX, and multivariate GARCH — was completed afterward and is documented in `report/paper.docx`, but that later code was never folded back into the `.qmd` source.

## Reproducing

Requires [Quarto](https://quarto.org/) and R with the packages listed in `analysis.qmd`'s setup chunk (`astsa`, `forecast`, `rugarch`, `quantmod`, `xts`, `tsDyn`, and others). Render with:

```bash
quarto render analysis.qmd
```

## Author

Anthony Le — Denison University, MATH 422, Fall 2024
