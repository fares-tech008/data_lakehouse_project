# Data Lakehouse on Databricks (Medallion Architecture, PySpark + Delta Lake)

A guided learning project: a small data lakehouse built on **Databricks** with Bronze, Silver and Gold layers. It takes sales data from two source systems (CRM and ERP, both CSV files), cleans it with **PySpark**, and builds a star schema as **Delta** tables.

> **Credits:** The dataset, the business case and the overall design (CRM + ERP sources, Bronze/Silver/Gold, the star schema) come from the free **Data With Baraa** SQL data warehouse tutorial (MIT licensed, © 2024 Baraa Khatib Salkini). In that tutorial the warehouse is built in SQL Server. Here I rebuilt it on Databricks with PySpark, Spark SQL and Delta Lake, as a way to practice these tools. The original copyright notice is kept in [LICENSE](LICENSE).

---

## Architecture

| Layer | What it does | Tables (catalog `workspace`) |
|-------|--------------|------------------------------|
| **Bronze** | Reads each CSV from a Unity Catalog Volume and saves it as a Delta table, unchanged (schema inferred). | `bronze.crm_cust_info`, `crm_prd_info`, `crm_sales_details`, `erp_cust_az12`, `erp_loc_a101`, `erp_px_cat_g1v2` |
| **Silver** | One notebook per table: trims text, standardises codes, casts types, removes bad rows, renames columns. | `silver.crm_customers`, `crm_products`, `crm_sales`, `erp_customers`, `erp_customer_location`, `erp_product_category` |
| **Gold** | Joins the Silver tables into a star schema. | `gold.dim_customers`, `gold.dim_products`, `gold.fact_sales` |

## Repository structure

```
data_lakehouse_project/
├── datasets/
│   ├── source_crm/        # cust_info.csv, prd_info.csv, sales_details.csv
│   └── source_erp/        # CUST_AZ12.csv, LOC_A101.csv, PX_CAT_G1V2.csv
├── script/
│   ├── init_lakehouse.ipynb       # Creates the schemas and the Volume
│   ├── bronze/bronze_layer.ipynb  # Loads all 6 CSVs into Bronze tables
│   ├── silver/
│   │   ├── crm/                   # 3 notebooks (customers, products, sales)
│   │   ├── erp/                   # 3 notebooks (customers, location, product category)
│   │   └── silver_orchestration.ipynb   # Runs the 6 Silver notebooks
│   └── gold/
│       ├── gold_dim_customers.ipynb
│       ├── gold_dim_products.ipynb
│       ├── gold_fact_sales.ipynb
│       └── gold_orchestration.ipynb     # Runs the 3 Gold notebooks
├── LICENSE
└── README.md
```

## Technologies

Databricks, Apache Spark (PySpark and Spark SQL), Delta Lake, Unity Catalog Volumes.

## How to run

You need a Databricks workspace with Unity Catalog and a catalog named `workspace`.

1. **Import the `script/` folder** into your Databricks workspace. Keep the sub-folders (`bronze`, `silver/crm`, `silver/erp`, `gold`), because the orchestration notebooks use relative paths like `./crm/silver_crm_cust_info`.
2. **Run `init_lakehouse`.** It creates the schemas `bronze`, `silver`, `gold` and the Volume `workspace.bronze.raw_sources`.
3. **Upload the CSV files** from `datasets/` into the Volume, keeping the two sub-folders, so the paths become:
   - `/Volumes/workspace/bronze/raw_sources/source_crm/cust_info.csv` (and the other 2 CRM files)
   - `/Volumes/workspace/bronze/raw_sources/source_erp/CUST_AZ12.csv` (and the other 2 ERP files)
4. **Run the notebooks in this order:**
   1. `bronze/bronze_layer`
   2. `silver/silver_orchestration`
   3. `gold/gold_orchestration`
5. **Check the result:**

```sql
SELECT * FROM workspace.gold.fact_sales LIMIT 10;
```

## What I learned

- The Medallion pattern (Bronze, Silver, Gold) on Databricks
- Reading CSVs from a Unity Catalog Volume and writing Delta tables
- Cleaning data with PySpark (trimming, code mapping, type casting, renaming)
- Building a star schema (two dimensions and one fact table)
- Running notebooks in order with `dbutils.notebook.run`

## Limitations (honest notes)

- Guided tutorial project: the design is not original. The PySpark/Databricks version is my own rebuild.
- Every run overwrites the tables (full reload). There is no incremental loading.
- There are no automated tests and no data quality checks or logging.
- The Bronze layer infers the schema, which is simple but not production style.

For a project I designed and debugged myself, see my [airline-operations-lakehouse](https://github.com/fares-tech008/airline-operations-lakehouse), which has incremental loading, a data quality gate, an audit log, 28 automated tests and CI.

## License

MIT License. See [LICENSE](LICENSE). Original work © 2024 Baraa Khatib Salkini; modifications © 2026 Fares Bittar.

## About me

I'm **Fares Bittar**, an informatics student teaching myself data engineering (SQL, Python, Spark). This repository is one of my learning projects.
