# Data Dictionary — Meridian Industrial Supplies Ltd.

**Project:** Industrial Sales & Operations Analytics  
**Last Updated:** April 2026  
**Author:** Reeth  

---

## Overview

This document describes every table and column used in the Meridian Industrial Supplies analytics project. For each column, the data type, description, source, and any transformations applied are documented.

**Data Sources:**
- **Kaggle FMCG Daily Sales Data (2022–2024):** Primary transaction dataset, relabelled to Meridian context
- **ONS Retail Sales Index:** UK government monthly retail sales volumes (market benchmark)
- **ONS Producer Price Index:** UK government monthly industrial goods price indices (price benchmark)
- **Mockaroo Synthetic Data:** CRM lead pipeline and marketing campaign tables (generated to mirror proprietary schemas)
- **Kaggle Warehouse Inventory Dataset:** SKU-level stock data for inventory health analysis

---

## Table 1: transactions_clean.csv

**Source:** Kaggle FMCG Daily Sales Data 2022–2024  
**Rows:** ~186,000 (after cleaning)  
**Purpose:** Central fact table for all sales analysis, dashboard KPIs, and demand forecasting  
**Cleaning Notebook:** `02_notebooks/02_data_cleaning.ipynb`

| Column | Data Type | Description | Source / Transformation |
|--------|-----------|-------------|------------------------|
| date | datetime | Date of transaction | Parsed from string using `pd.to_datetime()` |
| year | int | Calendar year extracted from date | Derived: `df['date'].dt.year` |
| month | int | Calendar month (1–12) extracted from date | Derived: `df['date'].dt.month` |
| quarter | int | Calendar quarter (1–4) extracted from date | Derived: `df['date'].dt.quarter` |
| week | int | ISO week number (1–53) extracted from date | Derived: `df['date'].dt.isocalendar().week` |
| product_id | string | Unique product/SKU identifier | Renamed from original `sku` column |
| brand | string | Product brand identifier (e.g., MiBrand1, ReBrand2) | Original column, unchanged |
| segment | string | Product sub-segment (e.g., Milk-Seg1, SnackBar-Seg2) | Original column, unchanged |
| category | string | Original FMCG product category (Milk, Juice, ReadyMeal, SnackBar, Yogurt) | Original column, retained for audit trail |
| product_line | string | Meridian product line classification | Derived: mapped from `category` — Milk→MRO Supplies, Juice→Warehouse Equipment, ReadyMeal→Industrial Hardware, SnackBar→Safety Products, Yogurt→Packaging |
| channel | string | Sales channel (Retail, Discount, E-commerce) | Original column, unchanged |
| region | string | UK sales region (North, Midlands, South East, South West, London) | Replaced: original Polish regions (PL-Central, PL-North, PL-South) replaced with random UK region assignment using `np.random.seed(42)` for reproducibility. In production, regions would be derived from customer postcode data in CRM/ERP |
| pack_type | string | Product packaging type (Single, Multipack, Carton) | Original column, unchanged |
| unit_price_gbp | float | Unit price in GBP (£) | Renamed from original `price_unit` column. Currency relabelled to GBP for UK context |
| promotion_flag | int | Whether a promotion was active (0 = no, 1 = yes) | Original column, unchanged |
| delivery_days | int | Number of days for delivery (1–5) | Original column, unchanged |
| stock_available | int | Stock available at time of transaction | Original column, unchanged |
| delivered_qty | int | Quantity delivered to customer | Original column, unchanged |
| units_sold | int | Number of units sold in transaction | Original column, unchanged |
| customer_type | string | Customer segment (Trade, Retail, Government, Contractor) | Derived: randomly assigned with weighted probabilities (Trade 45%, Retail 30%, Government 15%, Contractor 10%) using `np.random.seed(42)`. Weighting reflects typical UK B2B industrial distribution revenue mix. In production, this would come from CRM customer master data |
| revenue_gbp | float | Transaction revenue in GBP (£) | Calculated: `units_sold × unit_price_gbp`. Rows with zero or negative revenue removed during cleaning |

