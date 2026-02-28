
# 🏪 ShopData — Data Warehouse & Analytics Project

A complete **data warehousing and analytics solution** built on PostgreSQL using the **Medallion Architecture** (Bronze → Silver → Gold). This project demonstrates end-to-end data engineering: from raw CSV ingestion, through data cleansing and transformation, to business-ready analytical views and reports.

---

## 🏗️ Data Architecture

The project follows the **Medallion Architecture** with three layers:

![Data Architecture](docs/data_architecture.png)

| Layer | Purpose | Implementation |
|-------|---------|----------------|
| **🥉 Bronze** | Raw data ingestion from CSV source files | Tables created via `bronzeLayer.sql`, data loaded with Python (`psycopg2`) |
| **🥈 Silver** | Data cleansing, standardization & normalization | Transformations in `silverLayer.sql` (deduplication, type casting, formatting) |
| **🥇 Gold** | Business-ready dimension, fact & report views | Star schema views + analytical reports in `goldLayer.sql` |

---

## 📖 Project Overview

### What This Project Does

1. **Data Ingestion** — Loads raw CSV data from two source systems (CRM & ERP) into PostgreSQL bronze tables.
2. **Data Cleansing** — Deduplicates records, standardizes formats (gender, marital status, country names), fixes data types, and handles nulls.
3. **Data Modeling** — Creates a **star schema** with dimension views (`dim_customers`, `dim_products`) and a fact view (`fact_sales`).
4. **Advanced Analytics** — Builds report views with KPIs like customer lifetime value, product revenue segmentation, recency analysis, and spending tiers.

### Skills Demonstrated

- SQL Development & PostgreSQL
- Data Architecture (Medallion Architecture)
- ETL Pipeline Development
- Data Modeling (Star Schema)
- Data Analytics & Reporting
- Python (data loading with `psycopg2`)

---

## � Data Sources

Data is sourced from two systems:

### CRM (Customer Relationship Management)
| File | Description |
|------|-------------|
| `cust_info.csv` | Customer demographics (ID, name, gender, marital status) |
| `prd_info.csv` | Product catalog (name, cost, line, dates) |
| `sales_details.csv` | Sales transactions (orders, quantities, prices) |

### ERP (Enterprise Resource Planning)
| File | Description |
|------|-------------|
| `CUST_AZ12.csv` | Customer details (birth date, gender) |
| `LOC_A101.csv` | Customer locations (country) |
| `PX_CAT_G1V2.csv` | Product categories & subcategories |

---

## 🥉 Bronze Layer — `bronzeLayer.sql`

Creates the `bronze` schema and raw tables that mirror the CSV structure exactly.

**Tables created:**
- `bronze.crm_cust_info` — CRM customer info
- `bronze.crm_prd_info` — CRM product info
- `bronze.crm_sales_details` — CRM sales transactions
- `bronze.erm_customers` — ERP customer details
- `bronze.erm_customer_location` — ERP customer locations
- `bronze.erm_product_category` — ERP product categories

> Data is loaded into bronze tables using a Python script with `psycopg2`.

---

## 🥈 Silver Layer — `silverLayer.sql`

Runs as a **single execution block** (`DO $$ BEGIN...END $$`). Cleans and transforms bronze data into analysis-ready tables.

**Key transformations:**
| Table | Transformations Applied |
|-------|------------------------|
| `silver.crm_cust_info` | Deduplication via `ROW_NUMBER()`, name trimming, gender/marital status standardization |
| `silver.crm_prd_info` | Category ID extraction, product line mapping (R→Road, M→Mountain, etc.), end date via `LEAD()` |
| `silver.crm_sales_details` | Integer→Date conversion, negative sales/price correction, null handling |
| `silver.erm_customers` | "NAS" prefix removal, future birth date nullification, gender standardization |
| `silver.erm_customer_location` | Dash removal from IDs, country name standardization (USA→United States, DE→Germany) |
| `silver.erm_product_category` | Direct copy (data is clean) |

---

## 🥇 Gold Layer — `goldLayer.sql`

Creates the **star schema** views (dimensions + facts) and **advanced report views**, all in a single executable script.

