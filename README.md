# Sector Rotation Strategy Backtester

**Test whether your market rotation strategy actually works before risking capital.**

Most traders talk about rotating sectors. Few test the idea systematically. This project applies quantitative portfolio-analysis principles — hypothesis, backtest, risk-adjusted metrics, benchmark comparison — to test market-rotation rules rather than assume they work.

Input a rotation rule (e.g., "rotate defensive when VIX > 25"). Get back total return, Sharpe ratio, max drawdown, and win rate. See if it actually beats buy-and-hold.

**Current status**: The backtesting engine, metrics, and research framework are built and working on realistic simulated data. The next phase — and the single highest-value upgrade — is plugging in real historical prices (see [Real Market Data](#real-market-data--priority-upgrade) below).

---

## What It Does

```
Your Rotation Rule  →  Historical Backtester  →  Risk Metrics Dashboard
"VIX > 25?"            5 years of data              Sharpe | Drawdown | Return
                       Daily calculations           vs S&P 500 benchmark
```

In one click:
- 📊 Compare your strategy vs S&P 500 (same period, same timeline)
- 📈 Equity curve chart (visual proof of concept)
- 🎯 Sector allocation over time (see when rotations happened)
- 📋 Key metrics: Return, Sharpe ratio, max drawdown, win rate
- ⚡ Results in <100ms (no waiting)

**Try it now**: Open `index.html` in any browser. Simulated data loads instantly. Test different rotation rules. See what works.

---

## Why This Matters

### The Problem
Retail investors rotate sectors based on hunches. Institutional managers validate with data.

### The Solution
This tool lets you:
1. **Test ideas empirically** — "Does VIX-based rotation beat buy-and-hold?"
2. **Compare risk properly** — Sharpe ratio (return per unit of risk), not just returns
3. **Benchmark correctly** — Against S&P 500 over identical periods
4. **Iterate fast** — Change your rule, re-run, see new results instantly

### Example Results
**Strategy**: Rotate to defensive sectors (XLV, XLP) when VIX > 25  
**Backtest**: 2020–2024

| Metric | Strategy | S&P 500 |
|--------|----------|---------|
| **Total Return** | +87.5% | +62.3% |
| **Sharpe Ratio** | 1.21 | 0.89 |
| **Max Drawdown** | -18.2% | -25.4% |
| **Win Rate** | 52.3% | — |

**Interpretation**: Strategy outperformed by 25% with lower drawdown. Better risk-adjusted returns (higher Sharpe).

## Quick Start

**Zero setup. Just open a file.**

```bash
git clone https://github.com/yourusername/sector-rotation-backtester.git
cd sector-rotation-backtester
open index.html
```

Or: Download `index.html` and double-click it.

**That's it.** No build step. No dependencies to install. React and Chart.js load from CDN.

### First backtest (30 seconds)
1. Choose a rotation rule from the dropdown (e.g., "VIX threshold")
2. Set the threshold (e.g., VIX > 25)
3. Set date range (defaults: 2020–2024)
4. Click "Run backtest"
5. See results: equity curve, metrics, allocation chart

## How It Works

**The algorithm** (plain English):

```
For each trading day from START to END:
  1. Check if your rotation rule triggers
  2. Rebalance portfolio to new sector allocation
  3. Calculate daily return based on sector prices
  4. Track vs S&P 500 (buy-and-hold)
  
After all days:
  5. Compute Sharpe ratio, max drawdown, win rate
  6. Visualize equity curves and allocation shifts
```

**Sector ETFs tracked:**
- **XLV** (Health Care) — defensive
- **XLE** (Energy) — cyclical
- **XLK** (Tech) — growth
- **XLF** (Financials) — cyclical
- **XLY** (Consumer Disc) — growth
- **XLRE** (Real Estate) — defensive
- **XLP** (Consumer Staples) — defensive

**Current rotation rules:**
- **VIX Threshold**: Switch to defensive (XLV, XLP) when volatility spiked. Stay growth-heavy when calm.
- **Unemployment Rate**: Rotate to growth when employment strong (< 4.0%). Defensive when weak (> 4.5%).
- **Equal Weight**: Baseline — hold all sectors equally (control comparison)

## Investment Research

This project is framed as a research question, not just a tool. The distinction matters: a tool answers "how do I build this?" Research answers "is this claim true, and how do I know?"

### Research Question

**Does volatility-based sector rotation improve risk-adjusted returns relative to a passive S&P 500 benchmark?**

Sub-questions this framework is built to investigate:
- Does a VIX threshold provide a useful signal for reallocating between defensive and growth sectors?
- Does the strategy outperform passive S&P 500 exposure after transaction costs?
- Does the relationship survive out-of-sample testing, or is it an artifact of the period tested?
- How sensitive are results to the chosen VIX threshold?
- Does the strategy perform consistently across different market regimes?

### Hypothesis

**H1**: Rotating into defensive sectors (XLV, XLP) when VIX exceeds a threshold reduces drawdown without materially sacrificing return, producing a higher Sharpe ratio than buy-and-hold.

**H0 (null)**: VIX-triggered rotation provides no meaningful improvement in risk-adjusted return over passive holding — any apparent outperformance is attributable to the specific period or threshold tested, not a real signal.

### Methodology

1. Define the rotation rule (e.g. VIX > threshold → shift toward XLV/XLP; below threshold → shift toward XLK/XLY)
2. Backtest daily over a specified window, rebalancing whenever the trigger condition changes
3. Compute total return, Sharpe ratio, max drawdown, and win rate against an SPY buy-and-hold benchmark over the identical period
4. Compare strategy vs. benchmark before drawing any conclusion about whether the rule adds value

### Results (Current — Simulated Data)

| Metric | Strategy | S&P 500 |
|--------|----------|---------|
| **Total Return** | +87.5% | +62.3% |
| **Sharpe Ratio** | 1.21 | 0.89 |
| **Max Drawdown** | -18.2% | -25.4% |
| **Win Rate** | 52.3% | — |

These numbers are from simulated price data — read them as a proof of methodology, not a validated edge. They confirm the pipeline calculates correctly; they don't yet confirm the hypothesis.

### Robustness Testing

A single backtest, on one period, at one threshold, proves very little — it's trivially easy to curve-fit a rule to one dataset. The tests below are what would separate a real signal from a lucky backtest. They're the next phase of this project, unlocked once real data is in:

| Test | What it checks | Values to test |
|------|----------------|-----------------|
| **Threshold sensitivity** | Does the effect hold near the chosen cutoff, or only at one hand-picked value? | VIX > 20, 25, 30, 35 |
| **Holding period sensitivity** | Does the edge survive different rebalancing frequencies? | Daily, weekly, monthly |
| **Regime testing** | Does it work everywhere, or only in one period (e.g. only during 2020's crash)? | Bull, bear, high-vol, low-vol |
| **Out-of-sample validation** | Does a rule tuned on one window hold on an untouched later window? | Fit 2015–2020, test 2021–2024 |
| **Transaction costs** | Does the edge survive realistic slippage and commissions? | 0.05%–0.15% per trade |

If a result only shows up at one threshold, one holding period, and one regime, it's very likely noise — not a strategy.

### Limitations

- Results above use simulated price data, not verified historical prices (see [Real Market Data](#real-market-data--priority-upgrade))
- No transaction costs currently modeled — this biases results in the strategy's favor
- Rebalancing is instantaneous and assumes unlimited liquidity
- No statistical significance testing (e.g. bootstrapped confidence intervals) has been applied to the return differential yet
- Regime-dependence hasn't been tested — see Robustness Testing above

### Conclusion (So Far)

This project isn't yet claiming "VIX-based rotation works." It's a framework built to test that claim honestly, including the ways it might fail. The mechanism and metrics are built and validated on simulated data. The actual test — real prices, threshold sweeps, regime splits, out-of-sample validation — is the next phase.

## Metrics (What They Mean)

| Metric | What It Is | Why It Matters |
|--------|-----------|----------------|
| **Total Return** | % gain from start to end | Absolute performance |
| **Sharpe Ratio** | Return per unit of risk | Most important. >1.0 = good. >2.0 = excellent. |
| **Max Drawdown** | Worst peak-to-trough loss | How painful is the downside? |
| **Win Rate** | % of days you beat S&P 500 | Consistency — how often are you right? |

**What winning looks like:**
- Higher total return AND lower drawdown = outperformance
- Higher Sharpe = better returns for the risk taken
- Win rate >50% = beating benchmark on most days

## Real Market Data — Priority Upgrade

**This is the single biggest thing that upgrades this project from prototype to research.**

Right now it runs on realistic *simulated* data, which is enough to validate the mechanics — the rebalancing logic, the metrics, the charts all work. But simulated data can't answer the actual research question ("does this rule work?") because there's nothing real to be right or wrong about.

The full pipeline this project is built toward:

```
Yahoo Finance / FRED → cleaning → signals → backtest → transaction costs → risk metrics → benchmark → visualisation
```

To pull real historical data (sector ETFs, VIX, unemployment):

```bash
pip install -r requirements.txt
python data/fetch_real_data.py
```

Takes ~2 minutes. Full walkthrough in **SETUP_REAL_DATA.md**.

Once real data is in, the [Robustness Testing](#robustness-testing) section below becomes runnable — that's what turns this into a genuine research project rather than a framework.

## What's Included

- **index.html** — Complete app (React + Chart.js). Open and go.
- **README.md** — You're reading it
- **LICENSE** — MIT (open source)
- **SETUP_REAL_DATA.md** — Guide to use real market data (yfinance)
- **CONTRIBUTING.md** — How to extend with custom rules

## Why This Project

**For interviews:**
- "Show me how you validate investment ideas" → Here's my framework
- "How do you think about risk?" → Sharpe ratio, drawdown, not just returns
- "Can you build analytical tools?" → Full pipeline: logic → calculation → visualization

**For roles:**
- Investment analyst: "I empirically test strategies before recommendation"
- Quant finance: "I build backtesting systems"
- Data/analytics: "I turn strategy ideas into measurable validation"
- MBA admissions: "This signals quantitative rigor"

**For yourself:**
- Test if YOUR rotation ideas actually work
- Learn how professionals validate strategies
- Build a reference for future quant work

## Extend It

**Add your own rotation rule:**

Edit `index.html` in the `calculateReturns()` function:

```javascript
if (rotationRule.type === 'my_custom_rule') {
  if (myCondition) {
    allocation = { 'SPY': 0.1, 'Defensive': 0.7, 'Growth': 0.2, 'Other': 0 };
  } else {
    allocation = { 'SPY': 0.1, 'Defensive': 0.2, 'Growth': 0.7, 'Other': 0 };
  }
}
```

Re-run and see how your idea performs historically.

## Roadmap (Ideas)

1. **Real market data integration** ← do this first, unlocks everything below
2. **Robustness testing** — threshold sweeps, holding-period sensitivity, regime splits (see Investment Research)
3. **Walk-forward validation** — out-of-sample testing
4. **Transaction cost modeling** — slippage, commissions
5. More metrics (Sortino ratio, Calmar ratio, information ratio)
6. Custom rotation rule builder (GUI instead of code editing)
7. Multi-asset rotation (bonds, commodities)
8. Export results to CSV/PDF

## Performance & Constraints

- **Speed**: 5 years of daily data → results in <100ms (no waiting)
- **Browser**: Works in Chrome, Firefox, Safari, Edge
- **Data**: Currently simulated (realistic). Real data available via yfinance.
- **Assumptions**: Instant rebalancing, no slippage, no commission — see [Limitations](#limitations) above for the full list

Survivorship bias note: doesn't currently account for delisted companies, though impact is minimal for the sector ETFs used (2020+).

## Contributing

See **CONTRIBUTING.md** for contribution guidelines, architecture, and how to add features.

## License

MIT License — use it, modify it, share it.

## Disclaimer

**Educational tool only.** Not investment advice. Backtest results don't guarantee future performance. Always consult a financial advisor.

---

**Questions?** Open an issue. **Improvements?** PRs welcome.

Built for rigorous portfolio analysis. 📊
