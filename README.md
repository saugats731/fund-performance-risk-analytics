# Quantitative Performance & Risk Analytics: The Compounding Fallacy & Fund Benchmarking

An applied quantitative investment analysis and business intelligence project demonstrating:
1. Why **arithmetic averaging misrepresents multi-period returns** (The Compounding Trap).
2. Institutional risk-adjusted performance attribution (**Sharpe Ratio**, **Annualized Volatility**, **CAPM Beta**, and **R²**) across simulated share classes and a market benchmark.
3. Resolving **non-trading day calendar gaps** in Power BI / DAX time-intelligence functions.

---

## The Compounding Fallacy: Arithmetic Mean vs. Geometric Return (CAGR)

A frequent error in performance evaluation is averaging multi-period returns arithmetically. Because capital scales multiplicatively rather than additively, arithmetic averages systematically overstate investment outcomes:

```text
Initial Capital: $100
├── Year 1: +50% gain  ──> Portfolio Value: $150
└── Year 2: -50% loss  ──> Portfolio Value: $75
```

### The Conflict
* **Arithmetic Average:** ((+50%) + (-50%)) / 2 = **0.00%** *(misleadingly suggests capital was preserved)*
* **Actual Realized Net Return:** (75 - 100) / 100 = **-25.00%**
* **True Compound Annual Growth Rate (CAGR):**
  $$\text{CAGR} = \left(\frac{75}{100}\right)^{\frac{1}{2}} - 1 = \sqrt{0.75} - 1 \approx \mathbf{-13.40\% \text{ p.a.}}$$

> **Core Financial Takeaway:** Returns are path-dependent. A 50% drawdown requires a **+100% gain** just to restore the initial balance. Geometric annualization (CAGR) accounts for the fluctuating capital base, providing the only mathematically valid metric for multi-year capital growth.

---

## Analytical Scope & Raw Data Architecture

Using simulated monthly Net Asset Values (NAV) spanning **December 2019 to October 2025**, two distinct fund share classes were evaluated against a broad market benchmark:

* **Share Class 1 (SC1):** Growth-oriented, low-correlation strategy.
* **Share Class 2 (SC2):** Capital-efficiency, market-linked strategy.
* **Base Re-indexing:** Because price levels vary significantly (SC1 starts at 150.20, SC2 at 320.00, and Benchmark at 1,020.32), all series are rebased to an index of 100 as of 31.12.2019 to ensure true comparability:
  $$\text{Rebased Price}_t = \left(\frac{\text{Current Price}_t}{\text{Starting Price}_0}\right) \times 100$$

### Raw Excel Dataset Overview
Before ingestion into the Power BI semantic model, monthly closing NAVs and benchmark index values were structured as follows:

![Excel Raw Data Preview](excel_data_raw.png)

---

## Summary of Results & Key Metrics

| Metric | Share Class 1 (SC1) | Share Class 2 (SC2) | Benchmark | Strategic Interpretation |
| :--- | :---: | :---: | :---: | :--- |
| **2025 YTD Return** | 10.28% | 15.85% | 7.53% | SC2 captured current market momentum |
| **Annualized Return (CAGR)** | 17.78% | 10.58% | — | SC1 dominates long-term capital accumulation |
| **Annualized Volatility (σ_ann)** | 12.88% | 6.67% | — | SC2 offers half the risk profile of SC1 |
| **Sharpe Ratio** | 1.38 | 1.59 | — | **SC2 delivers superior risk-adjusted efficiency** |
| **Market Beta (β)** | 0.37 | 0.77 | 1.00 | SC1 is defensive/decoupled; SC2 tracks benchmark |
| **Coefficient of Determination (R²)** | 0.02 | 0.36 | 1.00 | SC1 driven by manager alpha; SC2 driven by market beta |

---

## Econometric & Methodological Framework

### 1. Geometric Annualization (CAGR)
Annualized return is calculated using a point-to-point approach to isolate true capital appreciation:
```dax
Ann. Return SC_1 = 
VAR StartPrice = CALCULATE(SUM(SimulatedData[ShareClass1]), FIRSTDATE('Calendar'[Date]))
VAR EndPrice = CALCULATE(SUM(SimulatedData[ShareClass1]), LASTDATE('Calendar'[Date]))
VAR TotalMonths = COUNTROWS(DISTINCT('SimulatedData'[Date]))
VAR NumYears = TotalMonths / 12
RETURN
    ( (EndPrice / StartPrice) ^ (1 / NumYears) ) - 1
```

### 2. Risk Scaling via the Square Root of Time
Risk cannot be measured point-to-point; it requires calculating periodic dispersion across continuous holding periods. Monthly sample standard deviation is scaled to an annual standard:
```dax
Ann. Volatility SC1 = 
STDEVX.S(
    'SimulatedData',
    'SimulatedData'[SC1_Return]
) * SQRT(12)
```
*Why multiply by √12 instead of 12?* Financial returns follow a random-walk assumption where variance scales linearly with time ($t$), meaning standard deviation (volatility) scales with the square root of time ($\sqrt{t}$). Multiplying directly by 12 assumes volatility moves in the exact same direction every single month, severely overstating systemic risk.

### 3. Data Engineering: Solving Non-Trading Day Gaps
* **The Problem:** Standard DAX time-intelligence functions (`DATEADD`, `FIRSTDATE(DATESMTD)`) navigate continuous Gregorian calendars. In financial markets, equities do not trade on weekends or bank holidays. Comparing Monday close to Sunday yields blank or null values.
* **The Solution:** Transitioned the calculation logic from static dates to a sequential active trading-day index querying actual transaction dates (`StockData`) directly, successfully skipping non-trading gaps.

---

## Interactive Dashboard Preview

### Portfolio Performance Overview
![Performance Overview](dashboard_overview.png)

### Share Class Deep Dive & Regression Analysis
![Risk Metrics & Scatter](risk_metrics_scatter.png)

---

## Repository Structure

```text
├── LICENSE
├── README.md                              <-- Project documentation & econometric methodology
├── fund_performance_risk_engine.pbix     <-- Interactive Power BI report file
├── simulated_fund_benchmark_data.xlsx    <-- Cleaned simulated NAV & benchmark dataset
├── excel_data_raw.png                    <-- Preview of raw monthly time series
├── dashboard_overview.png                 <-- Dashboard overview preview image
├── risk_metrics_scatter.png               <-- Regression & risk deep-dive visual
├── cagr_geometric_annualization.txt       <-- DAX: Point-to-point geometric CAGR
├── dax_annualized_volatility.txt          <-- DAX: Volatility scaled by sqrt(12)
└── daxtrading_calendar_workaround.txt     <-- DAX: Non-trading session offset logic
```
