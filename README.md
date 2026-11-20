# Inventory & Supply Chain Analytics: Part 1

Reconciling, classifying and forecasting a full year of retail inventory data (millions of transactions) with Python and Pandas. Part 1 of a multi-part supply chain controlling and audit series.

<!-- TODO: add one chart image here, e.g. the ABC efficiency curve from the notebook (cell with the k1/k2 plot). Save it as docs/images/abc-efficiency.png -->
![ABC efficiency curve](docs/images/abc-efficiency.png)

---

## About this project

I'm a Master's student in management control and audit. This project applies controlling and audit thinking to inventory data: first prove the numbers reconcile, then classify the products, then forecast demand.

**Data:** public inventory case-study data (2016 sales, purchases, beginning and ending inventory, invoices, prices).
Sources: [PwC data & analytics case studies](https://www.pwc.com/us/en/careers/university-relations/data-and-analytics-case-studies-files.html) · [Kaggle mirror](https://www.kaggle.com/datasets/bhanupratapbiswas/inventory-analysis-case-study/data).
This is a publicly available case-study dataset. It is not client work.

---

## Roadmap

- [x] Data preparation and memory optimization
- [x] Stock reconciliation
- [x] ABC/XYZ classification
- [x] Demand forecasting method comparison 
- [x] Lead time analysis
- [ ] Reorder point modeling 
- [ ] Economic Order Quantity (EOQ)
- [ ] Carrying cost analysis
- [ ] Inventory turnover and process optimization

---

## Key findings

### 1. The inventory data reconciles
For every store-brand combination, **beginning inventory + purchases − sales = ending inventory, with zero difference.**

| | Units |
|---|---:|
| Beginning inventory | 4,219,275 |
| + Purchases | 33,584,377 |
| − Sales | 32,917,876 |
| **= Ending inventory** | **4,885,776** |

The reconciliation also holds at the brand level for each product group below (permanent, discontinued, added and same-year SKUs).

### 2. Product lifecycle
Of **11,237 SKUs sold** in the year:

| Group | SKUs |
|---|---:|
| Permanent (sold, in opening and closing stock) | 6,933 |
| Discontinued mid-year (in opening stock, not in closing) | 1,106 |
| Added mid-year (not in opening stock, in closing) | 2,665 |
| Entered and left within the year | 744 |

The 6,933 permanent SKUs generated about **$430.9M in sales** (30.8M units). Within the discontinued group, 16 SKUs vanished with zero stock, and 459 were sold without being repurchased. Within the added group, 177 were purchased but never sold.

### 3. ABC/XYZ classification
- **ABC:** instead of the fixed 80/20 rule, the A cutoff is data-driven, set at the peak of an efficiency curve. Result: **class A is the top ~8.5% of SKUs and covers ~61.6% of sales.**
- **XYZ:** based on the coefficient of variation of weekly demand (X ≤ 50%, Y 50-100%, Z > 100%), with stockout weeks excluded so they don't inflate variability.

| Tier | Share of sales |
|---|---:|
| AX | 39.3% |
| AY | 19.0% |
| AZ | 3.3% |
| BX | 12.5% |
| BY | 7.3% |
| BZ | 1.2% |
| CX | 7.6% |
| CY | 7.1% |
| CZ | 2.7% |

Demand groups used for forecasting: **4,143 X/Y SKUs, 1,567 Z SKUs and 1,223 "ghost" SKUs** (under 1 unit per week). **439 SKUs with stockout weeks** were excluded from forecasting, so that stockouts aren't mistaken for low demand.

### 4. Demand forecasting
Methods compared per SKU on weekly demand: simple moving averages (3, 4, 5, 8, 13 weeks), weighted moving averages (same windows), simple exponential smoothing with optimized alpha, Holt's linear trend and damped Holt. The best method per SKU is chosen by wMAPE and by bias.

<!-- TODO: fill with real results after re-running the fixed tournament. Only write what the output shows.
- Evaluation scheme (e.g. one-step-ahead forecasts, holdout weeks 40-52)
- Overall wMAPE and bias per method
- How many SKUs each method wins
- Takeaway in one sentence -->

### 5. Lead time
Average lead time and its variability (coefficient of variation) are computed for each brand-vendor pair and classified X (≤ 0.2), Y (≤ 1) or Z (> 1) by reliability.

<!-- TODO: add 1-2 findings, e.g. average lead time overall, share of unreliable vendor-SKU pairs. -->

---

## Memory optimization

The raw data takes **about 2.2 GB** when loaded in Pandas. Downcasting data types with a min/max strategy and storing the data as Parquet brought it down to **about 0.5 GB (roughly 77% smaller)**.

<!-- TODO: replace with exact before/after figures from df.memory_usage(deep=True).sum()
TODO: state that you verified no overflow, e.g. min/max assertions for each downcast column.
Check especially: Purchase['lead'] is int8 (range -128 to 127) -->

---

## Repository structure

```
.
├── <!-- TODO: notebook name -->.ipynb     # main analysis
├── <!-- TODO: memory optimization file -->  # converts CSV to optimized Parquet
├── CSVDATA/                                 # raw CSV files (not included, see Data setup)
└── PraquetDATA/                             # optimized Parquet files (not included)
```

## Data setup

1. Download the dataset from one of the sources above.
2. Place the CSV files in `CSVDATA/`.
3. <!-- TODO: command or script to create the Parquet files in PraquetDATA/ -->
4. Open the notebook and run all cells.

<!-- TODO: Python version and libraries, e.g. pandas, numpy, statsmodels, matplotlib, tqdm, pyarrow -->

---

## Methodology notes and limitations

- Weekly buckets: 52 weeks per year (days beyond week 52 are folded into week 52).
- Only one year of data, so seasonality can't be validated across years.
- <!-- TODO: forecast validation limits, e.g. which weeks were held out and what that means for the results -->
- <!-- TODO: assumptions behind the ABC cutoff and the CV thresholds, in your own words -->
- Exponential smoothing parameters are optimized per SKU with `statsmodels`; flat series use a fallback.

---

## Next steps
EOQ analysis, carrying cost analysis, and inventory turnover and process optimization (see the roadmap above).
