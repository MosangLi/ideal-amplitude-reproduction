# Ideal Amplitude Factor Reproduction

> A reproduction of the Ideal Amplitude Factor introduced in the Kaiyuan Securities research report *The Hidden Structure of the Amplitude Factor — Market Microstructure Research Series (7)*.

![Status](https://img.shields.io/badge/status-completed-brightgreen)
![Market](https://img.shields.io/badge/market-China%20A--Share-red)
![Platform](https://img.shields.io/badge/platform-BigQuant-orange)
![Language](https://img.shields.io/badge/language-Python-blue)
![Period](https://img.shields.io/badge/period-2019--2024-lightgrey)

## Overview

The conventional amplitude factor has negative stock-selection ability in the China A-share market, but its performance is not always stable.

The original research decomposes the conventional amplitude factor into two components based on the relative price state of each stock:

- High-price amplitude: `V_high`
- Low-price amplitude: `V_low`

The Ideal Amplitude Factor is constructed by standardizing these two components cross-sectionally and calculating their difference:

```text
IdealAmplitude = ZScore(V_high) - ZScore(V_low)
```

This project evaluates the factor using monthly IC, RankIC, quintile portfolios, a long-short portfolio and lookback-window robustness tests.

## Research Hypothesis

The central hypothesis is:

> Price amplitude observed during a stock's relatively high-price state has stronger negative predictive power than amplitude observed during its relatively low-price state.

The Ideal Amplitude Factor is therefore expected to be a negative factor:

```text
Lower factor value  → Higher expected subsequent return
Higher factor value → Lower expected subsequent return
```

The portfolio construction follows this direction:

```text
Long portfolio:  Group 1 — stocks with the lowest factor values
Short portfolio: Group 5 — stocks with the highest factor values
```

## Factor Construction

The factor is calculated for every stock at each month-end signal date.

### 1. Daily Amplitude

Daily price amplitude is defined as:

```text
Amplitude_t = High_t / Low_t - 1
```

### 2. Price-State Classification

For each stock, the most recent `N` trading days are sorted by daily closing price.

Under the baseline parameters `N=20` and `λ=25%`:

```text
Low-price state  = 5 days with the lowest closing prices
High-price state = 5 days with the highest closing prices
```

### 3. Conditional Amplitude

```text
V_low  = Mean daily amplitude during the low-price state
V_high = Mean daily amplitude during the high-price state
```

### 4. Cross-Sectional Processing

At each signal date, `V_high` and `V_low` are processed separately across the stock universe:

1. Winsorization at the 1st and 99th percentiles
2. Z-score standardization

```text
Z_high = ZScore(Winsorize(V_high))
Z_low  = ZScore(Winsorize(V_low))
```

### 5. Ideal Amplitude Factor

```text
IdealAmplitude = Z_high - Z_low
```

## Baseline Configuration

| Parameter | Value |
|---|---:|
| Market | China A-share |
| Research period | 2019–2024 |
| Data frequency | Daily |
| Factor frequency | Monthly |
| Lookback window `N` | 20 trading days |
| Split ratio `λ` | 25% |
| Price-state field | Close |
| Daily amplitude | `high / low - 1` |
| Number of portfolio groups | 5 |
| Factor direction | Negative |
| Portfolio weighting | Equal weight |
| Transaction costs | Not included |

## Reproduction Results

### IC Analysis

The factor exhibits a persistent negative relationship with subsequent monthly stock returns.

| Metric | Original Report | Reproduction |
|---|---:|---:|
| IC Mean | -0.067 | -0.0364 |
| RankIC Mean | Not reported | -0.0517 |
| Annualized RankICIR | Not reported | -2.16 |
| Negative RankIC Month Ratio | 84.2% | 76.06% |

The negative RankIC indicates that stocks with lower Ideal Amplitude Factor values tend to generate higher subsequent returns.

### Quintile Portfolio Performance

| Portfolio | Average Monthly Return | Annualized Return |
|---|---:|---:|
| Group 1 | 1.50% | 15.83% |
| Group 2 | 1.61% | 17.87% |
| Group 3 | 1.28% | 13.69% |
| Group 4 | 1.09% | 11.16% |
| Group 5 | 0.45% | 2.21% |
| Group 1 − Group 5 | 1.05% | 12.88% |

Three of the four adjacent group relationships satisfy a decreasing-return relationship.

Group 2 outperforms Group 1, so the quintile returns are not strictly monotonic. Nevertheless, the lower-factor portfolios substantially outperform the highest-factor portfolio.

### Long-Short Portfolio

The long-short portfolio is defined as:

```text
Long-Short Return = Group 1 Return - Group 5 Return
```

| Metric | Result |
|---|---:|
| Cumulative return | 104.79% |
| Annualized return | 12.88% |
| Annualized volatility | 9.95% |
| Return-to-risk ratio | 1.29 |
| Maximum drawdown | -11.23% |
| Monthly win rate | 77.46% |

The long-short portfolio remains profitable over the full research period, although its performance weakens during parts of 2024.

## Robustness Test

The factor was evaluated using four lookback windows while keeping the price-state split ratio fixed at `λ=25%`.

| Lookback `N` | Mean RankIC | Annualized ICIR | Negative Month Ratio | Long-Short Annual Return | Long-Short Win Rate |
|---:|---:|---:|---:|---:|---:|
| 10 | -0.0414 | -1.86 | 67.61% | 11.22% | 69.01% |
| 20 | -0.0517 | -2.16 | 76.06% | 12.88% | 77.46% |
| 40 | -0.0461 | -1.71 | 69.01% | 10.20% | 67.61% |
| 60 | -0.0457 | -1.62 | 64.79% | 9.53% | 64.79% |

All tested lookback windows produce:

- Negative mean RankIC
- Negative annualized ICIR
- More than 50% negative-RankIC months
- Positive long-short annual returns

These results indicate that the factor's effectiveness is not restricted to one specific lookback window.

## Conclusion

The main conclusion of the original research report is successfully reproduced at a baseline level.

During the 2019–2024 period, the Ideal Amplitude Factor demonstrates:

- Persistent negative stock-selection ability
- Negative mean RankIC across multiple lookback windows
- Broadly decreasing quintile portfolio returns
- Positive long-short portfolio performance
- Reasonable robustness to the lookback-window parameter

The reproduced long-short annual return of `12.88%` is lower than the `23.3%` reported in the original research.

Potential sources of this difference include:

- Different research periods
- Different market-data sources
- Different price-adjustment methods
- Different stock-universe filters
- Different return and trading assumptions
- Different outlier-treatment methods
- Different neutralization procedures

The project should therefore be classified as a successful baseline reproduction rather than an exact numerical replication.

## Current Limitations

The current implementation does not include:

- ST and *ST stock exclusion
- Newly listed stock exclusion
- Suspension filtering
- Limit-up and limit-down trading restrictions
- Industry neutralization
- Market-cap neutralization
- Transaction costs
- Slippage
- Liquidity and capacity constraints

The reported results represent a research-level factor evaluation and should not be interpreted as directly tradable performance.

## Repository Structure

```text
ideal-amplitude-reproduction/
├── README.md
├── notebooks/
│   └── ideal_amplitude_reproduction.ipynb
└── outputs/
    ├── monthly_ic.csv
    ├── monthly_group_returns.csv
    ├── performance_summary.csv
    └── lookback_robustness.csv
```

Large stock-level datasets and raw market data are intentionally excluded from the repository.

## Notebook

The complete research Notebook, including factor construction, evaluation results, charts and robustness tests, is available on BigQuant:

- [View the BigQuant Notebook](https://bigquant.com/codesharev3/efbc81de-d742-4849-a24f-396b898fef2a)

A BigQuant account is required to view and clone the source code. Summary results and tables can be viewed without signing in.

## Data Policy

This repository does not contain:

- Raw or licensed market data
- Company-internal data or documents
- Slack messages or internal task information
- Account credentials, API keys or access tokens
- Live-trading configurations
- Unauthorized copies of the original research report

Only research code, documentation and aggregated evaluation results are included.

## Reference

Kaiyuan Securities, *The Hidden Structure of the Amplitude Factor — Market Microstructure Research Series (7)*, May 16, 2020.

- [Original research page on BigQuant](https://bigquant.com/wiki/doc/w5WH1P01Bl)

## Disclaimer

This project is intended solely for research, education and report-reproduction purposes. It does not constitute investment advice.

Historical backtest performance does not guarantee future results. The findings may be affected by data quality, sample selection, reproduction assumptions and trading rules.
