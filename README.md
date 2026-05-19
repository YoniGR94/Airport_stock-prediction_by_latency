# Airport Stock Prediction by Latency

**Can operational performance predict airline stock value?**  
A panel data study of U.S. domestic carriers, 2003–2025.

---

## Overview

This project investigates whether flight delay metrics — a direct, objective measure of service quality — can predict monthly stock returns for publicly traded U.S. airlines.

While prior research has examined the link between customer satisfaction and firm value using broad survey-based indices (primarily the American Customer Satisfaction Index, ACSI), no previous study directly tested a measurable operational signal against stock returns within a single industry over a long panel. This project fills that gap.

The analysis covers eight major U.S. carriers over more than two decades, combining monthly delay records from the Bureau of Transportation Statistics (BTS) with monthly stock price data from Alpha Vantage.


*The full project can be seen here:*

[Project as HTML](https://yonigr94.github.io/Airport_stock-prediction_by_latency/airport-stock-prediction-markdown.html)

---

## Key Finding

The preferred model (Model 5, FE with carrier, month, and year fixed effects) finds:

> **Each additional minute of average flight delay is associated with approximately a 0.27% decrease in the carrier's stock value within the same month** (statistically significant at p < 0.003).

This result is consistent in sign and direction across all six model specifications, supporting robustness of the central finding. The effect size is modest relative to broader market forces, which is expected — a single operational metric is unlikely to explain a large share of short-term return variance.

<img src="https://github.com/YoniGR94/Airport_stock-prediction_by_latency/blob/main/graphs/final_5_FE_table.png?raw=true" width="170"/>


Notable auxiliary findings:

most of latency time is due to carrier or airport delays

<img src="https://github.com/YoniGR94/Airport_stock-prediction_by_latency/blob/main/graphs/Delay_Composition_by_Airline.png?raw=true" width="700"/>


it is hard to visually see a pattern in stock grow using time factor only

<img src="https://github.com/YoniGR94/Airport_stock-prediction_by_latency/blob/main/graphs/Grow_Month_Year.png?raw=true" width="700"/>

---


## Research Question

> Is there a statistically significant relationship between average flight delay minutes and monthly stock returns for U.S. airline carriers?

---

## Data Sources

| Source | Content | Period |
|---|---|---|
| [BTS OT Delay](https://www.transtats.bts.gov/OT_Delay/OT_DelayCause1.asp) | Monthly delay data by carrier and airport | 2003–2025 |
| [Alpha Vantage API](https://www.alphavantage.co/documentation/) | Monthly stock prices (open, close, high, low, volume) | 2002–2025 |
| [FRED – WJFUELUSGULF](https://fred.stlouisfed.org/series/WJFUELUSGULF) | U.S. Gulf Coast jet fuel prices | 2003–2025 |

---

## Airlines Covered

| Ticker | Carrier |
|---|---|
| AAL | American Airlines |
| DAL | Delta Air Lines |
| UAL | United Airlines |
| ALK | Alaska Airlines |
| JBLU | JetBlue Airways |
| LUV | Southwest Airlines |
| HA | Hawaiian Airlines |
| ULCC | Frontier Airlines |

Spirit Airlines was excluded due to its bankruptcy filings in 2023 and 2025,
which introduce confounding effects unrelated to service quality.

---

## Methodology

The dependent variable is the log monthly stock return: `log(close / open)`.

The primary explanatory variable is `av_delay` — average delay minutes per flight for each carrier-month.

To isolate the effect of delays from confounding factors, all models use **Fixed Effects (FE) panel regression** via the `fixest` package in R, controlling for:

- **Carrier identity** — absorbs time-invariant firm characteristics (size, route network, business model)
- **Month of year** — controls for seasonality in both delays and market activity
- **Year** (in the preferred model) — controls for macroeconomic shocks and industry-wide events

Six model specifications were estimated and compared using AIC:

| Model | Specification |
|---|---|
| 1 | OLS: `log(grow) ~ av_delay + month + carrier` |
| 2 | FE: `log(grow) ~ av_delay \| carrier + month` |
| 3 | FE + fuel: `log(grow) ~ av_delay + fuel_change \| carrier + month` |
| 4 | FE on-time rate: `log(grow) ~ on_time_pct \| carrier + month` |
| **5** | **FE + year (preferred): `log(grow) ~ av_delay \| carrier + month + year`** |
| 6 | FE with lags: `log(grow) ~ av_delay + lag1 + lag2 + lag3 \| carrier + month` |

Heteroskedasticity was detected via Breusch-Pagan test (BP = 63.01, p < 0.001). All models use heteroskedasticity-robust standard errors clustered at the carrier level. VIF values were all below 2, ruling out multicollinearity concerns.


## Repository Structure

```
.
├── data/
│   ├── Airline_Delay_Cause_26.csv    # BTS delay data
│   ├── delay_data.csv                # Processed delay panel
│   ├── DAL_month.csv                 # Monthly stock data per carrier
│   ├── UAL_month.csv
│   ├── ... (one file per carrier)
│   ├── jet_fuel_data.csv             # FRED jet fuel prices
│   └── av_delay_df_ex.csv            # Sample of merged dataset
├── graphs/                           # some of the graphs photos
├── airport_stock_prediction_markdown.Rmd   # Full analysis notebook
└── README.md
```

> **Note:** Raw stock data files are not included in this repository. To reproduce the analysis, request a free API key from [Alpha Vantage](https://www.alphavantage.co/) and uncomment the API call blocks in the `.Rmd` file.

---


## Limitations

- **Monthly resolution:** Stock trading and investor reactions occur at daily frequency.
- **Carrier expectations:** Investors and passengers may hold different delay tolerance thresholds for different carriers.
- **Flight-level vs. passenger-level delays:** The dataset weights all flights equally regardless of aircraft size.
- **Bigger reasons:** some factors might effect the stock more than the latency: price, safety, geographic coverage.
- **Sample size:** number of carriers is 8, lower than usually used in FE models.
---

## Broader Implications

The findings are relevant beyond aviation. Any industry where service quality can be measured by an objective,
publicly available timeliness metric — logistics and parcel delivery, public transit operators, hospital emergency waiting times 
— may exhibit a similar relationship between operational performance and market valuation.

---

## Author

**Yonatan Getahon**  
M.Sc. Engineering and Management, Business Analytics  
