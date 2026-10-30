# BoC Rate Decision Model

Machine learning model that predicts the next Bank of Canada policy rate decision (**cut / hold / hike**) as probabilities.

Open `boc_rate_decision_ml.ipynb` in Google Colab and run all cells. No API keys needed. CPU runtime is enough.

## What it does

1. **Downloads data automatically**
   - Bank of Canada (Valet API): policy rate, 3-month T-bill, 2y / 10y / long Government of Canada yields, CPI, CPI-trim, CPI-median
   - FRED: Canadian unemployment, Fed funds rate, US 2y yield, WTI oil, USD/CAD
   - Yahoo Finance: ZFL, Canadian sector ETFs, US sector ETFs (in CAD), XIU (TSX 60)
2. **Builds the decision calendar** from the BoC's fixed announcement dates since 2001, plus any unscheduled moves, and labels each meeting as cut, hold or hike.
3. **Engineers features as of the day before each meeting**, with publication lags on macro data so there is no look-ahead.
4. **Tunes and tests five approaches**: logistic regression, XGBoost, MLP, LSTM and an ensemble.
5. **Evaluates honestly** with a walk-forward test: the models are refit before every meeting and predict only that meeting.
6. **Predicts the next meeting** and can re-test any past meeting out-of-sample.

## Features

| Block | Inputs |
|---|---|
| Market pricing | 3m T-bill minus policy rate, 2y minus policy rate, 10y-2y slope, recent changes in the 2y yield |
| Global | Fed funds vs BoC, Canada-US 2y spread, oil, USD/CAD |
| Inflation and labour | CPI, CPI-trim, CPI-median, core gap to the 2% target, unemployment and its change |
| Policy state | Current rate, last decision, recent direction, meetings since last change |
| Assets | ZFL returns and volatility, sector ETF returns relative to the TSX 60 |

## ETFs used

| Sector | US sector in CAD (rebuilt) | Canadian native |
|---|---|---|
| Utilities | ZXLU (XLU x USD/CAD) | XUT, ZUT |
| Health Care | ZXLV (XLV) | ZUH |
| Consumer Discretionary | ZXLY (XLY) | XCD |
| Communication Services | ZXLC (XLC) | ZMT |
| Industrials | ZXLI (XLI) | ZDJ |
| Materials | ZXLB (XLB) | XMA |
| Financials | ZXLF (XLF) | XFN, ZEB |
| Energy | ZXLE (XLE) | XEG |
| Real Estate | ZXLR (VNQ) | XRE |
| Technology | ZXLK (XLK) | XIT |
| Consumer Staples | ZXLP (XLP) | XST |

Two data choices worth knowing:

- **ZFL** started in 2010. Before that, its return is estimated from the change in the long Canadian bond yield (about 15 years of duration), then spliced with the real ETF.
- **BMO ZXL\* ETFs** have short histories, so they are rebuilt as the US sector ETF times USD/CAD, which is what they hold and gives 20+ years of data.

## Evaluation

| Period | Use |
|---|---|
| 2002 to 2011 | Hyperparameter tuning |
| 2012 to 2018 | Validation, used to choose the model |
| 2019 onward | Holdout, never used for choosing; this is the score to trust |

The main metric is log loss, which rewards well-calibrated probabilities. Results are compared against three baselines: always predicting hold, repeating the last decision, and a **market-only** model that uses just the T-bill and 2y yield spreads. Bond markets usually price the BoC well, so the market-only model is the one to beat.

## Usage

- **Next meeting**: section 13 prints cut / hold / hike probabilities for the selected model, the best full-feature model and the market-only model.
- **Any past meeting**: in section 14, set `MEETING = "2026-10-28"` (or any decision date) and rerun that cell. Models are trained only on earlier meetings.
- **New year**: add the BoC's published announcement dates to the `FAD` dictionary in section 2.

## Notes

- About 200 decisions in total, so the sample is small. Read the output as probabilities, not certainties.
- If a data source changes or is down, the download step prints which series are missing.
- For research and education only. Not investment advice.
