# SQL Data Warehouse & Analytics Project

A portfolio project that demonstrates the end-to-end development of a **SQL Server data warehouse**, from raw CRM/ERP source data through ingestion, transformation, quality validation, and business-ready analytical models.

The project follows a **Medallion Architecture** with Bronze, Silver, and Gold layers and uses a **star-schema-oriented analytical model** in the Gold layer.

---

## Project Overview

This project simulates a real-world analytics data platform that integrates data from multiple source systems:

- **CRM data:** customers, products, and sales transactions
- **ERP data:** customer attributes, locations, and product categories
- **SQL Server data warehouse:** Bronze, Silver, and Gold layers
- **Analytical model:** customer and product dimensions plus a sales fact table

The goal is to transform raw operational data into clean, standardized, and business-ready datasets that can support reporting and analytics.

---

## Architecture

The warehouse is organized into three layers:

```text
CRM / ERP CSV Sources
        |
        v
+-------------------+
|   Bronze Layer    |
| Raw / minimally   |
| transformed data  |
+-------------------+
        |
        v
+-------------------+
|   Silver Layer    |
| Cleaned, validated|
| and standardized  |
+-------------------+
        |
        v
+-------------------+
|    Gold Layer     |
| Business-ready    |
| dimensions + fact |
+-------------------+
```

### Bronze Layer

Stores raw source data with minimal transformation.

Main responsibilities:
- Create raw warehouse tables
- Load CSV files using `BULK INSERT`
- Refresh tables before each load

### Silver Layer

Cleans, standardizes, and transforms Bronze data.

Examples of transformations include:
- Trimming unwanted spaces
- Standardizing gender and marital-status values
- Handling missing product costs
- Cleaning customer and location keys
- Converting integer date representations into SQL dates
- Validating and correcting sales values
- Removing duplicate customer records using `ROW_NUMBER()`
- Deriving product validity periods using `LEAD()`

### Gold Layer

Provides business-ready analytical views based on the cleaned Silver data.

The Gold layer contains:

- `gold.dim_customers`
- `gold.dim_products`
- `gold.fact_sales`

These objects form a dimensional model suitable for reporting and analytics.

---

## Data Sources

### CRM

| Source | Description |
|---|---|
| `cust_info.csv` | Customer master data |
| `prd_info.csv` | Product information and product history |
| `sales_details.csv` | Sales transaction details |

### ERP

| Source | Description |
|---|---|
| `CUST_AZ12.csv` | Customer birth date and gender |
| `LOC_A101.csv` | Customer country/location |
| `PX_CAT_G1V2.csv` | Product categories and subcategories |

---

## Gold Data Model

### Customer Dimension

`gold.dim_customers`

Contains:
- Customer surrogate key
- Customer ID and number
- Name
- Country
- Marital status
- Gender
- Birthdate
- Customer creation date

### Product Dimension

`gold.dim_products`

Contains:
- Product surrogate key
- Product ID and number
- Product name
- Category and subcategory
- Maintenance classification
- Product cost
- Product line
- Product start date

Historical product versions are filtered so the analytical view represents the current product record.

### Sales Fact

`gold.fact_sales`

Contains:
- Order number
- Customer key
- Product key
- Order date
- Ship date
- Due date
- Sales amount
- Quantity
- Price

---

## SQL Concepts Demonstrated

This project demonstrates practical SQL Server skills, including:

- Database and schema creation
- DDL
- Tables and views
- Stored procedures
- `BULK INSERT`
- `TRUNCATE TABLE`
- `CASE`
- `TRIM` and `REPLACE`
- `ISNULL`
- `CAST` and date conversion
- `ROW_NUMBER()`
- `LEAD()`
- Window functions
- CTE-style analytical thinking
- Joins
- Data standardization
- Duplicate handling
- Data-quality validation
- Fact and dimension modeling
- Star-schema design
- ETL / ELT workflow design
- Error handling with `TRY...CATCH`

---

## Data Quality Framework

Quality checks are included for both Silver and Gold layers.

### Silver checks

The project validates:
- Null and duplicate business keys
- Leading/trailing spaces
- Standardized categorical values
- Missing or negative product costs
- Invalid date ranges
- Invalid order/ship/due-date relationships
- Sales consistency: `Sales = Quantity × Price`
- Invalid birth dates
- Country standardization

### Gold checks

The project validates:
- Uniqueness of customer surrogate keys
- Uniqueness of product surrogate keys
- Connectivity between the sales fact and its customer/product dimensions

Scripts:

`tests/quality_checks_silver.sql`

`tests/quality_checks_gold.sql`

---

## Project Structure

