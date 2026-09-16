# AWS Lakehouse Medallion Pipeline

[![Databricks](https://img.shields.io/badge/Databricks-Lakehouse-FF3621?logo=databricks)](https://databricks.com)
[![AWS](https://img.shields.io/badge/AWS-Cloud-FF9900?logo=amazon-aws)](https://aws.amazon.com)
[![Python](https://img.shields.io/badge/Python-3.x-3776AB?logo=python)](https://www.python.org)
[![PySpark](https://img.shields.io/badge/PySpark-Data_Engineering-E25A1C?logo=apache-spark)](https://spark.apache.org)

A production-ready **Medallion Architecture** data pipeline built on **Databricks** and **AWS**, transforming raw AdventureWorks sales data through Bronze, Silver, and Gold layers for enterprise analytics.

---

## 🏗️ Architecture Overview

This pipeline implements the **Medallion Architecture** pattern:

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   BRONZE    │ --> │   SILVER    │ --> │    GOLD     │
│  Raw Data   │     │  Cleaned    │     │ Analytics   │
│  (CSV → ∆)  │     │  Validated  │     │ Star Schema │
└─────────────┘     └─────────────┘     └─────────────┘
```

### **Bronze Layer** 🥉
- **Purpose:** Raw data ingestion with no transformations
- **Source:** CSV files in Unity Catalog Volumes
- **Schema:** All columns as strings + audit metadata
- **Tables:** 7 raw tables (sales orders, customers, products, addresses, territories)
- **Audit Columns:**
  - `_ingested_at`: Timestamp of ingestion
  - `_source_file`: Source filename for lineage

### **Silver Layer** 🥈  
- **Purpose:** Cleaned, typed, and validated data
- **Transformations:**
  - Type casting (int, decimal, timestamp, boolean)
  - Geospatial parsing (lat/long from WKT POINT)
  - Data quality checks (duplicate detection)
  - Referential integrity validation (FK checks)
- **Tables:** 7 curated tables ready for analytics

### **Gold Layer** 🥇
- **Purpose:** Business-level aggregates and dimensional models
- **Schema:** Star schema for BI tools
- **Tables:**
  - `dim_date` — Calendar dimension (2011-2014, 1,461 dates)
  - `dim_product` — Product dimension (504 products)
  - `dim_customer` — Customer dimension with territory enrichment (19,820 customers)
  - `fact_sales` — Sales fact table (121,317 transactions)

---

## 📂 Repository Structure

```
AWS-Lakehouse_Medallion_Pipeline/
├── 01_bronze_ingestion.ipynb        # Raw data ingestion from CSV
├── 02_silver_transformation.ipynb   # Type casting + data quality
├── 03_gold_aggregated.ipynb         # Star schema dimensions + facts
└── README.md                        # This file
```

---

## 🚀 Getting Started

### Prerequisites

- **Databricks Workspace** (AWS-hosted)
- **Unity Catalog** enabled
- **Serverless Compute** or interactive cluster
- **Unity Catalog Volume** with AdventureWorks CSV files at:
  ```
  /Volumes/workspace/bronze/landing/AdventureWorksCSV-main/AdventureWorksCSV-main/
  ```

### Setup

1. **Clone this repository** into your Databricks workspace:
   ```bash
   # In Databricks Repos
   Repos → Add Repo → https://github.com/EvansTanui/AWS-Lakehouse_Medallion_Pipeline.git
   ```

2. **Create Unity Catalog schemas:**
   ```sql
   CREATE SCHEMA IF NOT EXISTS workspace.bronze;
   CREATE SCHEMA IF NOT EXISTS workspace.silver;
   CREATE SCHEMA IF NOT EXISTS workspace.gold;
   ```

3. **Upload AdventureWorks CSV files** to your Unity Catalog Volume:
   - Place CSV files in: `/Volumes/workspace/bronze/landing/AdventureWorksCSV-main/AdventureWorksCSV-main/`

4. **Run notebooks in order:**
   - ✅ `01_bronze_ingestion.ipynb`
   - ✅ `02_silver_transformation.ipynb`
   - ✅ `03_gold_aggregated.ipynb`

---

## 📊 Data Model

### Bronze Tables (7)
- `workspace.bronze.sales_order_header`
- `workspace.bronze.sales_order_detail`
- `workspace.bronze.customer`
- `workspace.bronze.sales_territory`
- `workspace.bronze.product`
- `workspace.bronze.product_category`
- `workspace.bronze.address`

### Silver Tables (7)
Same structure as Bronze, but with:
- Proper data types (int, decimal, timestamp)
- Boolean flags (TRUE/FALSE → boolean)
- Parsed geospatial coordinates
- Validated foreign keys

### Gold Tables (4)
**Dimensions:**
- `workspace.gold.dim_date` — 1,461 dates (2011-2014)
- `workspace.gold.dim_product` — 504 products
- `workspace.gold.dim_customer` — 19,820 customers

**Facts:**
- `workspace.gold.fact_sales` — 121,317 sales transactions

---

## 🔧 Key Features

✅ **Idempotent Pipeline** — Can be re-run safely (overwrite mode)  
✅ **Data Quality Checks** — Duplicate detection on primary keys  
✅ **Referential Integrity** — FK validation between tables  
✅ **Audit Trail** — Ingestion timestamps + source file tracking  
✅ **Geospatial Parsing** — Extracts lat/long from WKT POINT format  
✅ **Type Safety** — Explicit casting with validation  

---

## 📈 Sample Queries

### Top 5 Customers by Revenue
```sql
SELECT 
  c.account_number,
  c.territory_name,
  SUM(f.line_total) AS total_revenue
FROM workspace.gold.fact_sales f
JOIN workspace.gold.dim_customer c ON f.customer_id = c.customer_id
GROUP BY c.account_number, c.territory_name
ORDER BY total_revenue DESC
LIMIT 5;
```

### Monthly Sales Trend
```sql
SELECT 
  d.year,
  d.month,
  d.month_name,
  SUM(f.line_total) AS monthly_revenue,
  COUNT(DISTINCT f.sales_order_id) AS order_count
FROM workspace.gold.fact_sales f
JOIN workspace.gold.dim_date d ON f.order_date_key = d.date_key
GROUP BY d.year, d.month, d.month_name
ORDER BY d.year, d.month;
```

---

## 🛠️ Technologies

- **Databricks** — Unified analytics platform
- **PySpark** — Distributed data processing
- **Delta Lake** — ACID transactions and time travel
- **Unity Catalog** — Unified governance
- **AWS** — Cloud infrastructure

---

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

---

## 👤 Author

**Evans Tanui**  
[GitHub](https://github.com/EvansTanui) | [LinkedIn](https://linkedin.com/in/evanstanui)

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!  
Feel free to check the [issues page](https://github.com/EvansTanui/AWS-Lakehouse_Medallion_Pipeline/issues).

---

## ⭐ Show Your Support

Give a ⭐️ if this project helped you!
