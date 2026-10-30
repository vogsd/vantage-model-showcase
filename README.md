# Vantage Model — ML Stock Ranking for Indian Equities (NSE)

An end-to-end machine-learning system that ranks ~600 liquid NSE stocks every week, holds the top 5–10, and steps aside to cash when the market turns. Built, validated and deployed solo, from raw exchange data to an automated daily pipeline and dashboard.

> **Code is private.** This repository shows the problem, approach, validation and results. Happy to walk through the system live in an interview.

![Vantage Model vs Nifty 50 and Nifty Midcap 150, March to October 2026](images/results_mar_oct_2026.png)

---

## Results at a glance

Walk-forward backtest with 0.2% cost per side. Models are retrained every month using **only data available at that time**.

| Period | Vantage, 5 stocks | Vantage, 10 stocks | Nifty 50 |
|---|---|---|---|
| **2011–16** (weak market years) | **+21.6% / yr** (worst fall −21%) | **+19.6% / yr** (−19%) | +5.0% / yr (−25%) |
| **2017–26** | **+62.5% / yr** (worst fall −18%) | **+42.6% / yr** (−18%) | +12.4% / yr (−35%) |
| **Mar–Oct 2026** | +50.1% (−11.5%) | +28.0% (−8.7%) | −9.8% (−11.2%) |

![Year-by-year returns, 2011 to 2026](images/results_by_year.png)

**Is the edge real or luck?**

| Test | Result |
|---|---|
| Picking skill (rank correlation of model score vs next-month return), 2014–26, 153 months | 0.090, t = 6.8, **p ≈ 2 × 10⁻¹⁰** |
| Same test on an independent early model, 2011–16, 72 months | 0.128, t = 5.1, **p ≈ 2 × 10⁻⁶** |
| Top-5 picks vs random picks from the same stocks (permutation test) | **p ≤ 0.001** |
| Monthly excess return over Nifty, 2017–26 | t = 5.2, **p < 10⁻⁶** |

The system went live in October 2026. Everything before that is simulated.

---

## How it works

![System architecture](images/architecture.png)

1. **Data.** Daily NSE price and volume files for every stock traded at the time, so delisted stocks are included and there is no survivorship bias. Splits and bonuses are adjusted. A liquidity filter keeps ~600 tradable stocks.
2. **Features.** 46 per stock covering trend, volatility and liquidity, plus market-regime context (breadth, broad-market trend, dispersion), each as a raw value and a cross-sectional rank.
3. **Models.** An ensemble of 10 gradient-boosting models with varied settings predicts each stock's relative rank over the next month. Their percentile ranks are averaged, and the number of models that agree is tracked. A separate probability model estimates the chance of a strong one-month move and is used for re-entry timing.
4. **Portfolio rules.** Weekly: buy the top-ranked stocks with equal weights, hold while they stay in a rank band, protect large gains, and check liquidity before buying. Crash protection moves to cash when the broad market's trend and breadth break down, and fast re-entry signals bring it back.
5. **Output.** A daily automated run after the market close produces the next-day order list and a self-updating dashboard.

**Stack:** Python · pandas · NumPy · scikit-learn (gradient boosting) · Matplotlib · HTML/SVG dashboard · Dhan API · scheduled daily automation.

---

## Validation: how overfitting was kept out

- **Walk-forward only.** Every month the models are retrained on data up to that point and tested on the following month. No future data touches any decision.
- **Two decades, two regimes.** A change is accepted only if it improves both the weak 2011–16 period and the strong 2017–26 period.
- **Costs included.** 0.2% per side on every trade, and buys only in stocks with enough daily turnover.
- **Robustness.** Random mixes of 7 of the 10 ensemble models give similar results (median +59% / yr for 5 stocks, 2017–26), so the result doesn't hinge on one lucky setting.
- **Over 15 ideas tested and rejected** when they failed out of sample: alternative targets, LambdaRank, regime and HMM models, momentum neutralisation, moving-average slope features, euphoria filters, model-agreement filters and others.

---

## What the research found

- **The model wins mostly by avoiding losers, not by finding winners.** Stocks in the top half of the ranking fall 10%+ in a month about 9–10% of the time. The bottom 1% do so 29% of the time, and plain momentum picks 16–24%.
- **It is not a momentum clone.** Only 10–16% of its picks overlap a standard momentum list, and with the same treatment it beats plain momentum (+32% vs +28% / yr with no rules, 2017–26) with a much smaller worst fall (−39% vs −52%).
- **Picking skill depends on the market.** It is strongest when the market falls (0.20) or is flat (0.11), and near zero in euphoric months. In a mania everything rises together, so nothing in price data separates the leaders.
- **The rules matter as much as the model.** On the same rankings, the portfolio rules roughly double the return and halve the worst fall (top 5: +32% / −39% without rules, +62.5% / −18% with rules).

---

## Dashboard

A self-updating HTML dashboard runs after every market close. Stock names and position sizes are hidden in these screenshots.

**Performance vs benchmarks**
![Dashboard: performance](images/dashboard_performance.png)

**Model-health scorecard.** Is the model still working, and is the market giving it anything to work with?
![Dashboard: scorecard](images/dashboard_scorecard.png)

**Market health**
![Dashboard: market health](images/dashboard_market_health.png)

**Weekly ranking** (names hidden)
![Dashboard: ranking](images/dashboard_ranking.png)

---

## Demo video

▶️ **[Watch the 3-minute walkthrough](VIDEO_LINK_HERE)**

---

## About me

**Soham Dutta.** Data scientist and chemical engineer (B.E., Jadavpur University) with about six years in data science and analytics at TCS, Ipsos MMA, EY and Ingenero, now running a retail pharmacy business alongside independent quantitative research. I build end-to-end data and ML systems single-handedly using AI-assisted development.

- GitHub: [github.com/soham1993](https://github.com/soham1993)
- LinkedIn: LINKEDIN_LINK_HERE

---

*Disclaimer: results before October 2026 are simulated on historical data and are not a live track record. Index comparisons use NIFTYBEES and MID150BEES ETF prices (price only). Past or simulated performance does not guarantee future results. Nothing here is investment advice or a recommendation to buy or sell any security.*
