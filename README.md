# Sales Data Pipeline — Databricks + Snowflake

An end-to-end data engineering pipeline that ingests raw sales data, transforms it using PySpark on Databricks, loads it into Snowflake, and visualizes key business metrics on a dashboard.

---

## Architecture

```
Raw Data (Python-generated CSV)
        ↓
PySpark Transformation (Databricks)
        ↓
Snowflake Data Warehouse
        ↓
Snowflake Dashboard
```

---

## Tech Stack

- **Python** — raw data generation
- **Apache PySpark** — data cleaning and transformation
- **Databricks** — PySpark execution environment
- **Snowflake** — cloud data warehouse
- **SQL** — analytical queries and reporting

---

## Project Structure

```
sales-data-pipeline/
│
├── generate_sales_data.py        # Generates 10,000 rows of fake sales data
├── sales_transformation.ipynb    # PySpark notebook — cleaning & transformation
└── README.md
```

---

## Pipeline Steps

### 1. Data Generation
Generated a synthetic sales dataset of 10,000 records using Python, covering:
- 10 products across Electronics category
- 5 regions (North, South, East, West, Central)
- 500 unique customers, 50 sales reps
- Orders from January 2023 to December 2024
- Intentional nulls (~3%) and mixed order statuses to simulate real-world data

### 2. PySpark Transformation (Databricks)
Loaded raw data into Databricks and performed the following transformations:
- Dropped rows with null `customer_id` or `region`
- Filtered to only `Completed` orders
- Added derived columns:
  - `gross_revenue` = quantity × unit_price
  - `net_revenue` = gross_revenue × (1 - discount)
  - `order_year` and `order_month` extracted from `order_date`
- Result: **5,675 clean records** saved as a Delta table

### 3. Snowflake Loading
Loaded the cleaned dataset into Snowflake:
- Database: `SALES_DB`
- Schema: `ANALYTICS`
- Table: `SALES_DATA_CLEAN`

### 4. SQL Analytics
Wrote analytical queries in Snowflake to answer key business questions:

**Revenue by Region**
```sql
SELECT 
    region,
    COUNT(order_id)            AS total_orders,
    ROUND(SUM(net_revenue), 2) AS total_revenue
FROM sales_db.analytics.sales_data_clean
GROUP BY region
ORDER BY total_revenue DESC;
```

**Top 5 Products by Revenue**
```sql
SELECT 
    product,
    COUNT(order_id)            AS total_orders,
    ROUND(SUM(net_revenue), 2) AS total_revenue
FROM sales_db.analytics.sales_data_clean
GROUP BY product
ORDER BY total_revenue DESC
LIMIT 5;
```

**Monthly Revenue Trend**
```sql
SELECT 
    order_year,
    order_month,
    ROUND(SUM(net_revenue), 2) AS monthly_revenue
FROM sales_db.analytics.sales_data_clean
GROUP BY order_year, order_month
ORDER BY order_year, order_month;
```

### 5. Dashboard
Built a Snowflake dashboard with 3 tiles:
- Bar chart — Revenue by Region
- Bar chart — Top 5 Products by Revenue
- Line chart — Monthly Revenue Trend (2023–2024)

---

## Key Findings

- **West** region generated the highest revenue (~$1.86M)
- **Laptops** were the top revenue-generating product (~$3.35M) despite not being the most ordered
- Revenue showed consistent monthly performance across 2023–2024

---

## How to Run

1. Clone this repo
2. Run `generate_sales_data.py` locally to generate `sales_data_raw.csv`
3. Upload `sales_data_raw.csv` to Databricks
4. Run `sales_transformation.ipynb` in Databricks
5. Load the output CSV into Snowflake under `SALES_DB.ANALYTICS.SALES_DATA_CLEAN`
6. Run the SQL queries in a Snowflake worksheet

---

## Author
Built as a portfolio project to demonstrate data engineering skills across PySpark, Databricks, and Snowflake.