### Dimension Views
| View | Description |
|------|-------------|
| `gold.dim_customers` | Customer dimension joining CRM info + ERP details + ERP location |
| `gold.dim_products` | Product dimension joining CRM products + ERP categories (active products only) |

### Fact View
| View | Description |
|------|-------------|
| `gold.fact_sales` | Sales fact table linking transactions to customer and product dimensions |

### Report Views
| View | Description |
|------|-------------|
| `gold.rpt_product_report` | Product KPIs: revenue segmentation, order metrics, lifespan, recency, avg revenue |
| `gold.rpt_customer_report` | Customer KPIs: spending segmentation, order metrics, lifespan, recency, avg revenue |

### Data Model

![Data Model](docs/data_model.png)

---

## 📈 Advanced Data Analysis — `advancedDataAnalysis.sql`

Standalone analytical queries for deeper insights:

- **Monthly Sales Summary** — Revenue, customer count, and quantity by month/year
- **Cumulative Yearly Sales** — Running total of sales across years
- **Product Revenue Trends** — Year-over-year product revenue comparison with trend analysis
- **Product Cost Categories** — Segmentation into High/Medium/Low cost tiers
- **Customer Spending Segmentation** — VIP / Regular / New customer classification
- **Customer Type Distribution** — Count of customers per segment

---

## 📂 Repository Structure

```
ShopData/
│
├── datasets/                          # Raw source data (CSV files)
│   ├── source_crm/                    # CRM system exports
│   │   ├── cust_info.csv              # Customer demographics
│   │   ├── prd_info.csv               # Product catalog
│   │   └── sales_details.csv          # Sales transactions
│   └── source_erp/                    # ERP system exports
│       ├── CUST_AZ12.csv              # Customer details
│       ├── LOC_A101.csv               # Customer locations
│       └── PX_CAT_G1V2.csv           # Product categories
│
├── docs/                              # Documentation & diagrams
│   ├── data_architecture.png          # Medallion architecture diagram
│   ├── data_model.png                 # Star schema diagram
│   ├── data_flow.png                  # Data flow diagram
│   ├── data_integration.png           # Integration diagram
│   ├── ETL.png                        # ETL process diagram
│   ├── data_catalog.md                # Field descriptions & metadata
│   ├── naming_conventions.md          # Naming guidelines
│   ├── data_layers.pdf                # Layer documentation
│   └── Project_Notes_Sketches.pdf     # Project planning notes
│
├── bronzeLayer.sql                    # Bronze: schema & table creation
├── silverLayer.sql                    # Silver: data cleansing & transformation
├── goldLayer.sql                      # Gold: dimension, fact & report views
├── advancedDataAnalysis.sql           # Advanced analytical queries
├── exploreData.sql                    # Data exploration queries
├── dashboard.pbix                     # Power BI dashboard
├── dashboard.pdf                      # Dashboard export (PDF)
└── README.md                          # This file
```

---

## 🚀 How to Run

### Prerequisites
- **PostgreSQL** installed and running
- **Python 3.x** with `psycopg2` package (for CSV loading)
- **Power BI Desktop** (optional, for dashboard)

### Execution Order

```bash
# 1. Create bronze tables
psql -f bronzeLayer.sql

# 2. Load CSV data into bronze tables (Python script)
python load_data.py

# 3. Transform data into silver layer
psql -f silverLayer.sql

# 4. Create gold layer views (dimensions, facts, reports)
psql -f goldLayer.sql

# 5. Run advanced analysis queries (optional)
psql -f advancedDataAnalysis.sql
```

> **Note:** Each SQL script is idempotent — safe to re-run at any time. Tables/views are dropped and recreated on each execution.

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| **PostgreSQL** | Database engine |
| **Python + psycopg2** | CSV data loading |
| **Power BI** | Dashboard & visualization |
| **Draw.io** | Architecture & data model diagrams |
| **Git + GitHub** | Version control |

---

## 🛡️ License

This project is licensed under the [MIT License](LICENSE). You are free to use, modify, and share this project with proper attribution.

---

## 🌟 About Me

Hi! I'm **Islam Benchaiba**, a data engineering enthusiast passionate about building robust data pipelines and analytical solutions.

📧 **Contact:** mi.benchaiba@esi-sba.dz