**Cleaning Steps Applied:**
1. Dates parsed from string to datetime; year, month, quarter, week columns extracted
2. Column `sku` renamed to `product_id`; `price_unit` renamed to `unit_price_gbp`
3. Duplicate rows removed (deduplicated on date + product_id + region + channel)
4. UK regions assigned (random, reproducible via seed 42)
5. Customer type assigned (weighted random, reproducible via seed 42)
6. Product line hierarchy created by mapping FMCG categories to industrial product lines
7. Revenue calculated as units_sold × unit_price_gbp
8. Rows with zero or negative revenue removed

---

## Table 2: ons_rsi_clean.csv

**Source:** ONS Retail Sales Index — Reference Tables (mainreferencetables.xlsx)  
**Sheet Used:** Table 1 M (monthly, chained volume, seasonally adjusted)  
**Rows:** 36 (Jan 2022 – Dec 2024)  
**Purpose:** Market benchmark for the Sales Overview dashboard (Page 1 dual-axis chart)  
**Cleaning Notebook:** `02_notebooks/02_data_cleaning.ipynb`

| Column | Data Type | Description | Source / Transformation |
|--------|-----------|-------------|------------------------|
| date | datetime | First day of month (e.g., 2022-01-01) | Parsed from ONS format "2022 Jan" using `pd.to_datetime(format='%Y %b')` |
| rsi_all_retail | float | Retail Sales Index — All Retailing Including Automotive Fuel (Index 2023=100) | Extracted from column "All Retailing, Including Automotive Fuel". Represents total GB retail sales volume. Used as overall market benchmark |
| rsi_non_food | float | Retail Sales Index — Predominantly Non-food Stores (Index 2023=100) | Extracted from column "Predominantly Non-food Stores". Closest RSI sub-sector to industrial/non-food goods. More relevant comparison for Meridian than total retail |

**Cleaning Steps Applied:**
1. Loaded from sheet "Table 1 M" with 7 metadata rows skipped
2. Period column parsed from "2022 Jan" format to datetime
3. Non-data rows filtered out using regex (kept only rows starting with 4-digit year)
4. Filtered to 2022–2024 date range
5. Values converted to numeric (coercing any non-numeric to NaN)
6. Duplicate dates removed (sheet contains two vertically stacked tables: index values and year-on-year % changes; only index values retained)

**Notes:**
- Index base year is 2023=100. Values above 100 indicate sales volume above 2023 average; below 100 indicates below
- Seasonally adjusted figures are used, removing calendar and seasonal effects
- The monthly period consists of 4 weeks except March, June, September, and December which are 5 weeks

---

## Table 3: ons_ppi_clean.csv

**Source:** ONS Producer Price Index (mm22.xlsx)  
**Sheet Used:** data (single sheet containing annual, quarterly, and monthly data stacked vertically)  
**Rows:** 36 (Jan 2022 – Dec 2024)  
**Purpose:** Price benchmark for the Price Benchmarking dashboard (Page 5 dual-axis chart)  
**Cleaning Notebook:** `02_notebooks/02_data_cleaning.ipynb`

| Column | Data Type | Description | Source / Transformation |
|--------|-----------|-------------|------------------------|
| date | datetime | First day of month (e.g., 2022-01-01) | Parsed from ONS format "2022 JAN" (uppercase) using `pd.to_datetime(format='%Y %b')` |
| ppi_manufactured | float | PPI Output Domestic — All Manufactured Products excl. Duty (Index 2015=100) | Column 488 (SIC: C). Broad industrial benchmark. Used as proxy for MRO Supplies and Safety Products product lines |
| ppi_fabricated_metal | float | PPI Output Domestic — Other Fabricated Metal Products n.e.c. (Index 2015=100) | Column 338 (SIC: C2599). Covers tools, hardware, fittings. Mapped to Industrial Hardware product line |
| ppi_metal_structures | float | PPI Output Domestic — Metal Structures and Parts of Structures (Index 2015=100) | Column 324 (SIC: C2511). Covers racking, shelving, structural storage. Mapped to Warehouse Equipment product line |
| ppi_packaging | float | PPI Output Domestic — Light Metal Packaging (Index 2015=100) | Column 335 (SIC: C2592). Direct match for Packaging product line |

