# Quantitative Performance & Risk Analytics: The Compounding Fallacy & Fund Benchmarking

An applied quantitative investment analysis and business intelligence project demonstrating:
1. Why **arithmetic averaging misrepresents multi-period returns** (The Compounding Trap).
2. Institutional risk-adjusted performance attribution (**Sharpe Ratio**, **Annualized Volatility**, **CAPM Beta**, and **$R^2$**) across simulated share classes and a market benchmark.
3. Resolving **non-trading day calendar gaps** in Power BI / DAX time-intelligence functions.

---

## The Compounding Fallacy: Arithmetic Mean vs. Geometric Return (CAGR)

A frequent error in performance evaluation is averaging multi-period returns arithmetically. Because capital scales multiplicatively rather than additively, arithmetic averages systematically overstate investment outcomes:

```text
Initial Capital: $100
├── Year 1: +50% gain  ──> Portfolio Value: $150
└── Year 2: -50% loss  ──> Portfolio Value: $75