```text
sql-data-warehouse-project/
│
├── datasets/
│   ├── source_crm/
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   │
│   └── source_erp/
│       ├── CUST_AZ12.csv
│       ├── LOC_A101.csv
│       └── PX_CAT_G1V2.csv
│
├── scripts/
│   ├── init_database.sql
│   │
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   │
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   │
│   └── gold/
│       └── ddl.gold.sql
│
├── tests/
│   ├── quality_checks_silver.sql
│   └── quality_checks_gold.sql
│
└── LICENSE
```

---

## How to Run

### 1. Prerequisites

Use:

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)

The scripts use SQL Server-specific features such as `BULK INSERT`, stored procedures, schemas, and `TRY...CATCH`.

### 2. Clone the repository

```bash
git clone https://github.com/bhaskar-nb/sql-data-warehouse-project.git
cd sql-data-warehouse-project
```

### 3. Create the warehouse

Run:

```text
scripts/init_database.sql
```

This creates the `Datawarehouse` database and the `bronze`, `silver`, and `gold` schemas.

**Warning:** the initialization script drops and recreates the database if it already exists.

### 4. Create Bronze tables

Run:

```text
scripts/bronze/ddl_bronze.sql
```

### 5. Configure the CSV paths

Before executing the Bronze loading procedure, update the file paths in:

```text
scripts/bronze/proc_load_bronze.sql
```

The procedure currently contains machine-specific Windows paths. Replace them with the actual location of the repository on your SQL Server environment.

### 6. Load Bronze data

Create the procedure and execute:

```sql
EXEC bronze.load_bronze;
```

### 7. Create Silver tables

Run:

```text
scripts/silver/ddl_silver.sql
```

### 8. Load and transform Silver data

Create and execute:

```sql
EXEC silver.load_silver;
```

### 9. Create Gold views

Run:

```text
scripts/gold/ddl.gold.sql
```

### 10. Run quality checks

Execute:

```text
tests/quality_checks_silver.sql
tests/quality_checks_gold.sql
```

---

## Example Analytical Queries

Once the Gold layer is created, the views can be queried for analytics.

### Total sales

```sql
SELECT
    SUM(sales_amount) AS total_sales
FROM gold.fact_sales;
```

### Sales by product

```sql
SELECT
    p.category,
    p.subcategory,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
JOIN gold.dim_products p
    ON f.product_key = p.product_key
GROUP BY
    p.category,
    p.subcategory
ORDER BY
    total_sales DESC;
```

### Sales by country

```sql
SELECT
    c.country,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
JOIN gold.dim_customers c
    ON f.customer_key = c.customer_key
GROUP BY
    c.country
ORDER BY
    total_sales DESC;
```

### Customer sales contribution

```sql
SELECT
    c.customer_key,
    c.first_name,
    c.last_name,
    SUM(f.sales_amount) AS total_sales
FROM gold.fact_sales f
JOIN gold.dim_customers c
    ON f.customer_key = c.customer_key
GROUP BY
    c.customer_key,
    c.first_name,
    c.last_name
ORDER BY
    total_sales DESC;
```

---

## Key Engineering and Analytics Workflow

```text
Source CSV Files
      ↓
Database Initialization
      ↓
Bronze DDL
      ↓
Bronze Load Procedure
      ↓
Silver DDL
      ↓
Silver ETL / Data Cleaning
      ↓
Silver Quality Checks
      ↓
Gold Dimension + Fact Views
      ↓
Gold Quality Checks
      ↓
Analytics / Reporting
```

---

## What This Project Demonstrates

This project demonstrates the ability to work beyond isolated SQL queries and build a structured analytics data pipeline.

Key capabilities demonstrated:

- Designing a layered data warehouse
- Integrating CRM and ERP sources
- Building repeatable SQL loading procedures
- Applying data cleaning and standardization rules
- Using window functions for deduplication and historical logic
- Designing analytical dimensions and fact tables
- Implementing data-quality checks
- Creating reusable business-ready views
- Documenting an end-to-end SQL project

---

## Technologies

- **Database:** Microsoft SQL Server
- **IDE:** SQL Server Management Studio (SSMS)
- **Language:** T-SQL
- **Data Format:** CSV
- **Architecture:** Medallion Architecture
- **Data Model:** Dimensional / Star Schema

---

## Author

**Bhaskar Nakka**

Computer Science Engineering Graduate | Aspiring Data Analyst

Skills demonstrated in this project:
`SQL` • `T-SQL` • `SQL Server` • `Data Warehousing` • `ETL` • `Data Cleaning` • `Data Modeling` • `Data Quality` • `Window Functions`

---

## License

This project is available under the MIT License. See [LICENSE](LICENSE) for details.