**Product Line to PPI Category Mapping:**

| Meridian Product Line | PPI Category | SIC Code | Rationale |
|-----------------------|-------------|----------|-----------|
| MRO Supplies | Manufactured Products (broad) | C | No single PPI category covers MRO; broad manufactured goods index is the most defensible proxy |
| Warehouse Equipment | Metal Structures and Parts | C2511 | Warehouse equipment (racking, shelving) is structurally classified under metal structures |
| Industrial Hardware | Other Fabricated Metal Products | C2599 | Fabricated metal products include tools, hardware, and fittings |
| Safety Products | Manufactured Products (broad) | C | Safety products span multiple SIC categories; broad index used as proxy |
| Packaging | Light Metal Packaging | C2592 | Direct category match |

**Cleaning Steps Applied:**
1. Loaded from single "data" sheet with column titles in row 0 and metadata in rows 1–6
2. Monthly data extracted starting from row 347 (file contains annual rows 7–74, quarterly rows 75–346, monthly rows 347+)
3. Only 4 relevant Output Domestic PPI columns selected from 602 total columns
4. Period column parsed from "2022 JAN" uppercase format to datetime
5. Filtered to 2022–2024 date range
6. Values converted to numeric

**Notes:**
- Index base year is 2015=100. All product lines show values above 100, indicating price levels higher than 2015
- Output PPI (Domestic) measures prices UK manufacturers charge to UK customers — the correct benchmark for a UK distributor's selling prices
- PPI category mapping is approximate. The ONS PPI uses SIC-based industry categories that do not map cleanly to Meridian's fictional product lines. Each mapping decision is documented above with rationale

---

## Table 4: crm_leads.csv (Synthetic)

**Source:** Mockaroo (mockaroo.com) — synthetic data generated to specification  
**Rows:** 1,000  
**Purpose:** Lead pipeline and conversion funnel analysis (Dashboard Page 3)  
**Location:** `00_raw_data/Synthetic/CRM Lead Pipeline.csv`

| Column | Data Type | Description | Source / Transformation |
|--------|-----------|-------------|------------------------|
| lead_id | string | Unique lead identifier (format: LD-1, LD-2, ...) | Mockaroo Row Number with formula `'LD-' + this.to_s` |
| enquiry_date | date | Date of initial customer enquiry | Mockaroo Datetime, range: Jan 2022 – Dec 2024, format dd/mm/yyyy |
| lead_source | string | Channel through which lead originated | Mockaroo Custom List: Website, Email Campaign, Trade Show, Referral, Online Directory, Cold Call (random, equal probability) |
| product_category | string | Product category the lead enquired about | Mockaroo Custom List: MRO Supplies, Warehouse Equipment, Industrial Hardware, Safety Products, Packaging (random, equal probability) |
| region | string | UK region of the prospective customer | Mockaroo Custom List: North, Midlands, South East, South West, London (random, equal probability) |
| response_time_hours | int | Hours between enquiry and first response | Mockaroo Number, min: 1, max: 96, no decimals |
| stage | string | Current stage in the sales pipeline | Mockaroo Custom List: Enquiry, Qualified, Proposal Sent, Won, Lost (random, equal probability). Note: guide specified weighted distribution 30/20/20/15/15 — to be adjusted in Python during cleaning if needed |
| deal_value_gbp | float | Estimated or actual deal value in GBP (£) | Mockaroo Number, min: 500, max: 85,000, 2 decimal places |
| customer_type | string | Type of customer organisation | Mockaroo Custom List: Trade, Retail, Government, Contractor (random, equal probability) |

