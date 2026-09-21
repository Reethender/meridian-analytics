# Industrial Sales & Operations Analytics — Meridian Industrial Supplies Ltd.

An end-to-end data analytics portfolio project replicating the analytical function of a UK-based B2B wholesale distributor: from raw data procurement through dashboard delivery, demand forecasting, and price benchmarking against UK government indices.

---

## Business Context

Meridian Industrial Supplies Ltd. is a fictional UK-based B2B wholesale distributor of industrial hardware, MRO (maintenance, repair and operations) supplies, warehouse equipment, safety products, and packaging materials. The company operates across five UK regions (North, Midlands, South East, South West, London) serving Trade, Retail, Government, and Contractor customer segments.

This project builds Meridian's complete analytics function: sales performance monitoring, inventory health management, lead pipeline tracking, campaign attribution, competitor price benchmarking against ONS indices, and 13-week demand forecasting using Holt-Winters exponential smoothing.

---

## Data Sources

| Dataset | Source | Date Range | Purpose |
|---------|--------|------------|---------|
| FMCG Daily Sales Data | [Kaggle](https://www.kaggle.com/datasets/beatafaron/fmcg-daily-sales-data-to-2022-2024) | 2022–2024 | Primary transaction dataset (relabelled to Meridian context) |
| ONS Retail Sales Index | [ONS](https://www.ons.gov.uk/businessindustryandtrade/retailindustry/datasets/retailsalesindexreferencetables) | 2022–2024 | Market benchmark for sales performance (Page 1 dual-axis chart) |
| ONS Producer Price Index | [ONS](https://www.ons.gov.uk/economy/inflationandpriceindices/datasets/producerpriceindex) | 2022–2024 | Price benchmark for competitive positioning (Page 5) |
| Logistics Warehouse Dataset | [Kaggle](https://www.kaggle.com/datasets/ziya07/logistics-warehouse-dataset) | Point-in-time | Inventory health analysis (overstock/understock flagging) |
| CRM Lead Pipeline | Mockaroo (synthetic) | 2022–2024 | Lead pipeline and conversion funnel analysis |
| Marketing Campaigns | Mockaroo (synthetic) | 2022–2024 | Campaign performance and channel ROI analysis |

---

## Methodology

### Data Preparation (Python / pandas)
- Systematic audit of all source datasets before cleaning (null checks, duplicate detection, data type validation)
- Transaction data relabelled from FMCG context to UK industrial distributor context with documented mapping
- UK regions and customer types assigned with weighted probabilities using fixed random seed (reproducible)
- ONS RSI extracted from multi-tab Excel workbook (Table 1 M — monthly, seasonally adjusted, chained volume)
- ONS PPI extracted from 602-column dataset; 4 Output Domestic PPI categories selected and mapped to Meridian product lines using SIC codes
- Full data dictionary documenting every table, column, data type, source, and transformation

### Dashboard (Power BI)
- Star schema data model: transactions as central fact table, ONS tables joined on year-month key
- 6-page interactive dashboard with cross-page slicers (Year, Quarter, Region, Customer Type, Product Line)
- Dual-axis benchmarking charts comparing Meridian performance against ONS RSI and PPI indices
- Inventory health traffic light system with conditional formatting (red/amber/green)
- Campaign ROI and Cost Per Lead analysis by marketing channel

### Demand Forecasting (Python / statsmodels)
- Weekly aggregation of transaction data (~155 data points per product line)
- Holt-Winters Exponential Smoothing with additive trend and additive seasonality (52-week period)
- 26-week holdout test set for model evaluation
- 13-week forward forecast per product line
- MAPE calculated and documented for all 5 product lines
- Fallback to trend-only model where insufficient data for seasonal estimation

---

## Key Findings

- **Revenue Growth vs Market:** Meridian revenue grew significantly from 2022 to mid-2023 while the ONS RSI remained relatively flat (100–107 range), suggesting market share gain rather than market-driven growth
- **Customer Mix:** Trade accounts represent 44.8% of revenue, consistent with UK B2B industrial distribution norms. Contractor segment at 10% represents an expansion opportunity
- **Inventory Health:** 2,801 of 3,204 SKUs (87%) flagged as understock, with £3.02M in estimated overstock waste value. This signals a systemic procurement planning gap
- **Forecast Accuracy:** MRO Supplies (16.6% MAPE) and Industrial Hardware (15.8% MAPE) show reliable forecast accuracy. Warehouse Equipment (43.8%) and Safety Products (65.6%) require longer data history or alternative modelling approaches
- **Campaign Performance:** PPC shows highest ROI by channel; Exhibition shows highest cost per lead. Trade Press delivers lowest cost per lead

---

## Dashboard

**[Live Dashboard Link]** — *(To be added after publishing to Power BI Service / Tableau Public)*

### Pages:
1. **Sales Overview** — KPI cards, revenue by product line/region/customer type, dual-axis Meridian vs ONS RSI benchmark
2. **Inventory Health** — Traffic light SKU table, overstock/understock KPIs, stock value treemap, procurement risk indicators
3. **Lead Pipeline** — Conversion funnel, win rate by lead source, pipeline value by region, win rate over time
4. **Campaign Performance** — ROI by channel, cost per lead, spend vs leads analysis, monthly spend trend
5. **Price Benchmarking** — Meridian unit price vs ONS PPI dual-axis chart, average price by product line
6. **Demand Forecast** — 13-week forecast by product line, MAPE accuracy table, procurement recommendation

---

## Folder Structure

```
meridian-analytics/
├── 00_raw_data/                  # All downloaded files, never edited
│   ├── FMCG Daily Sales Data/
│   ├── Logistics Warehouse Dataset (supplement)/
│   ├── ONS Producer Price Index/
│   ├── ONS Retail Sales Index/
│   ├── Synthetic/
│   │   ├── CRM Lead Pipeline.csv
│   │   └── Marketing Campaign Performance.csv
│   └── Warehouse Inventory Dataset/
├── 01_cleaned_data/              # Outputs from Python cleaning scripts
│   ├── transactions_clean.csv
│   ├── ons_rsi_clean.csv
│   ├── ons_ppi_clean.csv
│   ├── inventory_clean.csv
│   ├── forecasts_output.csv
│   └── forecast_mape.csv
├── 02_notebooks/                 # Jupyter notebooks
│   ├── 01_data_audit.ipynb
│   ├── 02_data_cleaning.ipynb
│   └── 03_demand_forecasting.ipynb
├── 03_dashboard/                 # Power BI / Tableau files
│   └── Meridian_VS.pbix
├── 04_outputs/                   # Charts and exports
│   └── forecast_all_product_lines.png
├── 05_docs/                      # Documentation
│   └── data_dictionary.md
├── README.md
└── requirements.txt
```

---

## How to Run

### Prerequisites
- Python 3.10+
- Power BI Desktop (free) for dashboard viewing

### Setup
```bash
git clone https://github.com/YOUR_USERNAME/meridian-analytics.git
cd meridian-analytics
pip install -r requirements.txt
```

### Notebook Run Order
1. `02_notebooks/01_data_audit.ipynb` — Data audit and exploration
2. `02_notebooks/02_data_cleaning.ipynb` — Cleaning, transformation, and ONS extraction
3. `02_notebooks/03_demand_forecasting.ipynb` — Holt-Winters model and MAPE evaluation

### Dashboard
Open `03_dashboard/Meridian_VS.pbix` in Power BI Desktop. All data sources are relative paths to `01_cleaned_data/`.

---

## Skills Demonstrated

| Skill | Application |
|-------|------------|
| Python (pandas, numpy, statsmodels) | Data cleaning, feature engineering, time series forecasting |
| Power BI | 6-page interactive dashboard, DAX measures, data modelling, conditional formatting |
| Statistics | Holt-Winters ETS, MAPE evaluation, train/test split methodology |
| Data Modelling | Star schema, many-to-one relationships, calculated columns |
| UK Government Data | ONS RSI and PPI extraction, SIC code mapping, index interpretation |
| Documentation | Data dictionary, README, methodology documentation |
| Domain Knowledge | S&OP, inventory management (days of supply, overstock/understock), B2B sales analytics |

---

## Limitations

1. **Synthetic CRM and marketing data** — Lead pipeline and campaign tables are Mockaroo-generated. Distributions are configurable but do not reflect real conversion rates
2. **Region and customer type are modelled** — Assigned programmatically with fixed random seed; in production these would come from CRM/ERP master data
3. **PPI category mapping is approximate** — ONS SIC codes do not map precisely to fictional product lines; mapping rationale documented in data dictionary
4. **No real-time data refresh** — Dashboard uses static CSVs; production implementation would use DirectQuery or scheduled pipeline
5. **Safety Products forecast accuracy is low (65.6% MAPE)** — Insufficient data history for seasonal estimation; documented as requiring longer collection period

---

## Author

**Reethender Reddy Vedira**

---

*Project completed April 2026*