**Notes:**
- This table is entirely synthetic. It was generated to mirror the schema and approximate distributions of a commercial CRM system (e.g., Salesforce, HubSpot)
- In a production environment, this data would come from CRM API exports or database views
- The equal probability distribution on lead_source and stage does not reflect real-world conversion rates. This is documented as a known limitation

---

## Table 5: marketing_campaigns.csv (Synthetic)

**Source:** Mockaroo (mockaroo.com) — synthetic data generated to specification  
**Rows:** 120 (approximately one per month per channel across 3 years)  
**Purpose:** Campaign performance and channel ROI analysis (Dashboard Page 4)  
**Location:** `00_raw_data/Synthetic/Marketing Campaign Performance .csv`

| Column | Data Type | Description | Source / Transformation |
|--------|-----------|-------------|------------------------|
| campaign_id | string | Unique campaign identifier (format: CP-1, CP-2, ...) | Mockaroo Row Number with formula `'CP-' + this.to_s` |
| month | date | Month of campaign activity | Mockaroo Datetime, range: Jan 2022 – Dec 2024, format yyyy-MM-dd |
| channel | string | Marketing channel used | Mockaroo Custom List: Email, LinkedIn, Trade Press, Direct Mail, PPC, Exhibition (random, equal probability) |
| spend_gbp | int | Campaign spend in GBP (£) | Mockaroo Number, min: 800, max: 15,000, no decimals |
| impressions | int | Number of impressions/views generated | Mockaroo Number, min: 2,000, max: 120,000, no decimals |
| clicks | int | Number of click-throughs | Mockaroo Formula: `(impressions * random(0.01, 0.08)).round` — calculates clicks as 1–8% of impressions, rounded to whole number |
| leads_generated | int | Number of leads attributed to campaign | Mockaroo Number, min: 5, max: 180, no decimals |
| revenue_attributed_gbp | int | Revenue attributed to campaign in GBP (£) | Mockaroo Number, min: 3,000, max: 95,000, no decimals |

**Notes:**
- This table is entirely synthetic
- Click-through rate is derived from impressions (1–8% range), creating a realistic relationship between impressions and clicks
- Revenue attribution in a real company would use multi-touch attribution models. The single-value attribution here is a simplification
- In production, this data would come from marketing automation platforms (e.g., HubSpot, Marketo) or advertising platform APIs

---

## Table 6: Inventory Dataset (to be cleaned)

**Source:** Kaggle Warehouse Inventory Dataset  
**Files:** `Consumables Report - Oct. 2022.xlsx`, `Food Report - Oct. 2022.xlsx`  
**Purpose:** Inventory health analysis, overstock/understock flagging (Dashboard Page 2)  
**Status:** Pending cleaning — will be processed in Phase 3 preparation

---

## Relationships Between Tables

| Primary Table | Join Key | Joins To | Join Key | Join Type |
|---------------|----------|----------|----------|-----------|
| transactions_clean | date (year-month) | ons_rsi_clean | date | Left join on year-month |
| transactions_clean | date (year-month) | ons_ppi_clean | date | Left join on year-month |
| transactions_clean | product_id | inventory (TBD) | product_id | Left join |
| crm_leads | — | — | — | Standalone table |
| marketing_campaigns | — | — | — | Standalone table |

---

## Known Limitations

1. **Region and customer type are modelled, not observed.** Both fields are assigned programmatically with a fixed random seed. In production, these would come from CRM customer master records or ERP system data
2. **Product line mapping is a relabelling exercise.** The underlying sales patterns are from FMCG data, not industrial goods. The numerical patterns (seasonality, volume) are analytically valid; the labels are contextual
3. **PPI category mapping is approximate.** MRO Supplies and Safety Products use the broad manufactured goods index because no single PPI category covers these product lines precisely
4. **Synthetic CRM and marketing data does not reflect real conversion rates or channel performance.** Distributions are configurable but not derived from actual commercial data
5. **No real-time data refresh.** All data is static CSV. In production, this would connect to a cloud data warehouse via DirectQuery (Power BI) or Live Connection (Tableau)
